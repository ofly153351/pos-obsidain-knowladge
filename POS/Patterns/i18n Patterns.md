---
tags: [pos, pattern, i18n, locales, typescript, dictionary]
created: 2026-05-22
updated: 2026-05-27
---

# 🌏 i18n Patterns

← [[POS Frontend MOC]] · [[POS Project MOC]]

Locale files: `locales/th.json` + `locales/en.json`  
Type: `typeof th` inferred from `locales/th.json`  
Helper: `getDictionary(locale)` → typed dictionary object

---

## Rules

1. **Every UI string** must be in both locale files — never hardcode
2. Keys added to `th.json` must also be added to `en.json` simultaneously
3. Locale key path = component/feature namespace (e.g. `purchasing.supplierUI`)
4. Dictionary types are exported from components that consume them

---

## Key Structure

```json
{
  "purchasing": {
    "title": "Supplier Management",
    "supplierUI": {
      "allSuppliers": "All Suppliers",
      "addSupplierBtn": "Add Supplier",
      "...": "..."
    },
    "addSupplierModal": {
      "subtitle": "...",
      "...": "..."
    }
  },
  "stock": {
    "categories": {
      "tabs": { "tabTypes": "Types", "..." },
      "toolbar": { "..." },
      "...": {}
    }
  }
}
```

---

## Dictionary Type Pattern

Components export their own dictionary type:

```ts
// components/purchasing/add-supplier-modal.tsx
export type AddSupplierDict = {
  createSupplier: string;
  contactPerson: string;
  // ...
  addSupplierModal: {
    subtitle: string;
    // ...55 keys
  };
};
```

Parent component's `dictionary` prop includes it:
```ts
type SupplierManagerProps = {
  dictionary: {
    // ...base keys
    supplierUI: SupplierUI;
    addSupplierModal: { ...55 keys };
  };
};
```

---

## Key Scale Reference

| Feature | Namespace | Key Count |
|---------|-----------|-----------|
| Supplier UI | `purchasing.supplierUI` | ~40 keys |
| Add supplier modal | `purchasing.addSupplierModal` | ~55 keys |
| Category management | `stock.categories` | 59 keys |
| Receipt settings | `settings.receipt` | ~30 keys |
| Warehouse receive | `stock.receiveGoods` | 80+ keys |

---

## 59-Key Category Structure

```
stock.categories:
  tabs.tabTypes/tabUnits/tabBrands
  toolbar.searchTypes/Units/Brands, statusAll/Active/Inactive
          addType/Unit/Brand
  columns.colOrder, colNameType/Unit/Brand, colProductCount,
          colStatus, colLastModified, colActions
  status.legendActive/Inactive/Edit/Delete/Drag
  count.countItems
  pagination.showing/of/perPage
  empty.emptyTitle/emptyAdd
  sidebar.overviewTitle/Total/Active/Inactive/TotalProducts
          popularTitle/ViewAll
  tools.toolsTitle, toolsImport/Export/Sort + Desc
  import.importTitle/DropText/Browse/Cancel/Confirm
        importPreviewTitle, importColName/Desc/Status
  delete.deleteTitle/Message/Warning/Cancel/Confirm
```

---

## Adding Keys Checklist

1. Add to `locales/th.json` (Thai text)
2. Add same path to `locales/en.json` (English text)
3. Update the TypeScript dictionary type in the component file
4. Pass the new key through the prop chain from page → component

---

## Related
- [[Category Management]] — 59-key example
- [[Supplier Management]] — ~100-key example
- [[POS Project Overview]] — locale file paths
- [[POS Dev Commands]] — type-check catches missing keys
