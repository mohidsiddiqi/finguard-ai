# FinGuard AI — Test Plan (MVP Acceptance)

Owner: **Mohid Siddiqi**
Scope: Validate Phase‑1 FRs/NFRs end‑to‑end for MVP.

---

## 1) Objectives & Scope

* Prove **ingestion** (CSV, OFX/QFX, Manual), **transactions**, **trends**, **budgets**, **AI tips/risk**, and **sandbox webhooks** work for a single tenant.
* Enforce **security baseline** (auth, RLS, HTTPS, signatures, rate limits) and **quality bars** (performance p95 targets, reliability).
* Cover **UI flows** per wireframes; APIs per `api-contract.md` / `openapi.yaml`.

Out of scope (MVP): live production connectors, HA/scale tests, enterprise auth.

---

## 2) Test Environments

* **Local Dev (Docker Compose)**: API, AI, worker, Postgres, Redis, proxy (HTTPS via self‑signed or local CA).
* **Staging/VPS (single node)**: same services; public HTTPS with Let’s Encrypt; error tracking enabled.

### Config

* `.env.example` completed; test creds separate from prod.
* Feature flags: only **one** sandbox provider enabled.

---

## 3) Test Data

* **User**: `test+user@finguard.local` / password `Test1234!` (or generated).
* **Accounts**: `HBL Checking` (checking).
* **Samples**: `/samples/sample.csv`, `/samples/sample.ofx`, `/samples/sample.qfx` (as committed).
* **Manual Txn**: Groceries `-4500` on `2025-09-03`.

---

## 4) Test Types & Suites

### 4.1 Acceptance (User Journeys)

| ID | Journey                    | Steps                                                                                                                | Expectation                                                                                  |
| -- | -------------------------- | -------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| A1 | First‑run happy path (CSV) | Register → Login → Create Account → Upload CSV → List Transactions → Trends → Create Budget → View Budgets → AI Tips | No errors; data visible; trends correct; budget status computed; at least one tip displayed. |
| A2 | First‑run (OFX)            | Register/Login → Upload OFX → Auto‑created account visible → Transactions appear → Trends OK                         | Account auto‑created; no dupes; mapping correct.                                             |
| A3 | Manual entry               | Login → Add Manual Transaction → List/Filter shows row                                                               | Row persists; filters work.                                                                  |
| A4 | Sandbox link demo          | Login → Connect sandbox → Trigger webhook (provider tool) → Activity shows event → Transactions updated              | Signature verified; idempotent on replay.                                                    |

### 4.2 API Contract Tests (map to FR IDs)

* **Auth**: FR‑01/02/03 — register/login/me; token/refresh behavior.
* **Accounts**: FR‑04/05 — create/list, validation.
* **Imports**: FR‑06/07/09 — upload (202 Accepted), status trail, results JSON.
* **Manual**: FR‑08 — field rules, ownership checks.
* **Transactions**: FR‑14/15 — filters, pagination, category patch.
* **Trends**: FR‑16 — group by category/month, totals match.
* **Budgets**: FR‑17/18/19 — create/list/update, status vs actuals.
* **AI**: FR‑20/21 — valid tips/risk response shapes.
* **Links/Webhooks**: FR‑10/11/12/13 — link created, signature verified, idempotency, auto‑create accounts.

### 4.3 Security & Compliance

* **Auth/RLS**: cannot read/modify another user’s resources.
* **Transport**: HTTPS enforced (staging); secure cookie flags present.
* **Signatures**: invalid/missing signature rejected for webhooks.
* **Rate Limits**: login (5/min), uploads (3/10min) return 429 as configured.
* **Secrets**: no secrets in logs/images; env‑based only.

### 4.4 Performance (NFR p95 targets)

* CSV 1k rows ≤ **5s** end‑to‑end (async worker) on dev box.
* OFX 1k txns ≤ **5s** (async worker) on dev box.
* GET /transactions (50 rows) p95 ≤ **300ms**.
* GET /trends (3‑month) p95 ≤ **500ms**.
* GET /budgets p95 ≤ **300ms**.

### 4.5 Accessibility & UX

* Keyboard focus order logical; labels on forms; bottom‑nav discoverable on mobile.
* Contrast meets WCAG 2.1 AA on primary text/buttons.
* Clear empty/loading/error states; helpful validation messages.

---

## 5) Test Cases (Selected)

### 5.1 Auth & Profile

```gherkin
Scenario: Register and login (happy path)
  Given I POST /auth/register with a unique email and strong password
  When I POST /auth/login with those credentials
  Then I receive 200 with an access_token and a refresh cookie
  And GET /me returns my id, email, country, currency
```

