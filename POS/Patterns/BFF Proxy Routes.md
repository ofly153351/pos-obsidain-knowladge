---
tags: [pos, pattern, bff, proxy, routes, nextjs, api]
created: 2026-05-22
updated: 2026-06-19
---

# 🔀 BFF Proxy Routes

← [[POS Frontend MOC]] · [[POS Backend MOC]] · [[POS Project MOC]]

Next.js Route Handlers in `app/api/stores/[storeId]/` proxy to Go backend at `http://localhost:8080` (env: `API_BASE_URL`).

Proxy logic: `lib/api-proxy.ts` — forwards `pos-access-token` cookie as `Authorization: Bearer` header.

**Body forwarding rule (updated 2026-06-04):**
- **JSON body:** `await request.text()` → pass as `body` string
- **File upload (multipart):** `await request.blob()` → pass as `body` Blob — **never use `request.text()` for multipart**, it destroys the binary boundary
- **Raw passthrough:** `request.body ?? undefined` (ReadableStream) — may have Node.js duplex issues, prefer `blob()`

---

## Documents

| BFF (`/api/stores/:storeId/documents/...`) | Backend (`/api/v1/stores/:storeID/documents/...`) |
|--------------------------------------------|--------------------------------------------------|
| `GET/POST /documents` | `GET/POST .../documents` |
| `GET/DELETE /documents/:id` | `GET/DELETE .../documents/:docID` |
| `PUT /documents/:id/status` | `PUT .../documents/:docID/status` |
| `POST /documents/bulk` | `POST .../documents/bulk` |
| `GET /documents/:id/print` | `GET .../documents/:docID/print` → HTML |
| `GET /documents/:id/pdf` | `GET .../documents/:docID/pdf` → PDF |
| `GET /documents/:id/wht-cert` | `GET .../documents/:docID/wht-cert` → HTML |
| `GET /documents/statement` | `GET .../documents/statement` → PDF |
| `POST /documents/:id/pay` | `POST .../documents/:docID/pay` |
| `POST /documents/:id/convert-tax` | `POST .../documents/:docID/convert-tax` |
| `POST /documents/:id/convert-do` | `POST .../documents/:docID/convert-do` ← ใบส่งของ |
| `POST /documents/:id/convert` | `POST .../documents/:docID/convert` ← quotation→invoice |

---

## Store

| BFF | Backend |
|-----|---------|
| `GET /stores/:storeId` | `GET /api/v1/stores/:storeID` |
| `PUT /stores/:storeId` | `PUT /api/v1/stores/:storeID` (FormData — logo upload) |
| `GET/POST /stores/:storeId/bank-accounts` | `GET/POST .../bank-accounts` |
| `DELETE /stores/:storeId/bank-accounts/:accountId` | `DELETE .../bank-accounts/:accountID` |

---

## Sales

| BFF | Backend |
|-----|---------|
| `* /sales/` | `* /sales/` |

---

## Receipts (Warehouse)

| BFF (`/receipts/...`) | Backend (`/warehouse/receipts/...`) |
|-----------------------|--------------------------------------|
| `GET/POST /receipts` | `GET/POST /warehouse/receipts` |
| `GET/PUT /receipts/:id` | `GET/PUT /warehouse/receipts/:id` |
| `POST /receipts/:id/confirm` | `POST /warehouse/receipts/:id/confirm` |
| `POST /receipts/:id/cancel` | `POST /warehouse/receipts/:id/cancel` |
| `POST /receipts/:id/attachments` | `POST /warehouse/receipts/:id/attachment` |
| `GET /receipts/:id/print` | `GET /warehouse/receipts/:id/print` |

---

## Receipt Settings

| BFF | Backend |
|-----|---------|
| `GET/PUT /receipt-settings` | `GET/PUT .../receipt-settings` |

---

## Stock

