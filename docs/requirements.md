# FinGuard AI — Functional Requirements (Detailed, Phase 1)

**Conventions**
- Base path: `/api` (JSON everywhere except file uploads).
- Auth: All endpoints require auth **except** `POST /auth/register`, `POST /auth/login`, and provider **webhooks** (which require signature verification).
- Dates: `YYYY-MM-DD`. Periods: `YYYY-MM`. Timestamps: ISO 8601 UTC (e.g., `2025-09-03T12:34:56Z`).
- Amounts: client may send decimal (two places) or integer minor units; backend **normalizes** to a consistent internal format and returns decimals in responses for readability.
- Categories: free text (max 50 chars) for MVP; no global taxonomy enforced.

---

## Shared Types (Schemas)

### User (profile fields)
- `id` (string, UUID/ULID)
- `email` (string, email)
- `country` (string, 2-letter ISO, e.g., `PK`)
- `currency` (string, 3-letter ISO, e.g., `PKR`)

### Account
- `id` (string)
- `name` (string, 1–80)
- `institution` (string, 0–80)
- `type` (enum: `checking` | `savings` | `credit` | `cash`)

### Transaction
- `id` (string)
- `account_id` (string)
- `date` (`YYYY-MM-DD`)
- `amount` (decimal; **negative for expenses, positive for income** in MVP)
- `merchant` (string, 0–120)
- `description` (string, 0–255)
- `category` (string, 0–50, optional)

### Budget
- `id` (string)
- `period` (`YYYY-MM`)
- `category` (string, 1–50)
- `amount` (decimal, > 0)
- `threshold_pct` (integer 1–100)
- Derived in list: `spent` (decimal), `percent` (decimal 0–100)

---

## A) Auth & Accounts

### FR-01 — Register
**Purpose:** Create a new user.

**Request**
```json
POST /auth/register
{ "email": "user@example.com", "password": "StrongPass!234", "country": "PK", "currency": "PKR" }
```

**Response**
- `201 Created`
```json
{ "id": "u_123", "email": "user@example.com" }
```

**AC (Gherkin)**
```gherkin
Given a unique email and strong password
When I POST /auth/register
Then I receive 201 with my user id and email
And I can subsequently log in with those credentials
```

---

### FR-02 — Login
**Purpose:** Get access token (and set refresh cookie).

**Request**
```json
POST /auth/login
{ "email": "user@example.com", "password": "StrongPass!234" }
```

**Response**
- `200 OK`
```json
{ "access_token": "jwt...", "token_type": "bearer" }
```
(Refresh token is returned via HttpOnly Secure cookie.)

**AC**
```gherkin
Given a registered user
When I POST /auth/login with valid credentials
Then I receive 200 with an access_token
And a refresh cookie is set
```

---

### FR-03 — Me (profile)
**Purpose:** Return the **user profile fields**.

**Request**
```
GET /me
```

**Response**
- `200 OK`
```json
{ "id": "u_123", "email": "user@example.com", "country": "PK", "currency": "PKR" }
```

**AC**
```gherkin
Given I am authenticated
When I GET /me
Then I receive 200 with id, email, country, currency
```

---

### FR-04 — Create Account
**Purpose:** Create a financial account.

**Valid body**
```json
POST /accounts
{ "name": "HBL Checking", "institution": "HBL", "type": "checking" }
```
- `name`: required, 1–80
- `institution`: optional, 0–80
- `type`: required, one of `checking|savings|credit|cash`

**Response**
- `201 Created`
```json
{ "id": "acc_1", "name": "HBL Checking", "institution": "HBL", "type": "checking" }
```

**AC**
```gherkin
Given I am authenticated
When I POST /accounts with {name, type} valid
Then I receive 201
And the account appears in GET /accounts
```

---

### FR-05 — List Accounts
```
GET /accounts
```

**Response**
- `200 OK` → only **my** accounts
```json
[
  { "id": "acc_1", "name": "HBL Checking", "institution": "HBL", "type": "checking" }
]
```

**AC**
```gherkin
Given I own N accounts
When I GET /accounts
Then I see exactly those N accounts and no others
```

---

## B) Imports: CSV / OFX-QFX / Manual

### FR-06 — Upload CSV
**What is a “valid CSV”?**
- Has a **header row** with these columns (order can vary):  
  `date, amount, merchant, description, category, account`
- `date`: `YYYY-MM-DD`
- `amount`: number (expenses negative; income positive)  
- `merchant`: optional string
- `description`: optional string
- `category`: optional string
- `account`: **must match** an existing Account `name` you own  
  (If not found: **reject** with a clear error, or create the account if `auto-create` flag is introduced later; MVP = **reject**.)

**Request**
```
POST /transactions/upload-csv  (multipart/form-data)
file=<csv>
```

