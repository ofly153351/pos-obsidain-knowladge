---
tags: [pos, component, settings, store]
created: 2026-05-22
updated: 2026-05-27
---

# ⚙️ Settings Components

← [[POS Frontend MOC]] · [[POS Project MOC]]

## File Map

| File | Role |
|------|------|
| `components/store/store-management-panel.tsx` | Two-panel: store list left + edit form right |
| `components/settings/receipt-payment-settings.tsx` | Receipt design + payment channels |

## store-management-panel.tsx

Two-panel layout (same pattern as [[Supplier Management]]):
- Left: store list
- Right: edit form for selected store

**Selected store card:**
- Background: `from-violet-600 to-violet-700` gradient
- Text: `text-white` — NOT `text-violet-900` or `text-violet-700`

**Switch store button:** gradient CTA with `text-white` (see [[POS UI Mistakes]])

## receipt-payment-settings.tsx

Full feature documentation → [[Receipt Payment Settings]]

**Quick reference:**
- Queries: `["receipt-settings", storeId]` + `["store", storeId]`
- Preview iframe uses `RenderPanelPreviewHTML()` (backend) — no toolbar, minimal scrollbar
- Payment channels stored as JSONB, merged with `CHANNEL_META` for icons/names on frontend
- Toggle switch must use `left-0.5 top-0.5` + `translate-x-0`/`translate-x-5` (see [[POS UI Mistakes]])

## Related
- [[Receipt Payment Settings]] — complete feature docs
- [[POS Today Theme]] — store card gradient tokens
- [[BFF Proxy Routes]] — receipt-settings proxy