| BFF | Backend |
|-----|---------|
| `POST /stock/add` | `POST /stock-movements/in` |
| `POST /stock/out` | `POST /stock-movements/out` |
| `POST /stock/adjust` | `POST /stock-movements/adjust` |
| `GET /stock/movements` | `GET /stock-movements` |
| `POST /stock-movements/transfer` | `POST /stock-movements/transfer` (location→location) |
| `GET /stock/products/:productId` | `GET /stock/products/:productID` |

---

## Locations & Warehouse Inventory (2026-06, [[Multi-Location Inventory]])

| BFF | Backend |
|-----|---------|
| `GET/POST /locations` · `GET /locations/tree` | location CRUD + hierarchy |
| `GET/PATCH/DELETE /locations/:locationId` (+ `/products`) | per-location |
| `PATCH/DELETE /locations/zones` · `/locations/floors` | rename/delete groups |
| `GET /warehouses/:warehouseId/inventory/products` | sellable/storage/total split |
| `GET /warehouses/:warehouseId/inventory` · `POST .../inventory/:productId/allocate` | cross-store transfers awaiting allocation |
| `POST /warehouses/:warehouseId/transfer` | warehouse transfer |

---

## Finance & Reports (2026-06, [[Finance & Reports]])

| BFF | Backend |
|-----|---------|
| `GET/POST /expenses` · `GET/PATCH/DELETE /expenses/:expenseId` · `GET /expenses/summary` | expense module |
| `GET/POST /expense-categories` · `PATCH/DELETE /expense-categories/:categoryId` | categories |
| `GET /finance/pnl` · `/finance/summary` · `/finance/inventory` | finance aggregations |

---

## Credit Sales · Promotions · Stock Count · Members · Customers · QR (2026-06)

| BFF | Feature |
|-----|---------|
| `/credit-sales` (+ `/summary`, `/:id`, `/:id/payments`, `/:id/cancel`, `/:id/statement`) | [[Credit Sales]] |
| `/promotions` (+ `/:promotionId`) | [[Promotions]] |
| `/stock-count-sessions` (+ `/:sessionId`, `/:sessionId/apply`) | [[Inventory & Stock Counting]] |
| `/members` (+ `/:userId`) | [[User Management]] |
| `/customers/:customerId/children` · `/tree` | [[Customer Management]] |
| `/promptpay-qr` | [[Customer Display]] (dynamic PromptPay QR) |

---

## BFF Route Handler Boilerplate

```ts
// GET — no body needed
export async function GET(request: Request, { params }: Ctx) {
  const { storeId } = await params;
  return proxyApiRequest(request, `/api/v1/stores/${storeId}/documents`);
}

// POST with JSON body
export async function POST(request: Request, { params }: Ctx) {
  const { storeId } = await params;
  const body = await request.text();
  return proxyApiRequest(request, `/api/v1/stores/${storeId}/bank-accounts`, { method: "POST", body });
}

// POST/PUT with file upload (multipart) — use blob(), never text()
export async function POST(request: Request, { params }: Ctx) {
  const { storeId } = await params;
  const body = await request.blob(); // ← binary-safe; preserves multipart boundary
  return proxyApiRequest(request, `/api/v1/stores/${storeId}/suppliers`, { method: "POST", body });
}

// PUT with FormData re-constructed (store logo via formData() → FormData obj)
export async function PUT(request: Request, { params }: Ctx) {
  const { storeId } = await params;
  const formData = await request.formData();
  return proxyApiRequest(request, `/api/v1/stores/${storeId}`, { method: "PUT", body: formData });
}
```

### Activity Logs

| BFF | Backend |
|-----|---------|
| `GET /stores/:storeId/activity-logs` | `GET .../activity-logs?page&limit&module&action&date_from&date_to` |

---

## Related
- [[API Client Patterns]] — how frontend calls these routes
- [[Backend Architecture]] — Go Fiber route registration
- [[Document Management]] — document action routes
- [[Document PDF & WHT]] — print/pdf/wht-cert routes
