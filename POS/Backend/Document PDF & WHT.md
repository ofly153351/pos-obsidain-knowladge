---
tags: [pos, backend, document, pdf, wht, invoice, bill, delivery-order, compliance, platform]
created: 2026-05-29
updated: 2026-06-01
---

# 🧾 Document PDF & WHT (Backend)

← [[POS Backend MOC]] · [[Backend Architecture]]

Rendering code lives in **`internal/platform/`** (pure, no business-logic imports).
Orchestration stays in **`internal/modules/document/`** (uses auth, gorm, idgen).

---

## Platform Package Structure

```
internal/platform/
├── receipthtml/        ← POS cash receipt HTML (ใบเสร็จหน้าร้าน)
├── dochtml/            ← Document HTML templates
│   ├── types.go        ← StoreInfo, DocData, DocItem, BankAccountInfo, WHTCertData
│   │                     BuildPromptPayQRDataURI() lives here
│   ├── helpers.go      ← formatMoney, fmtThaiDate, isTaxDoc, titleTH/EN
│   ├── render.go       ← RenderDocumentHTML (router), RenderWHTCertHTML
│   ├── invoice.go      ← invoiceTmpl (ใบแจ้งหนี้/ใบกำกับภาษี, monochrome)
│   ├── bill.go         ← billTmpl (ใบวางบิล, payment box)
│   ├── taxinvoice.go   ← taxInvoiceTmpl (ใบกำกับภาษีอย่างเต็ม, black thead)
│   ├── deliverynote.go ← deliveryNoteTmpl (ใบส่งของ/ใบกำกับภาษี)
│   └── wht.go          ← whtTmpl (ภ.ง.ด.3/53 certificate)
└── docpdf/             ← Document PDF (gofpdf)
```

---

## Document Types → Templates

| Type | Template | Notes |
|------|----------|-------|
| INVOICE | `invoiceTmpl` | Paginated, monochrome |
| TAX_INVOICE | `taxInvoiceTmpl` | Black thead, no logo |
| BILL | `billTmpl` | Payment callout box |
| QUOTATION | `invoiceTmpl` | Same layout as invoice |
| DELIVERY_ORDER | `deliveryNoteTmpl` | See below |
| CREDIT_NOTE | `invoiceTmpl` | |

---

## Delivery Note Template (`deliverynote.go`)

Title: **ใบส่งของ / ใบกำกับภาษี** — serves as both delivery + tax invoice.

### Layout sections
1. **Header** — store info (no logo), document title + original badge, top-info table
2. **Info boxes** — Customer (name/address/phone/tax_id) + Document info (DO No., Invoice Ref., due_date, delivery_date, sales_zone)
3. **Delivery location box** — `DeliveryAddress`, `DeliveryContact`, `DeliveryPhone`
4. **Items table** — columns: ลำดับ / รหัสสินค้า / รายการสินค้า / จำนวน / หน่วย / ราคา/หน่วย / ส่วนลด / จำนวนเงิน
5. **Bottom area** (flex):
   - Left: Payment method box (cash/transfer/credit/cheque) + **bank accounts** + **PromptPay QR** (right-aligned in box)
   - Right: Summary table + remarks
6. **Signature area** — 3 boxes: ผู้อนุมัติ / ผู้ส่งสินค้า / ผู้รับสินค้า
7. **Footer** — document no + TIN + date

### Style (post-2026-06-01)
- All borders: `#ccc` (gray, not black)
- `thead`: `background:#000; color:#fff` (black with white text)
- `td`: full border `.5px solid #ccc` (not bottom-only)
- `border-radius:6px` on all bordered boxes
- `overflow:hidden` on `.items` table

### PromptPay QR
`BuildPromptPayQRDataURI(promptPayID, amount) template.URL` — in `dochtml/types.go`.  
Normalizes Thai phone/tax-id, builds EMV QR payload, CRC16, encodes PNG as `data:image/png;base64,...`.  
Returns `template.URL` to bypass Go `html/template` sanitization (`#ZgotmplZ` trap).  
In delivery note: shown right of payment methods, `width:26mm height:26mm`.

### Bank accounts in payment box
`StoreInfo.BankAccounts []BankAccountInfo` — fetched from `store_bank_accounts` table in `RenderDocumentPrint`.  
Rendered with left-border style, format: `BankName  AccountNo  (AccountName)`.

---

## Store Bank Accounts

Migration `014_store_bank_accounts.sql`:
```sql
store_bank_accounts (id, store_id, bank_code, bank_name, account_no, account_name, created_at)
```

CRUD: `GET/POST /stores/:storeID/bank-accounts`, `DELETE /stores/:storeID/bank-accounts/:accountID`  
Frontend: Bank selector (15 Thai banks) + account no + account name in Settings page → `store-management-panel.tsx`.  
BFF: `app/api/stores/[storeId]/bank-accounts/[accountId]/`

---

## Invoice style (post-2026-06-01)
Same style changes as delivery note:
- All borders `#ccc`, thead `background:#000; color:#fff`
- `td` full border, `border-radius:6px + overflow:hidden` on `.items`
- `th.l` class for left-aligned description column
- Header label: "รายการสินค้า / รายละเอียด (PRODUCT DESCRIPTION)"

---

## PayInvoice — skip TAX_INVOICE when DO exists

`PayInvoice` in `service.go`:
```go
// If a DO already exists for this invoice → skip TAX_INVOICE
s.db.Model(&Document{}).Where("source_document_id = ? AND type = ?", id, TypeDeliveryOrder).Count(&doCount)
if doCount > 0 { return src, nil }
// else → createTaxInvoiceFrom(...)
```
Reason: the DO is titled "ใบส่งของ / ใบกำกับภาษี" and already serves as the combined document.

---

## Source Document ID

Migration `015_document_source_id.sql` — `documents.source_document_id VARCHAR(30) FK → documents.id`.  
`ConvertToDeliveryOrder` sets `SourceDocumentID = &src.ID` and copies `DueDate` from source invoice.  
Included in list query SELECT and `DocumentListItem` DTO → used by frontend for DO "ชำระแล้ว" button.

---

## WHT Certificate
- `receiver_type`: `individual` → ภ.ง.ด.3, `company` → ภ.ง.ด.53
- WHT base = `doc.Subtotal` (pre-VAT)

---

## §86/4 Compliance
All 7 required fields present. Store TIN warned if empty.

---

## Migrations
| File | Content |
|------|---------|
| `009` | `stores.tax_id` |
| `010` | `documents.customer_address`, `customer_phone`, `document_items.unit` |
| `011` | `stores.fax`, `email`, `website` |
| `013` | Delivery order fields (delivery_date, delivery_address, etc.) |
| `014` | `store_bank_accounts` table |
| `015` | `documents.source_document_id` FK |

---

## Related
- [[Document Management]] — frontend consumer
- [[Store Module]] — store settings, bank accounts UI
- [[BFF Proxy Routes]] — `/print`, `/pdf`, `/wht-cert`, `/convert-do`
- [[Backend Bug Fixes]] — VAT rounding, paginateWithLimits bug
