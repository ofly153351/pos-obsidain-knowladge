---
tags: [pos, backend, bugs, fixes, gorm, go]
created: 2026-05-22
updated: 2026-05-27
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

## Related
- [[GORM Patterns]] — GORM-specific fixes
- [[Backend Architecture]] — transfer flow fix context
- [[API Client Patterns]] — print helper bug
- [[Warehouse Receive Flow]] — discount_value bug context
