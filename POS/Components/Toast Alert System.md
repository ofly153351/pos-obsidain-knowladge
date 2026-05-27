---
tags: [pos, component, ui, toast, alert, notifications]
created: 2026-05-24
---

# 🔔 Toast & Alert System

← [[POS Frontend MOC]] · [[POS Project MOC]]

## Files

| File | Role |
|------|------|
| `components/ui/toast.tsx` | Floating toast (bottom-right, auto-dismiss) |
| `components/ui/alert.tsx` | Inline persistent alert banner |

`<Toaster />` is mounted once in `app/layout.tsx` — **do NOT add it again**.

---

## Usage

### `toast` — Async Action Results

Use for: save, delete, API mutations, print feedback.

```tsx
import { toast } from "@/components/ui/toast";

toast.success("บันทึกสำเร็จ");         // 3.5s auto-dismiss
toast.error("เกิดข้อผิดพลาด");         // 5s
toast.info("กำลังดำเนินการ...");        // 3s
toast.warning("คำเตือน");              // 4s
toast.error("ข้อความ", 8000);          // custom duration ms
```

`useToast()` hook also available — returns the same `toast` object.

### `<Alert>` — Inline Persistent Banners

Use for: form-level validation errors, persistent state warnings, non-dismissible info.

```tsx
import { Alert } from "@/components/ui/alert";

<Alert tone="error">กรุณาเลือก warehouse ก่อน</Alert>
<Alert tone="warning" onDismiss={() => setError("")}>{error}</Alert>
<Alert tone="success">อัปโหลดสำเร็จ</Alert>
<Alert tone="info" className="mt-2">ข้อมูลเพิ่มเติม</Alert>
```

Tones: `"success" | "error" | "info" | "warning"`

---

## Architecture

`toast.tsx` uses a **module-level store** — no Zustand, no React Context:
- Module-level `toasts[]` array + `Set<listener>`
- `useSyncExternalStore` for React integration
- Call `toast.*()` from anywhere (component, mutation callback, utility function)

```
toast.success() → pushes to module array → notifies listeners → <Toaster> re-renders
```

---

## Migration Rule

**Never** write inline notification divs:

```tsx
{/* ❌ Wrong */}
<div className="border-rose-200 bg-rose-50 text-rose-700 rounded p-3">Error</div>

{/* ✅ Correct — async/transient */}
toast.error("Error message");

{/* ✅ Correct — inline/persistent */}
<Alert tone="error">Error message</Alert>
```

See [[POS UI Mistakes]] for the full anti-pattern guide.

---

## Related
- [[POS UI Mistakes]] — inline div anti-pattern
- [[Warehouse Receive Flow]] — uses toast for autosave feedback + Alert for validation
- [[Product Form Drawer]] — uses toast for save/delete
- [[Supplier Management]] — uses toast for CRUD results
