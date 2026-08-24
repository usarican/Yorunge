---
name: realtime-streaming
description: "The full-stack live data contract in Yörünge — FastAPI SSE (EventSourceResponse) telemetry for onboarding Step 2, streaming LLM interview responses, keep-alive pings, reconnection, and the frontend hooks useTelemetrySSE / useInterviewStream. Use when producing or consuming any streamed event, or debugging a stalled progress bar, dropped connection, or duplicated log line."
---

# Real-Time Streaming Contract (SSE)

Two streams exist, and both sides of each must change together.

| Stream | Producer | Consumer |
| :-- | :-- | :-- |
| Analysis telemetry | `GET /api/v1/onboarding/telemetry/{assessment_id}` | `useTelemetrySSE.ts` → Step2Telemetry |
| Interview tokens | `interview.py` (SSE/WS) | `useInterviewStream.ts` → Step4Interview |

## Event payload — single source of truth

```jsonc
{ "type": "log", "step": "ast", "log": "Kotlin AST taraması...", "progress": 40 }
{ "type": "done", "progress": 100 }
{ "type": "error", "message": "Repo klonlanamadı" }
```

The `progress` field is monotonic and reaches exactly `100` on completion. The frontend advances to Step 3 on `progress === 100`. If a node is added to the LangGraph flow, redistribute the progress values — never emit 100 early, and never let the bar sit at 95 because the last node forgot to emit.

## Backend

```python
@router.get("/telemetry/{assessment_id}")
async def telemetry(assessment_id: UUID, user: User = Depends(get_current_user)):
    async def event_generator():
        async for update in agent_stream(assessment_id):
            yield {"event": "message", "data": update.model_dump_json()}
    return EventSourceResponse(event_generator(), ping=15)
```

- `ping=15` (15s keep-alive) is mandatory — Coolify/Traefik and browsers drop idle connections otherwise.
- Authorize the stream like any other endpoint: the caller must own the assessment. An unguarded `assessment_id` in a URL is a data leak.
- Handle client disconnect: `await request.is_disconnected()` or catch `asyncio.CancelledError`, then stop the generator and release the DB session. The LangGraph run continues on the server (it is checkpointed); the stream is only a view.
- Never hold an `AsyncSession` open for the life of the stream — acquire per event batch.
- Emit an `error` event and close cleanly on failure. Silently ending the stream leaves the UI frozen at whatever percent it reached.
- Frames must be small, JSON, and one line. No pickled objects, no tokens or secrets in a frame.

## Frontend

```ts
useEffect(() => {
  const es = new EventSource(`/api/v1/onboarding/telemetry/${assessmentId}`);
  es.onmessage = (e) => { /* parse, append, advance */ };
  es.onerror = () => { /* surface state, let the browser retry, close after N failures */ };
  return () => es.close();
}, [assessmentId]);
```

- The cleanup `es.close()` is not optional — React strict mode mounts twice, and a leaked EventSource duplicates every log line.
- The effect depends on `assessmentId` only. Putting a store value in the dep array reopens the stream on every log.
- `EventSource` cannot send an `Authorization` header. Either proxy through a Next.js route handler that attaches it, or use a short-lived signed token in the query string — do not disable auth on the endpoint to make it work.
- Wrap store writes so a burst of events doesn't re-render the whole wizard (see `frontend-state-data` on narrow selectors).
- Show a reconnecting state after an error rather than a dead progress bar; give up after a bounded number of retries and offer a retry button.
- On `done`, close the stream and invalidate the assessment query so the report renders from authoritative server data.

## Interview streaming

Token-level streaming appends to the last assistant message; only the final complete turn is written to `interviews.chat_history`. A partial turn interrupted by a disconnect must not be persisted as if it were complete.

## Verification

Manual check is required — `docker compose up --build`, run onboarding Step 2, confirm the bar reaches 100 and Step 3 opens, then reload mid-stream and confirm state recovers without duplicate log lines.
