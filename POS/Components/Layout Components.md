---
tags: [pos, component, layout, navigation]
created: 2026-05-22
---

# 🏗️ Layout Components

← [[POS Frontend MOC]] · [[POS Project MOC]]

## File Map

| File | Role |
|------|------|
| `components/layouts/user-workspace-layout.tsx` | Main shell: sidebar + topbar + `<main>` padding |
| `components/layouts/user-workspace-shell.tsx` | Shell variant with nav tabs (legacy pages) |
| `components/navigation/user-workspace-sidebar.tsx` | Dark indigo sidebar with expandable nav groups |
| `components/navigation/user-workspace-topbar.tsx` | Header bar — locale toggle, store switcher, profile menu |
| `components/navigation/user-profile-menu.tsx` | Dropdown: edit profile, logout |
| `components/sales/cashier-modal.tsx` | Full-screen cashier overlay (mounts over workspace) |

## Sidebar Styling

Container: `bg-indigo-950`

```
Active:     border-l-[3px] border-violet-400 bg-violet-900 text-white rounded-r-lg
Inactive:   text-violet-300 hover:bg-violet-900/50 hover:text-white hover:rounded-r-lg
Sub active:   bg-violet-800/80 text-violet-200
Sub inactive: text-violet-400 hover:bg-violet-900/60 hover:text-white hover:rounded-xl
Icon:         bg-violet-900/60 text-violet-300
Store card:   border border-violet-800/50 bg-violet-900/30
```

Selected store card: `from-violet-600 to-violet-700 text-white`  
Switch store button: gradient + `text-white` (NOT `text-violet-7xx` — see [[POS UI Mistakes]])

## Workspace Padding

Main content area has `p-6 lg:p-8`. Pages that need full-bleed (e.g. [[Supplier Management]], [[Warehouse Receive Flow]]) use:

```tsx
<div className="-mx-6 -my-6 lg:-mx-8 lg:-my-8 flex flex-col overflow-hidden"
     style={{ height: "calc(100dvh - 4.5rem)" }}>
```

The `4.5rem` accounts for the topbar height.

## Related
- [[POS Today Theme]] — all sidebar token values
- [[Supplier Management]] — uses full-bleed breakout layout
