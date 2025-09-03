# Security Baseline — MVP (Phase 0 Stub)

Will be expanded in Phase 2; items below are **minimum gates**.

## Authentication & Tokens
- [ ] JWT access + rotating refresh
- [ ] Refresh tokens stored as secure HTTP-only cookies

## Password Security
- [ ] Argon2/bcrypt password hashing
- [ ] Strong password policy (length, complexity)

## Data Access
- [ ] Per-user row-level access (no cross-user exposure)

## Transport Security
- [ ] HTTPS-only via proxy (Caddy/Nginx) with Let’s Encrypt
- [ ] Cookie flags: HttpOnly, Secure, SameSite

## Rate Limiting & Audit
- [ ] Basic API rate limits on auth & uploads
- [ ] Audit logs for auth, uploads, budgets, webhook events

## Data Minimization & Secrets
- [ ] Minimal PII at rest
- [ ] Encrypted secrets/credentials (no secrets in code)

## **NEW — Webhooks & File Safety**
- [ ] **Webhook signature verification** (e.g., Plaid JWT / TrueLayer TL-Signature)
- [ ] **Idempotent** ingestion for webhook & file imports (dedupe by id/hash)
- [ ] **File upload safety** (type/size checks; reject dangerous content)
- [ ] **OFX/QFX parser hygiene** (library updates, robust error handling)
