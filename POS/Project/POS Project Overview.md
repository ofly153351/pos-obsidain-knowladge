---
tags: [pos, project, setup, stack]
created: 2026-05-22
updated: 2026-06-19
---

# POS Project Overview

← [[POS Project MOC]]

## Paths

| Project | Path | Port |
|---------|------|------|
| Frontend | `/Users/obx/projects/pos-frontend` | `3000` |
| Backend  | `/Users/obx/projects/pos-backend`  | Go/Fiber `8080` |

**Branch:** `fix-kaew/dev` (active — Wutthichai). Older work landed on `fix-of/dev`.

## Stack

| Layer | Tech |
|-------|------|
| Framework | Next.js 15 App Router (`app/[locale]/`) |
| Language | TypeScript strict |
| Styling | Tailwind CSS 4 (`@theme inline`) |
| Data fetching | TanStack Query (`useQuery`, `useMutation`) |
| Global state | Zustand |
| i18n | `locales/th.json` + `locales/en.json` |
| API proxy | BFF: `app/api/stores/[storeId]/...` → Go backend |
| Backend | Go Fiber v2 + GORM v2 + PostgreSQL |

See: [[BFF Proxy Routes]] · [[i18n Patterns]]

## App Router Folder Structure

```
app/[locale]/
├── (user)/
│   └── (workspace)/
│       ├── sales/
│       ├── dashboard/
│       ├── products/ (categories/)        ← catalog (new /products route, 28ea8d1)
│       ├── inventory/ (counts/)           ← stock truth + counting
│       ├── stock/ (categories/ warehouses/)
│       ├── warehouse/
│       │   ├── overview/
│       │   └── receive/[id]/step-1|step-2|step-3|view
│       ├── purchases/ (suppliers/)        ← purchasing+receiving unified (b08be4d)
│       ├── credit-sales/                  ← ขายเชื่อ/ยืมสินค้า
│       ├── promotions/
│       ├── customers/
│       ├── finance/ (expenses/ pnl/)
│       ├── reports/ (summary/ inventory-value/)
│       ├── profile/
│       ├── documents/ (pending/)
│       └── settings/ (staff/ receipt-payment/ storage-locations/ activity-logs/)
├── customer-display/                      ← second screen (outside workspace)
├── print/ (invoice/[saleId]  document/[documentId])
└── (admin)/
```

## Recent Commits

### `fix-kaew/dev` — major delivery 2026-06-09 → 06-16 (Wutthichai)
A large multi-feature batch. New feature notes: [[Finance & Reports]], [[Credit Sales]], [[Promotions]], [[User Management]], [[Customer Display]], [[Multi-Location Inventory]], [[Customer Management]], [[Dashboard Command Center]].

| Area | FE / BE commits |
|------|-----------------|
| Multi-location inventory | FE `5f1692f`/`2fd42c7`/`034265e`/`115ed2e`/`98c81c9`/`8158acc` · BE `f8fcd68`/`212bc61`/`872e551`/`3ae2f4b`/`840131f`/`3262a55` |
| Finance & Reports | FE `481e3a6`/`1e74e34` · BE `8b40faa`/`43edea2`/`330f8e9` |
| Credit sales + Promotions + Stock count | FE `649e9ab`/`321e0e0` · BE `48e7393` |
| User management & permissions | FE `c0ed329` · BE `d9b0180`/`7bfa210` |
| Customer display (screen 2) | FE `b70ffb4` · BE `e7216e4` |
| Dashboard command center | FE `2071e7d`/`155a2ee`/`4a814a9` · BE `dd29120` |
| Customers redesign | FE `bbd9af9` |
| Products /products route + module overhaul | FE `28ea8d1`/`a93b8d6` |
| Purchasing+receiving unified workspace | FE `b08be4d`/`fb37eca`/`fa71115` · BE `cd64dd1`/`79848f4` |
| Profile/account settings | FE `1e85949` |
| Security / production hardening | FE `5bf31e2`/`e435f53` · BE `45408fa`/`c296d75` |

### `fix-of/dev` (earlier)

> Document module shipped 2026-05-29: see [[Document Management]] + [[Document PDF & WHT]]. FE: `a25b206` (page overhaul + receipt panel), `02d371e` (types/locales). BE: `4daaaf3` (PDF/WHT/§86·4/Bill), `51086e4` (handler/logo/routing), `db4131f` (store tax_id migration).

| Commit | Description |
|--------|-------------|
| `4c847df` | Upscale supplier UI + standard fade transitions |
| `6094217` | Redesign supplier management 2-panel master-detail |
| `644c5e4` | Receipt & payment settings with live backend preview |
| `e4c1518` | Storage location UI overhaul + global button style |
| `76ed465` | Storage location routing + sidebar nav improvements |
| `2ac279c` | Selling Price shows base_price |
| `13a9f8d` | VAT toggle as persistent header button |
| `d80ae67` | Replace all sky-\* with explicit violet-\* |
| `0ae80cf` | POS Today design system + AGENTS.md |

## Key Rules (AGENTS.md)
- No new libraries (no Framer Motion, no new npm packages)
- All text via i18n — never hardcode strings
- Use `toast.*` / `<Alert>` — never inline `border-rose-200` divs
- `button:not(:disabled) { cursor: pointer }` is global — do NOT add `cursor-pointer` to individual buttons

## Related
- [[POS Dev Commands]]
- [[POS Today Theme]]
- [[Backend Architecture]]
- [[BFF Proxy Routes]]
