---
tags: [pos, feature, inventory, stock, counting, frontend]
created: 2026-06-09
author: Wutthichai (fix-kaew/dev)
---

# 📋 Inventory & Stock Counting

← [[POS Features MOC]] · [[Stock Components]] · [[POS Project MOC]]

Dedicated **inventory module** split out from the product catalog (added 2026-06-09 by Wutthichai, branch `fix-kaew/dev`). Inventory = stock truth; the product module = catalog/pricing. They no longer share a screen.

## Routes / files
- `/inventory` → `components/stock/inventory-manager.tsx`
- `/inventory/counts` → `components/stock/stock-count-manager.tsx`
- `components/stock/inventory-types.ts`
- `components/stock/stock-adjust-drawer.tsx`
- nav: sidebar + `nav-config.tsx` point Inventory submenu to `/inventory`

## InventoryManager
- **KPI dashboard:** total products, low-stock, out-of-stock, stock value, movements today.
- **Inventory table:** current stock + bar, min, status (incl. **overstock**), storage hierarchy, last movement.
- No price / edit / delete here — purely stock-focused.

## StockAdjustDrawer
- Modes: **Receive / Decrease / Set-exact**, with calculation preview.
- Required reason + optional reference, two-step confirm.
- Writes via `addStock` / `adjustStock` → every change records a **movement log**.

## Stock counting (`/inventory/counts`)
- `stock-count-manager.tsx` — physical count session workflow (count vs system, variance).

## i18n
- ~145 new keys each in `th.json` / `en.json` (inventory + counting).

## Related
- [[Stock Components]] · [[Barcode & Labels]] · [[Warehouse Receive Flow]]
