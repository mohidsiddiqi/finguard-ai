# FinGuard AI — API Contract (MVP)

**Base URL**: `/api`
**Auth**: Bearer access token (JWT) in `Authorization: Bearer <token>`; refresh via secure HttpOnly cookie.
**Format**: JSON for requests/responses (except file uploads).
**Dates**: `YYYY-MM-DD`; **Timestamps**: ISO 8601 UTC (e.g., `2025-09-03T10:22:00Z`).
**Uploads**: `multipart/form-data` for CSV/OFX/QFX files (use proper `Content-Type`). citeturn0search14turn0search9

## Security: Webhooks (Sandbox)

* **Plaid**: verify `Plaid-Verification` header (JWT). Fetch JWK via `/webhook_verification_key/get`, verify the JWT, then trust the body. Reject if invalid/expired. citeturn0search0
* **TrueLayer**: verify `Tl-Signature` (JWS w/ detached payload). Validate signature/timestamp headers before enqueueing. Reject if invalid/expired. citeturn0search1turn0search16

---

## Error Schema

```json
{ "error": "code_string", "message": "Human-readable detail" }
```

Common: `400, 401, 403, 413, 429, 500`.

## Pagination

* Query: `page` (default 1), `page_size` (default 50, max 200).
* Response adds `total`, `page`, `page_size`.

---

## Auth

### POST /auth/register

Request

```json
{ "email": "user@example.com", "password": "StrongPass!234", "country": "PK", "currency": "PKR" }
```

201

```json
{ "id": "u_123", "email": "user@example.com" }
```

### POST /auth/login

Request

```json
{ "email": "user@example.com", "password": "StrongPass!234" }
```

200

```json
{ "access_token": "jwt...", "token_type": "bearer" }
```

(Sets refresh cookie.)

### POST /auth/refresh

200 → new `access_token`.

### GET /me

200

```json
{ "id": "u_123", "email": "user@example.com", "country": "PK", "currency": "PKR" }
```

---

## Accounts

### POST /accounts

Body

```json
{ "name": "HBL Checking", "institution": "HBL", "type": "checking" }
```

201 → Account

### GET /accounts

200 → `[Account]`

---

## Transactions — Uploads & Manual

### POST /transactions/upload-csv  (multipart/form-data)

Form: `file=<csv>`
**CSV header (order free):** `date,amount,merchant,description,category,account`

* `date`: `YYYY-MM-DD`
* `amount`: expenses negative; income positive
* `account`: must match an existing Account `name` (MVP reject if not found)

202

```json
{ "job_id": "job_abc123", "status": "queued" }
```

### POST /transactions/upload-ofx  (multipart/form-data)

Form: `file=<ofx|qfx>`
**Valid OFX/QFX**: parses OK; transactions include **FITID** (unique id), **DTPOSTED** (date), **TRNAMT** (amount), with NAME/MEMO optional; create Account if missing (provider = `ofx`). citeturn0search2turn0search7turn0search12
202 → `{ job_id, status }`

### POST /transactions/manual

```json
{
  "account_id": "acc_1",
  "date": "2025-09-03",
  "amount": -4500,
  "merchant": "Grocery Mart",
  "description": "weekly groceries",
  "category": "Groceries"
}
```

201 → Transaction

### GET /transactions

Query: `from`, `to`, `account_id`, `category`, `q`, `page`, `page_size`
200

```json
{ "items": [ { "id":"txn_1", "account_id":"acc_1", "date":"2025-08-01", "amount":-4500, "merchant":"Grocery Mart", "description":"weekly groceries", "category":"Groceries" } ],
  "total": 1, "page": 1, "page_size": 50 }
```

### PATCH /transactions/{id}

Body: `{ "category": "Transport" }`
200 → updated Transaction

---

## Imports

### GET /imports

200

```json
[
  { "id":"imp_1","source":"csv","filename":"aug.csv","status":"succeeded","rows_processed":250,"errors":0 },
  { "id":"imp_2","source":"ofx","filename":"card.qfx","status":"failed","rows_processed":0,"errors":1 }
]
```

---

## Trends (Derived)

### GET /trends

Query: `from`, `to`, `group_by=category|month`
200 (examples)

```json
{ "group_by": "category", "items": [ { "key": "Groceries", "total": 12500 } ] }
```

```json
{ "group_by": "month", "items": [ { "key": "2025-08", "total": 93500 } ] }
```

---

## Budgets

### POST /budgets

```json
{ "period": "2025-09", "category": "Groceries", "amount": 20000, "threshold_pct": 90 }
```

201 → Budget

### GET /budgets

200

```json
[
  { "id":"bgt_1","period":"2025-09","category":"Groceries","amount":20000,"spent":12500,"percent":62.5,"threshold_pct":90 }
]
```

### PATCH /budgets/{id}

Body: `{ "amount": 22000, "threshold_pct": 85 }`
200 → updated Budget

---

## Insights (AI)

### POST /advise/budget

```json
{ "period": "2025-09", "category": "Groceries", "context": { "planned": 20000, "spent": 12500 } }
```

200

```json
{ "tips": [ "Plan bulk purchases early in the month to avoid splurges." ] }
```

### POST /score/overrun

```json
{ "period": "2025-09", "by": "category" }
```

200

```json
{ "scores": [ { "category": "Groceries", "risk": 0.34 } ] }
```

### GET /insights

200 → `[ { "id":"ins_1", "type":"tip", "message":"...", "source":"AI", "created_at":"..." } ]`

---

## Sandbox Links & Webhooks (Demo-only)

### POST /links/sandbox/{provider}/connect

Path `provider`: `plaid|truelayer` (support at least one in MVP)
201

```json
{ "id": "link_1", "provider": "plaid", "status": "connected" }
```

### POST /webhooks/{provider}

Headers:

* **Plaid**: `Plaid-Verification: <jwt>` (verify with provider JWK) citeturn0search0
* **TrueLayer**: `Tl-Signature: <jws>` + timestamp header (validate JWS) citeturn0search1turn0search16

Body: provider JSON payload (sandbox)
200 on acceptance → enqueue job & record `WebhookEvent` (payload hash) for idempotency.

---
