---
tags: [pos, feature, activity-logs, audit, backend, frontend]
created: 2026-06-03
updated: 2026-06-04
---

# 📜 Activity Logs

← [[POS Features MOC]] · [[POS Backend MOC]] · [[POS Project MOC]]

ระบบบันทึก audit log ทุก action ของทุก module แยกตามร้านค้า

---

## Backend

### Migration

`init-db/016_activity_logs.sql`

```sql
CREATE TABLE activity_logs (
  id VARCHAR(30) PRIMARY KEY,
  store_id VARCHAR(30) NOT NULL REFERENCES stores(id) ON DELETE CASCADE,
  user_id VARCHAR(30) NOT NULL DEFAULT '',
  user_name VARCHAR(255) NOT NULL DEFAULT '',
  action VARCHAR(100) NOT NULL DEFAULT '',   -- create, update, delete, pay, cancel, …
  module VARCHAR(100) NOT NULL DEFAULT '',   -- product, sale, document, customer, …
  resource_id VARCHAR(30) NOT NULL DEFAULT '',
  method VARCHAR(10) NOT NULL DEFAULT '',
  path TEXT NOT NULL DEFAULT '',
  ip_address VARCHAR(50) NOT NULL DEFAULT '',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

Indexes on `store_id`, `created_at DESC`, `(store_id, module)`.

### Auto-Logging Middleware

`internal/middleware/activity_log.go`

- Fiber **after-middleware** — runs after handler via `c.Next()`
- Logs only successful responses (status < 400)
- Only logs mutating methods: POST, PUT, PATCH, DELETE
- Only logs routes with `:storeID` param
- Async goroutine — does not block response

```go
protected.Use(middleware.ActivityLog(deps.activityLogHandler.Service()))
```

Path parsing:
```
/api/v1/stores/{storeID}/products           POST  → module=product,   action=create
/api/v1/stores/{storeID}/products/{id}      PUT   → module=product,   action=update
/api/v1/stores/{storeID}/documents/{id}/pay POST  → module=document,  action=pay
/api/v1/stores/{storeID}/sales              POST  → module=sale,      action=create
```

Module map: `products→product`, `sales→sale`, `documents→document`, `customers→customer`, `stock-movements→stock`, `purchasing→purchasing`, etc.

### API

`GET /stores/:storeID/activity-logs`

Query params: `module`, `action`, `user_id`, `date_from`, `date_to`, `page`, `limit` (max 200)

---

## Frontend

### Page

`app/[locale]/(user)/(workspace)/settings/activity-logs/page.tsx` — server component  
`app/[locale]/.../settings/activity-logs/activity-logs-client.tsx` — client component

**Features:**
- Filter by module dropdown (11 options)
- Filter by action dropdown (13 options)
- Date range filter
- Pagination 50/page
- Auto-refresh every 30 seconds (`refetchInterval: 30_000`)
- Module badges (colored per module), Action badges (colored per action)

### Service

`services/activity-logs.ts` → BFF `app/api/stores/[storeId]/activity-logs/route.ts`

### i18n

Keys under `activityLogs` in `th.json` / `en.json`:
- `title`, `subtitle`, `refreshButton`, `filterLabel`
- `modules.*` — 11 module display names
- `actions.*` — 13 action display names
- `colUser`, `colModule`, `colAction`, `colTime`, `loading`, `empty`, `showing`

---

## Related
- [[Backend Architecture]] — middleware wiring
- [[BFF Proxy Routes]] — GET activity-logs route
- [[i18n Patterns]] — activityLogs dict section
