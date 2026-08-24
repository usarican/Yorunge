---
name: fastapi-backend
description: "Yörünge backend conventions for FastAPI + Python 3.12. Use when adding or changing anything under backend/app/ — API routers (auth, onboarding, evaluation, interview, roadmap, webhooks, knowledge), Pydantic v2 schemas, async SQLAlchemy 2.0 queries, services, dependency injection, config, error handling, or structlog logging."
---

# FastAPI Backend Conventions (Yörünge)

Authoritative spec: `docs/architecture/backend-architecture.md`. This skill is the *how*; the doc is the *what*.

## Layer rules — never skip a layer

```
api/v1/*.py   → HTTP only: validate, authorize, delegate. No business logic, no raw SQL.
agents/*.py   → LangGraph orchestration (see langgraph-agents skill).
services/*.py → External I/O: GitHub API, tree-sitter, embeddings. Framework-agnostic.
db/models/    → SQLAlchemy models only. No query helpers with business rules.
schemas/      → Pydantic v2 request/response. Never return an ORM model from a router.
core/         → database.py, security.py, logging.py. Infrastructure, imported by everyone.
```

A router that imports `AsyncSession` and writes a `select()` inline is acceptable for simple CRUD; anything touching two tables or an external API goes in `services/`.

## Router template

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

from app.core.database import get_session
from app.core.security import get_current_user
from app.db.models.user import User
from app.schemas.roadmap import RoadmapRead

router = APIRouter(prefix="/roadmap", tags=["roadmap"])


@router.get("/{roadmap_id}", response_model=RoadmapRead)
async def get_roadmap(
    roadmap_id: UUID,
    session: AsyncSession = Depends(get_session),
    user: User = Depends(get_current_user),
) -> RoadmapRead:
    roadmap = await roadmap_service.get_for_user(session, roadmap_id, user.id)
    if roadmap is None:
        raise HTTPException(status.HTTP_404_NOT_FOUND, "Roadmap bulunamadı")
    return RoadmapRead.model_validate(roadmap)
```

Non-negotiables in that template:
- Explicit type hints on **every** parameter and the return.
- `response_model=` always set — it is the contract the frontend types mirror (see `api-contract-sync`).
- Ownership check (`user.id`) is part of the query, not an `if` after fetching.
- Turkish user-facing messages, English identifiers/comments.

## Async SQLAlchemy 2.0

```python
stmt = (
    select(Assessment)
    .where(Assessment.user_id == user_id)
    .options(selectinload(Assessment.quizzes))   # never lazy-load in async context
    .order_by(Assessment.created_at.desc())
)
result = await session.execute(stmt)
assessments = result.scalars().all()
```

- `session.execute(select(...))` — not the legacy `session.query()`.
- Every relationship traversed in a response needs `selectinload`/`joinedload`. A lazy load under asyncio raises `MissingGreenlet`; do **not** "fix" it by expiring or by `expire_on_commit=False` blindly — add the eager load.
- One `AsyncSession` per request via `Depends(get_session)`. Never open a session inside a service that already received one.
- Background/Arq workers create their own session from the engine; they must not reuse a request-scoped one.

## Pydantic v2

- `model_config = ConfigDict(from_attributes=True)` on read schemas; use `Model.model_validate(orm_obj)`.
- v2 names only: `model_dump()`, `model_validate()`, `field_validator`, `Annotated[str, Field(...)]`. `.dict()`/`.parse_obj()`/`@validator` are v1 and must not appear.
- The 4-axis score JSONB is a real Pydantic model (`AxisScore` with `score`, `level`, `evidence: list[str]`), not `dict[str, Any]` — it is validated on write and on read.

## Config & secrets

All environment access goes through `app/config.py`:

```python
class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", extra="ignore")
    database_url: str
    anthropic_api_key: SecretStr
    github_client_secret: SecretStr
    encryption_key: SecretStr

settings = Settings()
```

`os.getenv` anywhere outside `config.py` is a bug. Secrets are `SecretStr` so they cannot leak into logs or tracebacks.

## Errors and logging

- Expected failures → `HTTPException` with the right status. Unexpected → let it propagate to the global handler in `main.py`, which logs with `structlog` and returns a generic 500.
- **Never** `except Exception: pass`, never return placeholder/dummy data to make an endpoint "work". If an upstream (GitHub, LLM) fails, surface a 502/503 with a Turkish message.
- Structured logs only: `log.info("ast_analysis_started", assessment_id=str(id), repo_count=len(repos))` — key/value, never f-strings.
- Bind `request_id` and `user_id` in middleware so every downstream log line carries them.

## Long-running work

AST parsing and RAG indexing never run inside the request. The router enqueues an Arq job and returns an id; progress reaches the browser over SSE (see `realtime-streaming`). A request handler that blocks more than ~2s is a design error.

## Rate limiting

LLM-backed endpoints (`interview`, quiz generation, `knowledge`) carry `slowapi` limits keyed on user id, e.g. `@limiter.limit("10/minute")`. Adding a new LLM-calling endpoint without a limit is incomplete work.

## Before you call it done

```bash
cd backend && ruff format . && ruff check . && pytest
```
