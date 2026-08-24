---
name: nextjs-app-router
description: "Yörünge frontend structure for Next.js 14+ App Router — route groups (onboarding) vs (dashboard), layout isolation (no sidebar in the wizard), Server vs Client Components, the 5-step stepper controller, loading/error boundaries, BFF routes under app/api, metadata, and Tailwind + Daylight Observatory tokens. Use when creating or modifying anything under frontend/src/app/ or frontend/src/components/."
---

# Next.js App Router Structure (Yörünge)

Spec: `docs/architecture/frontend-architecture.md`. Design tokens: `docs/design-system/DESIGN.md` (use the `ui-ux-pro-max` / `design-system` skills for visual decisions; this skill is about structure).

## Layout isolation — the one rule that is never bent

| Route group | Layout | Sidebar |
| :-- | :-- | :-- |
| `(onboarding)` | Daylight Stepper Header only | **NO sidebar. Ever.** |
| `(dashboard)` | Sidebar + Topbar shell | **Sidebar required** |

`(onboarding)/layout.tsx` must not import the sidebar, and no dashboard chrome may leak in via a shared wrapper. If a component is needed in both, it goes in `components/shared/` and receives its chrome from the layout, not the other way round.

Route groups `(...)` do not appear in the URL — they exist purely for this split.

## Server vs Client Components

Default to Server Components. Add `"use client"` only when the file needs one of: hooks (`useState`/`useEffect`/Zustand/React Query), event handlers, browser APIs (`EventSource`, `File`), or Framer Motion.

Push the boundary down: a page stays a Server Component and renders a small client leaf. Marking `page.tsx` as a client component to fix one button ships the whole subtree to the browser.

- No `async`/`await` data fetching inside a client component — fetch on the server or through React Query.
- Never import server-only modules (API secrets, node libs) into a `"use client"` file; use `import "server-only"` as a guard on those modules.

## The onboarding stepper

`(onboarding)/page.tsx` is the step controller; the five step components live in `components/onboarding/Step1Connect.tsx … Step5Report.tsx`. Rules:

- Current step lives in the Zustand store (`useOnboardingStore`), not in local state — Step 2's SSE handler advances it to Step 3.
- A step component owns only its own UI and validation; navigation is the controller's job.
- Forward navigation is gated on the step's completion predicate. Back navigation is free and must not discard answers already given.
- On mount, rehydrate from the server assessment so a refresh mid-wizard does not restart the flow.
- Mobile: stepper header collapses to number + percentage only.

## Files that belong in every route segment

- `loading.tsx` — a skeleton in the Daylight surface style, not a spinner-only screen.
- `error.tsx` — client component with a `reset()` retry and a Turkish message. Never let a failed assessment fetch render an empty page.
- `not-found.tsx` for dynamic segments like `tasks/[id]`.

## BFF routes (`app/api/`)

Only for: proxying a call that needs a server-held secret, webhook receivers, and same-origin SSE proxying to avoid CORS. Business logic stays in FastAPI. A Next.js route handler that queries a database or calls an LLM is in the wrong repo half.

## Conventions

- Components `PascalCase.tsx`, hooks `useThing.ts`, one component per file.
- No `any`, no non-null `!` to silence the compiler. Shared types come from `src/types/` (see `api-contract-sync`).
- `cn()` (clsx + tailwind-merge) for conditional classes; no string concatenation of Tailwind classes, and no arbitrary hex values — use the theme tokens.
- Images through `next/image`; fonts through `next/font` (`Inter` for UI, `Space Mono` for telemetry/code).

## Accessibility (non-optional here)

Quiz answers reachable and selectable by keyboard, `Esc` closes modals, focus is trapped in modals and restored on close, focus rings visible against `#0B0F19`, and the telemetry log region is `aria-live="polite"`.

## Verification

```bash
cd frontend && npm run lint && npx tsc --noEmit && npm run build
```
