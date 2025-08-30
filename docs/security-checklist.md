# Security Baseline — MVP (Phase 0 Stub)

The following controls are mandatory for the FinGuard AI MVP.  
This checklist will be expanded in Phase 2 (Architecture & Security).

- [ ] **Authentication & Tokens**
  - JWT access tokens with rotating refresh tokens
  - Refresh tokens stored as secure HTTP-only cookies

- [ ] **Password Security**
  - Argon2/bcrypt password hashing
  - Strong password requirements (min length, complexity)

- [ ] **Data Access**
  - Per-user row-level access enforced in the database
  - No cross-user data exposure

- [ ] **Transport Security**
  - HTTPS-only via proxy (Caddy/Nginx) with Let’s Encrypt TLS
  - Secure cookie flags (HttpOnly, Secure, SameSite)

- [ ] **Rate Limiting & Audit**
  - Basic API rate limits to prevent abuse
  - Audit logs for authentication and budget/transaction actions

- [ ] **Data Minimization**
  - Minimal personally identifiable information (PII) stored
  - All secrets and credentials encrypted in environment variables / secret manager