# Validation Matrix (Phase 1)

**Error shape (global)**

```json
{ "error": "code_string", "message": "Human-readable detail" }
```

**Primitives & Patterns**

* **email**: RFC5322‑lite, e.g. `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`
* **date**: `YYYY-MM-DD` regex `/^\d{4}-\d{2}-\d{2}$/` (server validates real calendar date)
* **period**: `YYYY-MM` regex `/^\d{4}-\d{2}$/`
* **currency**: ISO‑4217 uppercase `/^[A-Z]{3}$/`
* **country**: ISO‑3166‑1 alpha‑2 uppercase `/^[A-Z]{2}$/`
* **uuid/id**: server‑generated (UUID/ULID) strings
* **amount**: decimal, 2dp typical; negatives = expense; positives = income
* **category**: string 0–50
* **account name**: string 1–80

**File Constraints**

* CSV: extensions `.csv`, `Content-Type: text/csv`, size ≤ **5 MB**, UTF‑8
* OFX/QFX: extensions `.ofx` / `.qfx`, `Content-Type: application/x-ofx` (or vendor), size ≤ **2 MB**

---

## Endpoint Matrices

### POST /auth/register

| Field    | Type   | Required | Constraints            | Notes                |
| -------- | ------ | -------- | ---------------------- | -------------------- |
| email    | string | yes      | pattern=email, max 254 | unique               |
| password | string | yes      | min 8, max 72          | bcrypt/argon limits  |
| country  | string | no       | pattern=country        | defaults server‑side |
| currency | string | no       | pattern=currency       | defaults server‑side |

**Errors**: `auth_email_in_use (409)`, `validation_failed (400)`

---

### POST /auth/login

| Field    | Type   | Required | Constraints   |
| -------- | ------ | -------- | ------------- |
| email    | string | yes      | pattern=email |
| password | string | yes      | 1–72          |

**Errors**: `auth_invalid_credentials (401)`, `rate_limited (429)`

---

### GET /me

Headers: `Authorization: Bearer <token>`

**Errors**: `auth_token_missing (401)`, `auth_token_invalid (401)`

---

### POST /accounts

| Field       | Type   | Required | Constraints |         |        |        |
| ----------- | ------ | -------- | ----------- | ------- | ------ | ------ |
| name        | string | yes      | 1–80        |         |        |        |
| institution | string | no       | 0–80        |         |        |        |
| type        | enum   | yes      | \`checking  | savings | credit | cash\` |

**Errors**: `validation_failed (400)`

---

### POST /transactions/upload-csv (multipart)

| Part | Type | Required | Constraints     |
| ---- | ---- | -------- | --------------- |
| file | bin  | yes      | CSV rules above |

**Row validation**

* Headers must include: `date,amount,merchant,description,category,account` (case‑insensitive; order free)
* Per‑row rules: see CSV schema

**Errors**: `csv_header_missing (400)`, `csv_column_missing:<name> (400)`, `file_too_large (413)`, `file_type_not_allowed (400)`, `row_invalid_date (207 summary)`, `row_invalid_amount (207 summary)`, `row_unknown_account (207 summary)`

---

### POST /transactions/upload-ofx (multipart)

| Part | Type | Required | Constraints         |
| ---- | ---- | -------- | ------------------- |
| file | bin  | yes      | OFX/QFX rules above |

**Parsing**: requires `FITID`, `DTPOSTED`, `TRNAMT` per transaction. Auto‑create account if unknown (provider=`ofx`).

**Errors**: `ofx_parse_error (400)`, `ofx_unsupported (400)`, `file_too_large (413)`, `file_type_not_allowed (400)`

---

### POST /transactions/manual

| Field       | Type   | Required | Constraints         |
| ----------- | ------ | -------- | ------------------- |
| account\_id | string | yes      | must belong to user |
| date        | string | yes      | pattern=date        |
| amount      | number | yes      | non‑zero decimal    |
| merchant    | string | no       | 0–120               |
| description | string | no       | 0–255               |
| category    | string | no       | 0–50                |

**Errors**: `account_not_found (404)`, `validation_failed (400)`

---

### GET /transactions

| Query       | Type   | Required | Constraints                           |
| ----------- | ------ | -------- | ------------------------------------- |
| from        | string | no       | pattern=date                          |
| to          | string | no       | pattern=date                          |
| account\_id | string | no       | must belong to user                   |
| category    | string | no       | 0–50                                  |
| q           | string | no       | 0–120 (merchant/description contains) |
| page        | int    | no       | ≥1 (default 1)                        |
| page\_size  | int    | no       | 1–200 (default 50)                    |

**Errors**: `validation_failed (400)`

---

### PATCH /transactions/{id}

| Field    | Type   | Required | Constraints |
| -------- | ------ | -------- | ----------- |
| category | string | yes      | 0–50        |

**Errors**: `transaction_not_found (404)`, `validation_failed (400)`

---

### GET /imports

No parameters.

---

### GET /trends

| Query     | Type   | Required | Constraints  |         |
| --------- | ------ | -------- | ------------ | ------- |
| from      | string | no       | pattern=date |         |
| to        | string | no       | pattern=date |         |
| group\_by | enum   | yes      | \`category   | month\` |

**Errors**: `validation_failed (400)`

---

### POST /budgets

| Field          | Type   | Required | Constraints        |
| -------------- | ------ | -------- | ------------------ |
| period         | string | yes      | pattern=period     |
| category       | string | yes      | 1–50               |
| amount         | number | yes      | > 0                |
| threshold\_pct | int    | no       | 1–100 (default 90) |

**Errors**: `validation_failed (400)`

---

### PATCH /budgets/{id}

| Field          | Type   | Required | Constraints |
| -------------- | ------ | -------- | ----------- |
| amount         | number | no       | > 0         |
| threshold\_pct | int    | no       | 1–100       |

**Errors**: `budget_not_found (404)`, `validation_failed (400)`

---

### POST /advise/budget

| Field    | Type   | Required | Constraints     |
| -------- | ------ | -------- | --------------- |
| period   | string | yes      | pattern=period  |
| category | string | no       | 0–50            |
| context  | object | no       | any JSON object |

**Errors**: `validation_failed (400)`

---

### POST /score/overrun

| Field  | Type   | Required | Constraints      |
| ------ | ------ | -------- | ---------------- |
| period | string | yes      | pattern=period   |
| by     | enum   | no       | `category` (MVP) |

**Errors**: `validation_failed (400)`

---

### POST /links/sandbox/{provider}/connect

Path param `provider`: `plaid|truelayer` (MVP may enable one)

**Errors**: `provider_not_supported (400)`

---

### POST /webhooks/{provider}

Headers must include the provider’s signature header (`Plaid-Verification` or `Tl-Signature`). Body must be valid JSON.

**Errors**: `webhook_signature_invalid (401)`, `webhook_signature_missing (401)`, `webhook_payload_invalid (400)`