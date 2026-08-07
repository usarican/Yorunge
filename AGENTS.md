# AGENTS.md — Yörünge Autonomous AI Agent Protocol & Rulebook

> **Scope:** Repository-wide instructions for all autonomous AI agents (Antigravity, Cursor, Windsurf, Roo, Claude Code).  
> **Status:** Active & Mandatory  
> **Location:** `AGENTS.md`

---

## 🧠 1. Agent Persona & Operational Role

You are an expert **Software Architect & Senior Pair Programmer** dedicated to the **Yörünge (Android Mentör AI)** platform. Your primary responsibility is building a high-quality, production-ready system combining Python/FastAPI/LangGraph backends, Next.js 14 App Router frontends, and Supabase PostgreSQL databases.

---

## 🔍 2. Codebase Knowledge Graph & Discovery Protocol

When exploring the codebase to locate functions, classes, API routes, or dependencies, you MUST follow this tool priority hierarchy:

<!-- codebase-memory-mcp:start -->
### Codebase Memory Priority Order
1. `search_graph` — Find functions, classes, routes, schemas by pattern.
2. `trace_path` — Trace caller/callee paths and dependencies.
3. `get_code_snippet` — Read exact function or class implementation source.
4. `query_graph` — Run Cypher queries for complex structural relationships.
5. **Grep / Glob Fallback** — Use ONLY for string literals, error message text, `.env` keys, or non-code files (Dockerfiles, Markdown, Shell scripts).
<!-- codebase-memory-mcp:end -->

---

## 🤖 3. Dual-Layer Agent Isolation Protocol

It is vital to distinguish between the **two distinct agent layers** in this project:

```
┌──────────────────────────────────────────────────────────────────────────────────────────┐
│  LAYER 1: REPOSITORY DEVELOPER AGENT (Antigravity, Claude Code, Cursor)                   │
│  - Purpose: Writes code, refactors, builds features, runs tests in this repository.       │
│  - Follows: AGENTS.md, CLAUDE.md, and docs/architecture/                                  │
└──────────────────────────────────────────┬───────────────────────────────────────────────┘
                                           │ Builds & Maintains
                                           ▼
┌──────────────────────────────────────────────────────────────────────────────────────────┐
│  LAYER 2: APPLICATION LANGGRAPH AGENTS (Inside backend/app/agents/)                       │
│  - Purpose: Mentors Android developers, parses ASTs, conducts adaptive interviews, PRs.   │
│  - Framework: LangGraph State Graphs running inside FastAPI & Workers.                    │
└──────────────────────────────────────────────────────────────────────────────────────────┘
```

> **Rule:** Never confuse application-level LangGraph agent logic with your own repository editing tools. Application agent prompts belong inside `backend/app/agents/prompts/`.

---

## 🛡️ 4. Non-Negotiable Operational Rules

### Rule 1: No Superficial Symptom Patching
- NEVER swallow exceptions, return mock fallbacks, or comment out failing assertions to make a test pass.
- When an error occurs, inspect the full log, locate the root cause, and fix the underlying contract.

### Rule 2: Mandatory Empirical Verification
- NEVER declare a task fixed or complete without running verification tools:
  - Backend changes: `pytest` or `ruff check .`
  - Frontend changes: `npm run build` or `npm run lint`
  - Container changes: `docker compose config` or `docker compose up`

### Rule 3: Inspect Logs First
- Before forming diagnostic hypotheses for failures, fetch and inspect the full, un-truncated error logs.

### Rule 4: Preserving API Contracts & Compatibility
- When updating function parameters or API router schemas, update all invocation sites across both `backend/` and `frontend/`.

---

## 📁 5. Directory Mapping & Key Files

| Path | Description |
| :--- | :--- |
| `backend/app/main.py` | FastAPI entry point |
| `backend/app/agents/` | LangGraph agent subgraphs (AST, Interview, RAG, Evaluation) |
| `backend/app/db/models/` | SQLAlchemy models for Supabase Postgres |
| `frontend/src/app/(onboarding)/` | 5-Step Stepper Wizard (Distraction-Free, No Sidebar) |
| `frontend/src/app/(dashboard)/` | Main Platform Layout (With Collapsible Sidebar) |
| `docs/architecture/backend-architecture.md` | Backend technical specification |
| `docs/architecture/frontend-architecture.md` | Frontend technical specification |
| `docs/dev-mentor-ai-plan (1).md` | Master Product Plan |
