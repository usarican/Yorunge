---
name: langgraph-agents
description: "Build and modify Yörünge's LangGraph agent workflows under backend/app/agents/ — AST analyzer, quiz generator, adaptive scenario interview, 4-axis evaluator aggregator, GitHub PR reviewer, RAG QA. Use for agent state design, StateGraph nodes and edges, subgraphs, human-in-the-loop interrupt/resume, checkpointers, streaming, retries, and LLM cost control."
---

# LangGraph Agent Orchestration (Yörünge)

Graph topology and node names are specified in `docs/architecture/backend-architecture.md` §5. Match those names exactly — telemetry events, DB status values and the frontend stepper all key off them.

## State design

One typed state per graph, in `app/agents/state.py`:

```python
class OnboardingState(TypedDict):
    assessment_id: str
    repo_urls: list[str]
    ast_findings: Annotated[list[AstFinding], operator.add]   # accumulating channel
    quiz_questions: list[Question]
    quiz_answers: dict[str, str]
    axis_scores: dict[str, AxisScore]
    telemetry: Annotated[list[TelemetryEvent], operator.add]
```

- Fields that several nodes append to need an `Annotated[..., operator.add]` reducer; without it, parallel branches overwrite each other silently.
- Keep state serializable — it is checkpointed to Postgres. No `AsyncSession`, no HTTP clients, no file handles in state. Pass those via `config["configurable"]` or module-level factories.
- State is not a scratchpad: raw LLM responses stay local to a node; only structured, validated output is written back.

## Nodes

```python
async def ast_analysis_node(state: OnboardingState) -> dict:
    findings = await ast_parser_service.analyze(state["repo_urls"])
    return {
        "ast_findings": findings,
        "telemetry": [TelemetryEvent(step="ast", progress=40, log="Kotlin AST taraması tamamlandı")],
    }
```

- A node returns **only the keys it changed** — never the whole state.
- One responsibility per node. If a node both calls an LLM and writes to the DB, split it.
- Nodes are `async def`. Blocking libraries (tree-sitter) go through `asyncio.to_thread` or an Arq worker.
- Every node emits at least one `TelemetryEvent` so the Step 2 progress bar advances monotonically toward 100.

## Subgraphs

`ast_analyzer`, `scenario_interview`, `evaluator`, `pr_reviewer`, `rag_qa` are compiled subgraphs, each with its own state schema, added to the parent with an explicit input/output mapping. This keeps the PR-review flow (webhook-triggered) reusable without dragging onboarding state along.

## Human-in-the-loop — the quiz and interview pauses

```python
graph = builder.compile(checkpointer=AsyncPostgresSaver(pool))

# inside the node that needs the user
answers = interrupt({"questions": [q.model_dump() for q in state["quiz_questions"]]})

# resuming, from the API layer
await graph.ainvoke(Command(resume=answers), config={"configurable": {"thread_id": assessment_id}})
```

- **No checkpointer ⇒ no pause/resume.** Use `AsyncPostgresSaver` against Supabase; `MemorySaver` is for tests only.
- `thread_id` is the `assessment_id` (or `interview_id`) — stable, unique per user session, never reused.
- Everything before an `interrupt()` in the same node re-executes on resume. Put side effects (DB writes, GitHub calls) *after* the interrupt or in a separate node.
- Stale threads: a quiz abandoned past its TTL is marked `expired` by a sweeper querying the checkpointer, not left pending forever.

## Conditional edges

The `ScoreCheck` branch (`Quiz + AST ≥ Mid → interview, else → aggregator`) is a `add_conditional_edges` with a pure function over state. Keep the threshold in `app/config.py`, not inline — it will be tuned.

## LLM calls

- Anthropic via `ANTHROPIC_API_KEY` from `settings`; model id pinned in config, never hardcoded at the call site.
- Structured output through `llm.with_structured_output(PydanticModel)` — do not parse JSON out of free text.
- Bound the cost: max tokens per node, a turn cap on the interview subgraph, and a retry policy of at most 2 attempts with backoff. An unbounded `while` in a graph is an outage.
- Prompts live in `app/agents/prompts/` as module constants, not inline strings, so they can be diffed and evaluated.

## Streaming

Use `graph.astream(..., stream_mode=["updates", "custom"])`. `updates` drives telemetry; token-level interview streaming uses `astream_events` filtered to the chat node. The API layer converts these to SSE frames — see `realtime-streaming`.

## Failure handling

A node that fails after retries writes a failure telemetry event and routes to a terminal `failed` node that sets `assessment.status = "failed"`. It does not return fabricated scores. Partial results already in state are persisted so the user can retry from the last checkpoint.

## Verification

Graph changes need a `pytest` test that compiles the graph and asserts the node/edge set, plus at least one test driving the graph with a fake LLM (`FakeListChatModel`) through the interrupt and back.
