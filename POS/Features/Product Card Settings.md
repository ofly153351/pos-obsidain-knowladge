---
tags: [pos, feature, sale, cashier, card, settings, per-user, jsonb]
created: 2026-06-08
---

# 🎴 Product Card Settings (per-user)

← [[POS Features MOC]] · [[Sales Cashier]] · [[Backend Architecture]] · [[POS Project MOC]]

Merchant-tunable product-card appearance for the cashier grid, **persisted per user** (server + localStorage cache).

---

## Frontend

### ProductCard — `components/sales/product-card.tsx`
Pure/presentational. **Tailwind literal class maps → no safelist needed.**

```ts
const ASPECT = { "1/1":"aspect-square", "4/3":"aspect-[4/3]", "3/4":"aspect-[3/4]" };
const CLAMP  = { 1:"line-clamp-1", 2:"line-clamp-2", 3:"line-clamp-3" };
const SIZE   = { sm:{…}, md:{…}, lg:{…} }; // scales EVERY dimension per size
```

Anti-squeeze rules (the two bugs it prevents):
- **Image never squeezed** → image box uses a **fixed px height** from
  `resolveCardImageHeight(settings)` (lib/card-settings.ts, `CARD_IMAGE_HEIGHT[size][aspect]`) + `shrink-0`.
  > Updated 2026-06-09 (Wutthichai): switched **from CSS `aspect-ratio` → fixed px height**.
  > aspect-ratio got flex-squeezed into a thin strip in `cover` mode and went tiny in `contain`.
  > The resolved height is the single source of truth shared by the real grid **and** the modal
  > preview, so they always match. `contain` → `object-contain p-2`; card no longer uses `h-full`.
- **Long name never breaks layout** → `line-clamp-N` + inline `style.minHeight = N × 1.35em` → uniform card height, price row aligned.

`size` (sm/md/lg) scales font / padding / add-button / initials / badges together (not just grid column width — that was the "sizes feel wrong" fix). Grid column width = `minmax(CARD_SIZE_MIN[size], 1fr)` set by the parent.

Props: `{ item, config, qtyInCart, onAdd, labels }`. `item` is a minimal shape (`{id,name,price,stock,image?}`) so the same card renders real products AND modal preview samples.

### CardSettingsModal — `components/sales/card-settings-modal.tsx`
6 seg-controls (namePos, fit, aspect, lines, size, showStock) + **live preview** (reuses ProductCard) + reset/save. Escape/click-outside to close.

### Settings button
In `product-browser.tsx`, left of the grid/list toggle (`SlidersHorizontal` + "ตั้งค่าการ์ด") → opens modal (does **not** navigate; cart preserved).

### Grid = 3 rows per page
`ResizeObserver` measures container width → `cols = floor((w+gap)/(min+gap))` → `pageSize = cols × 3`. Recomputes on resize + size change.

### State / sync — `lib/card-settings.ts` + `services/card-settings.ts`
- `CardSettings` type + `DEFAULT_CARD_SETTINGS` + `CARD_SIZE_MIN`.
- localStorage key `pos-card-settings:<userId>` (from auth session) = fast cache.
- `saveCardSettings()` writes cache + dispatches `pos-card-settings-changed`; `cacheCardSettings()` writes cache **without** event (hydrate from server, avoids re-render loop).
- Flow: render from cache instantly → `fetchCardSettings()` reconciles with server → modal save writes cache (instant apply) + `updateCardSettings()` PUT server.

---

## Backend — `usersettings` module

- **Migration `019`**: `users.card_settings JSONB NOT NULL DEFAULT '{}'`.
- Routes: `GET/PUT /api/v1/me/card-settings` — user id from **JWT claims** (never sent by client).
- `CardSettings` implements `driver.Valuer` + `sql.Scanner` for JSONB round-trip — **do not** raw-`Update([]byte)`/`Scan(&[]byte)` (→ jsonb↔bytea error). Same pattern as receipt_settings PaymentChannels.
- Service `normalize()` clamps every field to allowed values (invalid → default); unset (`{}` / zero Size) → defaults.
- BFF: `app/api/me/card-settings/route.ts` (GET/PUT proxy).
- Tests: `internal/tests/usersettings/` (5 unit + verified real DB round-trip incl `showStock:false`).

Stored JSON shape:
```json
{"fit":"contain","size":"lg","lines":3,"aspect":"3/4","namePos":"top","showStock":false}
```

---

## Related
- [[Sales Cashier]] — cashier modal hosting the grid
- [[Backend Architecture]] — module wiring, JSONB Valuer/Scanner pattern
- [[Receipt Payment Settings]] — sibling per-store settings (this is per-user)
