---
tags: [pos, backend, bugs, fixes, gorm, go]
created: 2026-05-22
updated: 2026-06-04
---

# 🐛 Backend Bug Fixes

← [[POS Backend MOC]] · [[Backend Architecture]] · [[POS Project MOC]]

---

## Sale 500 Error — `stock_movements.created_by` FK

**Bug:** `INSERT INTO stock_movements` failed with FK violation on `created_by`.  
**Cause:** Passing `"system"` — a non-existent user ID — for the `created_by` foreign key.  
**Fix:** Pass `cashierUserID` (the authenticated user's UUID from JWT).

---

## Products Not Showing in Cashier — Wrong BFF Proxy Paths

**Bug:** Products and stock movements not loading in cashier mode.  
**Cause:** BFF Route Handlers pointed to wrong backend endpoints.

| Wrong | Correct |
|-------|---------|
| `/stock/add` | `/stock-movements/in` |
| `/stock/movements` | `/stock-movements` |

See [[BFF Proxy Routes]].

---

## Cross-Store Transfer — No Change

**Bug:** After transfer, destination store showed no stock change.  
**Cause:** Code was staging to `warehouse_inventory` table instead of the live `stocks` table.  
**Fix:** Changed insert target to `stocks` directly.

See [[Backend Architecture]] for the correct transfer flow.

---

## GORM v2 `Take(&string)` — Silent Empty String

**Bug:** GORM `Take` with a string pointer returns `""` silently when no row found (no error).  
**Cause:** GORM v2 treats primitive pointer destinations differently.  
**Fix:** Always use an anonymous struct for single-field lookups.

```go
// ❌ Wrong — silent empty string, no error
var id string
db.Model(&ProductType{}).Where("...").Take(&id)

// ✅ Correct — proper not-found error
var row struct{ ID string `gorm:"column:id"` }
db.Model(&ProductType{}).Where("...").Take(&row)
```

Affected: `cloneOrFindProductType`, `cloneOrFindProductUnit`, `cloneOrFindProductBrand`.  
See [[GORM Patterns]].

---

## Clone Product Missing Fields (`special_price`)

**Bug:** Cloned products were missing `special_price`, `special_price_start_at`, `special_price_end_at`.  
**Fix:** Added these three fields to both the SELECT and INSERT in the clone function.

---

## `unit_type` Column Error

**Bug:** Queries referencing `unit_type` column failed after migration.  
**Cause:** Column was dropped in migration `005`.  
**Fix:** Removed all references to `unit_type` from GORM queries.

---

## Discount `discount_value = 0` → 400 Error (Fixed 2026-05-24)

**Bug:** Saving receipt items with no discount returned 400.  
**Cause:** `calculateDiscount` in `warehouse_receipt/util.go` returned error when `discount_type = ""` but `discount_value` was a non-nil `*float64` pointing to `0`.  
Frontend always sends `discount_value: 0` for items without discounts.

```go
// ❌ Wrong
if discountValue != nil {
    return 0, fmt.Errorf("discount_type is empty but discount_value is set")
}

// ✅ Fix
if discountValue != nil && *discountValue != 0 {
    return 0, fmt.Errorf("discount_type is empty but discount_value is set")
}
```

File: `pos-backend/internal/modules/warehouse_receipt/util.go`

---

## Print HTML — Wrong API Helper

**Bug:** `fetchGoodsReceiptPrintDocument` returned undefined HTML.  
**Cause:** Used `authorizedRawRequest` instead of `authorizedApiRequest`.  
`authorizedRawRequest` returns the raw envelope `{ success, message, data: { html } }`.  
Code read `doc.html` (top-level) — not `doc.data.html`.

**Fix:** Switch to `authorizedApiRequest<GoodsReceiptPrintDocument>` and read `doc.data.html`.

See [[API Client Patterns]] for the full pattern explanation.

---

## Receipt Preview Panel — Wrong Render Function

**Bug:** Receipt preview was too large and had toolbar/overlay in 360px settings panel.  
**Cause:** Used `receipthtml.RenderPreviewHTML()` which adds dark overlay + toolbar + paper card.  
**Fix:** Created `receipthtml.RenderPanelPreviewHTML()` — white bg only, 3px minimal scrollbar injected via `<style>`.

File changed: `internal/modules/receipt_settings/service.go`  
New function: `internal/platform/receipthtml/template.go`

See [[Receipt Payment Settings]].

---

---

## Store Update — NOT NULL Constraint Violation (SQLSTATE 23502)

**Bug:** `PUT /stores/:storeID` threw `null value in column "tax_id" violates not-null constraint`.  
**Cause:** Repository `Update` payload used `nilIfEmpty(update.TaxID)` — returned `nil` for empty string, but `tax_id`, `fax`, `email`, `website` are `NOT NULL DEFAULT ''`.  
**Fix:** Use bare string value for NOT NULL columns; reserve `nilIfEmpty()` for nullable columns only.

```go
// ✅ NOT NULL columns — empty string is valid, nil is NOT
"tax_id":  update.TaxID,
"fax":     update.Fax,
"email":   update.Email,
"website": update.Website,

// ✅ nullable columns — nilIfEmpty OK
"phone":   nilIfEmpty(update.Phone),
"address": nilIfEmpty(update.Address),
```

Also fixed: `GetByID` and `ListByUser` were not SELECTing `tax_id`, `fax`, `email`, `website` at all.

See [[Store Module]] for full schema + repository patterns.

---

---

## parked_bills — note NOT NULL + customer_id nullable (2026-06-03)

**Bug 1:** `INSERT INTO parked_bills` failed — `null value in column "note"`.  
**Cause:** Code sent `nil` when `bill.Note == ""`.  
**Fix:** Always send `bill.Note` (empty string is valid — column is NOT NULL DEFAULT '').  
Migration `017`: `ALTER TABLE parked_bills ALTER COLUMN note SET DEFAULT ''; ALTER ... SET NOT NULL;`

**Bug 2:** `null value in column "customer_id"` for walk-in customers.  
**Fix:** Only set `customer_id` in insert payload when non-empty. Column dropped NOT NULL.  
Migration `018`: `ALTER TABLE parked_bills ALTER COLUMN customer_id DROP NOT NULL;`

File: `internal/modules/parkedbill/repository.go`

---

## Supplier File Upload 500 — BFF reads body as text (2026-06-03)

**Bug:** POST/PUT supplier with logo → 500 Internal Server Error.  
**Cause:** BFF route used `request.text()` which converts multipart binary to string, destroying the boundary.  
**Fix:** Use `request.blob()` instead — preserves binary data; `buildForwardHeaders` correctly forwards `Content-Type` including the boundary.

```ts
// ❌ Wrong — destroys multipart
const body = await request.text();

// ✅ Correct — binary-safe
const body = await request.blob();
return proxyApiRequest(request, url, { body, method: "POST" });
```

Affected routes: `app/api/stores/[storeId]/suppliers/route.ts` and `.../[supplierId]/route.ts`

See [[BFF Proxy Routes]].

---

## AdjustStock FK violation — location_id empty string (2026-06-02)

**Bug:** `AdjustStock` failed with FK constraint on `location_id`.  
**Cause:** Passed `&""` (pointer to empty string) — FK expects NULL or valid ID.  
**Fix:** Resolve via `findDefaultStockLocation`, use `*string` nil pointer when no location.

---

## MinIO MINIO_PUBLIC_URL double bucket name (2026-06-03)

**Bug:** Image URLs rendered as `https://media.phiraphat.site/pos-assets/pos-assets/products/...`  
**Cause:** `MINIO_PUBLIC_URL` was set to `https://media.phiraphat.site/pos-assets` — the bucket name was already appended by the SDK path.  
**Fix:** Set `MINIO_PUBLIC_URL=https://media.phiraphat.site` (no bucket path suffix).

---

## Related
- [[GORM Patterns]] — GORM-specific fixes
- [[Store Module]] — NOT NULL pattern, contact fields
- [[Backend Architecture]] — transfer flow fix context
- [[API Client Patterns]] — print helper bug
- [[Warehouse Receive Flow]] — discount_value bug context
