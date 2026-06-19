---
tags: [pos, feature, promotions, sales, backend, frontend]
created: 2026-06-19
author: Wutthichai (fix-kaew/dev)
---

# 🎯 Promotions

← [[POS Features MOC]] · [[Sales Cashier]] · [[POS Project MOC]]

Promotion engine (BE `48e7393`, FE `321e0e0`/`bbd9af9`).

## Frontend
- `/promotions` page.
- `components/promotions/promotion-engine.ts` — client-side rule evaluation applied to the cart.
- `components/promotions/promotion-types.ts` (~380 lines of rule/condition types).

## Backend endpoints (`internal/modules/promotion/`)
- `GET/POST /…/promotions`
- `PUT /…/promotions/:promotionID`
- `DELETE /…/promotions/:promotionID`

Migrations: `024_promotions.sql`, `027_promotions_audit.sql`, `028_promotion_usages.sql` (usage tracking + audit).

## Related
- [[Credit Sales]] · [[Customer Management]] · [[Sales Cashier]]
