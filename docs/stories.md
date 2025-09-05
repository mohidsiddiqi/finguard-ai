## FILE: docs/stories.md

# FinGuard AI — User Stories (Brief, Phase 1)

> Format: **As a \[role], I want \[goal], so that \[benefit].**
> Columns: **ID, Epic, Story (one‑liner), Priority (P0/P1/P2), Estimate (pts), FR Link**.
> Acceptance Criteria live in **FRs (docs canvas)** and are referenced by **FR‑IDs**.

---

## Epic Definitions

* **EP‑AUTH**: Authentication & Profile
* **EP‑ACCT**: Accounts
* **EP‑IMPORT**: Imports (CSV/OFX)
* **EP‑MANUAL**: Manual Transactions
* **EP‑TXN**: Transactions & Filters
* **EP‑TRENDS**: Trends (Derived)
* **EP‑BUDGETS**: Budgets (Create/List/Update)
* **EP‑AI**: AI Tips & Risk Scores
* **EP‑LINK**: Sandbox Bank Link
* **EP‑WEBHOOKS**: Webhook Receiver & Idempotency
* **EP‑ACTIVITY**: Activity & Import History

---

## Stories

| ID    | Epic        | Story (one‑liner)                                                                                     | Priority | Est | FR Link     |
| ----- | ----------- | ----------------------------------------------------------------------------------------------------- | -------- | --- | ----------- |
| US‑01 | EP‑AUTH     | As a visitor, I want to **register** with email/password so that I can create an account.             | P0       | 3   | FR‑01       |
| US‑02 | EP‑AUTH     | As a user, I want to **log in** and receive an access token so that I can access my data.             | P0       | 3   | FR‑02       |
| US‑03 | EP‑AUTH     | As a user, I want to **view my profile** so that I can confirm country/currency and account identity. | P1       | 1   | FR‑03       |
| US‑04 | EP‑ACCT     | As a user, I want to **create an account** (e.g., HBL Checking) so that I can attach transactions.    | P0       | 2   | FR‑04       |
| US‑05 | EP‑ACCT     | As a user, I want to **list my accounts** so that I can choose them when filtering or adding data.    | P0       | 1   | FR‑05       |
| US‑06 | EP‑IMPORT   | As a user, I want to **upload a CSV** so that transactions are imported into my accounts.             | P0       | 5   | FR‑06       |
| US‑07 | EP‑IMPORT   | As a user, I want to **upload an OFX/QFX** so that bank statements import reliably.                   | P0       | 5   | FR‑07       |
| US‑08 | EP‑MANUAL   | As a user, I want to **add a manual transaction** so that I can record cash or missing entries.       | P0       | 3   | FR‑08       |
| US‑09 | EP‑ACTIVITY | As a user, I want to **view import history** so that I can verify results and see errors if any.      | P1       | 2   | FR‑09       |
| US‑10 | EP‑LINK     | As a user, I want to **connect a sandbox test bank** so that demo transactions sync automatically.    | P1       | 3   | FR‑10       |
| US‑11 | EP‑WEBHOOKS | As the system, I need to **verify webhook signatures** so that only trusted events are processed.     | P0       | 3   | FR‑11       |
| US‑12 | EP‑WEBHOOKS | As the system, I need **idempotent ingestion** so retries/replays don’t create duplicates.            | P0       | 3   | FR‑12       |
| US‑13 | EP‑WEBHOOKS | As the system, I need to **auto‑create accounts** when new provider accounts appear in webhooks.      | P1       | 2   | FR‑13       |
| US‑14 | EP‑TXN      | As a user, I want to **list transactions with filters** so that I can quickly find what I need.       | P0       | 3   | FR‑14       |
| US‑15 | EP‑TXN      | As a user, I want to **update a transaction’s category** so that my reports stay accurate.            | P1       | 2   | FR‑15       |
| US‑16 | EP‑TRENDS   | As a user, I want to **see spending by category/month** so that I can understand patterns.            | P1       | 3   | FR‑16       |
| US‑17 | EP‑BUDGETS  | As a user, I want to **create a budget** for a period/category so that I can plan spending.           | P0       | 3   | FR‑17       |
| US‑18 | EP‑BUDGETS  | As a user, I want to **view budgets with status vs actuals** so I can track progress.                 | P0       | 3   | FR‑18       |
| US‑19 | EP‑BUDGETS  | As a user, I want to **update a budget** so that I can adjust amounts or thresholds.                  | P1       | 2   | FR‑19       |
| US‑20 | EP‑AI       | As a user, I want to **receive AI budgeting tips** so that I get actionable suggestions.              | P2       | 2   | FR‑20       |
| US‑21 | EP‑AI       | As a user, I want to **see overrun risk scores by category** so that I can prevent overspend.         | P2       | 2   | FR‑21       |
| US‑22 | EP‑ACTIVITY | As a user, I want an **Activity view** (webhooks/imports/jobs) so that I trust the system’s behavior. | P2       | 2   | FR‑09/11/12 |

---

## Notes

* **Priorities**: P0 = MVP critical; P1 = important for demo value; P2 = nice‑to‑have polish within MVP window.
* **Estimates** are rough story points for planning; refine in Phase 2.
* Each story’s **Gherkin AC** lives in the FRs doc under the corresponding **FR‑ID**.
* Keep UX copy consistent with `docs/ux-copy.md` and validation with `docs/schemas/validation_matrix.md`.
