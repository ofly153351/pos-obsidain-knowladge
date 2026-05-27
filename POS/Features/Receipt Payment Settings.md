---
tags: [pos, feature, settings, receipt, payment-channels]
created: 2026-05-26
---

# 🧾 Receipt & Payment Settings

← [[POS Features MOC]] · [[Settings Components]] · [[POS Project MOC]]

## Files

| Project | File | Role |
|---------|------|------|
| **Frontend** | `components/settings/receipt-payment-settings.tsx` | Main UI component |
| Frontend | `services/receipt-settings.ts` | API: `getReceiptSettings()`, `updateReceiptSettings()` |
| Frontend | `types/receipt-settings.ts` | `ReceiptSettingsData`, `PaymentChannelSetting`, `UpdateReceiptSettingsInput` |
| Frontend | `app/api/stores/[storeId]/receipt-settings/route.ts` | BFF proxy (GET + PUT) |
| **Backend** | `internal/modules/receipt_settings/` | module: model, repo, service, handler, errors, util |
| Backend | `init-db/006_receipt_settings.sql` | migration: `store_receipt_settings` table (unique per store_id) |

## Backend

- **Routes:** `GET /api/v1/stores/:storeID/receipt-settings` · `PUT /api/v1/stores/:storeID/receipt-settings`
- **GET behavior:** auto-creates default row if none exists (upsert on GET)
- **PUT behavior:** upsert
- **`promptpay_id`** lives on the store record, NOT in `store_receipt_settings`
- **`payment_channels`** stored as JSONB in `store_receipt_settings`
- **idgen prefix:** `"rset"`

## Frontend Component

### Queries

```ts
useQuery(["receipt-settings", storeId])   // receipt design settings
useQuery(["store", storeId])              // store info for preview (name, address, phone, logo_url, promptpay_id)
```

### Local Draft State

Component keeps a local draft state synced from server on load.  
On save: `useMutation` → PUT → `invalidateQueries`.

### Payment Channels

`payment_channels` is JSONB in the backend.  
Frontend merges with static `CHANNEL_META` object to get icons and display names.

### Receipt Preview Panel

Preview is an `<iframe>` that loads HTML from the backend preview endpoint.

> ⚠️ Must use `RenderPanelPreviewHTML()` — NOT `RenderPreviewHTML()`.
> - `RenderPreviewHTML` adds dark overlay + toolbar + paper card → overflows 360px panel
> - `RenderPanelPreviewHTML` = white bg only + 3px minimal scrollbar injected via `<style>`

```go
// backend: internal/modules/receipt_settings/service.go
receipthtml.RenderPanelPreviewHTML(receiptBytes)  // ✅ use this
// NOT: receipthtml.RenderPreviewHTML(receiptBytes)
```

### PromptPay Display

Shows real `promptpay_id` from store API (read-only), with a link to store settings page.

## Toggle Switch Fix

The receipt settings component had the toggle knob bug.  
Fix: `left-0.5 top-0.5` on the knob span + `translate-x-0` off / `translate-x-5` on.  
See [[POS UI Mistakes]].

## Related
- [[Settings Components]] — component map
- [[API Client Patterns]] — `authorizedApiRequest` for settings queries
- [[BFF Proxy Routes]] — receipt-settings BFF route
- [[POS UI Mistakes]] — receipt preview panel note, toggle knob bug
- [[Backend Architecture]] — module registration in `app.go`
