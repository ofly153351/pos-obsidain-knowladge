---
tags: [pos, feature, credit-sales, sales, backend, frontend]
created: 2026-06-19
author: Wutthichai (fix-kaew/dev)
---

# 🧾 Credit Sales (ขายเชื่อ / ยืมสินค้า)

← [[POS Features MOC]] · [[Sales Cashier]] · [[POS Project MOC]]

Credit-sale / goods-loan feature (BE `48e7393`, FE `649e9ab`/`321e0e0`). A credit sale is a **genuine product-backed sale** — it reuses the real sale pipeline, not a parallel one.

## Key architecture
`internal/app/creditsale.go` builds the handler by **reusing the `sale` service** (server-authoritative pricing, customer benefit resolution, stock deduction). So a credit sale deducts stock and resolves member discounts exactly like a cash sale; only payment/settlement differs.

## Backend endpoints (`internal/modules/creditsale/`)
- `GET /…/credit-sales/summary` (registered before `/:creditSaleID`)
- `GET/POST /…/credit-sales`
- `GET /…/credit-sales/:creditSaleID`
- `POST /…/credit-sales/:creditSaleID/payments` — record a repayment
- `POST /…/credit-sales/:creditSaleID/cancel`
- `GET /…/credit-sales/:creditSaleID/statement`

Migrations: `023_credit_sales.sql`, `026_credit_payments_store_fk.sql`.

## Frontend
- `/credit-sales` page → `components/.../credit-sales` + `types/credit-sale.ts`.
- VIP KPI card + button wiring; entry point also linked from the customers page.
- Services: `services/credit-sales.ts`.

## Related
- [[Promotions]] · [[Customer Management]] · [[Sales Cashier]]
