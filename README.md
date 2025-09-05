# FinGuard AI

*Secure, AI-powered budgeting from CSVs, OFX/QFX, manual entry, and sandbox bank links.*

FinGuard AI is an MVP for automated personal finance tracking.
Import transactions (CSV/OFX/QFX or manual), view trends, set budgets, and get AI tips with overrun risk — all deployable with Docker Compose.

---

## 🚀 MVP Features

* **Multi-source ingestion:** CSV • OFX/QFX • Manual entry
* **Sandbox bank connection (demo-only):** link a test bank; **webhooks** ingest events into Transactions
* **Transactions:** list, filter, categorize
* **Trends:** spend by **category** and **month**
* **Budgets:** create and track **status vs actuals**
* **AI insights:** budgeting tips + **overrun risk scoring**
* **Auth & Accounts:** register/login/me + basic account management
* **One-command bring-up:** Docker Compose (local + single-node VPS)

---

## 🛠️ Tech Stack

**Frontend:** React (Vite, TypeScript, Tailwind)
**Backend:** FastAPI (core API) + FastAPI AI microservice
**Worker:** Celery/RQ with Redis
**Database:** PostgreSQL
**Infra:** Docker Compose, Caddy/Nginx, S3-compatible storage

---

## 📂 Repository Structure

```
finguard-ai/
├── apps/
│   ├── web/          # React PWA
│   ├── api/          # FastAPI core API
│   │   └── integrations/
│   │       ├── plaid/
│   │       └── truelayer/
│   ├── ai/           # FastAPI AI service
│   └── worker/       # Celery/RQ worker
├── infra/
│   ├── docker/       # Dockerfiles, proxy config
│   └── docker-compose.yml
├── docs/
│   ├── prd.md
│   ├── api-contract.md
│   ├── wireframes/wireframes.md
│   ├── schemas/
│   │   ├── csv_v1.md
│   │   ├── ofx_mapping_v1.md
│   │   ├── validation_matrix.md
│   │   └── error_codes.md
│   ├── test-plan.md
│   ├── ux-copy.md
│   └── phase-1-checklist.md
├── samples/
│   ├── sample.csv
│   ├── sample.ofx
│   └── sample.qfx
├── openapi.yaml
└── README.md
```

---

## ⚡ Quick Start (MVP)

```bash
# clone
git clone https://github.com/mohidsiddiqi/finguard-ai.git
cd finguard-ai

# envs (edit as needed)
cp .env.example .env

# bring up the stack
docker compose up
```

* Services: **API**, **AI**, **worker**, **Postgres**, **Redis**, **proxy**.
* Use **/samples** files to try imports immediately.

---

## 📚 Docs & Specs

* **API Contract:** `docs/api-contract.md` • **OpenAPI:** `openapi.yaml`
* **CSV format:** `docs/schemas/csv_v1.md` • **OFX/QFX mapping:** `docs/schemas/ofx_mapping_v1.md`
* **Validation & Error Codes:** `docs/schemas/validation_matrix.md`, `docs/schemas/error_codes.md`
* **Wireframes:** `docs/wireframes/wireframes.md`
* **Test Plan:** `docs/test-plan.md`

---

## 🔐 Security (MVP baseline)

* JWT access + rotating refresh (HttpOnly, Secure, SameSite)
* Argon2/bcrypt password hashing
* Per-user row-level access (no cross-tenant data)
* HTTPS via proxy; minimal PII
* Rate limits + audit logs
* **Webhooks:** signature verification (Plaid/TrueLayer sandbox) + **idempotent** ingestion
* **File uploads:** allowlist extensions/MIME, size limits, stored outside webroot

---

## 📅 Project Status

* **Current Phase:** **Phase 1 — Requirements & UX** (sign-off pending)
* **Next:** **Phase 2 — Architecture & Security** (DB schema, migrations, Dockerfiles, scaffolding)

For details, see [`/docs`](./docs).

---

## 👤 Author

**Mohid Siddiqi**
Senior Full-Stack Developer • Cloud & Security • Infrastructure Automation
[LinkedIn](https://www.linkedin.com/in/mohid-siddiqi) • [mohidsiddiqi@gmail.com](mailto:mohidsiddiqi@gmail.com)
