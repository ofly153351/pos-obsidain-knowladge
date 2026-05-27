---
tags: [pos, feature, warehouse, receive, wizard, autosave, draft]
created: 2026-05-24
---

# 📥 Warehouse Receive Flow

← [[POS Features MOC]] · [[POS Project MOC]]

3-step wizard for recording incoming stock into a warehouse.  
Lifecycle: `draft → confirmed` or `draft → cancelled`.  
Only draft receipts can be edited. Only `manage` permission can confirm/cancel.

---

## File Structure

```
components/warehouse/
├── receive-goods-flow.tsx   ← barrel re-export (7 lines) — keeps page files unchanged
├── receive-flow.tsx         ← main shell: Stepper, ContextBar, wizard state/logic
├── receive-step1.tsx        ← Step 1: header form + view mode (presentational)
├── receive-step2.tsx        ← Step 2: items table + product browser (presentational)
├── receive-step3.tsx        ← Step 3: summary, stock impact, attachment (presentational)
├── receive-cards.tsx        ← SummaryCard + ReceiptStatusBadge (no circular deps)
└── receive-shared.ts        ← pure TS: types, formatters, session helpers, builders
```

> **Circular import rule:** step files import `SummaryCard` from `./receive-cards`, NOT from `./receive-flow`.  
> `receive-flow.tsx` imports from all steps → any step importing back = cycle.

**All state and logic lives in `ReceiveWizard`.** Step components are purely presentational.

Barrel re-export:
```ts
// receive-goods-flow.tsx
export { ReceiveIndexPage as ReceiveGoodsIndexPage, ... } from "./receive-flow";
```

---

## Page Routes

| Route | Component | Props |
|-------|-----------|-------|
| `/warehouse/receive` | `ReceiveIndexPage` | `dictionary, locale` |
| `/warehouse/receive/new` | `ReceiveNewPage` | `dictionary, locale` |
| `/warehouse/receive/[id]/step-1` | `ReceiveWizard` | `step={1}` |
| `/warehouse/receive/[id]/step-2` | `ReceiveWizard` | `step={2}` |
| `/warehouse/receive/[id]/step-3` | `ReceiveWizard` | `step={3}` |
| `/warehouse/receive/[id]/view` | `ReceiveWizard` | `step="view"` |

---

## 3-Step Flow

### Step 1 — Header Info
Required: `warehouse_id`, `received_at`  
Optional: supplier, reference number, VAT %, VAT included, note  
Autosaves (800ms debounce) + mirrors to `sessionStorage: receive-goods:{id}:step-1`

### Step 2 — Line Items
- Selected items table at top (qty, location, unit price, discount — inline editable)
- Product browser below: search by name/SKU/barcode, scan, import from PO
- Autosaves (800ms debounce) + mirrors to `sessionStorage: receive-goods:{id}:step-2`

### Step 3 — Review & Confirm
- Financial summary (subtotal, discount, net, total)
- Stock impact preview (before/change/after table)
- Items table with line totals
- Attachment upload
- Confirm → receipt becomes immutable

---

## SessionStorage Keys

| Key | Type | Content |
|-----|------|---------|
| `receive-goods:{receiptId}:step-1` | `HeaderForm` | Header form state |
| `receive-goods:{receiptId}:step-2` | `Record<string, ItemFormRow>` | Items keyed by productId |

On wizard mount: sessionStorage checked first → preserves unsaved edits across page refresh.

---

## Autosave Mechanism

```
useEffect triggers when: step active + draft + required fields filled + payload ≠ last signature
  → debounce 800ms
  → call API (PUT receipt or POST items with replace_existing: true)
  → update signature ref
```

Signatures stored in `useRef` (not state) to avoid re-render loops.  
Items: `upsertGoodsReceiptItems(id, { items, replace_existing: true })` — always full replace.

---

## TanStack Query Keys

```ts
["warehouse", "receive", "index", "drafts"]
["warehouse", "receive", "index", "recent"]
["warehouse", "receive", "new", "warehouses"]
["warehouse", "receive", receiptId]
["warehouse", "receive", "warehouses"]
["warehouse", "receive", "suppliers"]
["warehouse", "receive", "products"]           // enabled: step === 2
["warehouse", "receive", "purchase-orders"]    // enabled: step === 2
["warehouse", "receive", "locations", warehouseId]
["warehouse", "receive", receiptId, "stock-impact"]  // enabled: step === 3 || view
```

---

## Step Navigation Logic (`receive-shared.ts`)

```ts
function getEditableReceiptStep(receipt): 1 | 2 | 3 | "view" {
  if (receipt.status === "confirmed") return "view";
  if (!receipt.warehouse_id || !receipt.received_at) return 1;
  if (!receipt.items?.length) return 2;
  return 3;
}
```

