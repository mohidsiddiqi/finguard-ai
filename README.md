# FinGuard AI
**Track expenses, set budgets, and get AI-driven financial tips — all from a simple CSV.**

FinGuard AI is an MVP for automated personal finance tracking.  
It allows users to upload CSVs, view transactions and spending trends, set budgets, and receive AI-powered tips and overrun risk alerts.

---

## 🚀 MVP Features

* Import via **CSV/OFX/QFX** or **manual entry** → parse, store, list, and filter transactions
* **Sandbox bank connection (demo-only)** with **webhook ingestion** into Transactions
* Categorize expenses and view spending trends
* Create budgets and track status vs actuals
* AI-powered budgeting tips + overrun risk scoring
* User authentication and basic account management
* One-command deployment with Docker Compose (local + single-node VPS)

---

## 🛠️ Tech Stack

* **Frontend**: React (Vite, TypeScript, Tailwind)
* **Backend**: FastAPI (core API) + FastAPI AI microservice
* **Worker**: Celery/RQ with Redis
* **Database**: PostgreSQL
* **Infra**: Docker Compose, Nginx/Caddy, S3-compatible storage

---

## 📂 Repository Structure

```
finguard-ai/
├── apps/                  # web, api, ai, worker
├── infra/                 # docker, proxy config
├── docs/                  # SDLC artifacts (prd.md, security-checklist.md, risk-register.md, etc.)
└── README.md
```

---

## ⚡ Quick Start (MVP, stub)

```bash
# clone the repo
git clone https://github.com/mohidsiddiqi/finguard-ai.git
cd finguard-ai

# bring up the stack (dev mode)
docker compose up
```

---

## 📅 Project Status

* **Current Phase:** Phase 0 — Problem Framing
* Next: Phase 1 — Requirements & UX (user stories, wireframes, API contracts)

For detailed SDLC plan and design docs, see [`/docs`](./docs).

---

## 👤 Author

**Mohid Siddiqi**
Senior Full-Stack Developer | Cloud & Security | Infrastructure Automation
[LinkedIn](https://www.linkedin.com/in/mohid-siddiqi) | [mohidsiddiqi@gmail.com](mailto:mohidsiddiqi@gmail.com)