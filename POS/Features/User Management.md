---
tags: [pos, feature, users, members, permissions, roles, backend, frontend]
created: 2026-06-19
author: Wutthichai (fix-kaew/dev)
---

# 👥 User Management & Permissions

← [[POS Features MOC]] · [[Auth Pages]] · [[POS Project MOC]]

Store-scoped member roles & permissions (BE `d9b0180`, FE `c0ed329`).

## Backend (`internal/modules/member/`)
Endpoints (all store-scoped):
- `GET /…/members` · `POST /…/members` (add) · `PATCH /…/members/:userID` (update role/status) · `DELETE /…/members/:userID` (remove)

Migration `031_store_members_role_status.sql` adds **role + status** to `store_members`.
`internal/modules/auth/security.go` extended for permission checks. Cashier read-access to product catalog & tier discounts hardened separately (BE `7bfa210`).

## Frontend
- `/settings/staff` page — invite/manage members.
- `lib/use-store-role.ts` (~85 lines) — hook exposing the current user's role for the active store; gates UI.
- `lib/store-storage.ts` — persists active store role; `types/member.ts`, `services/members.ts`.

## Related
- [[Auth Pages]] · [[Settings Components]] · [[Activity Logs]]
