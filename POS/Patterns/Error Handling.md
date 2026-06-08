---
tags: [pos, pattern, error, validation, toast, inline, frontend, backend]
created: 2026-06-04
updated: 2026-06-04
---

# 🚨 Error Handling Patterns

← [[POS Frontend MOC]] · [[POS Backend MOC]] · [[POS Project MOC]]

---

## Backend — Structured Errors

`internal/platform/httpx/response.go`

### Error codes

| Code | HTTP | Helper |
|------|------|--------|
| `VALIDATION_ERROR` | 422 | `Err422(c, field, msg)` / `ValidationError(c, ...fields)` |
| `NOT_FOUND` | 404 | `ErrNotFound(c, msg)` |
| `FORBIDDEN` | 403 | `ErrForbidden(c, msg)` |
| `CONFLICT` | 409 | `ErrConflict(c, msg)` |
| `BAD_REQUEST` | 400 | `ErrBadRequest(c, msg)` |
| `INTERNAL_ERROR` | 500 | `ErrInternal(c)` |

### Response shape

```json
{
  "success": false,
  "message": "validation failed",
  "error": {
    "code": "VALIDATION_ERROR",
    "fields": [
      { "field": "name", "message": "name is required" },
      { "field": "price", "message": "must be >= 0" }
    ]
  }
}
```

### Module field mapping

Each module's `writeXxxError()` maps sentinel errors to field names:

```go
// product handler
case errors.Is(err, ErrInvalidProductName):
    return httpx.Err422(c, "name", err.Error())
case errors.Is(err, ErrInvalidBasePrice):
    return httpx.Err422(c, "base_price", err.Error())

// customer handler
case errors.Is(err, ErrInvalidCustomerName):
    return httpx.Err422(c, "full_name", err.Error())
case errors.Is(err, ErrInvalidLevel):
    return httpx.Err422(c, "level", err.Error())
```

Internal errors always log with context and return `INTERNAL_ERROR` (no detail exposed).

---

## Frontend — Error Utilities

### ApiError (extended)

`services/api.ts`

```ts
class ApiError extends Error {
  status: number;
  code?: ApiErrorCode;           // "VALIDATION_ERROR" | "NOT_FOUND" | …
  fields?: ApiFieldError[];      // [{ field, message }]

  fieldMap(): Record<string, string>  // field → first error message
}
```

### lib/form-errors.ts

```ts
// Map ApiError fields into state, returns true if any field errors found
setFromApiError(err: ApiError, setErrors: Fn): boolean

// Human-readable fallback message
friendlyMessage(err: unknown): string
// → uses err.message from backend; falls back by status code
```

### FieldError component

`components/ui/field-error.tsx`

```tsx
<FieldError message={fieldErrors.name} />
// → renders red ✕ text below input; renders nothing if message is falsy
```

---

## Usage Pattern in Components

```tsx
const [fieldErrors, setFieldErrors] = useState<Record<string, string>>({});

async function handleSave() {
  try {
    await saveProduct(form);
    toast.success("บันทึกสำเร็จ");
  } catch (err) {
    if (err instanceof ApiError) {
      const hadFields = setFromApiError(err, setFieldErrors);
      if (!hadFields) toast.error(friendlyMessage(err));  // non-field error
    } else {
      toast.error(friendlyMessage(err));
    }
  }
}

// JSX
<input ... />
<FieldError message={fieldErrors.name} />
```

---

## Implemented In

| Component | Uses |
|-----------|------|
| `stock-manager.tsx` | `friendlyMessage()` for delete errors + toast |
| `customer-network-manager.tsx` | `friendlyMessage()` replaces hardcoded string |
| `supplier-manager.tsx` (EditSupplierModal) | `friendlyMessage()` |

---

## Related
- [[Backend Architecture]] — structured error format definition
- [[Toast Alert System]] — `toast.success/error/warning`
- [[API Client Patterns]] — `ApiError`, `authorizedApiRequest`
