---
tags: [pos, soul, philosophy, principles, identity]
created: 2026-06-09
---

# 🫀 SOUL — POS System

> The "why" behind the code. Read this before the specs.
> When a decision is unclear, decide in the direction this document points.

← [[POS Project MOC]]

---

## What this is

A **point-of-sale system for Thai retail shops** — built first for a gas/อุปกรณ์ shop,
designed to fit any small store: front-counter selling, stock, documents, and the
back-office that keeps them honest.

Two repos, one product:
- **pos-backend** — Go (Fiber + GORM + PostgreSQL), clean architecture, the source of truth.
- **pos-frontend** — Next.js App Router, the fast hands of the cashier.

It runs on a single Ubuntu box, exposed through a Cloudflare tunnel, deployed with one script (`golive.sh`).
No Kubernetes, no microservices. **Boring infrastructure so the product can be interesting.**

---

## Who we serve

The **cashier** at the counter, mid-rush, one hand on a barcode scanner.
The **owner** who opens the shop and wants the day to "just work."
Neither of them reads manuals. Both of them notice every extra tap and every half-second of lag.

Every feature is judged by one question: **does it make the next sale faster or the books more correct?**
If it does neither, it doesn't ship.

---

## Core beliefs

1. **Speed at the counter is sacred.** The cashier flow is the heart. A modal that opens
   instantly, a card that doesn't reflow, a checkout that never double-charges — these matter
   more than any dashboard. Optimistic where safe, confirmed where money moves.

2. **Correctness of money and stock is non-negotiable.** Totals, VAT, change, stock movements,
   and documents must always reconcile. The backend validates everything; the frontend never
   trusts itself for the final number.

3. **The merchant configures, we don't hardcode.** Receipt layout, payment channels, product-card
   appearance, tax mode — tunable without a deploy. Per-store settings live with the store;
   per-user preferences live with the user (and follow them across devices).

4. **Thai-first, bilingual always.** Every string is in `th.json` / `en.json`. Thai is the
   working language of the shop; English is there for whoever needs it. No hardcoded text, ever.

5. **It should survive a bad network.** localStorage caches, graceful fallbacks, "saved locally"
   when the server hiccups. The shop doesn't stop because the internet did.

6. **Errors must be honest and human.** Structured errors from the backend (`code` + field-level
   messages), red text under the field that's wrong, a toast that says what actually happened —
   never a silent failure or a bare "internal server error."

---

## Design soul — POS Today (violet)

- **One brand colour: violet** (`#7C3AED`). No rainbow gradients, no competing hues. Status colours
  (emerald=ok, rose=danger, amber=warn) earn their place; everything else is violet + slate + white.
- **Calm, generous, uncramped.** Rounded cards, soft shadows, real whitespace. The screen should
  feel premium even on a cheap counter monitor.
- **Motion has a purpose.** Smooth enter/exit (150–250ms), `Escape` closes everything, no animation
  for decoration's sake. If it doesn't guide attention, it doesn't move.
- **Numbers are monospace + tabular.** Prices, totals, stock counts line up. Money is read at a glance.
- **Uniformity beats cleverness.** Cards in a row are the same height; images never squeeze; long
  names clamp. Predictable layout > pixel-perfect special cases.

See [[POS Today Theme]] · [[POS UI Mistakes]] · [[react-smooth-ux]]

---

## Engineering soul

- **Clean architecture, thin routes.** handler → service → repository. Business logic never lives
  in HTTP handlers or React components. See [[go-clean-arch]] · [[Backend Architecture]].
- **One standard envelope, one error shape.** Every response wrapped; every error carries a code +
  fields. See [[Error Handling]] · [[go-http-envelope]].
- **Tests come with the change, not after.** Backend features ship with tests in
  `internal/tests/<module>/`. New logic → write the test first, then hand it over.
- **Reuse the proven pattern.** JSONB? Use the Valuer/Scanner pattern. New page? Use the existing
  service layer + TanStack Query + BFF proxy. Don't reinvent; extend.
- **The frontend never talks to the DB, the backend never renders UI.** BFF proxy routes bridge
  them; tokens are injected server-side, never exposed.
- **Migrations are forward-only, numbered, idempotent.** `init-db/NNN_*.sql`, auto-run on boot.

---

## Domain language (speak the shop's words)

| Code | Shop |
|------|------|
| sale / cashier | หน้าขาย |
| parked bill | พักบิล / เรียกบิล |
| receipt / abbreviated tax invoice | ใบเสร็จ / ใบกำกับภาษีอย่างย่อ |
| tax invoice, delivery note, quotation | ใบกำกับภาษี, ใบส่งของ, ใบเสนอราคา |
| stock / warehouse | สต็อก / คลัง |
| store vs user settings | ตั้งค่าร้าน vs ตั้งค่าผู้ใช้ |

When naming things, prefer the word the cashier would say.

---

## Non-negotiables (the lines we don't cross)

- ❌ No hardcoded user-facing text — i18n or it doesn't exist.
- ❌ No new libraries when the stack already solves it (Zustand, TanStack Query, toast, Tailwind).
- ❌ No money/stock math trusted from the client as final.
- ❌ No silent catch — surface it (toast / field error / log).
- ❌ No secrets in the repo, in chat, or baked into git remote URLs.
- ❌ No writing to the old monolithic knowledge file — the structured vault is the truth.

---

## How we ship

- Work on `fix-of/dev`. Commit small, message the *why*.
- Promote to `main` when ready — **production deploys from `main`**.
- `golive.sh` on the server: sync to `origin/main` (hard reset), build, PM2, tunnel. One command.
- After every code change, update the relevant note in this vault.

See [[POS Deployment]] · [[POS Dev Commands]].

---

## The test of a good change

> Would the cashier feel it as *faster or clearer*?
> Would the owner trust the number more?
> Could the next developer understand it from the vault alone?

If yes to all three — it has soul. Ship it.
