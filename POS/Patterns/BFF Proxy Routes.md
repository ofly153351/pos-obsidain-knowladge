---
tags: [pos, pattern, bff, proxy, routes, nextjs, api]
created: 2026-05-22
updated: 2026-05-27
---

# 🔀 BFF Proxy Routes

← [[POS Frontend MOC]] · [[POS Backend MOC]] · [[POS Project MOC]]

Next.js Route Handlers in `app/api/stores/[storeId]/` proxy to Go backend at `http://localhost:8080` (env: `API_BASE_URL`).

**Key rule:** `storeId` in the BFF URL path is NOT forwarded to the backend as a path param. The backend resolves store access from the **JWT token** and resource ownership.

Proxy logic: `lib/api-proxy.ts` — strips `/api/stores/[storeId]/`, forwards `pos-access-token` cookie or `Authorization` header.

---

## Stock

| BFF (`/api/stores/:storeId/...`) | Go Backend (`/api/v1/...`) |
|----------------------------------|---------------------------|
| `POST /stock/add` | `POST /stock-movements/in` |
| `GET /stock/movements` | `GET /stock-movements` |

---

## Sales

| BFF | Backend |
|-----|---------|
| `* /sales/` | `* /sales/` |

---

## Receipts (Warehouse Receive)

| BFF (`/api/stores/:storeId/receipts/...`) | Go Backend (`/api/v1/warehouse/receipts/...`) |
|------------------------------------------|----------------------------------------------|
| `GET/POST /receipts` | `GET/POST /warehouse/receipts` |
| `GET/PUT /receipts/:id` | `GET/PUT /warehouse/receipts/:id` |
| `POST /receipts/:id/items` | `POST /warehouse/receipts/:id/items` |
| `POST /receipts/:id/confirm` | `POST /warehouse/receipts/:id/confirm` |
| `POST /receipts/:id/cancel` | `POST /warehouse/receipts/:id/cancel` |
| `POST /receipts/:id/attachments` ← plural | `POST /warehouse/receipts/:id/attachment` ← singular |
| `GET /receipts/:id/stock-impact` | `GET /warehouse/receipts/:id/stock-impact` |
| `GET /receipts/:id/print` | `GET /warehouse/receipts/:id/print` |
| `POST /receipts/generate-document-no` | `POST /warehouse/receipts/generate-document-no` |

> Note: BFF uses `attachments` (plural), backend uses `attachment` (singular). Route Handler translates.

---

## Receipt Settings

| BFF | Backend |
|-----|---------|
| `GET/PUT /receipt-settings` | `GET/PUT /api/v1/stores/:storeID/receipt-settings` |

---

## Suppliers / Purchasing

| BFF | Backend |
|-----|---------|
| `GET/POST /suppliers` | `GET/POST /suppliers` |
| `GET/PUT/DELETE /suppliers/:id` | `GET/PUT/DELETE /suppliers/:id` |
| `GET/POST /purchase-orders` | `GET/POST /purchase-orders` |

---

## Early Bugs Fixed

| Wrong BFF Path | Correct Path | Result |
|----------------|--------------|--------|
| `/stock/add` | `/stock-movements/in` | 404 → fixed |
| `/stock/movements` | `/stock-movements` | 404 → fixed |

See [[Backend Bug Fixes]].

---

## BFF Route Handler Boilerplate

```ts
// app/api/stores/[storeId]/receipt-settings/route.ts
import { authorizedProxy } from "@/lib/api-proxy";

export async function GET(req: Request, { params }: { params: { storeId: string } }) {
  return authorizedProxy(req, `/api/v1/stores/${params.storeId}/receipt-settings`);
}

export async function PUT(req: Request, { params }: { params: { storeId: string } }) {
  return authorizedProxy(req, `/api/v1/stores/${params.storeId}/receipt-settings`, "PUT");
}
```

---

## Related
- [[API Client Patterns]] — how frontend calls these routes
- [[Backend Architecture]] — Go Fiber route registration
- [[Backend Bug Fixes]] — wrong proxy paths
- [[Warehouse Receive Flow]] — receipts BFF usage
