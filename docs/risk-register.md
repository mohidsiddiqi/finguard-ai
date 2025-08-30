# Risk Register — Phase 0 (FinGuard AI)

This register captures known risks for the MVP, along with likelihood, impact, and planned mitigations.  
It will be updated in each phase as new risks are identified.

| ID  | Risk                               | Likelihood (1–3) | Impact (1–3) | Mitigation                                         | Owner |
|-----|------------------------------------|------------------|--------------|----------------------------------------------------|-------|
| R1  | CSV parsing edge cases (weird headers, encodings, missing fields) | 2 | 3 | Queue parsing in worker; add retry logic; provide a sample CSV template | Mohid |
| R2  | Async job flakiness (failed/duplicate tasks in Celery/RQ) | 2 | 3 | Use idempotent tasks, retries, and Redis-backed queue | Mohid |
| R3  | Security drift (forgetting baseline controls) | 2 | 3 | Maintain `docs/security-checklist.md`; review at end of each sprint | Mohid |
| R4  | Scope creep (jumping to bank aggregators or advanced AI early) | 3 | 2 | Keep aggregators flagged for Phase 4; block extras in Phase 0–3 | Mohid |
| R5  | Budget/category semantics unclear (thresholds, category rules) | 2 | 2 | Freeze entity fields in API contracts during Phase 1; validate with tests | Mohid |