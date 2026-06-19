---
tags: [pos, feature, customer-display, promptpay, broadcastchannel, frontend]
created: 2026-06-19
author: Wutthichai (fix-kaew/dev)
---

# 🖥️ Customer Display (Screen 2)

← [[POS Features MOC]] · [[Sales Cashier]] · [[POS Project MOC]]

Second-screen customer-facing display (FE `b70ffb4`, BE `e7216e4`).

## How it works
- Page `app/[locale]/customer-display/page.tsx` — opens on a second monitor, **outside** the workspace layout.
- `lib/customer-display.ts` (~86 lines) syncs the live cart from the cashier to the display via the **`BroadcastChannel`** API (same-origin, no backend round-trip).
- Shows a **dynamic PromptPay QR** for the current total: `GET /stores/:storeID/promptpay-qr` (BE `payment` module, `internal/modules/payment/handler.go`) generates the QR payload server-side; `services/payment.ts` + BFF `app/api/stores/[storeId]/promptpay-qr/route.ts`.

## Related
- [[Sales Cashier]] · [[Receipt Payment Settings]] · [[BFF Proxy Routes]]
