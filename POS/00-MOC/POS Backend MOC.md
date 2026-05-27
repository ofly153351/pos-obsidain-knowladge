---
tags: [pos, moc, backend, go]
created: 2026-05-27
---

# 🔌 POS Backend — Architecture Map

← [[POS Project MOC]]

## Architecture
- [[Backend Architecture]] — Go Fiber v2, route→handler→service→repo, cross-store warehouse transfer

## Bugs Fixed
- [[Backend Bug Fixes]] — sale 500, proxy 404, GORM Take silent empty string, discount_value=0 400 error

## Data Layer
- [[GORM Patterns]] — `gorm:"->"` read-only, anonymous struct for single-field lookups, correlated subquery for `product_count`

## API & Proxy
- [[BFF Proxy Routes]] — full BFF → Go route table, `storeId` NOT forwarded as path param
- [[API Client Patterns]] — `authorizedApiRequest` (unwraps envelope) vs `authorizedRawRequest` (raw body)

## Documentation
- [[Backend Docs Map]] — all `pos-backend/docs/*.md` + rules for keeping docs updated

## Key Rules
- Route → Handler → Service → Repository — never skip layers
- All backend docs in English
- After any backend change → update `docs/` file for that module
- GORM `Take(&string)` → silent empty — always use anonymous struct
