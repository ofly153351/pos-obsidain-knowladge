---
tags: [pos, feature, auth, login, register, i18n]
created: 2026-06-08
---

# 🔐 Auth Pages (Login / Register)

← [[POS Features MOC]] · [[POS Project MOC]]

2-panel auth layout shared by login + register. Redesigned 2026-06-05.

## Files
- `components/auth/auth-shell.tsx` — 2-panel chrome + brand panel + language dropdown
- `components/auth/auth-form.tsx` — fields, validation, submit, redirect (all logic here)
- `app/[locale]/(user)/login/page.tsx`, `.../register/page.tsx` — server pages pass dict + `authBrand`

## Layout
- **Left (brand):** violet gradient + grid overlay + glass rings, logo tile, badge, hero, 4 feature cards, 3 stats, version footer.
- **Right (form):** white card — eyebrow → title → fields → submit; language **dropdown** top-right (TH/EN, rotate chevron, click-outside + Escape, active ✓).

## AuthForm logic (kept intact through redesign)
- Leading icons (Mail/Lock/Store/User/Phone), password show/hide toggle.
- **Login:** validates required + email format only (password length NOT enforced — accepts any existing pw; backend decides). Red error text under each field; server error → banner.
- **Register:** strength meter (score 0–4), confirm-password match (`confirmOf`, excluded from payload), required terms checkbox, 2-col grid via `half` field flag.
- Redirect: login + store_id → `/dashboard`; else → `/subscription`.
- i18n: `common.authBrand` + `login.*` / `register.*` (strength/terms/confirm keys), th/en.

## Gotchas fixed
- Login was redirecting via `/sales` bounce → now straight to `/dashboard`.
- Password-min-8 was blocking login for short existing passwords → moved to register only.
- Remember/terms checkmark: `peer-checked` on a nested svg doesn't work (sibling combinator) → use state-driven `opacity` instead.

## Related
- [[i18n Patterns]] · [[Error Handling]] · [[react-smooth-ux]]