**Response**
- `202 Accepted`
```json
{ "job_id": "job_abc123", "status": "queued" }
```

**AC**
```gherkin
Given a CSV with the required headers and valid rows
When I POST it to /transactions/upload-csv
Then I receive 202 with job_id
And after processing, rows appear under my user and are visible in GET /transactions
```

---

### FR-07 — Upload OFX/QFX
**What is a “valid OFX/QFX”?**
- File extension `.ofx` or `.qfx`
- Parses successfully via the OFX parser (e.g., ofxtools)  
- Contains a transaction list (`BANKTRANLIST` or credit card message set) with:
  - `FITID` (unique id per transaction)  
  - `DTPOSTED` (date)  
  - `TRNAMT` (amount)  
  - optional: `NAME`/`MEMO` fields for merchant/description
- Includes (or can be inferred) an **account identifier**.  
  **Mapping rule (MVP):** If the account doesn’t exist locally, **create it** (provider = `ofx`) and attach transactions.

**Request**
```
POST /transactions/upload-ofx  (multipart/form-data)
file=<ofx|qfx>
```

**Response**
- `202 Accepted`
```json
{ "job_id": "job_xyz789", "status": "queued" }
```

**AC**
```gherkin
Given a valid OFX or QFX file
When I POST it to /transactions/upload-ofx
Then I receive 202 with job_id
And after processing, transactions appear under an Account (created if needed) I own
```

---

### FR-08 — Manual Transaction Entry
**Fields**
```json
POST /transactions/manual
{
  "account_id": "acc_1",
  "date": "2025-09-03",
  "amount": -4500,
  "merchant": "Grocery Mart",
  "description": "weekly groceries",
  "category": "Groceries"
}
```
- `account_id`: required; must be mine
- `date`: required; `YYYY-MM-DD`
- `amount`: required; negative expenses / positive income
- `merchant`, `description`, `category`: optional

**Response**
- `201 Created` → returns the created Transaction

**AC**
```gherkin
Given I am authenticated and own an account
When I POST /transactions/manual with valid fields
Then I receive 201 and the transaction appears in GET /transactions
```

---

### FR-09 — Import History
**Purpose:** View recent imports and outcomes.

```
GET /imports
```

**Response**
```json
[
  { "id":"imp_1","source":"csv","filename":"aug.csv","status":"succeeded","rows_processed":250,"errors":0 },
  { "id":"imp_2","source":"ofx","filename":"card.qfx","status":"failed","rows_processed":0,"errors":1 }
]
```

**AC**
```gherkin
Given I have run imports
When I GET /imports
Then I see each job with source, filename, status, rows_processed, errors
```

---

## C) Sandbox Bank Link & Webhooks (Demo-only)

### FR-10 — Create Sandbox Link
```
POST /links/sandbox/{provider}/connect
```
- `provider` enum: `plaid` | `truelayer` (MVP supports one; keep the enum for future)
- Creates an `AccountLink` record (sandbox-only) for my user.

**Response**
- `201 Created`
```json
{ "id": "link_1", "provider": "plaid", "status": "connected" }
```

**AC**
```gherkin
Given I am authenticated
When I POST /links/sandbox/plaid/connect
Then an AccountLink record is created for me
And I receive 201 with link info
```

---

### FR-11 — Webhook Receiver (signature-verified)
```
POST /webhooks/{provider}
```
- Verifies provider signature header
  - `plaid`: JWT verification header (e.g., `Plaid-Verification`)  
  - `truelayer`: `Tl-Signature` HMAC
- On success: enqueue ingestion job; record `WebhookEvent` (payload hash for idempotency).
- On failure: return `400/401`.

**Response**
- `200 OK` on acceptance.

**AC**
```gherkin
Given a webhook POST with a valid signature
When it hits /webhooks/{provider}
Then the API returns 200 and enqueues an ingestion job
And the event is recorded with a payload hash for idempotency
```

---

### FR-12 — Idempotent Ingestion
**Purpose:** Prevent duplicates from retries/replays.

**AC**
```gherkin
Given a previously processed webhook payload
When the same payload is delivered again
Then no duplicate transactions are created
And the WebhookEvent is marked as duplicate/no-op
```

---

### FR-13 — Auto-Create Accounts from Connector
**Purpose:** Map unknown provider accounts.

**AC**
```gherkin
Given a webhook payload references a new provider account
When ingestion runs
Then an Account is created for my user (provider-labeled)
And incoming transactions attach to that Account
```

---

## D) Transactions & Trends

### FR-14 — List Transactions (filters, paging)
```
GET /transactions?from=YYYY-MM-DD&to=YYYY-MM-DD&account_id=&category=&q=&page=1&page_size=50
```
- Returns my transactions only.
- Supports optional filters and a free-text `q` matching merchant/description.
- Default sort: `date desc, id asc`.

