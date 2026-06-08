---
tags: [pos, feature, supplier, purchasing, master-detail, modal]
created: 2026-05-27
---

# 👥 Supplier Management

← [[POS Features MOC]] · [[Purchasing Components]] · [[POS Project MOC]]

## Files

| File                                           | Role                                     |
| ---------------------------------------------- | ---------------------------------------- |
| `components/purchasing/supplier-manager.tsx`   | 2-panel shell + sub-components           |
| `components/purchasing/add-supplier-modal.tsx` | Create supplier modal                    |
| `services/suppliers.ts`                        | API service + `CreateSupplierInput` type |

## Layout Pattern — 2-Panel Master-Detail

```tsx
<div className="-mx-6 -my-6 lg:-mx-8 lg:-my-8 flex flex-col overflow-hidden"
     style={{ height: "calc(100dvh - 4.5rem)" }}>
  <TopSearchBar />   {/* gradient violet-50 background */}
  <div className="flex flex-1 overflow-hidden">
    <LeftPanel className="w-[320px] shrink-0 border-r" />
    <RightPanel className="flex-1 overflow-hidden bg-violet-gradient" />
  </div>
</div>
```

- Negative margins break out of workspace `p-6 / p-8` padding
- `calc(100dvh - 4.5rem)` accounts for topbar height
- See [[Layout Components]] for the topbar height context

## Transition Pattern

```tsx
{/* key prop forces re-mount → CSS animation re-fires automatically */}
<div key={selectedId} className="smooth-fade-up h-full">
  <SupplierDetail ... />
</div>
```

> Rule: NEVER use `detail-slide-in` (horizontal from left) for detail panels.  
> Always use `smooth-fade-up` for panel content transitions.  
> See [[Animation Patterns]]

## Hero Header (Solid Violet — no gradient)

```
bg-violet-600
  │ pb-14  ← extra bottom padding for KPI overlap
  └── avatar + name (text-white) + action buttons
      ├── Call: bg-white/15 text-white backdrop-blur-sm hover:bg-white/25
      ├── Edit: bg-white text-violet-700 shadow-sm (solid white = primary)
      └── Delete: bg-white/15 text-white/90 hover:bg-red-400/30
```

KPI cards float over header with `-mt-8`:

```tsx
<div className="-mt-8 shrink-0 px-6 pb-1">
  <div className="flex gap-3 rounded-2xl bg-white border border-violet-100
                  p-4 shadow-[0_4px_24px_rgba(124,58,237,0.14)]">
    <KpiCard primary />   {/* Total purchase — bg-violet-600 */}
    <KpiCard />           {/* Outstanding */}
    <KpiCard />           {/* Credit limit */}
    <KpiCard />           {/* Remaining */}
  </div>
</div>
```

`KpiCard` with `primary` prop = `bg-violet-600` (solid, no gradient).

## 2×2 Info Card Grid

```tsx
<div className="grid grid-cols-2 gap-4">
  <InfoCard icon={<User />} title="Contact">  {/* name, phone, email */}
  <InfoCard icon={<CreditCard />} title="Payment">  {/* method, account, credit term */}
  <InfoCard icon={<MapPin />} title="Address">
  <InfoCard icon={<MessageSquare />} title="Notes">
</div>
```

Each `InfoCard` = `rounded-xl border border-violet-100 bg-white p-4 shadow-sm` with `h-6 w-6 rounded-lg bg-violet-100 text-violet-600` icon container.

## Supplier Card (Left Panel)

```tsx
<button className={`border-l-[3px] transition-all duration-200 ${
  isSelected ? "border-violet-500 bg-violet-50" : "border-transparent hover:bg-slate-50/80"
}`}>
  <InitialsAvatar />
  <div>
    <p className={isSelected ? "text-violet-900" : "text-slate-800"}>{supplier.name}</p>
    <span className="rounded-full bg-violet-50 px-1.5 text-[10px] text-violet-600 ring-1 ring-violet-100">
      เครดิต 30 วัน
    </span>
  </div>
</button>
```

## Add Supplier Modal

