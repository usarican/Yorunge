---
name: docker-coolify-deploy
description: "Container and deployment work for Yörünge — backend and frontend multi-stage Dockerfiles, docker-compose local stack, docker-entrypoint.sh with Alembic upgrade, environment variables, healthchecks, and Coolify/Hetzner VPS deploy concerns (SSL, SSE through the proxy, resource limits). Use when editing a Dockerfile, docker-compose.yml, entrypoint, or env/deploy configuration."
---

# Docker & Coolify Deployment

Reference definitions: `docs/architecture/backend-architecture.md` §6 and `frontend-architecture.md` §5. Keep the committed files consistent with those docs.

## Image rules

**Backend** — multi-stage `python:3.12-slim`: builder installs into `/install`, runner copies it. `git` and `curl` stay in the runner (repo cloning for AST analysis needs git). Entry via `docker-entrypoint.sh` → `alembic upgrade head` → uvicorn.

**Frontend** — `node:20-alpine`, four stages (base → deps → builder → runner), `output: 'standalone'` in `next.config.js`, non-root `nextjs:nodejs` user, only `.next/standalone`, `.next/static` and `public` copied into the runner. Shipping `node_modules` into the final image defeats the whole point.

Both:
- `.dockerignore` must exclude `.git`, `node_modules`, `.next`, `__pycache__`, `.venv`, `.env`. A missing `.dockerignore` silently multiplies image size and can bake secrets in.
- Layer order = dependency manifests copied and installed **before** source, or every code edit reinstalls everything.
- Pin base image tags; no `:latest`.
- No secrets in `ENV`, no secrets in build args — they persist in image layers.

## Build-time vs run-time env (Next.js)

`NEXT_PUBLIC_*` values are inlined at **build** time. Changing them in Coolify's runtime env does nothing until the image is rebuilt. Anything that differs per environment and must be runtime-configurable has to be read server-side, not via `NEXT_PUBLIC_`.

## docker-compose (local only)

- Backend mounts `./backend:/app` for hot reload; the frontend runs `npm run dev` locally rather than through the production image.
- Secrets come from a local `.env` that is gitignored. `docker-compose.yml` references `${VAR}` — it never contains a real value.
- `depends_on` alone does not wait for Postgres readiness; use a healthcheck with `condition: service_healthy`, otherwise the first `alembic upgrade` races the DB and the stack flaps on a cold start.
- Compose is the local stack. Production is Coolify — do not add production-only services here without also documenting them.

## Coolify / VPS specifics

- Deploy is git-push driven; a broken `alembic upgrade head` takes the service down on boot. Verify migrations locally with `docker compose up --build` first.
- **SSE through the proxy**: buffering breaks the telemetry stream. Ensure `X-Accel-Buffering: no` on SSE responses and keep the 15s keep-alive ping (see `realtime-streaming`). A progress bar that freezes in production but works locally is almost always proxy buffering or an idle timeout.
- Set a healthcheck endpoint (`GET /health`, no DB dependency) so Coolify restarts only genuinely dead containers.
- Uvicorn `--workers 4` is a VPS-size decision, not a constant — memory per worker matters on a small Hetzner box. Tune with the LLM/AST memory profile in mind, and remember worker count interacts with in-process rate limiting and any in-memory state (there should be none).
- Persistent volumes: only Postgres data locally; Supabase is managed in production. Never store uploaded CVs on the container filesystem — they vanish on redeploy.

## Verification

```bash
docker compose up --build      # full stack must come up clean
docker compose down
```
Confirm: migrations applied, `/health` returns 200, the frontend can reach the backend, and one SSE stream survives 60+ seconds idle.
