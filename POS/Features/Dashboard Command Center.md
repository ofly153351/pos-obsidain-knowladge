---
tags: [pos, feature, dashboard, kpi, frontend]
created: 2026-06-19
author: Wutthichai (fix-kaew/dev)
---

# 📊 Dashboard Command Center

← [[POS Features MOC]] · [[Layout Components]] · [[POS Project MOC]]

Dashboard redesigned into a "command center" (FE `2071e7d`, with warehouse KPI expansion BE `dd29120`).

## What changed
- `/dashboard` redesigned: chart fix + consolidated stock widget + expense widget (`155a2ee`).
- Warehouse dashboard KPI expansion + `COUNT_CORRECTION` stock-movement reason + UTF-8 encoding fix (BE `dd29120`, `internal/modules/warehouse_dashboard/`).
- BFF: `…/dashboard/route.ts`, `…/dashboard/warehouse/route.ts`.
- Notifications: clickable items **deep-link** to filtered pages (FE `2288f18`).
- Sidebar: independently scrollable nav area, notification dropdown, profile menu redesign (FE `4a814a9`/`6e2fc56`).

## Related
- [[Finance & Reports]] · [[Layout Components]] · [[Multi-Location Inventory]]
