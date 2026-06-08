---
tags: [pos, feature, documents, invoice, bill, pdf, receipt, delivery-order]
created: 2026-05-29
updated: 2026-06-01
---

# 📄 Document Management

← [[POS Features MOC]]

## Overview
Single page at `/[locale]/(user)/(workspace)/documents` managing all store documents (ใบแจ้งหนี้, ใบวางบิล, ใบกำกับภาษี, ใบเสนอราคา, ใบส่งของ, ใบลดหนี้) **plus** POS receipts (ใบเสร็จหน้าร้าน). One icon filter strip switches between the two worlds.

## Layout Shell
- Outer card: `flex h-full flex-col overflow-hidden rounded-xl border border-violet-100 bg-white shadow-sm`
- Header (icon + title + subtitle) → icon type filter → mode-specific body

## Icon Type Filter
`document-page-client.tsx` — centered icon buttons with CSS-only tooltip:

| Icon | Type | Mode |
|------|------|------|
| `LayoutGrid` | ทั้งหมด | document |
| `FileText` | INVOICE ใบแจ้งหนี้ | document |
| `FileDigit` | BILL ใบวางบิล | document |
| `FileBadge` | TAX_INVOICE | document |
| `FileQuestion` | QUOTATION | document |
| `FileMinus` | CREDIT_NOTE | document |
| `Truck` | DELIVERY_ORDER ใบส่งของ | document |
| `Receipt` | RECEIPT ใบเสร็จหน้าร้าน | **receipt** |

## Document Preview Panel — `document-preview-panel.tsx`
A4 types (INVOICE/TAX_INVOICE/BILL/QUOTATION/CREDIT_NOTE/**DELIVERY_ORDER**) → fixed **drawer** `w-[820px]`. Others → inline card `w-[360px]`.

### Header buttons (context-sensitive)
| Button | Condition |
|--------|-----------|
| **ชำระแล้ว** (emerald) | INVOICE, not paid, not cancelled |
| **สร้างเอกสาร ▾** dropdown | INVOICE, not paid, not cancelled → ใบกำกับภาษี / ใบส่งของ |
| **ชำระแล้ว** (emerald) | DELIVERY_ORDER with `source_document_id`, not paid |
| **ยกเลิก** (rose) | Any document not CANCELLED or COMPLETED |
| **พิมพ์** | Always |

Props added: `paymentStatus`, `documentStatus`, `sourceDocumentId` — all from document list item.

### Document actions flow
- **ชำระแล้ว** (INVOICE) → `payInvoice(id)` → marks paid + creates TAX_INVOICE (unless DO exists → skip TAX_INVOICE)
- **ชำระแล้ว** (DELIVERY_ORDER) → `payInvoice(sourceDocumentId)` — pays the linked invoice
- **ใบกำกับภาษี** → `convertToTaxInvoice(id)`
- **ใบส่งของ** → `convertToDeliveryOrder(id)` → creates DO with items + customer + due_date copied from invoice
- **ยกเลิก** → `PUT /status { status: "CANCELLED" }` with window.confirm

### source_document_id
Migration `015_document_source_id.sql` — FK on `documents.source_document_id`.  
`ConvertToDeliveryOrder` stores source invoice ID → used by DO's "ชำระแล้ว" button.  
DO also inherits `due_date` from source invoice.

## Sales → "เปิดบิลเชื่อ" flow (Cashier → Invoice)
When cashier selects network customer and clicks **"เปิดบิลเชื่อ"**:
1. Creates INVOICE via `createDocument`
2. **Post-invoice popup** appears asking to create ใบส่งของ
3. If confirmed → `convertToDeliveryOrder(invoiceId)` → DO with `source_document_id`
4. PayInvoice on this invoice will skip TAX_INVOICE (DO already serves as delivery+tax doc)

### Quotation mode in cashier
"ใบเสนอราคา" toggle button in checkout modal header:
- Active (violet solid): hides payment fields, shows **ยืนราคาถึง** presets (7/14/30/45 วัน) + date + notes
- "ตกลง" changes to "สร้างใบเสนอราคา" → creates QUOTATION document
- Toggling off restores normal payment mode

## Receipt Mode (`query.type === "RECEIPT"`)
POS sales history via `listSales()`. Stats / filter / multi-print / CSV export — unchanged.

## Backend
See [[Document PDF & WHT]] for delivery note template, bank accounts in payment box, PromptPay QR, and §86/4 compliance.

## Related
- [[Document PDF & WHT]] — backend rendering, delivery note, bank accounts
- [[BFF Proxy Routes]] — all document BFF routes
- [[Sales Cashier]] — cashier checkout, post-invoice popup, quotation mode
