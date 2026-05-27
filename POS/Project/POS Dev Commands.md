---
tags: [pos, project, commands, cli]
created: 2026-05-22
---

# POS Dev Commands

← [[POS Project Overview]]

## Frontend

```bash
cd /Users/obx/projects/pos-frontend
npm run dev                            # start dev server → port 3000
npx tsc --noEmit                       # type check (no output = clean ✅)
npx tsc --noEmit --skipLibCheck        # type check, skip lib types
git log --oneline -10                  # recent commits
```

## Backend

```bash
cd /Users/obx/projects/pos-backend
go run main.go                         # start backend → port 8080
go build ./...                         # compile check
```

## Pre-Commit Checklist

1. `npx tsc --noEmit` — zero errors
2. Add i18n keys to **both** `locales/th.json` and `locales/en.json`
3. Grep for `sky-` — see [[POS UI Mistakes]] for why `sky-*` is dangerous
4. `npm run dev` → open changed page in browser, check visually
5. No hardcoded Thai/English strings — must be in locale files
6. No inline `border-rose-200 bg-rose-50` divs — use [[Toast Alert System]]
