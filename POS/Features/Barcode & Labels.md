---
tags: [pos, feature, barcode, label, qr, pdf, print, frontend]
created: 2026-06-09
author: Wutthichai (fix-kaew/dev)
---

# 🏷️ Barcode Center & Label Printing

← [[POS Features MOC]] · [[Stock Components]] · [[POS Project MOC]]

Self-contained barcode/label engine — **no new npm deps** (hand-rolled). Added 2026-06-09 by Wutthichai, branch `fix-kaew/dev`.

## Engine (lib/)
- `lib/barcode.ts` — CODE128 / EAN-13 / EAN-8 / UPC-A generators.
- `lib/qr.ts` — QR generator.
- `lib/label.ts` — shared label layout + print window.
- `lib/label-raster.ts` — `foreignObject` → PNG raster.
- `lib/pdf.ts` — hand-rolled multi-page PDF (no library).

## UI
- `components/stock/barcode-modal.tsx` — single-product label.
- `components/stock/barcode-batch-modal.tsx` — **Barcode Center**: batch select,
  content flags, print quantity, printer modes, A4 layouts, live label preview.
- Export: foreignObject → PNG/PDF.

## Products table → ERP/WMS view
`components/stock/products-table.tsx` overhauled into an ERP-style table:
- Dedicated columns: **Barcode (+copy)**, stock-bar, **Location**, status.
- **Density toggle** (persisted), sticky header, floating sticky bulk-action bar.
- `product-detail-view.tsx` — full product detail page.
- `storage-assignment-card.tsx` — assign storage location.
- `product-card-grid.tsx` — grid variant for the stock module.

## i18n
- ~119 new keys each in `th.json` / `en.json`.

## Related
- [[Stock Components]] · [[Inventory & Stock Counting]] · [[Category Management]]
