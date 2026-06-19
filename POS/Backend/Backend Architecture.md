---
tags: [pos, backend, go, fiber, architecture, gorm]
created: 2026-05-22
updated: 2026-06-04
---

# 🔌 Backend Architecture

← [[POS Backend MOC]] · [[POS Project MOC]]

## Stack

| Layer | Tech |
|-------|------|
| Framework | Go Fiber v2 |
| ORM | GORM v2 + PostgreSQL |
| Auth | JWT (stored in `pos-access-token` cookie) |
| ID generation | Custom prefix IDs (e.g. `rset`, `whr`) |

## Request Pattern

```
Route (app.go)
  → Handler (handler.go)
    → Service (service.go)
      → Repository (repository.go)
        → GORM / PostgreSQL
```

Never skip layers. Service validates business rules. Repository handles DB queries only.

**Platform packages** (`internal/platform/`) are pure utilities with no business-logic imports — they can be used by any module. Modules **import** platform packages; platform packages do **not** import modules (one-way dependency).

## Module Registration

Every module registered in `internal/app/`:
- `app.go` — wire handler into fiber app
- `dependencies.go` — instantiate repo → service → handler

Pattern: `receipt_settings.go` → `NewReceiptSettingsHandler(db) → app.RegisterRoutes(handler)`

## Cross-Store Warehouse Transfer Flow

1. Deduct from Store A `stocks` (location with `quantity >= qty`, ordered by `is_sale_point ASC`)
2. Clone product chain for Store B (all idempotent via `LOWER(name)` match):
   - `producttype` → `productunit` → `productbrand` → `product`
3. Auto-create warehouse in Store B (tracks `source_store_id`)
4. Insert directly to Store B `stocks` at non-sale-point location

> The transfer **inserts to `stocks` directly** — NOT `warehouse_inventory` (which was the bug).

## Response Envelope

All responses use `httpx.Success` / `httpx.Error`:

```json
{ "success": true, "message": "OK", "data": <payload> }
{ "success": false, "message": "Error description", "error": <detail> }
```

File: `pos-backend/internal/platform/httpx/response.go`

This is why [[API Client Patterns]] distinguishes `authorizedApiRequest` (unwraps `.data`) from `authorizedRawRequest` (raw body).

## Key Modules

| Module path | Purpose |
|-------------|---------|
| `internal/modules/receipt_settings/` | Receipt design + payment channels |
| `internal/modules/warehouse_receipt/` | Receive goods wizard |
| `internal/modules/sale/` | Cashier / POS sales |
| `internal/modules/stock/` | Stock movements |
| `internal/modules/document/` | Document business logic only (CRUD, status, orchestration) |
| `internal/platform/receipthtml/` | POS receipt HTML (`RenderPanelPreviewHTML`) |
| `internal/platform/dochtml/` | Document HTML templates — Invoice, Bill, WHT cert (see [[Document PDF & WHT]]) |
| `internal/platform/docpdf/` | Document PDF renderers — Invoice, Bill, Statement (gofpdf + embedded fonts) |

| `internal/modules/activity_log/` | Auto-log all store mutations (added 2026-06-03) |
| `internal/modules/parkedbill/` | Hold/restore bills in cashier |
| `internal/modules/usersettings/` | Per-user prefs (card_settings JSONB) — `GET/PUT /me/card-settings` (added 2026-06-08) |
| `internal/modules/expense/` + `finance/` | Expenses + P&L/summary/inventory reports — see [[Finance & Reports]] (2026-06) |
| `internal/modules/creditsale/` | Credit sales — **reuses the `sale` service** for pricing/stock — see [[Credit Sales]] |
| `internal/modules/promotion/` | Promotion rules + usage audit — see [[Promotions]] |
| `internal/modules/member/` | Store members, roles & status — see [[User Management]] |
| `internal/modules/location/` + `provisioning/` | Location hierarchy + default warehouse/sale-location provisioning — see [[Multi-Location Inventory]] |
| `internal/modules/stock_movement/` | Location-aware in/out/transfer/adjust movements |
| `internal/modules/stockcount/` | Physical stock-count sessions — see [[Inventory & Stock Counting]] |
| `internal/modules/payment/` | Dynamic PromptPay QR (`GET /…/promptpay-qr`) — see [[Customer Display]] |
| `internal/modules/warehouse_dashboard/` | Warehouse KPI aggregations — see [[Dashboard Command Center]] |

---

## JSONB columns — Valuer/Scanner pattern

Store struct ↔ JSONB by implementing `driver.Valuer` (`Value() → json.Marshal`) +
`sql.Scanner` (`Scan(any) → json.Unmarshal`, handle `[]byte` **and** `string`) on the type,
and tag the field `gorm:"type:jsonb"`. Read/write through that field — **never** raw
`Update("col", []byte)` / `Scan(&[]byte)` (→ `jsonb is of type bytea` error / unreliable scan).
Examples: `usersettings.CardSettings`, `receipt_settings.PaymentChannels`.

---

## Structured Error Format (added 2026-06-04)

`internal/platform/httpx/response.go` now returns `APIError{code, fields}`:

```go
// Single field validation
httpx.Err422(c, "name", "name is required")

// Multiple fields
httpx.ValidationError(c,
  httpx.FieldError{Field: "name", Message: "required"},
  httpx.FieldError{Field: "price", Message: "must be >= 0"},
)

// Other codes
httpx.ErrNotFound(c, "product not found")
httpx.ErrForbidden(c, "user cannot operate this store")
httpx.ErrConflict(c, "product is already in use")
httpx.ErrInternal(c)
```

JSON response:
```json
{ "success": false, "message": "validation failed",
  "error": { "code": "VALIDATION_ERROR", "fields": [{"field":"name","message":"required"}] } }
```

Each module's `writeXxxError()` maps sentinel errors to field names.

---

## Activity Log Middleware

Fiber after-middleware that auto-logs every successful POST/PUT/PATCH/DELETE scoped to `:storeID` routes.

```go
protected.Use(middleware.ActivityLog(deps.activityLogHandler.Service()))
```

Path parsing extracts `module` and `action` from URL without touching existing handlers.

---

## Backend Tests

Location: `internal/tests/` — 27 tests, all passing.

| Package | Tests |
|---------|-------|
| `tests/httpx` | Err422, ErrNotFound, ErrForbidden, ErrConflict, ErrInternal, multi-field |
| `tests/customer` | ValidateCustomerInput (name, email, level) |
| `tests/sale` | Sentinel error distinctness |
| `tests/purchasing` | Sentinel error distinctness |
| `tests/parkedbill` | note/customerID nullable constraints |
| `tests/document` | Sentinel errors non-nil/non-empty |
| `tests/product` | Image replace/delete safety |
| `tests/store` | Logo replace/delete safety |

---

## Related
- [[Backend Bug Fixes]] — all critical bugs and their fixes
- [[GORM Patterns]] — database query patterns
- [[Document PDF & WHT]] — document HTML/PDF/WHT rendering
- [[BFF Proxy Routes]] — how frontend connects to backend
- [[Backend Docs Map]] — documentation files for each module
