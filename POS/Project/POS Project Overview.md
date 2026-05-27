---
tags: [pos, project, setup, stack]
created: 2026-05-22
updated: 2026-05-27
---

# POS Project Overview

← [[POS Project MOC]]

## Paths

| Project | Path | Port |
|---------|------|------|
| Frontend | `/Users/obx/projects/pos-frontend` | `3000` |
| Backend  | `/Users/obx/projects/pos-backend`  | Go/Fiber `8080` |

**Branch:** `fix-of/dev`

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
│       ├── stock/
│       │   └── categories/
│       ├── warehouse/
│       │   └── receive/[id]/step-1|step-2|step-3|view
│       ├── purchasing/
│       └── settings/
└── (admin)/
```

## Recent Commits (`fix-of/dev`)

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
