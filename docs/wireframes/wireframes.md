# FinGuard AI — Wireframes (Phase 1)

> Goal: A modern, elegant, **immediately familiar** finance app that first‑time, casual, and returning users can navigate without thinking. Keep flows shallow, copy clear, and states explicit (empty, loading, success, error). Responsive from 360px+.

---

## Global Layout & Navigation

- **Header (sticky):** logo · global search (cmd+/) · notifications · profile menu (me/logout)
- **Primary nav:**
  - **Desktop ≥ 1024px:** left **navigation rail** with icons+labels: *Home, Transactions, Budgets, Trends, Imports, Insights, Sandbox Link, Activity*
  - **Tablet/Mobile:** **Bottom navigation** (3–5 primaries): *Home, Transactions, Budgets, More* (sheet reveals *Trends, Imports, Sandbox, Activity, Settings*)
- **Page shell:** content container (max‑width 1280px), grid‑based sections, cards with soft shadows (no clutter), generous whitespace
- **Feedback:** toast for success/error, inline validation on forms, progress bars for long tasks, skeleton loaders while fetching, explicit empty states

### Global Components
- **Primary Button:** filled; **Secondary:** outline; **Destructive:** subtle danger
- **Filters Bar:** date range, account, category, search field, clear
- **Table/List:** responsive—table on desktop, card list on mobile
- **Modals/Drawers:** create/edit budget; manual transaction entry; import results
- **Badges/Chips:** category, status (queued/processing/done/failed/duplicate)

---

## 0) First‑Run vs Returning Logic

- **First‑time (no data):** Home shows three large CTAs: *Connect Test Bank*, *Import CSV/OFX*, *Add a Manual Transaction*. Each CTA includes 1‑line benefit and an arrow to the action. Empty states across other pages link back to these.
- **Returning:** Home shows personal KPIs and quick links (last import, budgets at risk, spend this month). A subtle “+” quick‑add menu provides *Manual Txn* and *New Budget* from anywhere.

---

## 1) Home (Dashboard)

**Purpose:** Overview + next best actions. Two variants: empty vs populated.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ Header                                                                   │
├─────┬────────────────────────────────────────────────────────────────────┤
│ Nav │  Welcome back, Mohid · Sep budget at a glance                      │
│     │  [ Connect Test Bank ]  [ Import CSV/OFX ]  [ Add Manual Txn ]     │
│     │                                                                     │
│     │  KPIs (cards, 3‑up):  [This Month Spend] [Top Category] [At Risk]  │
│     │                                                                     │
│     │  Trends (mini chart):  Spend by month (last 3)                      │
│     │  Budgets (mini list):  Groceries 62% · Transport 48% · Dining 80%   │
│     │  Insights (chips):  “Reduce dining” · “Subscriptions +Rs1200/mo”    │
└─────┴────────────────────────────────────────────────────────────────────┘
```

**Empty state copy:** “No data yet. Start by connecting the **Sandbox Test Bank** or uploading **CSV/OFX**.”

---

## 2) Auth (Login · Register · Profile)

**Login/Register** (single column center): email, password; strong inline validation; link to Register/Login toggle. Social auth omitted in MVP.

**Profile (/me):** shows id, email, country, currency; buttons for *Change Password* (deferred) and *Logout*.

```
┌──────────────┐
│  FinGuard AI │   [ Email ] [ Password ] ( Show )
│   Sign In    │   ( Sign In )   ( Create account )
└──────────────┘
```

---

## 3) Imports (CSV/OFX)

**Purpose:** Upload, see progress, review results. Drag‑and‑drop + click‑to‑upload; samples link.

```
┌───────────────────────────────────────────────────────────────┐
│ Imports                                                       │
│ ┌─────────────── Drop CSV/OFX or Click to Upload ───────────┐ │
│ │  Accepted: .csv .ofx .qfx · CSV header sample · Size limits│ │
│ └────────────────────────────────────────────────────────────┘ │
│ Recent Imports                                                │
│ ┌───────────────┬───────────┬──────────┬───────────┬────────┐ │
│ │ Filename      │ Source    │ Status   │ Rows      │ Errors │ │
│ ├───────────────┼───────────┼──────────┼───────────┼────────┤ │
│ │ aug.csv       │ csv       │ done     │ 250       │ 0      │ │
│ │ card.qfx      │ ofx       │ failed   │ 0         │ 1      │ │
│ └───────────────┴───────────┴──────────┴───────────┴────────┘ │
└───────────────────────────────────────────────────────────────┘
```

**States:** queued → processing → done/failed; each import row opens a details drawer with parsed/errored rows.

---

## 4) Manual Transaction (Drawer/Modal)

**Purpose:** Quick add without leaving context.

```
┌──────────────────────────── Add Transaction ─────────────────────────────┐
│ Account ▾  Date 📅  Amount (+income / −expense)                          │
│ Merchant (opt)  Description (opt)  Category (opt)                        │
│ [ Cancel ]                                  [ Save Transaction ]         │
└──────────────────────────────────────────────────────────────────────────┘
```

Inline hints clarify sign for expenses/income. On save → toast + list refresh.

---

## 5) Transactions (List + Filters)

**Purpose:** Fast scanning, powerful yet simple filtering.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ Transactions                                                             │
│ [Date ▾] [Account ▾] [Category ▾] [ Search… ] ( Clear )                  │
│ ┌──────────────────────────────────────────────────────────────────────┐ │
│ │ 2025‑09‑03  Grocery Mart        Groceries         −4,500              │ │
│ │           weekly groceries  ·  HBL Checking                          │ │
│ ├──────────────────────────────────────────────────────────────────────┤ │
│ │ 2025‑09‑02  Salary Credit       Income             +80,000             │ │
│ └──────────────────────────────────────────────────────────────────────┘ │
│   « Prev   Page 1 of 12   Next »                                        │
└──────────────────────────────────────────────────────────────────────────┘
```

