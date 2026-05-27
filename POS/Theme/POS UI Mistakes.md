---
tags: [pos, theme, anti-pattern, bugs, contrast]
created: 2026-05-22
updated: 2026-05-27
no-gradient: added 2026-05-27
---

# ⚠️ POS UI — Common Mistakes & Anti-Patterns

← [[POS Today Theme]] · [[POS Project MOC]]

---

## 🔴 Critical: `sky-*` Classes

`sky-*` utilities are remapped to violet via `@theme inline` in `globals.css`.
They are **not reliably remapped at runtime** — always use explicit `violet-*`.

```bash
# Safe search to catch sky-* usage:
grep -r "sky-" components/ --include="*.tsx"
```

**Bug history:** A greedy sed replacement turned `bg-sky-50` → `bg-violet-600` instead of `bg-violet-50`, making text invisible.

---

## 🔴 Text Contrast on `bg-violet-600`

`bg-violet-600 text-violet-700` → **invisible** (dark purple on dark purple).

| ❌ Wrong | ✅ Correct |
|---------|----------|
| `bg-violet-600 text-violet-700` | `bg-violet-600 text-white` |
| `bg-violet-600 text-violet-900` | `bg-violet-600 text-white` |

**Applies to:** selected store card, switch store button, any solid violet background.

---

## 🔴 Toggle Knob Asymmetry

**Bug:** `translate-x-0.5` without `left-0.5` → knob at x=2px off, but x=20px on (asymmetric).

```tsx
{/* ❌ Wrong */}
<span className={`absolute top-0.5 h-5 w-5 ... ${checked ? "translate-x-5" : "translate-x-0.5"}`} />

{/* ✅ Correct */}
<span className={`absolute left-0.5 top-0.5 h-5 w-5 ... ${checked ? "translate-x-5" : "translate-x-0"}`} />
```

Rule: `left-0.5` (2px from left) + `translate-x-0` off / `translate-x-5` (20px) on = 2px gap each side.

See: [[POS Today Theme]] for full toggle snippet.

---

## 🔴 Inline Error Divs

Never write inline validation styling:

```tsx
{/* ❌ Wrong */}
<div className="border-rose-200 bg-rose-50 text-rose-700 p-3 rounded">Error message</div>

{/* ✅ Correct */}
import { Alert } from "@/components/ui/alert";
<Alert tone="error">Error message</Alert>
```

See: [[Toast Alert System]]

---

## 🟡 Receipt Preview Panel

Settings receipt preview must use `RenderPanelPreviewHTML()` (backend), NOT `RenderPreviewHTML()`.
- `RenderPreviewHTML` adds dark overlay + toolbar + paper card → overflows the 360px panel
- `RenderPanelPreviewHTML` = white bg only, 3px minimal scrollbar injected via `<style>`

See: [[Receipt Payment Settings]]

---

## 🔴 Brand Gradients (violet→pink)

`from-violet-600 to-pink-500` and any colorful multi-hue gradient are **banned** (removed 2026-05-27).

| ❌ Wrong | ✅ Correct |
|---------|----------|
| `bg-gradient-to-br from-violet-600 to-pink-500` | `bg-violet-600` |
| `hover:from-violet-700 hover:to-pink-600` | `hover:bg-violet-700` |
| `bg-gradient-to-br from-violet-600 via-violet-700 to-purple-800` | `bg-violet-700` |
| `bg-gradient-to-br from-violet-400 to-violet-600` | `bg-violet-600` |
| `bg-gradient-to-r from-violet-700 to-purple-500` | `bg-violet-700` |
| `bg-gradient-to-b from-violet-600 to-violet-700` (active button) | `bg-violet-600` |

**Allowed (subtle tints only):**
- `from-violet-50 to-white` — barely-visible tint for headers/footers
- `from-emerald-50 to-white`, `from-slate-100 to-white` — status tints (DRAWER_BG)
- Page bg: `bg-[linear-gradient(160deg,...)]`

---

## 🟡 Modal Transition (Horizontal Slide)

`modal-slide-in` / `modal-slide-out` slide horizontally — feels wrong for centered modals.

```tsx
{/* ❌ Wrong — slides from left */}
className={isClosing ? "modal-slide-out" : "modal-slide-in"}

{/* ✅ Correct — normal fade */}
className={isClosing ? "fade-out" : "smooth-fade-up"}
```

See: [[Animation Patterns]]

---

## 🟡 `cursor-pointer` on Buttons

`button:not(:disabled) { cursor: pointer }` is set globally in `globals.css`.
Adding `cursor-pointer` to individual buttons is redundant.
Disabled buttons automatically get the `default` cursor.

---

## 🟡 Hardcoded Strings

Never hardcode UI text. Always use i18n:

```tsx
{/* ❌ Wrong */}
<button>บันทึก</button>

{/* ✅ Correct */}
<button>{dict.save}</button>
```

See: [[i18n Patterns]]