- Dimensions: `max-w-[780px] max-h-[92vh]`
- Top accent: `h-1.5 bg-violet-600`
- Header bg: `bg-gradient-to-r from-violet-50/60 to-white` (subtle tint — allowed)
- Header icon: `bg-violet-600 rounded-2xl shadow-lg`
- Footer bg: `bg-gradient-to-r from-violet-50/40 to-white` (subtle tint — allowed), save button `flex-[2]`
- Save button: `bg-violet-600 hover:bg-violet-700`

### New fields sent to backend (2026-05-27)

| Frontend field | Backend column | Type |
|----------------|---------------|------|
| `email` | `email` | TEXT |
| `line_id` | `line_id` | TEXT |
| `payment_method` | `payment_method` | TEXT CHECK ('promptpay','bank_account') |
| `promptpay_number` | `promptpay_number` | TEXT |
| `bank_name` | `bank_name` | TEXT |
| `bank_account_number` | `bank_account_number` | TEXT |
| `bank_account_name` | `bank_account_name` | TEXT |
| `credit_days` | `credit_days` | INTEGER DEFAULT 0 |

Migration: `007_supplier_extended_fields.sql`
- Enter: `smooth-fade-up`, Exit: `isClosing → "fade-out" → onAnimationEnd → onClose()`
- Two-column body: `grid grid-cols-2 divide-x divide-slate-100`
- **Left column:** company name, contact+phone (50/50), LINE+email (50/50), logo drag-drop
- **Right column:** payment method toggle (PromptPay/Bank), `key=` on conditional fields for animation, credit term pills (0/7/15/30/45/custom), address+notes with char counters

### Validation

| Field | Rule |
|-------|------|
| Phone | `/^0\d{9}$/` (10 digits Thai) |
| Email | RFC: `/^[^\s@]+@[^\s@]+\.[^\s@]+$/` |
| PromptPay | 10 or 13 digits |
| Bank account | 10–12 digits |
| Credit custom | 1–365 integer |

### Cancel Confirmation

When `isDirty === true` → clicking X/Cancel shows a dialog (`AlertTriangle` icon, amber bg).  
Confirm → `triggerClose()` → `isClosing = true` → CSS `fade-out` → `onClose()`.

## services/suppliers.ts

`CreateSupplierInput` fields added:
```ts
line_id?: string;
email?: string;
payment_method?: "promptpay" | "bank_account";
promptpay_number?: string;
bank_name?: string;
bank_account_number?: string;
bank_account_name?: string;
credit_days?: number;
```

## i18n Keys

~100 keys added under `purchasing.supplierUI` and `purchasing.addSupplierModal`.  
See [[i18n Patterns]] for key structure.

---

## EditSupplierModal — Full Fields (updated 2026-06-04)

File: `components/purchasing/supplier-manager.tsx` — `EditSupplierModal`

Expanded from 7 fields to full supplier data:

| Section | Fields |
|---------|--------|
| Logo | Upload/remove with preview (PNG/JPG ≤ 2MB) |
| Contact | ชื่อบริษัท*, ผู้ติดต่อ, โทร, email, LINE ID |
| Payment | Toggle: PromptPay / บัญชีธนาคาร / ไม่ระบุ |
| Credit | Preset buttons (0/7/15/30/45/60/90 วัน) + custom input |
| Address | ที่อยู่, เลขผู้เสียภาษี, หมายเหตุ |
| Status | Toggle พร้อม label active/inactive |

Modal: `max-w-3xl`, scrollable body, gradient header.

### File Upload Fix

BFF routes for supplier POST/PUT now use `request.blob()` (not `request.text()`) to preserve multipart boundary.

```ts
const body = await request.blob();
return proxyApiRequest(request, url, { body, method: "POST" });
```

See [[BFF Proxy Routes]] — multipart rule.

## Related
- [[Animation Patterns]] — `smooth-fade-up`, `fade-out`, `key` re-mount trick
- [[POS Today Theme]] — gradient hero tokens, KPI primary card, button-on-dark styles
- [[API Client Patterns]] — TanStack Query: `useQuery(["suppliers"])`, `useMutation`
- [[BFF Proxy Routes]] — supplier CRUD endpoints + multipart fix
- [[Toast Alert System]] — CRUD result notifications
