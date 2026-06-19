---
tags: [pos, feature, customers, crm, backend, frontend]
created: 2026-06-19
author: Wutthichai (fix-kaew/dev)
---

# 🧑‍🤝‍🧑 Customer Management

← [[POS Features MOC]] · [[Credit Sales]] · [[POS Project MOC]]

Customer page redesign + tax fields (FE `bbd9af9`).

## What changed
- Added **tax ID + branch** fields to customers (migration `029_customers_tax_branch.sql`); `types/customer.ts`.
- Redesigned `/customers` page; links out to [[Credit Sales]] (VIP/credit) entry points.
- Customer **hierarchy** (parent/child) exposed via BFF: `…/customers/:customerId/children`, `…/customers/:customerId/tree`.
- Member-tier discounts resolved server-side by the sale pipeline (`customer.NewSaleBenefitResolver`).

## Related
- [[Credit Sales]] · [[Promotions]] · [[Sales Cashier]]