**Response**
```json
{
  "items": [
    { "id":"txn_1","account_id":"acc_1","date":"2025-08-01","amount":-4500,"merchant":"Grocery Mart","description":"weekly groceries","category":"Groceries" }
  ],
  "total": 1,
  "page": 1,
  "page_size": 50
}
```

**AC**
```gherkin
Given I have transactions
When I filter by date range and category
Then I see only my matching rows with total and paging info
```

---

### FR-15 — Update Transaction Category
```
PATCH /transactions/{id}
{ "category": "Transport" }
```
- Only owner can update.
- Other fields immutable in MVP.

**AC**
```gherkin
Given I own a transaction
When I PATCH its category
Then I receive 200 and the new category is persisted
```

---

### FR-16 — Trends (derived)
```
GET /trends?from=YYYY-MM-DD&to=YYYY-MM-DD&group_by=category|month
```

**Responses (examples)**
```json
{ "group_by": "category", "items": [ { "key": "Groceries", "total": 12500 } ] }
```
```json
{ "group_by": "month", "items": [ { "key": "2025-08", "total": 93500 } ] }
```

**AC**
```gherkin
Given transactions exist in range
When I group by category
Then totals are returned per category
And when I group by month
Then totals are returned per YYYY-MM
```

---

## E) Budgets

### FR-17 — Create Budget
```
POST /budgets
{ "period": "2025-09", "category": "Groceries", "amount": 20000, "threshold_pct": 90 }
```

**AC**
```gherkin
Given I am authenticated
When I POST /budgets with valid fields
Then I receive 201
And the budget appears in GET /budgets
```

---

### FR-18 — List Budgets with Status vs Actuals
```
GET /budgets
```
**Response**
```json
[
  { "id":"bgt_1","period":"2025-09","category":"Groceries","amount":20000,"spent":12500,"percent":62.5,"threshold_pct":90 }
]
```

**AC**
```gherkin
Given I have budgets and transactions
When I GET /budgets
Then I see planned amount, spent so far, percent used, and threshold
```

---

### FR-19 — Update Budget
```
PATCH /budgets/{id}
{ "amount": 22000, "threshold_pct": 85 }
```

**AC**
```gherkin
Given I own a budget
When I PATCH allowed fields
Then I receive 200 and the changes are reflected in GET /budgets
```

---

## F) AI Insights (Thin Service)

### FR-20 — AI Budget Tips
```
POST /advise/budget
{ "period": "2025-09", "category": "Groceries", "context": { "planned": 20000, "spent": 12500 } }
```

**Response**
```json
{ "tips": [ "Plan bulk purchases early in the month to avoid splurges." ] }
```

**AC**
```gherkin
Given I post a valid period and context
When I call /advise/budget
Then I receive at least one actionable tip string
```

---

### FR-21 — Overrun Risk Score
```
POST /score/overrun
{ "period": "2025-09", "by": "category" }
```

**Response**
```json
{ "scores": [ { "category": "Groceries", "risk": 0.34 } ] }
```

**AC**
```gherkin
Given I post a valid period
When I call /score/overrun
Then I receive risk scores per category
```

---

## Errors & Limits (MVP)
- `400` invalid payload or invalid file (bad headers, parse failure)
- `401` missing/expired token
- `403` accessing another user’s resource
- `413` file too large (size limits enforced)
- `429` rate limited
- `500` unexpected

**File Safety (MVP gates)**
- CSV/OFX/QFX: allowlist extensions & MIME, max size (e.g., CSV ≤ 5 MB, OFX/QFX ≤ 2 MB), stored outside webroot, sanitized filenames.
- OFX/QFX must parse; rows missing required fields are skipped with reasons logged.

---
---
---

## Non-Functional Requirements (Phase 1)

### 1) Performance & Capacity
- **CSV ingest**: parse and persist **1,000 rows ≤ 5s** on dev hardware (async worker).
- **OFX/QFX ingest**: parse typical statement (**≤ 1,000 txns**) ≤ **5s** (async worker).
- **Transactions list**: server latency **p95 ≤ 300ms** for 50 rows, basic filters.
- **Trends**: p95 **≤ 500ms** for a 3-month range, category/month grouping.
- **Budgets list**: p95 **≤ 300ms** for 50 budgets.

**Verify**
- Load test: 3 back-to-back CSV/OFX uploads; measure worker completion times.
- API timing: capture server-side timings (structured logs) and confirm p95.

---

### 2) Reliability & Jobs
- **Idempotency**: file imports and webhooks must not create duplicates (hash/FITID/provider-id).
- **Retries**: worker retries failed jobs **up to 3** with exponential backoff.
- **At-least-once** processing: webhook events recorded **before** job enqueue; job reads from durable store.
- **Concurrency**: worker processes **≥ 2 jobs** in parallel without data races.

