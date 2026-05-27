---
tags: [pos, backend, gorm, go, database, patterns]
created: 2026-05-24
---

# 🗄️ GORM Patterns

← [[POS Backend MOC]] · [[Backend Architecture]] · [[POS Project MOC]]

---

## ⚠️ `Take(&string)` Returns Silent Empty String

**Never** use a plain string/primitive pointer as the destination for `Take`:

```go
// ❌ Wrong — returns "" silently when no row found (no error!)
var id string
db.Model(&ProductType{}).Where("store_id = ?", storeID).Take(&id)

// ✅ Correct — returns proper gorm.ErrRecordNotFound
var row struct{ ID string `gorm:"column:id"` }
db.Model(&ProductType{}).Where("store_id = ?", storeID).Take(&row)
if errors.Is(err, gorm.ErrRecordNotFound) { /* handle */ }
```

**Where it was fixed:** `cloneOrFindProductType`, `cloneOrFindProductUnit`, `cloneOrFindProductBrand`

---

## Read-Only Fields — `gorm:"->"`

For virtual/computed fields that GORM should read from DB but NEVER write back:

```go
// model.go
type ProductType struct {
    ID           string `gorm:"primaryKey"`
    Name         string
    // Read-only — populated by correlated subquery in ListByStore
    ProductCount int64  `json:"product_count" gorm:"column:product_count;->"`
}
```

The `->` tag means:
- ✅ GORM reads it from SELECT results
- ❌ GORM never includes it in INSERT or UPDATE statements
- Without `->`: GORM ignores it during SELECT (doesn't populate), but tries to write during UPDATE (fails or corrupts)

---

## Correlated Subquery for Aggregates

Pattern for adding a computed count to a list query (no migration needed):

```go
// repository.go — ListByStore
err := r.db.WithContext(ctx).
    Model(&ProductType{}).
    Select(`product_types.*,
        COALESCE(
            (SELECT COUNT(*) FROM products p WHERE p.product_type_id = product_types.id),
            0
        ) AS product_count`).
    Where("product_types.store_id = ?", storeID).
    Order("product_types.name ASC").
    Find(&items).Error
```

Applied to: `producttype`, `productunit` (subquery on `p.product_unit_id`), `productbrand` (on `p.brand_id`).

The `.Select()` string + `gorm:"->"` tag on the model field = GORM populates the field from the alias.

---

## `cloneOrFind` Pattern (Idempotent Product Chain Clone)

For cross-store product cloning — matches existing records by `LOWER(name)`:

```go
func (r *repo) cloneOrFindProductType(ctx context.Context, name string, storeID string) (string, error) {
    var row struct{ ID string `gorm:"column:id"` }
    err := r.db.WithContext(ctx).
        Model(&ProductType{}).
        Where("LOWER(name) = LOWER(?) AND store_id = ?", name, storeID).
        Take(&row).Error
    if err == nil {
        return row.ID, nil  // already exists
    }
    if !errors.Is(err, gorm.ErrRecordNotFound) {
        return "", err
    }
    // create new
    pt := &ProductType{ID: idgen.New("pt"), Name: name, StoreID: storeID}
    return pt.ID, r.db.WithContext(ctx).Create(pt).Error
}
```

---

## Related
- [[Backend Bug Fixes]] — GORM Take bug, clone missing fields
- [[Backend Architecture]] — repository layer context
- [[Category Management]] — product_count subquery usage
