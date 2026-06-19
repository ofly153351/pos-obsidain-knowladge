---
tags: [pos, feature, finance, reports, expenses, pnl, backend, frontend]
created: 2026-06-19
author: Wutthichai (fix-kaew/dev)
---

# 💰 Finance & Reports

← [[POS Features MOC]] · [[Backend Architecture]] · [[POS Project MOC]]

Finance & Reports module shipped on `fix-kaew/dev` (BE `8b40faa`/`43edea2`/`330f8e9`, FE `481e3a6`). Two backend modules — **`expense`** (records) and **`finance`** (aggregations) — feed four FE pages.

## Frontend pages
| Route | Purpose |
|-------|---------|
| `/finance/expenses` | Expense entry + expense categories CRUD |
| `/finance/pnl` | Profit & Loss statement |
| `/reports/summary` | Sales dashboard summary |
| `/reports/inventory-value` | Inventory valuation + dead-stock |

Types: `components/finance/finance-types.ts`, `components/finance/pnl-types.ts`, `components/reports/reports-types.ts`, `components/reports/summary-types.ts`.

## Backend endpoints
**Expenses** (`internal/modules/expense/`):
- `GET/POST /stores/:storeID/expenses` · `GET/PATCH/DELETE /…/expenses/:expenseID`
- `GET /…/expenses/summary` (register **before** `/:expenseID` so Fiber matches it first)
- `GET/POST /…/expense-categories` · `PATCH/DELETE /…/expense-categories/:categoryID` (delete = deactivate)

**Finance** (`internal/modules/finance/`):
- `GET /…/finance/pnl` · `GET /…/finance/summary` · `GET /…/finance/inventory`

## Notes
- `025_sale_items_unit_cost.sql` stores unit cost per sale item → enables true COGS / margin in P&L.
- Inventory report includes a **per-product dead-stock breakdown** (`330f8e9`).
- Dashboard expense widget surfaces recent expenses (FE `155a2ee`).
- Migrations: `020_expenses.sql`, `021_normalize_expense_payment_methods.sql`.

## Related
- [[Dashboard Command Center]] · [[Multi-Location Inventory]] · [[BFF Proxy Routes]]
