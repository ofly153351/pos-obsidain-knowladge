---
tags: [pos, component, sales, cashier, cart]
created: 2026-05-22
---

# 🛒 Sales / Cashier Component

← [[POS Frontend MOC]] · [[POS Project MOC]]

## Files

| File | Role |
|------|------|
| `components/sales/sales-manager.tsx` | Main cashier UI — cart, checkout, numpad |
| `components/sales/product-browser.tsx` | Product grid/list with category filter pills |
| `components/sales/cashier-modal.tsx` | Full-screen overlay that mounts the above |

## Layout

```
xl:grid-cols-[1fr_420px]
├── Product browser (flex-1)   — grid or list view, category filter pills
└── Cart panel (420px fixed)   — line items, VAT toggle, numpad, checkout
```

## localStorage Keys

| Key | Type | Default | Purpose |
|-----|------|---------|---------|
| `pos-sales-product-view` | `"grid" \| "list"` | `"grid"` | View preference across sessions |
| `pos-sales-apply-vat` | `"true" \| "false"` | `"true"` | VAT on/off state |

## VAT Toggle

Always-visible button in cart panel header.

```tsx
{/* ON */}
<button className="bg-violet-600 text-white">VAT 7%</button>
{/* OFF */}
<button className="border-violet-200 text-violet-400">VAT 7%</button>
```

## Exact Amount Button (ยอดเงินพอดี)

`applyQuickCash(amount, isExact=true)` — always sets paid amount to bill total.
- Idempotent: clicking multiple times does NOT accumulate
- Sets `paid === totalBill` exactly (not `+= totalBill`)

## Category Filter Pills

Active pill: `bg-violet-600 text-white rounded-xl`  
Inactive: `border-violet-200 text-violet-600 hover:bg-violet-50`

## Related
- [[POS Today Theme]] — button/pill styles
- [[Toast Alert System]] — checkout success/error notifications
- [[BFF Proxy Routes]] — `/sales` proxy
