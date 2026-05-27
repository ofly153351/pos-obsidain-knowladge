---
tags: [pos, backend, docs, documentation]
created: 2026-05-22
---

# 📚 Backend Docs Map

← [[POS Backend MOC]] · [[POS Project MOC]]

All docs live at `/Users/obx/projects/pos-backend/docs/` and must stay in **English**.

## Rule: Keep Docs Updated

- Backend module changed → update its `docs/*.md` file
- New endpoint → document in the relevant `docs/` file
- New API → add to `docs/api/` subfolder

---

## Module → Doc File

| Module | Doc File |
|--------|----------|
| Auth | `docs/auth.md` |
| Store | `docs/store.md` |
| Product | `docs/product.md`, `docs/product-structure.md` |
| Product types/units/brands | `docs/product-units.md`, `docs/product-brands.md` |
| Sales / Cashier | `docs/sales.md`, `docs/vat.md` |
| Parked bills | `docs/parked-bills.md` |
| Warehouse | `docs/warehouse.md` |
| Stock movements | `docs/stock-movements.md` |
| Purchasing / PO | `docs/purchasing.md` |
| Customer network | `docs/customer-network.md` |
| Dashboard | `docs/dashboard.md` |
| Invoice | `docs/invoice.md` |
| Subscription | `docs/subscription.md` |
| Admin | `docs/admin.md`, `docs/admin-subscription.md` |
| Architecture | `docs/api-development.md`, `docs/flow.md` |

## API Docs Subfolder

| File | Covers |
|------|--------|
| `docs/api/warehouse-dashboard.md` | `GET /api/v1/stores/:storeID/dashboard/warehouse` — 6 response sections, alert levels, movement types, period mapping |

## New Docs Created (2026-05-22)

| File | Covers |
|------|--------|
| `docs/warehouse.md` | Warehouse CRUD, cross-store transfer, clone behavior |
| `docs/stock-movements.md` | in/out/transfer/adjust, auto-location, BFF proxy map |
| `docs/purchasing.md` | Suppliers, supplier-products, PO lifecycle |
| `docs/parked-bills.md` | Hold bill CRUD, cart snapshot, no stock deduction |

## Related
- [[Backend Architecture]] — module list
- [[BFF Proxy Routes]] — route documentation
