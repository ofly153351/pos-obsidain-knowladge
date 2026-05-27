---
tags: [pos, backend, go, fiber, architecture, gorm]
created: 2026-05-22
updated: 2026-05-27
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
| `internal/platform/receipthtml/` | Receipt HTML rendering (`RenderPanelPreviewHTML`) |

## Related
- [[Backend Bug Fixes]] — all critical bugs and their fixes
- [[GORM Patterns]] — database query patterns
- [[BFF Proxy Routes]] — how frontend connects to backend
- [[Backend Docs Map]] — documentation files for each module