Redirect guards in `ReceiveWizard`:
- confirmed + not on view → redirect to view
- view + not confirmed → redirect to editable step
- on step 3 + maxAvailableStep < 3 → redirect backward

---

## `ItemFormRow` Type

```ts
type ItemFormRow = {
  discountValue: string;  // "0" = no discount
  locationId:   string;
  quantity:     string;  // "" = not selected, ">0" = selected
  unitPrice:    string;
}
```

All values are strings (form inputs). `buildDraftItemsPayload` converts to numbers and filters `quantity <= 0`.

---

## Barcode Scan Flow (Step 2)

1. User types scan code → Enter
2. Match against product `sku` or `barcode` (case-insensitive exact match)
3. Match → increment qty by 1, show success/duplicate feedback
4. No match → show error feedback
5. Scan input cleared either way

---

## PO Import Flow (Step 2)

1. Select PO from dropdown → click import
2. `getPurchaseOrder(selectedPoId)` → full PO with items
3. For each PO item: **add** quantity (stacks, not reset)
4. `unitPrice` = PO item `unit_cost` → product `cost_price`
5. Location = currently filtered location or first available

Backend validates: receipt qty per item ≤ PO outstanding qty.

---

## Location Auto-fix Effect

When `headerForm.warehouseId` changes → items whose `locationId` doesn't exist in new warehouse → reset to `locations[0].id`.

---

## Backend: `replace_existing` Behavior

| Value | Behavior |
|-------|----------|
| `true` | Delete ALL existing items, insert new list (used by autosave) |
| `false` | Append to existing list, check uniqueness across merged list |

---

## Backend: `discount_type` + `discount_value` Bug (Fixed 2026-05-24)

`discount_type = ""` + `discount_value = 0` (non-nil pointer) was causing 400.  
Frontend always sends `discount_value: 0` for items without discounts.

**Fix:** `if discountValue != nil && *discountValue != 0` (was `if discountValue != nil`)  
File: `pos-backend/internal/modules/warehouse_receipt/util.go`

See [[Backend Bug Fixes]].

---

## Backend: Stock Confirmation

`POST /warehouse/receipts/:id/confirm`:
- Requires `manage` permission
- Writes stock movements (increments at each item's location)
- Receipt status → `confirmed` — permanently immutable

---

## Backend: Validation Causes 400

| Error | Cause |
|-------|-------|
| `at least one receipt item is required` | empty items array |
| `location does not belong to receipt warehouse` | wrong warehouse |
| `sale point locations cannot be used` | `is_sale_point: true` location |
| `duplicate product and location combination` | same (productId, locationId) twice |
| `receipt quantity exceeds outstanding PO quantity` | qty > PO outstanding |
| `only draft warehouse receipt can be modified` | editing confirmed/cancelled |

---

## Print Bug Fix (Pop-up Blocked)

`window.open("", "_blank")` inside async functions is blocked by Safari/Firefox.

**Fix:** Hidden iframe + Blob URL:
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

See [[API Client Patterns]] for the `authorizedApiRequest` bug that was also part of this fix.

---

## BFF Proxy Routes

See [[BFF Proxy Routes]] for the complete receipts route table.

Key: `storeId` in BFF URL is NOT forwarded to backend — backend resolves from JWT.

---

## Services (`services/goods-receipts.ts`)

```ts
createGoodsReceiptDraft(input)
getGoodsReceipt(id)
listGoodsReceipts({ page, limit, status? })
updateGoodsReceipt(id, input)
upsertGoodsReceiptItems(id, { items, replace_existing })
confirmGoodsReceipt(id)
cancelGoodsReceipt(id)
uploadGoodsReceiptAttachment(id, file)
getGoodsReceiptStockImpact(id)
fetchGoodsReceiptPrintDocument(id)    // uses authorizedApiRequest (not Raw)
generateGoodsReceiptDocumentNo()
```

---

## Formatters (`receive-shared.ts`)

| Function | Notes |
|----------|-------|
| `formatDateTimeInput(value)` | ISO → `datetime-local` value (local timezone) |
| `formatDateTimeLabel(value)` | Thai date-time `th-TH` locale |
| `formatNumber(value)` | Thai locale, 0–2 decimals |
| `formatSignedNumber(value)` | prefixed `+`/`-` |
| `formatCurrency(value)` | Thai Baht with symbol |
| `formatFileSize(value)` | bytes → KB or MB |

---

## Related
- [[API Client Patterns]] — `authorizedApiRequest` vs `authorizedRawRequest`, print bug
- [[BFF Proxy Routes]] — receipts route map
- [[Backend Bug Fixes]] — discount_value=0 bug, print pop-up bug
- [[Toast Alert System]] — autosave feedback, error notifications
- [[Animation Patterns]] — ContextBar, stepper transitions
