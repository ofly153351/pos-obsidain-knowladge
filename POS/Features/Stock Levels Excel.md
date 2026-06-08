---
tags: [pos, feature, stock, excel, xlsx, import, export]
created: 2026-05-24
---

# 📊 Stock Levels — Excel Import/Export

← [[POS Features MOC]] · [[Stock Components]] · [[POS Project MOC]]

Migrated from CSV to Excel (`.xlsx`) using the `xlsx` library.

## File

`components/stock/stock-levels-section.tsx`

## Implementation

```ts
import * as XLSX from "xlsx";

// Export
const ws = XLSX.utils.json_to_sheet(rows);
const wb = XLSX.utils.book_new();
XLSX.utils.book_append_sheet(wb, ws, "Products");
XLSX.writeFile(wb, `products-export-${date}.xlsx`);

// Template download
XLSX.writeFile(templateWb, "product-import-template.xlsx");

// Import (parse uploaded file)
const buffer = await file.arrayBuffer();
const wb = XLSX.read(buffer, { type: "array" });
const ws = wb.Sheets[wb.SheetNames[0]];
const rows = XLSX.utils.sheet_to_json(ws);
```

## UI Changes

| Before | After |
|--------|-------|
| `<input accept=".csv">` | `<input accept=".xlsx,.xls">` |
| "อัปโหลดไฟล์ CSV" | "อัปโหลดไฟล์ Excel" |
| `exportLabel: "ส่งออก"` | `exportLabel: "ส่งออก Excel"` (th) |
| `exportLabel: "Export"` | `exportLabel: "Export Excel"` (en) |

The locale key `warehouse.exportLabel` was updated in both `locales/th.json` and `locales/en.json`.

---

## Product Import — 3-Step Preview Flow (added 2026-06-04)

File: `components/stock/import-product-modal.tsx`

**Flow:** `idle → preview → importing → done`

**Template columns (11):** ชื่อสินค้า, SKU, Barcode, ราคาขาย, ราคาทุน, สต็อก, สต็อกขั้นต่ำ, หน่วย, หมวดหมู่, แบรนด์, คำอธิบาย  
(Active removed — defaults to `true` on backend)

**Preview step:** Parses file → shows table with all rows, validates name/price, highlights errors before import.

**Auto-create logic (`LookupCache`):**
- Loads all existing types/units/brands once
- Resolves name → ID; creates via API if not found, caches result
- Category = `createProductType`, Unit = `createProductUnit`, Brand = `createProductBrand`

**Import result:** Progress bar per-row, success/fail count, error details per failed row.

```ts
class LookupCache {
  async resolveType(name: string): Promise<string>  // auto-creates if missing
  async resolveUnit(name: string): Promise<string>
  async resolveBrand(name: string): Promise<string>
}
```

## Related
- [[Stock Components]] — parent section
- [[i18n Patterns]] — locale key update pattern
- [[API Client Patterns]] — createProductType/Unit/Brand service calls
