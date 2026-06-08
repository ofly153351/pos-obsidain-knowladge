---
tags: [pos, feature, invoice, print]
created: 2026-05-28
---

# Invoice Print Feature

← [[POS Features MOC]]

## Overview

Print-quality invoice documents generated from sale receipts. Two formats:
- **Full A4** — formal Thai business invoice (210mm × 297mm)
- **Short Form** — compact A5 invoice

## Entry Point

Button `"ออกใบแจ้งหนี้"` on each sale row in `components/documents/documents-manager.tsx`:

```ts
window.open(`/${locale}/print/invoice/${sale.id}`, "_blank", "noopener,noreferrer")
```

`locale` from `useParams()` in the component.

## Route

`app/[locale]/print/invoice/[saleId]/page.tsx`

- Outside `(workspace)` route group → no sidebar, clean print page
- Uses locale layout only (validates locale, no UI chrome)
- Server Component → passes dict to client

## Components

### `components/invoice/invoice-print-client.tsx`
- `"use client"` — fetches sale data via `getSaleById(saleId).then(res => res.data)`
- Toolbar: format toggle (Full A4 / Short), Print button, Close button
- Format state drives which invoice component renders
- CSS `@media print` hides toolbar, shows only the invoice content
- `@page` size adapts: A4 for full, A5 for short

### `components/invoice/invoice-a4.tsx`
- Pure presentational, receives `sale: Sale` + `dict` + `invoiceNo`
- Inline `style={{ width: "210mm", ... }}` for print fidelity
- Sections: store header | divider | customer + payment | items table (with filler rows) | totals | signature area | footer
- Items table has 7 columns: #, description, unit, qty, unit price, discount, amount

### `components/invoice/invoice-short.tsx`
- `width: "148mm"` (A5 landscape width / A6 portrait)
- Compact: header centered, items 5-column table, summary, single signature block

## Invoice Number

Generated from sale_number:
```ts
function makeInvoiceNo(sale: Sale): string {
  const suffix = sale.sale_number.replace(/^SALE-/, "").slice(0, 15);
  return `INV-${suffix}`;
}
```

## Data Source

`services/sales.ts` → `getSaleById(saleId)` → returns `ApiResponse<Sale> & { data: Sale }`

**Important**: `getSaleById` returns `res.data` (wrapped), not the Sale directly.

## Type Updates

`types/sale.ts` — added fields to `Sale`:
- `sale_number`, `cashier_name`, `customer_phone`
- `store_name`, `store_address`, `store_phone`

`SaleItem` — added `sku`, `unit_type`

## Revenue Code §86/4 Compliance

Full ใบกำกับภาษี requires all 7 elements:
1. "ใบกำกับภาษี" — prominently displayed ✅ (hardcoded Thai, plus subtitle line)
2. Seller: ชื่อ + ที่อยู่ + เลขประจำตัวผู้เสียภาษี ✅ (from `sale.store_*` + `sale.store_tax_id`)
3. Buyer: ชื่อ + ที่อยู่ ✅ (from `sale.customer_name`, `customer_phone`; short-form exempts address)
4. Invoice number ✅ (derived from sale_number: `INV-{...}`)
5. Items: ชื่อ ชนิด ปริมาณ มูลค่า ✅ (items table)
6. VAT amount — **explicitly separate line** ✅ (always shown, even if 0.00)
7. Date ✅ (Buddhist-era format)

Short-form (อย่างย่อ) omits buyer TIN/address — legal for retail POS per Revenue Code.

**Warning shown** when `store_tax_id` is empty — amber italic "ยังไม่ได้ตั้งค่า".

### Store TIN Setup

Added to backend: `stores.tax_id VARCHAR(13)` (migration `009_store_tax_id.sql`).
Edit in: Settings → ตั้งค่าร้านค้า → Payment section → "เลขประจำตัวผู้เสียภาษี".

`store_tax_id` flows: `stores.tax_id` → `sale.store_tax_id` (JOIN in repository) → `Sale` type → invoice components.

## i18n

New namespace `"invoice"` in `th.json` and `en.json` with ~35 keys.

`SalesDictionary` type (`components/sales/types.ts`) — added `printInvoiceButton: string`.

## Print CSS Pattern

```ts
<style>{`
  @media print {
    .no-print { display: none !important; }
    .print-only { display: block !important; }
    @page { size: ${format === "full" ? "A4" : "A5"} portrait; margin: 0; }
  }
  @media screen {
    .print-only { display: none; }
  }
`}</style>
```

Both `no-print` and `print-only` divs are rendered — CSS controls visibility.
