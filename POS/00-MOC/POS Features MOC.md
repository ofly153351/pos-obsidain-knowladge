---
tags: [pos, moc, features]
created: 2026-05-27
---

# ✨ POS Features — Index

← [[POS Project MOC]]

## Feature Status

| Feature | Status | Main Files |
|---------|--------|-----------|
| [[Supplier Management]] | ✅ Done | `supplier-manager.tsx`, `add-supplier-modal.tsx` |
| [[Category Management]] | ✅ Done | `catalog-setup-section.tsx` |
| [[Stock Levels Excel]] | ✅ Done | `stock-levels-section.tsx` |
| [[Warehouse Receive Flow]] | ✅ Done | `receive-flow.tsx` + 5 sub-files |
| [[Product Form Drawer]] | ✅ Done | `product-form-modal.tsx` |
| [[Receipt Payment Settings]] | ✅ Done | `receipt-payment-settings.tsx` |
| [[Invoice Print Feature]] | ✅ Done | `components/invoice/`, `app/[locale]/print/invoice/[saleId]/page.tsx` |
| [[Document Management]] | ✅ Done | `components/documents/`, backend `internal/modules/document/` |
| [[Activity Logs]] | ✅ Done | `activity-logs-client.tsx`, `internal/modules/activity_log/`, middleware |
| [[Product Card Settings]] | ✅ Done | `product-card.tsx`, `card-settings-modal.tsx`, `internal/modules/usersettings/` (per-user JSONB) |
| [[Auth Pages]] | ✅ Done | `auth-shell.tsx`, `auth-form.tsx` (2-panel login/register) |
| [[Inventory & Stock Counting]] | 🟡 `fix-kaew/dev` | `inventory-manager.tsx`, `stock-count-manager.tsx`, `stock-adjust-drawer.tsx` |
| [[Barcode & Labels]] | 🟡 `fix-kaew/dev` | `barcode-batch-modal.tsx`, `lib/barcode.ts`, `lib/qr.ts`, `lib/pdf.ts` |

## Shared Infrastructure Used by All Features

| Concern | Note |
|---------|------|
| Transitions | [[Animation Patterns]] — always `smooth-fade-up` + `fade-out` |
| API calls | [[API Client Patterns]] — `useQuery` + `useMutation` + `authorizedApiRequest` |
| Proxy routing | [[BFF Proxy Routes]] |
| Notifications | [[Toast Alert System]] — `toast.success/error()` for async, `<Alert>` for validation |
| Error handling | [[Error Handling]] — `ApiError`, `friendlyMessage()`, `<FieldError>`, structured backend errors |
| Text content | [[i18n Patterns]] — all text in `th.json` + `en.json`, never hardcoded |
| Design | [[POS Today Theme]] — all features follow the violet token system |

## Cross-Feature Architectural Patterns

### 2-Panel Master-Detail
Used by: [[Supplier Management]]
- Fixed left list (`w-[320px]`) + flex-1 right panel
- `calc(100dvh - 4.5rem)` height, `-mx-6 -my-6` negative margin breakout
- `<div key={selectedId} className="smooth-fade-up">` for panel re-mount animation

### 3-Step Wizard
Used by: [[Warehouse Receive Flow]]
- Draft → Confirmed lifecycle, autosave 800ms debounce
- sessionStorage mirrors form state for refresh safety

### Left-Side Drawer
Used by: [[Product Form Drawer]]
- `translate-x-0` open / `-translate-x-full` closed, `transition-transform duration-300`

### Gradient Hero Header
Used by: [[Supplier Management]]
- `bg-gradient-to-br from-violet-600 to-pink-500`
- KPI cards float over with `-mt-8` + white card shadow

### Right-Side Drawer (A4) vs Inline Panel (A5)
Used by: [[Document Management]]
- A4 docs → fixed `slide-in-right` drawer `w-[820px]` + backdrop + Esc
- A5/receipt → inline `w-[360px]` card next to the table
- Preview body = backend HTML in `<iframe srcDoc>`, print via hidden-iframe blob

### Icon Filter Strip + CSS Tooltip
Used by: [[Document Management]]
- Centered `h-11 w-11` icon buttons, active `bg-violet-600 text-white`
- Tooltip = `group-hover:opacity-100` absolute caret box (no JS)
