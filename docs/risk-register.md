# Risk Register — Phase 0 (FinGuard AI)

| ID  | Risk                                                        | Likelihood (1–3) | Impact (1–3) | Mitigation                                                                 | Owner |
|-----|-------------------------------------------------------------|------------------|--------------|----------------------------------------------------------------------------|-------|
| R1  | CSV parsing edge cases (headers, encodings, missing fields) | 2                | 3            | Worker + retries; sample template; clear error logs                        | Mohid |
| R2  | OFX/QFX parsing issues or malformed files                   | 2                | 3            | Use maintained parser; strict validation; reject unsupported variants       | Mohid |
| R3  | Async job flakiness (dup/failed tasks)                      | 2                | 3            | Idempotent tasks; retries with backoff; Redis-backed queue                 | Mohid |
| R4  | Webhook forgery/replay                                     | 2                | 3            | Verify signatures (Plaid JWT / TL-Signature); enforce idempotency & clocks | Mohid |
| R5  | Sandbox ≠ production (data shape/latency drift)             | 2                | 2            | Clearly mark demo-only; document assumptions; make connector pluggable     | Mohid |
| R6  | Scope creep (live prod connectors, payments, multi-provider)| 3                | 2            | Keep prod connections OUT; one sandbox provider only; feature flags        | Mohid |
| R7  | Budget/category semantics unclear                           | 2                | 2            | Freeze entity fields in Phase 1 contracts; add tests                       | Mohid |
