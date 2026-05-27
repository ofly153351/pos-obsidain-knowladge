---
tags: [pos, component, purchasing, supplier, purchase-order]
created: 2026-05-22
updated: 2026-05-27
---

# 🛍️ Purchasing Components

← [[POS Frontend MOC]] · [[POS Project MOC]]

## File Map

| File | Role |
|------|------|
| `components/purchasing/purchase-list.tsx` | PO list view |
| `components/purchasing/purchase-form.tsx` | Create PO form |
| `components/purchasing/receive-modal.tsx` | Receive stock into warehouse from PO |
| `components/purchasing/supplier-manager.tsx` | Supplier CRUD — 2-panel master-detail |
| `components/purchasing/add-supplier-modal.tsx` | Full-featured create supplier modal |

## supplier-manager.tsx

Full redesign → see [[Supplier Management]] for complete documentation.

**Quick reference:**
- 2-panel: left `w-[320px]` list + right `flex-1` detail
- Negative margin breakout: `-mx-6 -my-6 lg:-mx-8 lg:-my-8`
- Height: `calc(100dvh - 4.5rem)`
- Transition: `smooth-fade-up` on `<div key={selectedId}>` — NOT slide-from-left

## add-supplier-modal.tsx

- Width: `max-w-[780px] max-h-[92vh]`
- Accent bar: `h-1.5 bg-gradient-to-r from-violet-500 via-violet-600 to-pink-500`
- Two-column body: `grid grid-cols-2 divide-x divide-slate-100`
- Payment method: PromptPay / Bank Account segmented toggle
- Credit term: pill buttons (0 / 7 / 15 / 30 / 45 / custom days)
- Exit: `isClosing` state → `fade-out` → `onAnimationEnd` → `onClose()`

## services/suppliers.ts

`CreateSupplierInput` extended with:
```ts
line_id?: string;
email?: string;
payment_method?: "promptpay" | "bank_account";
promptpay_number?: string;
bank_name?: string;
bank_account_number?: string;
bank_account_name?: string;
credit_days?: number;
```

## Related
- [[Supplier Management]] — full feature deep-dive
- [[Animation Patterns]] — modal/panel transition classes
- [[i18n Patterns]] — ~100 keys for supplier UI + add modal
- [[BFF Proxy Routes]] — purchasing proxy routes
