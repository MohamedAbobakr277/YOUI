<p align="center">
  <h1 align="center">YOUI</h1>
  <p align="center">
    <strong>An Agentic AI System That Designs a Unique Frontend for Every Prompt</strong>
  </p>
  <p align="center">
    <a href="#-features">Features</a> •
    <a href="#-architecture">Architecture</a> •
    <a href="#-tech-stack">Tech Stack</a> •
    <a href="#-getting-started">Getting Started</a> •
    <a href="#-project-structure">Project Structure</a> •
    <a href="#-milestones">Milestones</a> •
    <a href="#-team">Team</a> •
    <a href="#-license">License</a>
  </p>
</p>

---

## 📌 About

**YOUI** is an agentic AI system that turns a plain-language brief into a **distinctive, working frontend design** — not another generic hero-section-and-three-cards template.

It addresses a real problem: AI-generated frontends look the same regardless of the request. YOUI solves this with a **five-stage Perceive–Cognize–Act pipeline** that generates design tokens per prompt, selects from layout archetypes and style directions, validates the result against automated checks, and **remembers every past design** to guarantee uniqueness.

> Built as an academic project for the *Management Information Systems in the Age of Agentic AI* course — Faculty of Science, Computer Science Department.

---

## ✨ Features

| Feature | Description |
|:--------|:------------|
| 🤖 **Guided 5-Question Dialog** | Agent 1 walks the user through five design questions with smart suggestions tailored to previous answers |
| 🎨 **Unique Design Generation** | Design tokens (palette, typography, spacing) are generated per prompt — never recycled |
| 🧬 **Anti-Repetition Memory** | Every approved design is fingerprinted; new designs that are too similar are rejected and regenerated |
| ✅ **Automated Validation** | Schema checks, blocklist enforcement, and uniqueness scoring — all in code, not by the LLM |
| 🖼️ **Live Canvas Preview** | Sandboxed iframe preview of generated React pages with real-time editing |
| 📝 **Plain-Language Edits** | Edit the generated UI by describing changes in natural language |
| 🔄 **Version History & Revert** | Every build and edit is a snapshot; revert to any previous version |
| 📦 **Export & Download** | Download the approved Spec as JSON/PDF and the build as a ready-to-run ZIP |
| 🌍 **Bilingual (AR/EN)** | Full Arabic and English support with RTL layout |
| 🔒 **Security First** | JWT auth, user-scoped data, sandboxed code execution, prompt injection detection |

---

## 🏗 Architecture

YOUI is a **layered modular monolith** with an Orchestrator pattern and two LangGraph agents.

```
┌─────────────────────────────────────────────────┐
│                   React App                      │
│         Landing · Auth · Chat · Canvas           │
└──────────────────────┬──────────────────────────┘
                       │ REST + Streamed Replies
┌──────────────────────▼──────────────────────────┐
│                   API Layer                      │
│       Routes · JWT · Validation · Rate Limit     │
├──────────────────────┬──────────────────────────┤
│                 Orchestrator                      │
│      Phase Order · Build Limit · Retries         │
├─────────┬────────────┼────────────┬─────────────┤
│ Agent 1 │ Validation │  Agent 2   │             │
│Questions│  Schema    │  Build     │             │
│ & Spec  │ Blocklist  │  & Edit    │             │
│(LangGraph) Uniqueness│(LangGraph) │             │
├─────────┴────────────┴────────────┴─────────────┤
│               Infrastructure                     │
│   Repositories · LLM Client · Exporters          │
└──────────────────────┬──────────────────────────┘
                       │
                   MongoDB
```

### Design Rules

1. **Code controls the flow.** The LLM only classifies, extracts, and writes.
2. **Agents never call each other.** They exchange only the Spec JSON, through the Orchestrator.
3. **Every database query is filtered by `user_id`.**
4. **Every model call goes through the LLM client** with retry, fallback, and token cap.
5. **User text and generated code are never trusted.**

### Design Patterns Used

| Pattern | Purpose |
|:--------|:--------|
| Orchestrator (service layer) | Phase order, attempt limits, calling agents and validators |
| State Machine (LangGraph) | Agent 1 question loop; Agent 2 build/edit flow |
| Repository | All MongoDB access, user-scoped |
| Adapter with Retry & Fallback | Unified LLM client for every model call |
| Validation Pipeline | Ordered checks: schema → blocklist → uniqueness |
| Strategy (dict of functions) | Export formats and similarity measures |
| Dependency Injection | FastAPI `Depends`; fakes in tests |
| Snapshot Versioning (Memento) | Each build/edit is a new version document |

> 📄 Full architecture document: [`docs/architecture/ARCHITECTURE.md`](docs/architecture/ARCHITECTURE.md)

---

## 🛠 Tech Stack

| Layer | Technology |
|:------|:-----------|
| **Backend** | Python 3.12 · FastAPI · LangGraph · LangChain |
| **Frontend** | React 18 · TypeScript · Tailwind CSS · Vite |
| **Database** | MongoDB (Motor / Beanie async ODM) |
| **AI/LLM** | OpenAI API (with fallback model support) |
| **Auth** | JWT (httpOnly cookies) · Argon2 / bcrypt |
| **Export** | ReportLab (PDF) · JSON · ZIP |
| **DevOps** | Docker Compose · Git |

---

## 🚀 Getting Started

### Prerequisites

- **Python** ≥ 3.12
- **Node.js** ≥ 20
- **MongoDB** ≥ 7 (or Docker)
- **Git**

### Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/MohamedAbobaker/YOUI.git
cd YOUI