**Verify**
- Replay the same webhook payload twice → no new rows.
- Submit the same file twice → no duplicates (or produce a controlled “already imported” outcome).

---

### 3) Security (Baseline for MVP)
- **Auth**: JWT access + rotating refresh (HttpOnly, Secure cookie; SameSite=Lax/Strict).
- **Passwords**: Argon2/bcrypt with sane params; strong password policy.
- **Row-level access**: a user cannot access another user’s data (enforced in DB/service).
- **Transport**: HTTPS only (Caddy/Nginx, Let’s Encrypt).
- **Rate limits**: per-IP on auth, uploads, webhooks; sensible defaults (e.g., login **5/min**, uploads **3/10min**).
- **Audit**: auth events, uploads, webhook accept/ingest events are logged.
- **Webhooks**: signature verification (Plaid JWT / TrueLayer Tl-Signature); **reject** if invalid/expired; clock-skew tolerant (±5 min).
- **File safety**: allowlist extensions + MIME; size limits (**CSV ≤ 5MB**, **OFX/QFX ≤ 2MB**); store outside webroot; sanitize filenames.
- **Secrets**: only via env/secret store; never in VCS or plaintext images.
- **PII minimization**: store only what’s needed; no raw access tokens in logs.

**Verify**
- Security checklist walkthrough; unit/integration tests for auth, RLS, signatures; attempt invalid signature → `401/400`.

---

### 4) Observability & Ops
- **Structured logs** (JSON) with request id, user id (if present), route, latency, status.
- **Job logs**: imports/webhooks log `job_id`, counts, skip reasons; error trace on failure.
- **Error tracking**: enabled in prod (server + web).
- **Health endpoints**: `/healthz` (API/AI/worker basic liveness).
- **Metrics (lightweight)**: counts of imports, webhook acceptances, error rates (via logs or minimal metrics endpoint).

**Verify**
- Trigger a failing upload → see error trace + skip count.
- Trigger webhook → see accept log + enqueue log + completion log.

---

### 5) Usability & Accessibility
- Clear upload flow with progress states (queued, processing, done, failed).
- Empty states for Transactions, Budgets, Trends.
- Basic keyboard navigation; forms have labels; color contrast AA for primary UI.
- Helpful validation messages (field-level, not just generic).

**Verify**
- Manual UI checks; axe/lighthouse baseline scan (no critical A11y violations).

---

### 6) Compatibility (Clients/Browsers)
- **Web**: Latest Chrome/Edge/Firefox/Safari; responsive to ≥ 360px width.
- **PWA baseline**: manifest + icons; offline not required in MVP.

**Verify**
- Smoke tests on the listed browsers; mobile viewport sanity pass.

---

### 7) Data Management
- **Timezones**: store timestamps as UTC; display in user’s local TZ (browser).
- **Amounts**: internally consistent (document choice: decimal(18,2) **or** integer minor units); API returns decimals.
- **Retention**: imports + webhook payload hashes retained **≥ 30 days** (for audit); transactions/budgets retained until user deletes account.
- **Backups**: nightly DB backup in VPS; verify restore locally monthly.

**Verify**
- One successful backup + restore drill on staging or local from prod backup.

---

### 8) Rate Limiting & Quotas (MVP)
- Auth endpoints: **5/min/IP**.
- Upload endpoints: **3/10min/user**.
- Webhooks: accept bursts but queue; global processing concurrency limited to protect DB.

**Verify**
- Hit limits in tests → `429` with `Retry-After`.

---

### 9) API Behavior
- **Pagination**: `page` + `page_size` (default 50; max 200).
- **Errors**: JSON error body `{ "error": "code", "message": "details" }`.
- **Sorting**: Transactions default `date desc, id asc`.
- **Filtering**: strict schema; unknown params are ignored, not fatal.

**Verify**
- Contract tests for pagination, error shapes, default sorting.

---

### 10) Deployment & Environments
- **One-command up**: `docker compose up` brings API, AI, worker, DB, Redis, proxy.
- **Configs**: `.env.example` with all required vars; compose references only env.
- **Images**: pinned base images; non-root user where feasible.
- **VPS**: single-node, HTTPS via proxy; logs written to files/stdout and rotated.

**Verify**
- Fresh clone → `.env` from example → `docker compose up` succeeds on a clean machine.

---

### 11) Privacy & Legal (MVP posture)
- Sandbox connectors only (no live banking); clearly marked in UI.
- No selling/sharing data; data used only to render features.
- Basic disclaimer: “Insights are informational; not financial advice.”

**Verify**
- Text appears in README and app settings/about page.

---
