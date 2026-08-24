---
name: verify-before-done
description: "Yörünge's mandatory verification gate before reporting any code task complete. Use at the end of every change that touches backend/ or frontend/ — runs ruff, pytest, tsc, next build, and docker compose as applicable, and defines what counts as proof rather than assumption."
---

# Verify Before Done

CLAUDE.md rule #2 is absolute: **no task is complete without running verification**. "It should work" is not a result. Report the command you ran and its actual output.

## What to run, by blast radius

**Backend only**
```bash
cd backend && ruff format . && ruff check . && pytest -q
```

**Frontend only**
```bash
cd frontend && npm run lint && npx tsc --noEmit && npm run build
```

**Anything crossing the boundary** (new endpoint, contract change, SSE work, Docker/env change)
```bash
docker compose up --build
```
then exercise the actual path: hit the endpoint, or walk the wizard step in the browser.

**DB change** — additionally round-trip the migration:
```bash
alembic upgrade head && alembic downgrade -1 && alembic upgrade head
```

## Scenarios that require a manual pass, not just green tests

- Onboarding Step 2 telemetry: bar must reach 100 and auto-advance to Step 3; reload mid-stream must recover without duplicate log lines.
- Layout isolation: `(onboarding)` renders **no** sidebar; `(dashboard)` renders one.
- LangGraph interrupt: quiz submit resumes the same `thread_id` and does not re-run pre-interrupt side effects.
- Webhook: a tampered signature returns 401.

## Rules for interpreting results

- A failing test is a finding, not noise. Fix the cause; do not adjust the assertion, add `pytest.mark.skip`, or loosen a type to get green.
- Pre-existing failures unrelated to your change: say so explicitly with the output, and do not silently fold them into your work.
- A build that "passes" with new warnings you introduced is not clean — resolve them.
- Never mask a failure with a try/except, a fallback to mock data, or `// @ts-expect-error`. CLAUDE.md rule #1.

## Reporting

State plainly: what changed, which commands ran, what they returned, and anything left unverified and why. If a check could not be run (no Docker daemon, missing secret), say which one and what remains unproven — do not report completion as if it had run.
