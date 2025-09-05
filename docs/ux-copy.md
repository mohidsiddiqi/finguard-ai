# UX Copy — MVP

**Principles**: be clear, short, and reassuring. Prefer verbs over jargon. Always show the next best action. Tone: friendly, confident, non‑patronising.

---

## 1) Global Navigation & Labels

* **App name**: FinGuard AI
* **Primary nav**: Home · Transactions · Budgets · Trends · Imports · Insights · Sandbox Link · Activity · Profile
* **Buttons**: Save · Cancel · Upload · Connect Test Bank · Add Transaction · New Budget · Try Again · View Details · Clear Filters
* **Placeholders**: Search transactions… · Choose account… · Pick a category…

**Toasts (generic)**

* Success: “Saved.” / “Done.”
* Error: “Something went wrong.”
* Info: “Working on it…”

---

## 2) Home (Dashboard)

**Header**: “Welcome back, {{name}}”
**Empty state**: “No data yet. Start by connecting the **Sandbox Test Bank** or uploading **CSV/OFX**.”
**CTA buttons**: Connect Test Bank · Import CSV/OFX · Add Manual Transaction
**KPI captions**: This month · Top category · At risk

---

## 3) Auth & Profile

**Login title**: “Sign in to FinGuard AI”
**Register title**: “Create your account”
**Fields**: Email · Password
**Validation**

* Email: “Enter a valid email.”
* Password: “Use 8+ characters.”
  **Errors**
* Invalid creds: “Email or password is incorrect.”
* Rate limited: “Too many attempts. Try again in a minute.”
  **Profile (/me)**
* Title: “Your profile”
* Fields (readonly): Email · Country · Currency

---

## 4) Imports (CSV/OFX)

**Dropzone title**: “Drop CSV/OFX here or click to upload”
**Dropzone help**: “Accepted: .csv .ofx .qfx · CSV header sample · Max 5MB (CSV), 2MB (OFX/QFX)”
**Queued**: “Upload queued. We’ll process this shortly.”
**Processing**: “Processing… this can take a few seconds.”
**Done**: “Imported {{rows\_imported}} of {{rows\_total}} rows.”
**Failed**: “Import failed. View details for errors.”
**Row issues link**: “Download error report”
**Recent table headings**: Filename · Source · Status · Rows · Errors

**Inline errors**

* Missing header: “CSV header row is missing. Download the sample CSV.”
* Column missing: “Required column ‘{{name}}’ is missing.”
* File too large: “File exceeds the maximum size.”
* Type not allowed: “Unsupported file type.”

---

## 5) Manual Transaction (Drawer/Modal)

**Title**: “Add transaction”
**Fields**: Account · Date · Amount · Merchant (optional) · Description (optional) · Category (optional)
**Hint (amount)**: “Use negative for expenses, positive for income (e.g., −4500, +80000).”
**Success**: “Transaction added.”
**Validation**

* Account: “Choose an account.”
* Date: “Use YYYY‑MM‑DD.”
* Amount: “Enter a non‑zero amount.”

---

## 6) Transactions (List + Filters)

**Title**: “Transactions”
**Filters**: Date · Account · Category · Search
**Empty state**: “No transactions match your filters.”
**Clear action**: “Clear filters”
**Row subtitle**: “{{description}} · {{account\_name}}”

---

## 7) Trends

**Title**: “Trends”
**Controls**: Group by Category | Month · Date range
**Empty state**: “Not enough data to show trends yet.”
**Card captions**: “Spend by category” · “Spend by month”

---

## 8) Budgets

**Title**: “Budgets”
**Empty state**: “You don’t have any budgets yet.”
**New budget button**: “New budget”
**Budget row**: “{{spent}} / {{amount}} · {{percent}}% used”
**Threshold hint**: “Get an alert when you cross {{threshold\_pct}}%.”
**Create modal labels**: Period (YYYY‑MM) · Category · Amount · Threshold %
**Success**: “Budget saved.” / “Budget updated.”
**Validation**

* Period: “Use YYYY‑MM.”
* Amount: “Enter a positive amount.”
* Category: “Choose a category.”

---

## 9) Insights (AI)

**Title**: “Insights”
**Tip card header**: “Tip”
**Risk header**: “Overrun risk by category”
**Empty state**: “Insights will appear after you add some transactions.”

---

## 10) Sandbox Link (Demo‑only)

**Title**: “Connect Test Bank (Sandbox)”
**Lead**: “Fastest way to keep transactions updated—demo data only.”
**Button**: “Start Link”
**Explainers**: “What we access & why” · “Security & privacy”
**Success**: “Sandbox connected.”
**Status**: “Last sync: {{timestamp}}”
**Notice**: “Sandbox only—no real accounts.”

---

## 11) Activity (Webhooks & Jobs)

**Title**: “Activity”
**Tabs**: Webhooks · Imports · Jobs
**Headings**: Time · Source · Event/Status · Info
**Empty state**: “No recent activity.”

---

## 12) Error & Empty Patterns (Global)

**Generic error banner**: “We couldn’t complete that action.”
**Retry cta**: “Try again”
**Details cta**: “View details”
**Empty list**: “Nothing here yet.”

---

## 13) Footer & Legal

**Disclaimer**: “Insights are informational and not financial advice.”
**Privacy note**: “Sandbox connectors only; no live banking in MVP.”

---

## 14) Accessibility Aids (copy adjacent)

* “Skip to content”
* “Open navigation” / “Close navigation”
* “Open filters” / “Close filters”
* “Open details” / “Close details”

---

## 15) Microcopy Guidelines

* Prefer **verbs**: “Add transaction”, “Connect test bank”.
* Prefer **present tense** and **positive** feedback: “Saved.” not “Save successful.”
* Avoid jargon; if necessary, **explain inline** (“negative = expense”).
* Keep messages **≤ 120 characters**; tooltips **≤ 80**.