# 2. Copy and configure environment variables
cp .env.example .env
# Edit .env with your LLM API key, JWT secret, etc.

# 3. Start with Docker (recommended)
docker-compose up --build

# Backend  → http://localhost:8000
# Frontend → http://localhost:3000
```

### Manual Setup

<details>
<summary><strong>Backend</strong></summary>

```bash
cd backend
python -m venv .venv
source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

</details>

<details>
<summary><strong>Frontend</strong></summary>

```bash
cd frontend
npm install        # or: pnpm install
npm run dev        # or: pnpm dev
```

</details>

---

## 📁 Project Structure

```
YOUI/
├── backend/
│   └── app/
│       ├── api/                 # Routes, auth dependency, error format
│       │   ├── routes/          #   Auth, conversations, builds, users
│       │   ├── dependencies/    #   JWT extraction, service injection
│       │   └── middleware/      #   Error handler, rate limiter
│       ├── services/            # Orchestrator, auth, conversations, build jobs
│       ├── agents/
│       │   ├── agent1/          # Question dialog & Spec generation (LangGraph)
│       │   │   └── prompts/     #   Prompt templates (guard, extract, ask, generate)
│       │   └── agent2/          # Build & edit flow (LangGraph)
│       │       └── prompts/     #   Prompt templates (plan, generate, fix, edit)
│       ├── domain/              # Spec model, validation, fingerprint, similarity
│       ├── repositories/        # MongoDB data access (always user-scoped)
│       ├── llm/                 # Unified LLM client (retry, fallback, token cap)
│       ├── exporters/           # JSON, PDF, ZIP export
│       ├── config/              # Settings, blocklist YAML files
│       ├── schemas/             # Pydantic request/response schemas
│       └── models/              # MongoDB document models
├── backend/tests/
│   ├── unit/                    # Unit tests per module
│   └── conversations/           # Scripted end-to-end chat tests
│
├── frontend/src/
│   ├── pages/                   # Landing, Auth, Chat, Canvas
│   ├── components/              # UI components (chat, canvas, auth, layout, common)
│   ├── api/                     # API client (fetch + streaming)
│   ├── hooks/                   # React hooks (useAuth, useChat, useStream)
│   ├── store/                   # State management
│   ├── i18n/locales/            # Arabic & English translations
│   ├── types/                   # TypeScript type definitions
│   ├── utils/                   # Constants, formatters
│   └── styles/                  # Global CSS & Tailwind config
│
├── docs/
│   ├── architecture/            # Architecture doc + Mermaid diagrams
│   ├── srs/                     # Software Requirements Specification
│   └── ux/                      # UX wireframes and flows
│
├── .env.example                 # Environment variable template
├── docker-compose.yml           # Backend + Frontend + MongoDB
└── README.md
```

> **Layer rule:** `api → services → agents, domain → repositories, llm` — lower layers never import upper ones.

---

## 🗓 Milestones

| # | Milestone | What Gets Built |
|:-:|:----------|:----------------|
| **M1** | Core Pipeline | API layer, auth (JWT), chat & history, Orchestrator, Agent 1, validation, fingerprints, blocklist, repositories, LLM client, JSON/PDF export |
| **M2** | Build & Canvas | Agent 2 build, Canvas (sandboxed preview), background jobs |
| **M3** | Edits & Versions | Plain-language edits, version history, revert, ZIP download |
| **M4** | Hardening | Testing, evaluation, security audit, performance tuning |

---

## 📡 API Overview

| Endpoint | Method | Description |
|:---------|:------:|:------------|
| `/auth/register` | `POST` | Create account |
| `/auth/login` | `POST` | Login (sets JWT cookie) |
| `/auth/logout` | `POST` | Clear session |
| `/conversations` | `GET` | List conversations |
| `/conversations` | `POST` | Start a new conversation |
| `/conversations/{id}` | `GET` | Get conversation details |
| `/conversations/{id}/messages` | `POST` | Send message (streamed reply) |
| `/conversations/{id}/spec` | `GET` | Download Spec (JSON \| PDF) |
| `/conversations/{id}/builds` | `POST` | Start a build |
| `/conversations/{id}/builds` | `GET` | List build versions |
| `/conversations/{id}/edits` | `POST` | Apply an edit |
| `/conversations/{id}/builds/{v}/revert` | `POST` | Revert to version |
| `/conversations/{id}/download` | `GET` | Download build (ZIP) |
| `/users/me` | `DELETE` | Delete account & data |

---

## 🔒 Security

- **Authentication:** JWT in httpOnly, Secure, SameSite cookies with expiry
- **Authorization:** Every query filtered by `user_id`; other users' data returns 404
- **Prompt Injection:** Guard node classification + user text sent as data, never instructions
- **Generated Code:** Sandboxed iframe without `allow-same-origin`; strict CSP
- **Fingerprints:** Traits only, never content; never sent to client
- **Secrets:** Environment variables; `.env` never committed; per-user rate limits

---

## 👥 Team

| Name | Role |
|:-----|:-----|
| **Mohamed Abobakr Ahmed** | Team Lead |
| Mohamed Sayed Mohamed | Member |
| Abdelrahman Mohamed Hassan | Member |
| Fatma Abdelmotagly | Member |
| Dina Ebrahem | Member |

> **Supervisor:** Prof. Sameh Ahmed — Faculty of Science, Computer Science Department

---

## 📄 License

This project is developed for academic purposes as part of the *Management Information Systems in the Age of Agentic AI* course.

---

<p align="center">
  <sub>Built with ❤️ by the YOUI team</sub>
</p>
