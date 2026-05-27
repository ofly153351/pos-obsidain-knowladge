---
tags: [pos, feature, stock, categories, product-types, units, brands]
created: 2026-05-24
---

# 🗂️ Category Management

← [[POS Features MOC]] · [[Stock Components]] · [[POS Project MOC]]

## Files

| File | Role |
|------|------|
| `components/stock/catalog-setup-section.tsx` | Main component — 3-tab layout |
| `types/product.ts` | Added `product_count?: number`, `updated_at?: string` |
| `components/stock/types.ts` | Added `CategoriesDictionary` type (59 keys) |

**Route:** `/[locale]/stock/categories`

## Layout

```
Two-column: flex items-start gap-6
├── Table card (flex-1)
│   ├── Toolbar: search + status filter + add CTA
│   ├── Table: order badge, avatar, name/desc, product count, status, last-modified, actions
│   └── Pagination: client-side, options 10/25/50/100
└── Right sidebar (w-80)
    ├── 2×2 stats grid (total/active/inactive/total products)
    ├── Top-3 popular (gold/silver/bronze rank)
    └── Import/Export tool buttons
```

Tab bar: `bg-violet-600 text-white rounded-xl` (active) · each tab shows product count

## Key Design Patterns

### SSR Hydration Safety

```tsx
const [mounted, setMounted] = useState(false);
useEffect(() => setMounted(true), []);

const { data } = useQuery({
  queryKey: ["product-types", storeId],
  queryFn: ...,
  enabled: mounted,    // prevents SSR mismatch
});
```

### Cache Deduplication

Uses same query keys as `StockManager` — TanStack Query deduplicates automatically.

### Page Reset on Filter Change

```tsx
useEffect(() => {
  setPage(1);
}, [tab, search, statusFilter]);
```

### Avatar Color Cycling

```tsx
const AVATAR_COLORS = [/* 8 gradient combos */];
<div className={`bg-gradient-to-br ${AVATAR_COLORS[idx % AVATAR_COLORS.length]}`}>
  {initials}
</div>
```

### Hover-Reveal Action Buttons

```tsx
<tr className="group">
  <td>
    <div className="opacity-0 group-hover:opacity-100 transition-opacity">
      <button>Edit</button>
      <button>Delete</button>
    </div>
  </td>
</tr>
```

## Scaling Sizes (applied 2026-05-24)

| Element | Size |
|---------|------|
| Tab bar text | `text-base px-5 py-2.5` |
| Tab icons | `h-5 w-5` |
| Table header/rows | `px-6 py-4` |
| Avatar | `h-11 w-11 text-sm` |
| Product count badge | `text-sm px-3 py-1.5` |
| Status pills | `text-sm px-3.5 py-1.5`, dot `h-2 w-2` |
| Pagination buttons | `h-8 w-8 text-sm` |
| Sidebar | `w-80`, stats `text-2xl` |

## i18n — 59 Keys (`stock.categories`)

```
tabs:       tabTypes, tabUnits, tabBrands
toolbar:    searchTypes, searchUnits, searchBrands
            statusAll, statusActive, statusInactive
            addType, addUnit, addBrand
columns:    colOrder, colNameType/Unit/Brand, colProductCount,
            colStatus, colLastModified, colActions
pagination: showing, of, perPage
empty:      emptyTitle, emptyAdd
sidebar:    overviewTitle, overviewTotal/Active/Inactive/TotalProducts
            popularTitle, popularViewAll
tools:      toolsTitle, toolsImport/Export/Sort + Desc variants
import:     importTitle, importDropText, importBrowse, importCancel/Confirm,
            importPreviewTitle, importColName/Desc/Status
delete:     deleteTitle, deleteMessage, deleteWarning, deleteCancel/Confirm
```

## Backend: `product_count` Subquery

See [[GORM Patterns]] for the read-only `gorm:"->"` pattern.

```go
// model.go
ProductCount int64 `json:"product_count" gorm:"column:product_count;->"`

// repository.go
Select("product_types.*, COALESCE((SELECT COUNT(*) FROM products p WHERE p.product_type_id = product_types.id), 0) AS product_count")
```

Applied to: `producttype`, `productunit`, `productbrand` modules.

## Related
- [[GORM Patterns]] — `product_count` read-only field
- [[i18n Patterns]] — 59-key dictionary structure
- [[Stock Components]] — parent component context
- [[POS Today Theme]] — tab/pill/avatar styles
