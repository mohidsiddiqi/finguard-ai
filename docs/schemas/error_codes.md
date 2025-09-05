# Error Codes (Phase 1)

> Canonical codes, HTTP status, and client guidance. **Always** return the standard error shape.

| Code                        | HTTP | Message (example)                               | Client Action                  |
| --------------------------- | ---- | ----------------------------------------------- | ------------------------------ |
| validation\_failed          | 400  | One or more fields are invalid.                 | Highlight fields; re‑submit.   |
| auth\_token\_missing        | 401  | Authorization token is required.                | Re‑login; send bearer token.   |
| auth\_token\_invalid        | 401  | Authorization token is invalid or expired.      | Refresh or re‑login.           |
| auth\_invalid\_credentials  | 401  | Email or password is incorrect.                 | Inform user; throttle retries. |
| auth\_email\_in\_use        | 409  | An account with this email already exists.      | Offer login/reset.             |
| account\_not\_found         | 404  | Account not found.                              | Verify account id; refresh.    |
| transaction\_not\_found     | 404  | Transaction not found.                          | Refresh; check ownership.      |
| budget\_not\_found          | 404  | Budget not found.                               | Refresh.                       |
| provider\_not\_supported    | 400  | Provider is not supported.                      | Use available provider.        |
| file\_type\_not\_allowed    | 400  | File type is not allowed.                       | Enforce allowed list.          |
| file\_too\_large            | 413  | File exceeds maximum size.                      | Reduce size.                   |
| csv\_header\_missing        | 400  | CSV header row is missing.                      | Use sample format.             |
| csv\_column\_missing:<name> | 400  | Required CSV column is missing: <name>.         | Add column.                    |
| row\_invalid\_date          | 207  | Some rows skipped due to invalid dates.         | Show per‑row report.           |
| row\_invalid\_amount        | 207  | Some rows skipped due to invalid amounts.       | Show per‑row report.           |
| row\_unknown\_account       | 207  | Some rows skipped due to unknown account names. | Map or create account.         |
| ofx\_parse\_error           | 400  | Could not parse OFX/QFX file.                   | Check file; use sample.        |
| ofx\_unsupported            | 400  | OFX/QFX container not supported.                | Use supported statements.      |
| webhook\_signature\_missing | 401  | Webhook signature header is missing.            | Provider misconfig; log.       |
| webhook\_signature\_invalid | 401  | Webhook signature is invalid or expired.        | Reject; record event.          |
| webhook\_payload\_invalid   | 400  | Webhook payload is invalid JSON or schema.      | Reject; record event.          |
| rate\_limited               | 429  | Too many requests.                              | Back off; respect Retry‑After. |
| not\_found                  | 404  | Resource not found.                             | Verify path/id.                |
| internal\_error             | 500  | Something went wrong.                           | Retry later; alert ops.        |

**Notes**

* `207` indicates multi‑status summary for imports; details delivered in the upload result object.
* For `rate_limited`, include `Retry-After` header.
* Never leak sensitive details (e.g., which field failed password checks); keep messages helpful but generic.
