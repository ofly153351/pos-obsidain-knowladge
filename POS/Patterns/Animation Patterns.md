---
tags: [pos, pattern, animation, css, transition, tailwind]
created: 2026-05-24
updated: 2026-05-27
---

# 🎬 Animation Patterns

← [[POS Frontend MOC]] · [[POS Project MOC]]

All animations are **CSS keyframes** defined in `app/globals.css`.  
**No Framer Motion** — prohibited by AGENTS.md (no new libraries).

---

## Available Classes

| Class             | Keyframe                          | Duration | Use for                                    |
| ----------------- | --------------------------------- | -------- | ------------------------------------------ |
| `smooth-fade-up`  | opacity 0→1 + translateY(10px→0)  | 320ms    | Page content, panels, modals entering      |
| `smooth-fade`     | opacity 0→1                       | 220ms    | Simple backdrop fade-in                    |
| `fade-out`        | opacity 1→0 + scale(1→0.97)       | 180ms    | Modals/panels exiting                      |
| `field-slide-in`  | opacity 0→1 + translateX(-20px→0) | 200ms    | Conditional form field appearance          |
| `error-slide-in`  | opacity 0→1 + translateX(-12px→0) | 180ms    | Validation error messages                  |
| `modal-slide-in`  | opacity 0→1 + translateX(-100%→0) | 320ms    | ⚠️ Legacy only — do NOT use for new modals |
| `modal-slide-out` | opacity 1→0 + translateX(0→60px)  | 200ms    | ⚠️ Legacy only                             |
| `detail-slide-in` | opacity 0→1 + translateX(-80px→0) | 300ms    | ⚠️ Deprecated — use `smooth-fade-up`       |
| `slide-in-right`  | opacity 0→1 + translateX(100%→0)  | 300ms    | **Right-side drawers** (Document A4 preview) |

Delay helpers: `smooth-delay-1` (60ms) · `smooth-delay-2` (120ms)

---

## Rules

### ✅ Modals — Use `smooth-fade-up` + `fade-out`

```tsx
<div className={`... ${isClosing ? "fade-out" : "smooth-fade-up"}`}
     onAnimationEnd={() => { if (isClosing) onClose(); }}>
```

> ❌ Do NOT use `modal-slide-in` / `modal-slide-out` — horizontal slides feel wrong for centered modals.

### ✅ Detail Panels — Use `smooth-fade-up` with `key` Re-Mount

```tsx
{/* key prop forces React to unmount/remount → CSS animation re-fires automatically */}
<div key={selectedId} className="smooth-fade-up h-full">
  <SupplierDetail ... />
</div>
```

> ❌ Do NOT use `detail-slide-in` — slides from left, inconsistent with other pages.

### ✅ Conditional Form Fields — Use `field-slide-in`

```tsx
{/* key= on the wrapper triggers re-mount + animation when payment method changes */}
<div key={form.paymentMethod} className="field-slide-in">
  {form.paymentMethod === "promptpay" ? <PromptPayFields /> : <BankFields />}
</div>
```

### ✅ Left-Side Drawers — Use CSS `translate-x` Transition

```tsx
// No className animation — use Tailwind transition + JS state
<div className={`transition-transform duration-300 ease-out
  ${isOpen ? "translate-x-0" : "-translate-x-full"}`}>
```

---

## Reduced Motion

All animation classes are disabled under `prefers-reduced-motion: reduce` via `globals.css`:

```css
@media (prefers-reduced-motion: reduce) {
  .smooth-fade-up, .smooth-fade, .field-slide-in, .error-slide-in,
  .modal-slide-in, .modal-slide-out, .fade-out {
    animation: none !important;
  }
}
```

---

## `isClosing` Pattern for Exit Animation

```tsx
const [isClosing, setIsClosing] = useState(false);

function triggerClose() { setIsClosing(true); }
function handleAnimationEnd() { if (isClosing) onClose(); }

// In render:
<div className={isClosing ? "fade-out" : "smooth-fade-up"}
     onAnimationEnd={handleAnimationEnd}>
```

Cancel + dirty state:
```tsx
function handleCancel() {
  if (isDirty) { setShowCancelConfirm(true); }
  else { triggerClose(); }
}
```

---

## Keyframe Definitions (globals.css)

```css
@keyframes smooth-fade-up {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}
@keyframes fade-out {
  from { opacity: 1; transform: scale(1); }
  to   { opacity: 0; transform: scale(0.97); }
}
@keyframes field-slide-in {
  from { opacity: 0; transform: translateX(-20px); }
  to   { opacity: 1; transform: translateX(0); }
}
@keyframes error-slide-in {
  from { opacity: 0; transform: translateX(-12px); }
  to   { opacity: 1; transform: translateX(0); }
}
```

---

## Related
- [[Supplier Management]] — `smooth-fade-up` detail panel, `fade-out` modal
- [[Product Form Drawer]] — `translate-x` drawer slide
- [[POS UI Mistakes]] — `modal-slide-in` anti-pattern
