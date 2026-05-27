---
tags: [pos, feature, stock, product, form, drawer]
created: 2026-05-24
---

# 🗂️ Product Form Drawer

← [[POS Features MOC]] · [[Stock Components]] · [[POS Project MOC]]

Converted from centered modal (scale+opacity) to **left-side drawer** (slide translate).

## File

`components/stock/product-form-modal.tsx`

## Exports

```ts
export { ProductFormDrawer };
export { ProductFormDrawer as ProductFormModal };  // backwards compat alias
```

## Drawer Layout Structure

```
fixed inset-0 z-50
  ├── absolute inset-0  ← backdrop (click-outside-to-close)
  └── absolute left-0 top-0 h-full  ← drawer panel
      ├── header (shrink-0, violet gradient)
      ├── form (flex-1 overflow-y-auto)
      │   └── content p-5/p-6
      │       └── xl:grid xl:grid-cols-[1fr_268px]
      │           ├── form sections (left)
      │           └── POSPreviewCard (right, xl+ only)
      └── sticky footer bottom-0
```

## CSS

```tsx
// Drawer
className="w-full sm:w-[80vw] xl:w-[860px] sm:rounded-r-[1.5rem]"

// Animation
className="transition-transform duration-300 ease-out"
// Open:   translate-x-0
// Closed: -translate-x-full

// Body scroll lock
useEffect(() => {
  document.body.style.overflow = "hidden";
  return () => { document.body.style.overflow = ""; };
}, []);
```

## POS Preview Card (`POSPreviewCard`)

Live preview of how the product appears in the cashier:

```tsx
// Sticky positioning
className="sticky top-6"

// Live image from file input
const previewUrl = URL.createObjectURL(formState.image);
useEffect(() => () => URL.revokeObjectURL(previewUrl), [previewUrl]);

// Derived fields
categoryName = productTypes.find(t => t.id === formState.typeId)?.name
unitName = unitOptions.find(u => u.id === formState.unitId)?.name

// Special price display
{specialPrice > 0 && (
  <s>{basePrice}</s>
  <span>{specialPrice}</span>
)}
```

## i18n Keys Added

| Key | EN | TH |
|-----|----|----|
| `stock.form.posPreviewLabel` | "POS Preview" | "ตัวอย่าง POS" |
| `stock.form.posStatusActive` | "Active" | "เปิดใช้งาน" |
| `stock.form.posStatusInactive` | "Inactive" | "ปิดใช้งาน" |

All optional in `StockManagerDictionary["form"]` with fallback defaults.

## Related
- [[Animation Patterns]] — drawer slide transition pattern
- [[Stock Components]] — parent component
- [[POS Today Theme]] — violet gradient header
- [[Toast Alert System]] — save/delete notifications
