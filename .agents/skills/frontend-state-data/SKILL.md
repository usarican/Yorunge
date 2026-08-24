---
name: frontend-state-data
description: "Client state and server data in the Yörünge frontend — Zustand stores (onboardingStore: step, repos, quiz answers, telemetry), TanStack Query v5 (queries, mutations, query keys, optimistic updates, invalidation), and the lib/api.ts fetch wrapper with JWT auth. Use when adding state, fetching or mutating backend data, or debugging stale/duplicated state."
---

# Client State & Server Data

## The split — decide this first

| Kind of state | Where it lives |
| :-- | :-- |
| Wizard step, quiz answers in progress, selected repos, open modal, telemetry log buffer | **Zustand** |
| Assessment, roadmap, tasks, PR status, knowledge answers — anything the backend owns | **TanStack Query** |
| Derived values | Computed at render. Never a third copy. |

Copying server data into Zustand ("so it's easier to access") is the bug that produces two sources of truth and stale UI. If it came from an API, it stays in the query cache.

## Zustand

```ts
export const useOnboardingStore = create<OnboardingState>()((set) => ({
  currentStep: 1,
  telemetryLogs: [],
  setStep: (step) => set({ currentStep: step }),
  appendTelemetryLog: (log, progress) =>
    set((s) => ({ telemetryLogs: [...s.telemetryLogs, log], telemetryProgress: progress })),
}));
```

- **Always select narrowly**: `useOnboardingStore((s) => s.currentStep)`. Subscribing to the whole store re-renders every step component on each telemetry line — this is the #1 perf mistake in this app.
- Actions live in the store, not scattered in components. Components call `setQuizAnswer`, they don't `set()` raw state.
- Store shape mirrors `OnboardingState` in the architecture doc; extend it there too when you add a field.
- Persist only what's safe to restore (step, answers) via `persist` with a versioned name; never persist tokens or telemetry buffers.
- Non-reactive reads inside callbacks: `useOnboardingStore.getState()`.

## TanStack Query v5

```ts
export const qk = {
  assessment: (id: string) => ["assessment", id] as const,
  roadmap: (userId: string) => ["roadmap", userId] as const,
  task: (id: string) => ["task", id] as const,
};

export function useAssessment(id: string) {
  return useQuery({ queryKey: qk.assessment(id), queryFn: () => api.get<Assessment>(`/assessments/${id}`) });
}
```

- Query keys come from a single `qk` factory — hand-written key arrays drift and break invalidation.
- v5 object syntax only (`useQuery({ queryKey, queryFn })`); `isPending` not `isLoading`, `gcTime` not `cacheTime`, no `onSuccess` on queries (do it in the mutation or in a `useEffect` on data).
- Mutations invalidate what they changed: `onSettled: () => qc.invalidateQueries({ queryKey: qk.roadmap(userId) })`.
- Optimistic updates (task status toggles) follow cancel → snapshot → set → rollback in `onError` → invalidate in `onSettled`. Skipping the snapshot means a failed request leaves a lie on screen.
- Long-running server work (AST analysis) is **not** polled — it streams over SSE (`realtime-streaming`). When the stream completes, invalidate the assessment query rather than trusting the streamed payload as cache truth.
- `staleTime` is set deliberately per query: roadmap/report data is stable (minutes), task/PR status is short-lived (seconds).
- The `QueryClientProvider` lives in a single client component mounted from the root layout.

## `lib/api.ts`

One wrapper for every backend call:

- Attaches the JWT, sets base URL from `NEXT_PUBLIC_API_URL`, and parses FastAPI's error shape into a typed `ApiError` with a Turkish message.
- Returns typed data (`api.get<T>`); no `any` escapes into components.
- On 401 it clears the session and redirects to login — handled once here, not in each hook.
- Components never call `fetch` directly.

## Verification

`npm run lint && npx tsc --noEmit && npm run build`. For state changes, check the React Query devtools for duplicated keys and the profiler for re-render storms during Step 2 telemetry.
