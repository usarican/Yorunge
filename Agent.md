# Agent.md — Yörünge Autonomous AI Agent Entry Point

> **Note:** This repository uses [`AGENTS.md`](file:///Users/ibrahimutkusarican/Yorunge/AGENTS.md) as its primary, authoritative AI rulebook and operational protocol.

---

## ⚡ Quick Summary of Agent Rules

1. **Authoritative Guide:** Refer to [`AGENTS.md`](file:///Users/ibrahimutkusarican/Yorunge/AGENTS.md) for full discovery protocols, dual-layer agent rules, and safety guardrails.
2. **Codebase Discovery:** ALWAYS prefer MCP Knowledge Graph tools (`search_graph`, `trace_path`, `get_code_snippet`, `query_graph`) over `grep`/`glob` for finding functions, classes, and routes.
3. **Execution Guardrails:**
   - Never mask bugs with silent try-except blocks or mock data.
   - Always run verification (`pytest`, `npm run build`, `docker compose`) before reporting completion.
   - Respect layout isolation: `(onboarding)` routes MUST NOT have a sidebar, while `(dashboard)` routes MUST have a sidebar.
4. **Architecture Specifications:**
   - Backend Specs: [`docs/architecture/backend-architecture.md`](file:///Users/ibrahimutkusarican/Yorunge/docs/architecture/backend-architecture.md)
   - Frontend Specs: [`docs/architecture/frontend-architecture.md`](file:///Users/ibrahimutkusarican/Yorunge/docs/architecture/frontend-architecture.md)
   - Claude Guidelines: [`CLAUDE.md`](file:///Users/ibrahimutkusarican/Yorunge/CLAUDE.md)
