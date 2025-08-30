# FinGuard AI — PRD (Phase 0)
**Approved — 2025-08-29 — Mohid Siddiqi**

## Purpose
FinGuard AI is an MVP for automated personal finance tracking.  
It allows users to upload CSVs, view transactions and spending trends, set budgets, and receive AI-powered tips and overrun risk alerts.

## Scope (MVP)

### IN (Must-Have)
- **CSV to Transactions**: upload, parse, store, list, and filter  
- **Categorization & Trends**: computed from stored transactions  
- **Budgets**: create and view status vs actuals  
- **AI Insights**: budgeting tips + overrun risk via AI microservice endpoints  
- **Auth & Accounts**: register, login, “me” endpoint, and basic account management (MVP scope).  
- **Deployment**: Docker Compose for local dev and single-node VPS deployment; scalable/HA setup deferred to later phases.

### OUT (Explicitly Excluded for MVP)
- Live bank aggregators (Plaid/Yodlee) — feature-flagged for a later phase  
- Advanced analytics and non-essential AI prompts (beyond MVP loop)

## Objectives

- A user can register, log in, upload a CSV, and view stored transactions with categorized spending trends  
- A user can create a monthly budget and view its current status versus actuals  
- The application returns at least one AI-generated budgeting tip and an overrun risk score per budget category  
- The entire stack can be started with a single `docker compose up` command (API, AI, worker, web, Postgres, Redis, proxy)

## Definition of Done (Phase 0)

- `docs/prd.md` includes finalized **Scope (IN/OUT)** and **Objectives**, signed off with date and owner  
- `docs/security-checklist.md` exists with baseline headings (JWT + refresh, Argon2/bcrypt, row-level access, HTTPS/secure cookie, rate limits & audit, minimal PII, encrypted secrets)  
- `docs/risk-register.md` exists with seeded risks and one-line mitigations  
- Scope excludes live bank aggregators and non-essential analytics/AI prompts (explicitly marked OUT)  
- No changes are made to core entities or endpoints beyond what’s documented in Scope  
- Mohid (solo owner) has reviewed and marked this phase as **Approved**

## Exit Criteria (Phase 0 → Phase 1)

- Anyone reading `docs/prd.md` can clearly understand the MVP scope (what’s IN and what’s OUT)  
- Tech stack is confirmed: React PWA, FastAPI core API, FastAPI AI microservice, worker (Celery/RQ), Postgres, Redis, S3-compatible storage, proxy (Caddy/Nginx), Docker Compose  
- `docs/security-checklist.md` is present with baseline headings (to be expanded in Phase 2)  
- `docs/risk-register.md` is present with at least five identified risks and mitigations  
- Mohid has approved and committed all Phase 0 deliverables with a dated sign-off in `docs/prd.md`  
