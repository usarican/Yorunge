# Yörünge — Claude Code & AI Development Guide

This document serves as the primary guidance file for **Claude Code** and AI coding assistants working in the **Yörünge (Android Mentör AI)** codebase.

---

## 🚀 Quick Reference Commands

### Local Development (Docker Orchestrated)
- **Start All Services (Backend + Frontend + DB):**
  ```bash
  docker compose up --build
  ```
- **Stop All Services:**
  ```bash
  docker compose down
  ```

### Backend (Python 3.12 + FastAPI + LangGraph)
- **Navigate to backend directory:** `cd backend`
- **Install Dependencies:** `poetry install` (or `pip install -r requirements.txt`)
- **Run FastAPI Dev Server (Hot-reload):**
  ```bash
  uvicorn app.main:app --reload --port 8000
  ```
- **Run Tests:** `pytest`
- **Run Linting & Formatting:** `ruff check .` and `ruff format .`
- **Database Migrations (Alembic):**
  - Create migration: `alembic revision --autogenerate -m "description"`
  - Apply migration: `alembic upgrade head`

### Frontend (Next.js 14+ App Router + TypeScript)
- **Navigate to frontend directory:** `cd frontend`
- **Install Dependencies:** `npm install`
- **Run Next.js Dev Server:** `npm run dev`
- **Run Tests:** `npm test`
- **Run Build:** `npm run build`
- **Run Linter:** `npm run lint`

---

## 🏗️ Architecture & Project Structure

Yörünge is structured as a **Modular Monorepo**:

```
Yorunge/
├── backend/                        # Python FastAPI + LangGraph AI Engine
│   ├── app/
│   │   ├── api/v1/                 # REST & SSE Routers (onboarding, interview, tasks)
│   │   ├── agents/                 # LangGraph Agent Workflows (AST, Quiz, PR feedback, RAG)
│   │   ├── core/                   # Security (Fernet OAuth Token Encryption), DB engine
│   │   ├── db/models/              # SQLAlchemy Async Models (Supabase Postgres)
│   │   └── services/               # AST Parser, GitHub API Client, Vector Store
│   └── Dockerfile                      # Production Multi-Stage Container
├── frontend/                       # Next.js 14+ (App Router) + TypeScript
│   ├── src/
│   │   ├── app/
│   │   │   ├── (onboarding)/       # 5-Step Stepper Wizard Layout (NO Sidebar)
│   │   │   └── (dashboard)/        # Main App Layout (With Collapsible Sidebar)
│   │   ├── components/             # Daylight Observatory Design System components
│   │   ├── stores/                 # Zustand Stores (Onboarding state, Telemetry)
│   │   └── hooks/                  # SSE Telemetry & Streaming Chat Hooks
│   └── Dockerfile                      # Production Standalone Container
├── docker-compose.yml              # Local Development Stack Orchestration
├── docs/                           # Specs, Design System, & Architecture docs
│   ├── architecture/
│   │   ├── backend-architecture.md  # Backend technical spec & DB schema
│   │   └── frontend-architecture.md # Frontend technical spec & UI layout
│   └── dev-mentor-ai-plan (1).md   # Product & Phase Plan
├── CLAUDE.md                       # This file (Claude Code guidelines)
└── AGENTS.md / Agent.md            # Multi-agent repository rulebook
```

---

## 🎨 Code Style & Architectural Conventions

### 1. Python / FastAPI Guidelines
- **Type Annotations:** Explicit type hints on all function parameters and return types (`def parse_ast(repo_url: str) -> ASTResult:`).
- **Async/Await:** Use `async def` for FastAPI endpoints and DB queries (`AsyncSession` + `select()`).
- **Pydantic v2:** Use Pydantic models for request/response validation. Always inherit from `BaseModel`.
- **Secrets & Credentials:** Never hardcode API keys or OAuth secrets. Use `app/config.py` (`BaseSettings`).
- **Token Encryption:** Store GitHub OAuth tokens using `Fernet` encryption before writing to Supabase.

### 2. Next.js / TypeScript Guidelines
- **App Router Layout Isolation:**
  - `(onboarding)` route MUST NOT render a Sidebar. Stepper Header navigation only.
  - `(dashboard)` route MUST render full platform shell with Sidebar.
- **Design System:** Follow `docs/design-system/DESIGN.md` (Daylight Observatory dark theme: HSL dark blue background `#0B0F19`, cyan accents `#00F2FE`, `Inter` for UI, `Space Mono` for telemetry/code).
- **State Separation:** Use **Zustand** for transient wizard state and **React Query** for server-side state.

---

## 🛡️ Non-Negotiable AI Rules & Guardrails

1. **No Symptom Patching:** Never swallow exceptions or return dummy data to mask bugs. Fix the underlying root cause.
2. **Mandatory Verification:** Never mark a task as complete without running build/test verification (`npm run build`, `pytest`, or `docker compose`).
3. **Preserve Documentation:** Retain existing docstrings, licenses, and comments when modifying existing files.
4. **Exact File Edits:** When editing code, ensure target lines match exact file contents.
