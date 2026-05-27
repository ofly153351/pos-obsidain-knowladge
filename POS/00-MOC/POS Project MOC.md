---
tags: [pos, moc, index]
created: 2026-05-27
updated: 2026-05-27
---

# 🗺️ POS Project — Master Map of Content

> **POS Today** — Retail Point-of-Sale platform.
> Frontend: Next.js 15 + TypeScript + Tailwind CSS 4 + TanStack Query
> Backend: Go Fiber v2 + GORM + PostgreSQL · Branch: `fix-of/dev`

---

## 🏗️ Project
- [[POS Project Overview]] — paths, stack, app router structure, commit log
- [[POS Dev Commands]] — frontend, backend, pre-commit checklist

## 🎨 Design System
- [[POS Today Theme]] — full violet/purple palette, button variants, sidebar tokens, avatars
- [[POS UI Mistakes]] — contrast bugs, sky-\* remapping trap, toggle knob alignment

## 📦 Components
- [[Layout Components]] — workspace shell, sidebar, topbar, profile menu
- [[Sales Cashier]] — product browser, cart panel (420px), VAT toggle, exact-amount
- [[Stock Components]] — products table, warehouse, stock levels, catalog setup
- [[Purchasing Components]] — PO list/form, receive modal, supplier manager
- [[Settings Components]] — store management panel, receipt preview iframe
- [[Toast Alert System]] — module-level toast store + inline Alert banner

## ✨ Features
- [[Supplier Management]] — 2-panel master-detail, gradient hero header, add modal
- [[Category Management]] — 3-tab types/units/brands, stats sidebar, product_count subquery
- [[Stock Levels Excel]] — xlsx import/export replacing CSV
- [[Warehouse Receive Flow]] — 3-step wizard, autosave, barcode scan, PO import
- [[Product Form Drawer]] — left-side drawer, POS live preview card
- [[Receipt Payment Settings]] — live backend preview, payment channels JSONB

## 🔌 Backend
- [[Backend Architecture]] — route→handler→service→repo, cross-store transfer flow
- [[Backend Bug Fixes]] — sale 500 error, proxy 404, GORM Take trap, discount_value=0 bug
- [[GORM Patterns]] — read-only `->` tag, anonymous struct, correlated subquery
- [[Backend Docs Map]] — all `pos-backend/docs/*.md` files

## ⚙️ Patterns
- [[API Client Patterns]] — authorizedApiRequest vs authorizedRawRequest, envelope unwrapping
- [[BFF Proxy Routes]] — complete Next.js → Go Fiber route map
- [[Animation Patterns]] — smooth-fade-up, fade-out, no Framer Motion rule
- [[i18n Patterns]] — locale key structure, dictionary types, 100+ keys per feature
