---
tags: [pos, moc, frontend, components]
created: 2026-05-27
---

# 📦 POS Frontend — Component & Pattern Map

← [[POS Project MOC]]

## Shell & Navigation
- [[Layout Components]] — `user-workspace-layout`, sidebar (`bg-indigo-950`), topbar
- Active sidebar: `border-l-[3px] border-violet-400 bg-violet-900 text-white`

## Feature Components
| Component | File | Key Trait |
|-----------|------|-----------|
| [[Sales Cashier]] | `sales-manager.tsx` | `xl:grid-cols-[1fr_420px]`, VAT toggle, exact-amount |
| [[Stock Components]] | `catalog-setup-section.tsx` | 3-tab, client-side pagination |
| [[Purchasing Components]] | `supplier-manager.tsx` | 2-panel master-detail |
| [[Settings Components]] | `receipt-payment-settings.tsx` | live iframe preview |
| [[Toast Alert System]] | `ui/toast.tsx` + `ui/alert.tsx` | module-level store, no Context |

## Design
- [[POS Today Theme]] — complete token table (palette, inputs, buttons, badges)
- [[POS UI Mistakes]] — anti-patterns to catch in review

## Code Patterns
- [[Animation Patterns]] — `smooth-fade-up` for panels/modals, `fade-out` for exit, no slide-from-left
- [[i18n Patterns]] — 59 keys (categories), ~100 keys (supplier), dictionary type exports
- [[API Client Patterns]] — `useQuery` / `useMutation` setup, `authorizedApiRequest`
- [[BFF Proxy Routes]] — Route Handler boilerplate, storeId from JWT
