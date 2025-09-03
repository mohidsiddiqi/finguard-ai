# FinGuard AI — PRD (Phase 0)
**Approved — 2025-09-03 — Mohid Siddiqi**

## Purpose
FinGuard AI helps users track spending and budgets and get AI-driven tips.
MVP proves multi-source ingestion (CSV/OFX/QFX/Manual) and a demo-only (for now) bank connection via sandbox webhooks, plus budgets, trends, and AI insights.

## Scope (MVP)

### IN (Must-Have)
- **Ingestion**
  - **CSV** import → parse, store, list, filter
  - **OFX/QFX** import (industry format) → parse & store
  - **Manual Transaction Entry** Screen (single/add-in-bulk later)
  - **Aggregator Sandbox Integration (demo-only)**: connect a test bank in sandbox and **ingest webhook events** into Transactions (no production accounts)
- **Core App**
  - **Categorization & Trends** computed from stored transactions
  - **Budgets**: create and view status vs actuals
  - **AI Insights**: budgeting tips + overrun risk via AI microservice
  - **Auth & Accounts**: register, login, “me”, basic account management
- **Deployment**
  - Docker Compose for local dev and single-node VPS deployment (non-HA; hardening later)

### OUT (Explicitly Excluded for MVP)
- **Live/production** bank connections; paid traffic; multiple providers beyond one sandbox connector
- Payments, advanced analytics, and non-essential AI prompts
- High-availability/scaling; enterprise SSO/roles (defer)

## Objectives
- A user can register, log in, **import via CSV or OFX/QFX** and view stored transactions with categorized spending trends
- A user can **manually add a transaction** that appears alongside imported data
- The app **demonstrates a sandbox bank link + webhook ingestion** that creates/updates transactions in the user’s account (demo-only)
- A user can create a monthly budget and view status vs actuals
- The app returns at least one AI-generated budgeting tip and an overrun risk score per budget category
- The entire stack starts with a single `docker compose up` (API, AI, worker, web, Postgres, Redis, proxy)

## Definition of Done (Phase 0)
- `docs/prd.md` (this file) is finalized and signed
- `docs/security-checklist.md` stub exists (auth tokens, hashing, row-level access, HTTPS cookie, rate limits & audit, minimal PII, encrypted secrets)
- `docs/security-checklist.md` also lists **webhook signature verification**, **file upload safety**, and **idempotency** (see checklist)
- `docs/risk-register.md` exists with risks + mitigations (CSV/OFX parsing, async jobs, webhook security, sandbox drift, data semantics)
- Sample files added under `/samples` (`sample.csv`, `sample.ofx`, `sample.qfx`) and noted in README
- Owner sign-off added: **Approved — date — Mohid**

## Exit Criteria (Phase 0 → Phase 1)
- Anyone can read the PRD and know **what’s in/out**, including **sandbox-only** bank connections
- Tech stack confirmed: React PWA, FastAPI API, FastAPI AI service, worker (Celery/RQ), Postgres, Redis, proxy (Caddy/Nginx), Docker Compose
- Security checklist present (incl. webhook verification & file safety)
- Risk register present with mitigations
- Commit pushed & tagged as Phase-0 closure
