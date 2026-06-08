---
tags: [pos, component, sales, cashier, cart, quotation, invoice]
created: 2026-05-22
updated: 2026-06-01
---

# 🛒 Sales / Cashier Component

← [[POS Frontend MOC]] · [[POS Project MOC]]

## Files (post-refactor)

`sales-manager.tsx` was split from 2851 → 1273 lines into 14 focused files:

| File | Role |
|------|------|
| `sales-manager.tsx` | Orchestrator: state, effects, API calls, wiring |
| `checkout-summary-modal.tsx` | Payment / invoice / quotation modal |
| `cart-panel.tsx` | Cart display + header |
| `receipt-preview-modal.tsx` | Print preview iframe |
| `post-invoice-modal.tsx` | "สร้างใบส่งของ" confirmation popup |
| `actions-menu-modal.tsx` | Hold/restore/note/clear menu |
| `parked-bills-drawer.tsx` | Parked bills list + restore confirm |
| `hold-bill-modal.tsx` | Hold bill label input |
| `quantity-numpad.tsx` | Numpad for qty |
| `amount-numpad.tsx` | Numpad for amount/discount |
| `discount-editor-modal.tsx` | Per-item discount editor |
| `product-popup.tsx` | Long-press product detail |
| `use-numpad.ts` | Custom hook: numpad state + handlers |
| `utils/sales-calculations.ts` | Pure: formatCurrency, roundCurrency, getCartLine, etc. |
| `utils/numpad-utils.ts` | Pure: digit, backspace, decimal string ops |
| `product-browser.tsx` | Product grid/list with category filter |
| `cashier-modal.tsx` | Full-screen overlay mounting the above |

## Layout

```
xl:grid-cols-[1fr_420px]
├── ProductBrowser   — grid or list, category filter pills
└── CartPanel        — line items, VAT toggle, checkout button
```

## localStorage Keys

| Key | Default | Purpose |
|-----|---------|---------|
| `pos-sales-product-view` | `"grid"` | View preference |

## Checkout Modal — Settlement Modes

### ชำระทันที (เงินสด/บัตร)
Normal payment flow → `createSale()` → receipt preview → optional print.

### เปิดบิลเชื่อ (credit/invoice)
Available only for network customers (`selectedCustomerId` set).
- Creates INVOICE document via `createDocument`
- **Post-invoice popup** appears → option to create ใบส่งของ (DO)
- If DO created: `convertToDeliveryOrder(invoiceId)` → stores `source_document_id`
- PayInvoice on this invoice will skip TAX_INVOICE when DO exists

Due date presets: quick pills [7 / 14 / 30 / 45 วัน] + custom date input. Same pattern as [[Document Management]] invoice due-date.

### ใบเสนอราคา (quotation mode)
Toggle button in checkout modal header — violet solid when active:
- Hides payment fields (method, amount, quick-cash)
- Shows **ยืนราคาถึง** presets (7/14/30/45 วัน) + date picker + notes
- "ตกลง" → "สร้างใบเสนอราคา" → `createDocument({ type: "QUOTATION", valid_until, ... })`
- Cart cleared + modal closed on success

## Quotation toggle button pattern
```tsx
<button
  onClick={() => { setQuotationMode(m => !m); setQuotationValidUntil(""); }}
  className={quotationMode
    ? "border-violet-600 bg-violet-600 text-white"
    : "border-violet-200 bg-violet-50 text-violet-700 hover:bg-violet-100"}
>
  <FileText /> ใบเสนอราคา
</button>
```

## VAT Toggle
Always-visible in cart panel header. Toggle → recalculates live.

## Exact Amount (ยอดเงินพอดี)
`applyQuickCash(amount, isExact=true)` — sets paid = totalBill exactly, idempotent.

## Related
- [[Document Management]] — invoice settlement, quotation, delivery order
- [[Document PDF & WHT]] — DO template, bank accounts in payment box
- [[Toast Alert System]] — checkout success/error notifications
- [[BFF Proxy Routes]] — `/sales`, `/documents/convert-do`
