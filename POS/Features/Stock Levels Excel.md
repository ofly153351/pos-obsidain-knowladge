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

## Related
- [[Stock Components]] — parent section
- [[i18n Patterns]] — locale key update pattern
