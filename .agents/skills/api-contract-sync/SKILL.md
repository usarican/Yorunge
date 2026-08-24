---
name: api-contract-sync
description: "Keep the FastAPI Pydantic schemas and the frontend TypeScript types in sync across the Yörünge monorepo. Use whenever adding or changing an endpoint, a request/response field, an enum, or the 4-axis score shape — anything that crosses backend/app/schemas/ and frontend/src/types/."
---

# API Contract Sync (backend/schemas ⇄ frontend/types)

The monorepo has two type systems describing one contract. A change on one side that is not made on the other is an unfinished change, not a follow-up.

## The paired files

| Backend | Frontend |
| :-- | :-- |
| `app/schemas/assessment.py` | `src/types/assessment.ts` |
| `app/schemas/roadmap.py` | `src/types/roadmap.ts` |
| SSE event models | the event union in `useTelemetrySSE.ts` (see `realtime-streaming`) |

## Change checklist

When you touch a request or response shape:

1. Update the Pydantic model, with `response_model=` wired on the route.
2. Update the mirrored TS interface — same field names (`snake_case` on both sides; do not rename in transit).
3. Update the React Query hook and any component reading the field.
4. If it changed the DB, add the Alembic migration (`supabase-db-migrations`).
5. Reflect it in `docs/architecture/*.md` if the doc shows that shape.
6. Run both verifications: `pytest` and `npx tsc --noEmit && npm run build`.

## Rules that prevent drift

- **No `any`, no `as` casts** on API data. A cast hides exactly the mismatch this skill exists to catch.
- Optionality must match: Pydantic `str | None` ⇄ TS `string | null` (not `string?` unless the key is genuinely absent). Confusing "null" with "missing" produces runtime crashes on the report screen.
- Enums are shared literal unions on the frontend (`type AssessmentStatus = "pending" | "running" | "completed" | "failed"`) matching the Postgres enum exactly. Adding a status on the backend means adding it here, and handling it in the UI switch.
- Datetimes cross as ISO 8601 strings and are typed `string`, parsed at the edge — never typed `Date`.
- UUIDs are `string`.
- The 4-axis score object (`kotlin_idioms`, `jetpack_compose`, `software_principles`, `system_design`, each `{ score, level, evidence[] }`) is one named type on both sides. Never inline it twice.

## Preferred: generate instead of hand-writing

For larger surfaces, generate the client types from the live OpenAPI schema rather than transcribing them:

```bash
npx openapi-typescript http://localhost:8000/openapi.json -o src/types/api.d.ts
```

If that generation step is in place, hand-edited duplicates of generated types are forbidden — extend via composition instead. Regenerate whenever the backend contract changes and commit the diff.

## Errors are part of the contract too

FastAPI errors arrive as `{ "detail": ... }`. `lib/api.ts` normalizes that into a typed `ApiError`; new structured error payloads (e.g. validation details on quiz submission) need a matching TS type, not ad-hoc `catch (e: any)` handling.
