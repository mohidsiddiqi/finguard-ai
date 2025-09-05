> See also: ../implementation-notes/ingestion.md
---
# CSV Import Schema — v1.0 (MVP)

**Purpose**: Define a clear, friendly CSV format that first‑time and returning users can prepare without friction. Strict where it matters; forgiving on order/whitespace.

## File Basics
- **Encoding**: UTF‑8
- **Delimiter**: comma `,`
- **Quote**: double quotes `"` (only when needed)
- **Decimal**: dot `.` (e.g., `-4500.00`)
- **Line endings**: `
` or `
`
- **Max size**: **≤ 5 MB** (NFR)

## Required Header Row
Columns (order can vary):
```
date, amount, merchant, description, category, account
```
- **Order**: any
- **Case**: case‑insensitive
- **Extra columns**: allowed, **ignored** by MVP

## Field Rules (per row)
- **date**: `YYYY-MM-DD` (e.g., `2025-09-03`)
- **amount**: decimal; **negative = expense**, **positive = income** (e.g., `-4500`, `80000`)
- **merchant**: optional string (0–120)
- **description**: optional string (0–255)
- **category**: optional string (0–50)
- **account**: **must match an existing Account `name` owned by the user** (MVP rule: if not found → **row rejected**) 

> Tip: keep one account name consistent across uploads (e.g., `HBL Checking`).

## Validation & Errors
- Missing **header** → file rejected (`400`, code `csv_header_missing`)
- Missing **required column** → file rejected (`csv_column_missing:<name>`)
- Unknown **date** format → row skipped (`row_invalid_date`)
- Non‑numeric **amount** → row skipped (`row_invalid_amount`)
- Unknown **account** → row skipped (`row_unknown_account`)
- Too‑long text fields → row trimmed and/or skipped with reason
- Blank lines → ignored

**Outcome** (per upload): `queued → processing → done/failed`, with counts: `rows_total`, `rows_imported`, `rows_skipped` and sample errors.

## Examples
### Minimal valid CSV (unordered headers)
```csv
merchant,amount,category,description,date,account
Grocery Mart,-4500,Groceries,weekly groceries,2025-09-03,HBL Checking
Salary,+80000,Income,August payroll,2025-09-02,HBL Checking
Ride Hailer,-950,Transport,office commute,2025-09-01,HBL Checking
```

### With quotes/commas
```csv
merchant,amount,category,description,date,account
"Vendor, Inc.",-1200,Shopping,"sale, clearance",2025-09-03,HBL Checking
```

---