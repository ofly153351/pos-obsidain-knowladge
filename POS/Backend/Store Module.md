---
tags: [pos, backend, store, contact, repository, gorm, bank-accounts]
created: 2026-05-29
updated: 2026-06-01
---

# 🏪 Store Module

← [[POS Backend MOC]] · [[Backend Architecture]]

Module: `internal/modules/store/`

## Schema

```sql
CREATE TABLE stores (
  id            TEXT PRIMARY KEY,
  owner_user_id TEXT NOT NULL,
  name          TEXT NOT NULL,
  logo_url      TEXT,
  phone         TEXT,
  address       TEXT,
  promptpay_id  TEXT,
  currency_code TEXT NOT NULL DEFAULT 'THB',
  tax_id        VARCHAR(13) NOT NULL DEFAULT '',  -- migration 009
  fax           TEXT        NOT NULL DEFAULT '',  -- migration 011
  email         TEXT        NOT NULL DEFAULT '',  -- migration 011
  website       TEXT        NOT NULL DEFAULT '',  -- migration 011
  created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
)
```

```sql
-- migration 014
CREATE TABLE store_bank_accounts (
  id           VARCHAR(30) PRIMARY KEY,  -- prefix: sba-
  store_id     VARCHAR(30) NOT NULL REFERENCES stores(id) ON DELETE CASCADE,
  bank_code    VARCHAR(20) NOT NULL DEFAULT '',
  bank_name    VARCHAR(100) NOT NULL DEFAULT '',
  account_no   VARCHAR(50) NOT NULL DEFAULT '',
  account_name VARCHAR(100) NOT NULL DEFAULT '',
  created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW()
)
```

**Critical rule**: `tax_id`, `fax`, `email`, `website` are `NOT NULL DEFAULT ''`. Use `""` never `nil`. Using `nilIfEmpty()` causes SQLSTATE 23502.

## Bank Accounts CRUD

Routes: `GET/POST /stores/:storeID/bank-accounts` · `DELETE /stores/:storeID/bank-accounts/:accountID`

Service checks `UserCanManageStore` before every operation.  
`CreateBankAccount` generates ID with `idgen.Generate(PrefixStoreBankAccount)` (`"sba"`).

## PromptPay QR → StoreInfo

`GetDocument` now fetches `promptpay_id` from store alongside other contact fields.  
`RenderDocumentPrint` generates QR:
```go
if doc.StorePromptPayID != "" {
    docData.QRPaymentURL = dochtml.BuildPromptPayQRDataURI(doc.StorePromptPayID, doc.TotalAmount)
}
```
`QRPaymentURL` is `template.URL` (not `string`) to prevent `html/template` sanitization.

## StoreInfo (passed to dochtml)

```go
dochtml.StoreInfo{
    Name: ..., Address: ..., Phone: ...,
    Fax: ..., Email: ..., Website: ...,
    TaxID: ..., LogoURL: ...,
    PromptPayID: doc.StorePromptPayID,
    BankAccounts: []dochtml.BankAccountInfo{...},  // from store_bank_accounts
}
```

## Repository Patterns

### NOT NULL fields — bare string, not nilIfEmpty
```go
// ❌ Wrong
"tax_id": nilIfEmpty(update.TaxID),
// ✅ Correct
"tax_id": update.TaxID,
```
`phone` and `address` are nullable → `nilIfEmpty()` correct for those.

## Frontend — Bank Account Manager

`components/store/store-management-panel.tsx` — Payment section:
- Dropdown: 15 Thai banks (BBL, KBANK, SCB, KTB, BAY, TTB, GSB, BAAC, UOB, CIMBT, LHB, TISCO, KK, CITI, OTHER)
- Inputs: เลขบัญชี + ชื่อบัญชี
- List with delete button (trash icon, rose hover)
- Loaded via `listBankAccounts(storeId)` on store select
- Service: `services/stores.ts` — `listBankAccounts`, `createBankAccount`, `deleteBankAccount`
- BFF: `app/api/stores/[storeId]/bank-accounts/` POST needs `body: await request.text()`

## Migrations
| File | Content |
|------|---------|
| `009` | `stores.tax_id` |
| `011` | `stores.fax`, `email`, `website` |
| `014` | `store_bank_accounts` table |

## Related
- [[Document PDF & WHT]] — StoreInfo → dochtml, bank accounts in delivery note, PromptPay QR
- [[Backend Bug Fixes]] — NOT NULL violation pattern
- [[BFF Proxy Routes]] — bank-accounts BFF routes