Mobile collapses to stacked cards; filter bar becomes a sheet.

---

## 6) Trends (Category · Month)

**Purpose:** Quick comprehension of spend patterns.

```
┌──────────────────────────────────────────────────────┐
│ Trends  [ group: Category ▾ | Month ]  [ range: … ]  │
│ ┌───────────────┐   ┌──────────────────────────────┐ │
│ │ Category Pie  │   │ Month Line (last 6)         │ │
│ └───────────────┘   └──────────────────────────────┘ │
│ Table: Category · Total · %                          │
└──────────────────────────────────────────────────────┘
```

Charts are simple; data table always present for clarity.

---

## 7) Budgets (List + Create/Edit)

**Purpose:** See status vs actuals; create/edit quickly.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ Budgets  [ period: 2025‑09 ▾ ]  (+ New)                                  │
│ ┌──────────────────────────────────────────────────────────────────────┐ │
│ │ Groceries    12,500 / 20,000   62%  [██████████░░░░░░]  thresh 90%  │ │
│ │ Transport     4,800 / 10,000   48%  [█████░░░░░░░░░░]              │ │
│ └──────────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────┘
```

**New/Edit (modal):** period, category, amount, threshold%; inline validation; live preview of progress bar if current spend exists.

---

## 8) Sandbox Link (Demo‑Only)

**Purpose:** Showcase a **trustworthy** connector flow in sandbox.

```
┌─────────────────────────────────────────────────────────────┐
│ Connect Test Bank                                           │
│ [ Recommended ]  Fastest way to keep transactions updated   │
│ ( Start Link )                                              │
│ What we access & why ▸  Security & privacy ▸                │
└─────────────────────────────────────────────────────────────┘
```

“Recommended” label and short explainer increase conversion. After Link success, show linked accounts and last sync time.

---

## 9) Activity (Webhooks & Jobs)

**Purpose:** Transparency + debugging for demo.

```
┌──────────────────────────────────────────────────────────┐
│ Activity                                                 │
│ Tabs:  Webhooks  |  Imports  |  Jobs                     │
│ ┌───────────────┬───────────────┬───────────────┬──────┐ │
│ │ Time          │ Source        │ Event/Status  │ Info │ │
│ ├───────────────┼───────────────┼───────────────┼──────┤ │
│ │ 12:04:12      │ Plaid         │ TXN_UPDATED   │ 200  │ │
│ │ 12:02:07      │ Import (csv)  │ done          │ 250  │ │
│ └───────────────┴───────────────┴───────────────┴──────┘ │
└──────────────────────────────────────────────────────────┘
```

---

## 10) Insights (AI)

**Purpose:** Actionable, concise suggestions + risk list.

```
┌──────────────────────────────────────────────────────────────┐
│ Insights                                                     │
│ Tips (cards):                                                │
│  • “Plan bulk purchases early in month to avoid splurges.”   │
│ Risk by Category:  Groceries 0.34  ·  Dining 0.62            │
└──────────────────────────────────────────────────────────────┘
```

---

## Empty, Loading, Error States (Patterns)

- **Empty:** friendly icon + 1‑line description + primary action (e.g., *Import CSV/OFX*)
- **Loading:** skeletons for lists/cards/charts; avoid spinner‑only
- **Error:** specific message + recovery (retry, download sample, open details)

---

## Responsive Behavior

- **Mobile:** bottom nav; filters as sheet; tables to cards; charts stacked
- **Tablet:** two‑column grids; navigation rail optional (collapsible)
- **Desktop:** left rail + content; two/three‑column layouts, denser tables

---

## Tone & Microcopy (examples)

- **Uploads:** “Drop CSV/OFX or click to upload · Max 5MB CSV, 2MB OFX/QFX”
- **Manual:** “Use negative for expenses, positive for income (e.g., −4500, +80000)”
- **Sandbox:** “Demo‑only: connects to a test bank. No real accounts.”
- **Budgets:** “Threshold alerts when you cross 90% of budget.”

---

## Why this works (design guardrails)

- Recognition over recall; progressive disclosure (labels, visible options)
- Familiar patterns (nav rail/bottom nav, cards, modals)
- Clear status at all times (empty/loading/error states)
- Minimal cognitive load (short forms, inline validation, defaults)
- Accessibility‑minded: contrast‑friendly palette, keyboard reachable controls

---

## Next: UI Kit (tokens)

- **Typography:** Inter or system UI; sizes: 12/14/16/20/24/32
- **Spacing scale:** 4, 8, 12, 16, 24, 32
- **Radius:** 12–16 for cards; 8 for inputs; 999 for chips
- **Shadows:** subtle (y=2 blur=8) on cards; stronger on modals
- **Color:** neutral gray UI; accent green/blue; success/danger with accessible contrast

---

## Build Notes

- Implement wireframes as React routes: `/`, `/auth/*`, `/transactions`, `/budgets`, `/trends`, `/imports`, `/insights`, `/sandbox`, `/activity`
- Keep state in URL for filters; persist last used date range/account
- Reuse components (FilterBar, DataTable, Card, ChartWrapper, Drawer, Modal, Toast)

