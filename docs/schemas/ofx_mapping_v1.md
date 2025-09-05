> See also: ../implementation-notes/ingestion.md
---
# OFX/QFX Mapping — v1.0 (MVP)

**Purpose**: Map common OFX/QFX statement fields to FinGuard’s internal schema. Keep ingestion predictable, deduplicate by provider transaction id, and auto‑create missing accounts (provider=`ofx`).

## Supported Containers
- **Bank statement**: `BANKMSGSRSV1` → `STMTTRNRS` → `STMTRS` → `BANKTRANLIST`
- **Credit card**: `CREDITCARDMSGSRSV1` → `CCSTMTTRNRS` → `CCSTMTRS` → `BANKTRANLIST`

Both contain `STMTTRN` items.

## STMTTRN → Transaction Mapping
| OFX Tag    | Meaning                         | FinGuard Field   | Rules |
|------------|----------------------------------|------------------|-------|
| `FITID`    | Unique transaction id            | `external_id`    | **Required**; used for **idempotency**/dedupe |
| `DTPOSTED` | Posted date/time                 | `date`           | Parse to UTC; **store date only** (`YYYY-MM-DD`) |
| `TRNAMT`   | Amount                           | `amount`         | Decimal; sign from file; negative=expense |
| `NAME`     | Payee/merchant                   | `merchant`       | Optional |
| `MEMO`     | Description/memo                  | `description`    | Optional |
| `CURDEF`   | Currency (statement level)       | —                | Ignored for MVP; API returns decimals only |

> If both `NAME` and `MEMO` exist: use `NAME` as `merchant`, `MEMO` as `description`.

## Account Identification
Bank (`STMTRS`):
- `BANKACCTFROM` → `BANKID`, `ACCTID`, `ACCTTYPE`

Credit card (`CCSTMTRS`):
- `CCACCTFROM` → `ACCTID`

**Account key** (internal): `provider='ofx'`, `acct_ref = <BANKID|ACCTID>[:ACCTTYPE]`.

**Rule**: If no matching local Account exists, **auto‑create** one with:
- `name`: `${ACCTTYPE || 'Account'} (${masked ACCTID})`
- `institution`: `OFX`
- `type`: map from `ACCTTYPE` if present; else `checking`

## Deduplication & Idempotency
- Primary key: `FITID`
- Secondary guard: hash of (`FITID`,`TRNAMT`,`DTPOSTED`)
- Replays of same `FITID` must not create duplicates

## Date/Time
- Accept `DTPOSTED` formats like `YYYYMMDD`, `YYYYMMDDHHmmSS`, optional timezone; convert to date (`YYYY-MM-DD`).

## Error Handling
- File parse fails → upload `failed` with reason `ofx_parse_error`
- Missing `FITID`/`DTPOSTED`/`TRNAMT` on a row → row skipped with reason
- Unknown container (no `BANKTRANLIST`) → `ofx_unsupported`

---
