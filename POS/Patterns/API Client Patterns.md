---
tags: [pos, pattern, api, typescript, tanstack-query, axios]
created: 2026-05-24
---

# 🌐 API Client Patterns

← [[POS Backend MOC]] · [[POS Frontend MOC]] · [[POS Project MOC]]

---

## Backend Response Envelope

All Go backend responses wrap data in:

```json
{ "success": true,  "message": "OK",    "data": <payload> }
{ "success": false, "message": "Error", "error": <detail> }
```

File: `pos-backend/internal/platform/httpx/response.go`

---

## Two Request Helpers (`services/api.ts`)

| Helper | Returns | Use when |
|--------|---------|----------|
| `authorizedApiRequest<T>` | `ApiResponse<T>` — `.data` typed as `T` | **99% of cases** — standard JSON endpoints |
| `authorizedRawRequest<T>` | Raw Axios `response.data` (full envelope, NOT unwrapped) | Binary downloads, streaming, non-standard |

### `authorizedApiRequest` — Standard

```ts
const res = await authorizedApiRequest<GoodsReceiptPrintDocument>(
  "GET", `/api/stores/${storeId}/receipts/${id}/print`
);
const html = res.data.html;  // ✅ typed and unwrapped
```

- Calls `unwrapPayload` internally → validates `data` exists
- Returns full envelope with `.data` typed as `T`

### `authorizedRawRequest` — Raw (use carefully)

```ts
const raw = await authorizedRawRequest<any>("GET", url);
// raw = { success: true, message: "...", data: { html: "..." } }
// NOT: { html: "..." }
```

> **Bug pattern:** Using `authorizedRawRequest` and reading `res.html` instead of `res.data.html` → always undefined.  
> See [[Backend Bug Fixes]] for the print bug caused by this.

---

## TanStack Query Integration

### Query

```tsx
const { data, isLoading, error } = useQuery<SupplierList>({
  queryKey: ["suppliers", storeId],
  queryFn: async () => {
    const res = await authorizedApiRequest<SupplierList>("GET", "/api/stores/.../suppliers");
    return res.data;
  },
});
```

### Mutation with Invalidation

```tsx
const qc = useQueryClient();
const mutation = useMutation({
  mutationFn: (input) => authorizedApiRequest("POST", url, { data: input }),
  onSuccess: () => {
    qc.invalidateQueries({ queryKey: ["suppliers"] });
    toast.success("Saved");
  },
  onError: (err) => toast.error(err.message),
});
```

### Auth Token

JWT attached via interceptor in `services/api.ts`.  
Auth source: cookie `pos-access-token` or `Authorization` header.

---

## SSR Hydration Safety

For client components querying in App Router:

```tsx
const [mounted, setMounted] = useState(false);
useEffect(() => setMounted(true), []);

const { data } = useQuery({
  queryKey: ["..."],
  queryFn: ...,
  enabled: mounted,  // prevents hydration mismatch
});
```

See [[Category Management]] where this pattern is used.

---

## Print: Iframe + Blob URL (No Pop-up)

`window.open("", "_blank")` inside async functions is blocked by Safari/Firefox.

```ts
const blob = new Blob([html], { type: "text/html;charset=utf-8" });
const blobUrl = URL.createObjectURL(blob);
const iframe = document.createElement("iframe");
iframe.style.cssText = "position:fixed;width:0;height:0;opacity:0";
document.body.appendChild(iframe);
iframe.src = blobUrl;
iframe.onload = () => {
  iframe.contentWindow?.print();
  setTimeout(() => { URL.revokeObjectURL(blobUrl); iframe.remove(); }, 2000);
};
```

---

## Related
- [[Backend Architecture]] — response envelope definition
- [[Backend Bug Fixes]] — print helper bug, authorizedRawRequest trap
- [[BFF Proxy Routes]] — route map that these helpers call
- [[Warehouse Receive Flow]] — print implementation using this pattern
