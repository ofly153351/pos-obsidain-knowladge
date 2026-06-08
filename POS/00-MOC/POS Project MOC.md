---
tags: [pos, moc, index]
created: 2026-05-27
updated: 2026-06-04
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
- [[Sales Cashier]] — cashier refactor (14 files), quotation mode, post-invoice popup
- [[Stock Components]] — products table, warehouse, stock levels, catalog setup
- [[Purchasing Components]] — PO list/form, receive modal, supplier manager
- [[Settings Components]] — store management, bank accounts manager, receipt preview
- [[Toast Alert System]] — module-level toast store + inline Alert banner

## ✨ Features
- [[Document Management]] — invoice/DO/quotation actions, cancel, pay, convert, drawer preview
- [[Supplier Management]] — 2-panel master-detail, gradient hero header, add modal
- [[Category Management]] — 3-tab types/units/brands, stats sidebar, product_count subquery
- [[Stock Levels Excel]] — xlsx import/export + 3-step preview flow with auto-create Category/Unit/Brand
- [[Activity Logs]] — auto-log all store mutations, per-module filter UI, Fiber after-middleware
- [[Warehouse Receive Flow]] — 3-step wizard, autosave, barcode scan, PO import
- [[Product Form Drawer]] — left-side drawer, POS live preview card
- [[Receipt Payment Settings]] — live backend preview, payment channels JSONB

## 🔌 Backend
- [[Backend Architecture]] — route→handler→service→repo, cross-store transfer flow
- [[Document PDF & WHT]] — delivery note, bank accounts, PromptPay QR, PayInvoice logic
- [[Store Module]] — bank accounts CRUD, NOT NULL pattern, PromptPay QR generation
- [[Backend Bug Fixes]] — sale 500 error, proxy 404, GORM Take trap, discount_value=0 bug
- [[GORM Patterns]] — read-only `->` tag, anonymous struct, correlated subquery
- [[Backend Docs Map]] — all `pos-backend/docs/*.md` files

## 🔌 Backend (continued)
- [[Backend Architecture]] — structured error format, activity_log middleware, PM2, MinIO config

## ⚙️ Patterns
- [[API Client Patterns]] — authorizedApiRequest vs authorizedRawRequest, envelope unwrapping
- [[BFF Proxy Routes]] — complete Next.js → Go Fiber route map + multipart blob fix rule
- [[Animation Patterns]] — smooth-fade-up, fade-out, no Framer Motion rule
- [[i18n Patterns]] — locale key structure, dictionary types, 100+ keys per feature
- [[Error Handling]] — ApiError fields, friendlyMessage(), FieldError component, backend Err422

## 🚀 Deployment
- [[POS Deployment]] — golive.sh, PM2, Cloudflare Tunnel, MinIO env config, Navicat SSH
