---
tags: [pos, feature, inventory, locations, warehouse, stock, backend, frontend]
created: 2026-06-19
author: Wutthichai (fix-kaew/dev)
---

# 📍 Multi-Location Inventory

← [[POS Features MOC]] · [[Inventory & Stock Counting]] · [[Backend Architecture]] · [[POS Project MOC]]

Location-aware stock system delivered across 2026-06-13…16 on `fix-kaew/dev`. Stock now lives at **specific locations** (warehouse / sale point), not just per-store totals.

## Provisioning (`internal/modules/provisioning/`)
- On store setup, auto-provisions a **default warehouse + default sale location** (`f8fcd68`, migration `037_default_warehouse_sale_location.sql`).
- Products carry a **default storage location** (`5c36098`, migration `034_products_default_location.sql`).

## Location module (`internal/modules/location/`)
Zone/floor hierarchy with CRUD:
- `POST/GET /…/locations` · `GET /…/locations/tree`
- `PATCH/DELETE /…/locations/zones` · `…/locations/floors` (rename/delete groups)
- `GET /…/locations/:locationID` · `…/:locationID/products` · `PATCH/DELETE /…/locations/:locationID`

## Location-aware movements (`internal/modules/stock_movement/`)
- `POST /…/stock-movements/in | out | transfer | adjust`, `GET /…/stock-movements`
- **Transfers** between locations (`872e551`, migration `040_stock_transfers.sql`).
- **Quick adjustments** are location-scoped with reason idempotency (`212bc61`, migration `038_stock_adjustment_reason_idempotency.sql`).
- Destructive warehouse/location stock deletes are **guarded/restricted** (`3dfd12d`/`3262a55`, migrations `036_restrict_warehouse_location_stock_deletes.sql`).

## Sales deduct from the sale location
POS sales deduct stock from the **assigned sale location** (`3ae2f4b`, migrations `041_sale_location_idempotency.sql`, `042_sale_status_constraint.sql`). The `sale` repository picks the sale-point location.

## Warehouse-scoped inventory view (`840131f`)
- `GET /…/warehouses/:warehouseID/inventory/products` — read-only split of **พร้อมขาย / พื้นที่จัดเก็บ / รวมในคลัง** (sellable / storage / total-in-warehouse).
- FE: redesigned warehouse inventory workspace (`8158acc`) — `components/warehouse/inventory/`, `services/warehouse-inventory.ts`, `types/warehouse-inventory.ts`.
- Migration `043_product_stock_aggregate_contract.sql` defines the stock-aggregate contract; backend audits hardened in `4d2bcf0`/`bef36b9`.

## Related
- [[Inventory & Stock Counting]] · [[Warehouse Receive Flow]] · [[Stock Components]] · [[GORM Patterns]]
