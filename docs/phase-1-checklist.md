# Phase‑1 Checklist & Sign‑off — FinGuard AI (Requirements & UX)

**Purpose**: Single page to confirm all Phase‑1 artifacts are present, consistent, and ready for Phase‑2.

> ✅ Tick every box before moving on. Keep this doc in PRs as the gate.

---

## A) Deliverables Inventory (must exist in repo)

* [ ] **FRs** — `docs/requirements.md` (Functional Requirements complete with Gherkin AC)
* [ ] **NFRs** — `docs/requirements.md` (Non‑Functional Requirements, measurable targets)
* [ ] **API Contract** — `docs/api-contract.md`
* [ ] **OpenAPI** — `openapi.yaml` (3.1, paths match API Contract)
* [ ] **Wireframes** — `docs/wireframes/wireframes.md`
* [ ] **CSV Schema** — `docs/schemas/csv_v1.md`
* [ ] **Sample CSV** — `samples/sample.csv`
* [ ] **OFX/QFX Mapping** — `docs/schemas/ofx_mapping_v1.md`
* [ ] **Sample OFX/QFX** — `samples/sample.ofx`, `samples/sample.qfx`
* [ ] **Validation Matrix** — `docs/schemas/validation_matrix.md`
* [ ] **Error Codes** — `docs/schemas/error_codes.md`
* [ ] **User Stories (brief)** — `docs/stories.md`
* [ ] **Test Plan** — `docs/test-plan.md`
* [ ] **UX Copy** — `docs/ux-copy.md`
* [ ] **Implementation Notes (ingestion)** — `docs/implementation-notes/ingestion.md`

---

## B) Cross‑Doc Consistency (quick pass)

* [ ] **Entities align** across FRs, API Contract, and OpenAPI (User, Account, Transaction, Budget, ImportFile, AccountLink, WebhookEvent)
* [ ] **Endpoints align**: paths/verbs/params identical in `api-contract.md` and `openapi.yaml`
* [ ] **CSV/OFX rules** in schemas match **Validation Matrix** (headers, sizes, types)
* [ ] **Error codes** referenced in FRs/Test Plan exist in `error_codes.md`
* [ ] **Limits/targets** in NFRs match Validation Matrix (sizes, p95s, rate limits)
* [ ] **Security baseline** items appear in NFRs + Test Plan (JWT+refresh cookie, RLS, TLS, rate limits, webhook signatures, file safety, idempotency)

---

## C) Acceptance Smoke (local Docker Compose)

* [ ] `docker compose up` starts API, AI, worker, DB, Redis, proxy
* [ ] POST `/auth/register` + `/auth/login` + GET `/me` works
* [ ] Create **Account** (e.g., *HBL Checking*) works
* [ ] Upload **sample.csv** → returns `202` and completes → rows visible in GET `/transactions`
* [ ] Upload **sample.ofx**/**.qfx** → `202` → auto‑created account appears → rows visible → no duplicates on re‑upload
* [ ] Add **Manual Transaction** → appears in list & filters
* [ ] GET **/trends** returns expected totals for sample data
* [ ] Create **Budget** → GET **/budgets** shows `spent` & `% used`
* [ ] **AI endpoints** (`/advise/budget`, `/score/overrun`) return valid response shapes

*(Tip: record this quick run in `docs/test-runs/<YYYY-MM-DD>.md>`.)*

---

## D) Review Checklist (content quality)

* [ ] Copy in **UX** is clear, short, and consistent with wireframes
* [ ] All user‑visible numbers/dates examples use real‑looking, localized values
* [ ] All docs have **intro sentence** and **links** to related files
* [ ] README references `/docs` and `/samples`

---

## E) Approvals & Traceability

* [ ] Self‑review completed by **Mohid**
* [ ] (Optional) Friend/peer skim review completed

**Traceability table (fill)**

| FR‑ID                          | Endpoint(s)                   | Wireframe(s)      | Test Plan Case(s) |
| ------------------------------ | ----------------------------- | ----------------- | ----------------- |
| FR‑06 (CSV)                    | POST /transactions/upload-csv | Imports           | 5.3 CSV tests, A1 |
| FR‑07 (OFX)                    | POST /transactions/upload-ofx | Imports           | 5.4 OFX tests, A2 |
| FR‑08 (Manual)                 | POST /transactions/manual     | Manual Drawer     | 5.5               |
| FR‑14 (List)                   | GET /transactions             | Transactions      | 5.6               |
| FR‑16 (Trends)                 | GET /trends                   | Trends            | 5.6               |
| FR‑17/18/19 (Budgets)          | /budgets                      | Budgets           | 5.7               |
| FR‑10/11/12/13 (Link/Webhooks) | /links/sandbox/*, /webhooks/* | Sandbox, Activity | 5.9               |

*(Add/adjust rows as needed.)*

---

## F) Phase‑1 Exit Criteria (must be true)

* [ ] All deliverables in **A** exist and pass **B** consistency checks
* [ ] **C** Acceptance Smoke passes locally
* [ ] **Test Plan** ready to run at end of Phase‑3
* [ ] This checklist committed to repo

---

## Sign‑off

**Phase 1 Approved — \<YYYY‑MM‑DD> — Mohid Siddiqi**

Signature: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

---

## What happens in Phase‑2 (preview)

* Translate stories → epics/tasks; finalize **DB schema & migrations (Alembic)**
* Set up **project skeletons** (apps/api/ai/worker) and **Dockerfiles**
* Implement security scaffolding (JWT+refresh, cookie flags, RLS approach)
* Wire **basic CI** (lint/test/build) and env files (`.env.example`)
