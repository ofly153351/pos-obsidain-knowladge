---
tags: [pos, theme, tailwind, design-system, violet]
created: 2026-05-22
updated: 2026-05-27
no-gradient: true — all brand gradients (violet→pink) removed 2026-05-27
---

# 🎨 POS Today Theme — Design System

← [[POS Project MOC]] · See also: [[POS UI Mistakes]]

The entire app uses a **violet/purple** palette. Token file: `app/globals.css`.

> ⚠️ `sky-*` classes are remapped to violet via `@theme inline` in `globals.css`.  
> They may not work reliably at runtime — always use explicit `violet-*` classes.  
> See [[POS UI Mistakes]].

---

## Core Palette

| Role | Tailwind Classes | Note |
|------|-----------------|------|
| **Primary CTA** | `bg-violet-600 text-white hover:bg-violet-700` | no gradient — solid violet only |
| **Solid primary** | `bg-violet-600 text-white` | text MUST be white on violet-600 |
| **Secondary btn** | `border border-violet-200 bg-white text-violet-700 hover:bg-violet-50` | |
| **Ghost btn** | `text-violet-600 hover:bg-violet-50` | |
| **Danger btn** | `bg-red-600 text-white` / `border border-red-200 text-red-600` | keep red, not violet |
| **Page bg** | `bg-[linear-gradient(160deg,_#f5f3ff_0%,_#faf5ff_35%,_#f8fafc_100%)]` | violet-tinted gradient |
| **Card** | `bg-white border border-violet-100 shadow-sm rounded-xl` | |
| **Section bg** | `bg-violet-50/40` | empty states, table backgrounds |
| **Input** | `border-violet-200 focus:border-violet-400 focus:ring-2 focus:ring-violet-100` | |
| **Input error** | `border-rose-300 bg-rose-50/70` | |
| **Badge neutral** | `bg-violet-100 text-violet-700` | |
| **Avatar/Initials** | `bg-violet-600 text-white` | |
| **Sidebar bg** | `bg-indigo-950` | dark indigo, NOT violet |

---

## Button Rules

| Type | Classes |
|------|---------|
| CTA / Primary | `bg-violet-600 text-white hover:bg-violet-700` |
| Secondary | `border border-violet-200 bg-white text-violet-700 hover:bg-violet-50` |
| Ghost | `text-violet-600 hover:bg-violet-50` |
| Danger | `bg-red-600 text-white` or `border border-red-200 text-red-600 hover:bg-red-50` |
| Disabled | add `disabled:opacity-60` — never remove pointer-events |

> `button:not(:disabled) { cursor: pointer }` is global in `globals.css` — **do NOT** add `cursor-pointer` to individual buttons.

---

## Sidebar (Dark Indigo `bg-indigo-950`)

```
Active item:    border-l-[3px] border-violet-400 bg-violet-900 text-white rounded-r-lg
Inactive item:  text-violet-300 hover:bg-violet-900/50 hover:text-white hover:rounded-r-lg
Sub active:     bg-violet-800/80 text-violet-200
Sub inactive:   text-violet-400 hover:bg-violet-900/60 hover:text-white hover:rounded-xl
Icon inactive:  bg-violet-900/60 text-violet-300
Store card:     border border-violet-800/50 bg-violet-900/30
```

---

## Status Colors

| Status | Tailwind |
|--------|----------|
| Active / Success | `bg-emerald-100 text-emerald-700` (badge) |
| Warning | `bg-amber-100 text-amber-700` |
| Error / Danger | `bg-red-50 text-red-600` |
| Inactive / Neutral | `bg-slate-100 text-slate-500` |

---

## Buttons on Dark Backgrounds

When content sits on `bg-violet-600` or `bg-violet-700` (hero headers, modal headers):

```tsx
{/* Primary action → white solid */}
<button className="bg-white text-violet-700 shadow-sm hover:bg-violet-50" />

{/* Secondary/ghost → transparent white */}
<a className="bg-white/15 text-white backdrop-blur-sm hover:bg-white/25" />

{/* Danger on dark */}
<button className="bg-white/15 text-white/90 hover:bg-red-400/30" />
```

Status badge on dark bg (not the standard emerald/slate tokens):
```tsx
{/* Active on dark */}
<span className="bg-emerald-400/20 text-emerald-200 ring-1 ring-emerald-300/40" />
{/* Inactive on dark */}
<span className="bg-white/20 text-white/70 ring-1 ring-white/30" />
```

---

## Toggle Switch

```tsx
<button className={`relative h-6 w-11 rounded-full transition-colors ${checked ? "bg-violet-600" : "bg-slate-300"}`}>
  <span className={`absolute left-0.5 top-0.5 h-5 w-5 rounded-full bg-white shadow transition-transform
    ${checked ? "translate-x-5" : "translate-x-0"}`} />
</button>
```

> Must have `left-0.5` for symmetric 2px gap on both sides.  
> Off-state = `translate-x-0`, On-state = `translate-x-5`.  
> See [[POS UI Mistakes]] for the bug where this was `translate-x-0.5` (asymmetric).

---

## KPI Card — Primary Variant

The most important KPI in a row gets `primary` styling (solid violet, no gradient):

```tsx
{/* primary KPI */}
<div className="bg-violet-600 rounded-xl p-4 shadow-md shadow-violet-200/60">
  <div className="h-8 w-8 rounded-lg bg-white/20 text-white">{icon}</div>
  <p className="text-xl font-bold text-white tabular-nums">{value}</p>
  <p className="text-xs text-violet-100">{title}</p>
</div>
```

---

## Fonts

- `Sarabun` (Thai-compatible sans) — loaded in `globals.css`
- `JetBrains Mono` — for monetary values, codes, document numbers

---

## Related
- [[POS UI Mistakes]] — what NOT to do
- [[Layout Components]] — sidebar implementation
- [[Supplier Management]] — gradient hero header pattern
- [[Animation Patterns]] — transition classes in `globals.css`
