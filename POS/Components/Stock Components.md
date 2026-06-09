---
tags: [pos, component, stock, warehouse, products]
created: 2026-05-22
updated: 2026-05-27
---

# 📦 Stock Components

← [[POS Frontend MOC]] · [[POS Project MOC]]

## File Map

| File | Role |
|------|------|
| `components/stock/products-table.tsx` | Products table — Selling Price column shows `base_price` |
| `components/stock/warehouse-section.tsx` | Warehouse list + cross-store transfer |
| `components/stock/stock-levels-section.tsx` | Stock levels — xlsx import/export |
| `components/stock/catalog-setup-section.tsx` | Product types / units / brands tabs |

## products-table.tsx

- **Selling Price column** shows `base_price` (not `effective_price` or `selling_price`)
- Avatar initials with rotating `AVATAR_COLORS` (8 gradient combos)
- Hover-reveal action buttons: `opacity-0 group-hover:opacity-100`

## stock-levels-section.tsx — Excel

Migrated from CSV to Excel. See [[Stock Levels Excel]] for full details.
- Import: `import * as XLSX from "xlsx"`
- Export: `XLSX.writeFile(wb, "products-export-YYYY-MM-DD.xlsx")`
- Upload accepts: `.xlsx,.xls`

## catalog-setup-section.tsx

See [[Category Management]] for the full feature deep-dive.

Key patterns in this component:
- SSR safety: `useEffect(() => setMounted(true))` → all queries `enabled: mounted`
- Tab change / filter change → `useEffect([tab, search, statusFilter])` → reset page to 1
- Cache deduplication: same query keys as `StockManager` → no double requests

## warehouse-section.tsx

- Cross-store warehouse transfer UI
- Transfer creates cloned product chain in destination store
- See [[Backend Architecture]] for the transfer flow

## Stock module overhaul (2026-06-09, Wutthichai — `fix-kaew/dev`)

`products-table.tsx` rewritten into an **ERP/WMS table** (barcode+copy / stock-bar /
location / status columns, persisted density toggle, sticky header, floating bulk bar).
New siblings: `product-detail-view.tsx`, `product-card-grid.tsx`, `storage-assignment-card.tsx`,
`barcode-modal.tsx`, `barcode-batch-modal.tsx`. Inventory split into its own module.

See [[Barcode & Labels]] · [[Inventory & Stock Counting]].

## Related
- [[Category Management]] — full catalog-setup-section feature
- [[Stock Levels Excel]] — Excel import/export
- [[Barcode & Labels]] — barcode engine + ERP table
- [[Inventory & Stock Counting]] — /inventory module
- [[BFF Proxy Routes]] — stock proxy routes
- [[Backend Architecture]] — cross-store transfer flow