```gherkin
Scenario: Login throttling
  Given 6 rapid failed login attempts
  When I POST /auth/login again
  Then I receive 429 rate_limited with Retry-After
```

### 5.2 Accounts

```gherkin
Scenario: Create account and list
  Given I am authenticated
  When I POST /accounts with name and type
  Then I receive 201 and GET /accounts shows the new account
```

### 5.3 CSV Upload

```gherkin
Scenario: CSV happy path
  Given a CSV with headers date,amount,merchant,description,category,account
  And account names match existing accounts
  When I POST multipart to /transactions/upload-csv
  Then I receive 202 with job_id
  And within a short time the job completes and rows appear in GET /transactions
```

```gherkin
Scenario: CSV rejected for missing header
  Given a CSV without a header row
  When I upload it
  Then I receive 400 csv_header_missing
```

```gherkin
Scenario: CSV row skipped for unknown account
  Given a CSV with account name that does not exist
  When I upload it
  Then the upload completes with skipped rows and reason row_unknown_account
```

### 5.4 OFX/QFX Upload

```gherkin
Scenario: OFX happy path with auto-create account
  Given a valid OFX file with FITID, DTPOSTED, TRNAMT
  When I upload it to /transactions/upload-ofx
  Then I receive 202 and an account is auto-created if missing
  And transactions appear without duplicates on re-upload
```

```gherkin
Scenario: OFX parse error
  Given a malformed OFX file
  When I upload it
  Then I receive 400 ofx_parse_error
```

### 5.5 Manual Transaction

```gherkin
Scenario: Create manual transaction
  Given I own an account
  When I POST /transactions/manual with valid fields
  Then I receive 201 and the row appears in GET /transactions
```

### 5.6 Transactions & Trends

```gherkin
Scenario: Filter transactions and paginate
  Given multiple transactions exist across dates and categories
  When I GET /transactions with date range and category
  Then I only see matching rows with total and paging info
```

```gherkin
Scenario: Trends by category and month
  Given transactions exist within a date range
  When I GET /trends grouped by category
  Then I see totals per category
  When I GET /trends grouped by month
  Then I see totals per YYYY-MM
```

### 5.7 Budgets

```gherkin
Scenario: Create, list, update budget
  When I POST /budgets with period, category, amount
  Then I receive 201 and GET /budgets shows spent and percent
  When I PATCH /budgets/{id} to adjust amount
  Then GET /budgets reflects the change
```

### 5.8 AI Service

```gherkin
Scenario: Budget tips and risk scores
  When I POST /advise/budget with period/category/context
  Then I receive a list of tip strings
  When I POST /score/overrun with period
  Then I receive risk scores per category (0..1)
```

### 5.9 Sandbox Link & Webhooks

```gherkin
Scenario: Connect sandbox provider
  When I POST /links/sandbox/plaid/connect
  Then I receive 201 and an AccountLink exists for my user
```

```gherkin
Scenario: Webhook signature verified and idempotent
  Given a valid provider webhook with correct signature
  When I POST /webhooks/plaid
  Then I receive 200 and an ingestion job is enqueued
  And repeating the same payload does not create duplicate transactions
```

---

## 6) Performance Test Outline

* Use small script (e.g., k6/locust or Python + httpx) to:

  * Upload **1k-row CSV** thrice; record total processing time from upload to "done" state.
  * Hit `GET /transactions` (50 rows) 200 times; record p95 latency.
  * Hit `GET /trends` (3-month) 100 times; record p95 latency.
* Store results under `docs/perf-results/<date>.md`.

---

## 7) Security Test Checklist

* ❑ Cannot access other users’ `/transactions`, `/budgets`, `/accounts` (force id in URL).
* ❑ HTTPS only on staging; refresh token cookie flags set (HttpOnly, Secure, SameSite).
* ❑ Webhook invalid/missing signature → 401; clock skew outside window → 401/400.
* ❑ Rate limit enforced on `/auth/login`, uploads, and `/webhooks`.
* ❑ CSV/OFX size/type limits enforced; filenames sanitized; stored outside webroot.

---

## 8) Accessibility Checklist (Quick)

* ❑ Keyboard navigation works for forms, tables, modals/drawers.
* ❑ Form labels and aria‑describedby on inputs; error text linked to fields.
* ❑ Color contrast meets WCAG 2.1 AA for text/buttons.

---

## 9) Entry / Exit Criteria

**Entry**: FRs, NFRs, API Contract ready; samples available; env up.
**Exit**: All acceptance suites pass; p95 targets met; security checks green; known issues logged.

---

## 10) Reporting

* Test run log: `docs/test-runs/<YYYY-MM-DD>.md` (summary, failures, perf stats).
* Defects → GitHub issues with repro steps, logs, payloads.
