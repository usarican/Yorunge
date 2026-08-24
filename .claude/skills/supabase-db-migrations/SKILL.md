---
name: supabase-db-migrations
description: "Yörünge database work on Supabase Postgres — SQLAlchemy 2.0 async models under backend/app/db/models/, Alembic migrations, JSONB columns (scores_4_axis, chat_history), UUID keys, indexes, pgvector tables, and the docker-entrypoint upgrade flow. Use when adding or altering a table, column, relationship, enum, or index."
---

# Database & Migrations (Supabase Postgres)

The ER schema is defined in `docs/architecture/backend-architecture.md` §4. Any new table must be reflected there in the same change.

## Model conventions

```python
class Assessment(Base):
    __tablename__ = "assessments"

    id: Mapped[UUID] = mapped_column(primary_key=True, default=uuid4)
    user_id: Mapped[UUID] = mapped_column(ForeignKey("users.id", ondelete="CASCADE"), index=True)
    status: Mapped[AssessmentStatus] = mapped_column(
        SAEnum(AssessmentStatus, name="assessment_status"), default=AssessmentStatus.PENDING
    )
    scores_4_axis: Mapped[dict | None] = mapped_column(JSONB)
    created_at: Mapped[datetime] = mapped_column(server_default=func.now())
    completed_at: Mapped[datetime | None]

    user: Mapped["User"] = relationship(back_populates="assessments")
```

Rules:
- SQLAlchemy 2.0 style only — `Mapped[...]` + `mapped_column(...)`. No legacy `Column(...)` declarations.
- UUID primary keys with `default=uuid4` (application-side, so the id exists before flush).
- Every FK gets an explicit `ondelete` and an index. Postgres does **not** index FKs automatically.
- Timestamps are `server_default=func.now()`, timezone-aware.
- Status columns are Python `Enum`s mapped to native Postgres enums — never bare strings.
- JSONB columns are typed at the edge by Pydantic (`AxisScore`), so the DB stays flexible while the API stays strict.

## Alembic workflow

```bash
cd backend
alembic revision --autogenerate -m "add roadmap_nodes table"
# READ AND EDIT the generated file, then:
alembic upgrade head
```

Autogenerate is a draft, not an answer. Always review it for:
- Dropped tables/columns you did not intend (a model file not imported in `app/db/base.py` looks "removed" to Alembic — import it).
- Enum types: autogenerate does not create/drop Postgres enums reliably. Add `sa.Enum(..., name=...).create(op.get_bind())` explicitly.
- Server defaults and `NOT NULL` on a populated table: add the column nullable → backfill in the migration → set `NOT NULL`. A single-step `NOT NULL` add fails in production.
- A `downgrade()` that actually reverses the change.

Never edit a migration that has already run in a deployed environment; write a new one.

## Data-destructive changes

Dropping or renaming a column that holds user assessment data requires an explicit go-ahead from the user first. Renames must be `op.alter_column(..., new_column_name=...)`, not drop+add (which loses data).

## Deployment path

`docker-entrypoint.sh` runs `alembic upgrade head` before Uvicorn starts. Consequences to respect:
- Migrations must be idempotent-safe and fast — no full-table rewrites on large tables during boot.
- A broken migration takes the whole service down on deploy. Test with `docker compose up --build` locally before pushing to Coolify.

## pgvector

```python
from pgvector.sqlalchemy import Vector

class KnowledgeChunk(Base):
    __tablename__ = "knowledge_chunks"
    embedding: Mapped[list[float]] = mapped_column(Vector(1536))
```

The extension needs its own migration step (`op.execute("CREATE EXTENSION IF NOT EXISTS vector")`) before the first vector column, and an HNSW index for search — details in the `rag-pgvector` skill.

## Verification

```bash
alembic upgrade head && alembic downgrade -1 && alembic upgrade head   # round-trip
pytest
```
A migration that cannot round-trip is not finished.
