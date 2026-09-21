# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

_One paragraph an engineer can read in 60 seconds._

Northbeam is a validation prototype, not a product. It tests one hypothesis: if a growth dashboard leads with one highest-signal insight, a plain-language explanation, and a single recommended action, a Product Manager will act instead of abandoning the dashboard. The app is a TanStack Start (React 19) client-rendered SPA with a thin Supabase (Lovable Cloud) backend wired in for exactly two things — the guided overview's insight/evidence and the recorded decision — and everything else still hardcoded. What is genuinely built and reusable: nine-screen routing, a clean feature-folder architecture (data / state / presentation separated), a context-driven state machine that correctly distinguishes observed fact → correlation → confirmed cause and refuses to invent an insight when the data does not support one, the full journey from insight to a persisted decision (survives reload), four honest analysis states plus the analysing transition, and a credible twelve-chart control as the baseline to beat. What is duct tape: the "analysis" is a ~900 ms timer the reviewer manually flips between four canned states (not detection); the twelve-metric dataset, the funnel, the quotes and the experiment numbers are still static constants; date-range filters change selection, not numbers; Export logs a flag and produces no file. Treat the visual layer and the honest-states logic as the deliverable; treat the data and detection layers as placeholders you replace.

## Architecture (plain language)

- **Frontend:** Framework: TanStack Start v1 (file-based routing) on Vite 8, React 19, fully client-rendered SPA (__root is not ssr: false-gated because there is no auth; pages SSR their markup but the dashboard interactivity is client-side). TypeScript strict. Styling: Tailwind CSS v4 via src/styles.css (@theme tokens, no tailwind.config.js). shadcn/ui component library under src/components/ui/* (Radix primitives). Charts are inline SVG / CSS — recharts is installed but the product screens do not use it. Routing: src/routes/* are thin registrations only — each imports its screen component from src/features/* and defines a head() with per-page metadata. routeTree.gen.ts regenerates automatically. The root (src/routes/__root.tsx) wraps QueryClientProvider → PrototypeProvider → NorthbeamAppShell → <Outlet/> and registers 404 / error boundaries. Feature folders: each screen lives in src/features/<feature>/ with its display component, a sibling *-data.ts of pure constants, and (where it touches the DB) a *.functions.ts server-function module. Routes import components, never data-fetching logic directly.
- **Backend / data:** A **Supabase (Lovable Cloud)** project is live. RLS is enabled on every table; reads use a server-local **publishable-key** client (respects RLS as `anon`), writes go through the same client against `anon`-write policies. The service-role client is intentionally **not** used anywhere yet.  | Table | Holds | Wired to the UI? | |---|---|---| | `growth_metrics` | the twelve fictional metrics | seeded, **not yet wired** (control + section pages still read static `METRICS`) | | `funnel_steps` | the mobile-onboarding funnel, abnormal drop flagged | seeded, **not yet wired** (`/investigate` reads static funnel) | | `insights` + `insight_evidence` + `investigation_findings` | the highest-signal insight, evidence, and findings tagged `observed_fact` / `correlation` / `context` | **wired** — guided overview renders headline, takeaway, evidence from here | | `flagged_investigations` | the recorded decision per session (`session_id` + `insight_slug`, unique) | **wired** — "Flag for investigation" upserts a row; load restores it | | `research_quotes` | the three user quotes | seeded, shown via static copy | | `experiment_measures` | bounce / click / time-to-issue / export baselines + targets | seeded, experiment page reads static copy | | `prototype_events` | append-only behaviour log | write-only wired (an `investigation_flagged` event is logged on flag) |  Server functions live in `src/features/growth-data/growth-data.functions.ts` (`createServerFn`, zod-validated inputs): `getGuidedInsight`, `getFlaggedInvestigation`, `flagInvestigation` (upserts + emits event), and `logPrototypeEvent`. They are consumed via `@tanstack/react-query` options in `growth-queries.ts`. A per-browser `session_id` (stable in `localStorage`, `src/features/prototype/session-id.ts`) scopes the recorded decision; reloading the page re-fetches it so the "Investigation flagged ✓" state persists.
- **Key flows:** 1. **Guided journey (the hypothesis under test):** `/` (Guided Overview) → "Investigate mobile onboarding" CTA → `/investigate` (Mobile onboarding funnel, abnormal drop highlighted, hedged v4.8 driver) → "Flag for investigation" → `/decision` (Action completed) → "Return to overview" → `/` now shows "Investigation flagged ✓" and "You acted on this week's highest-signal issue." The decision is persisted to `flagged_investigations`, so it survives a reload and cannot be reached on `/decision` without a real prior flag (the screen prompts back to the investigation if `investigated` is false).
2. **Four honest overview states** (reviewer-controlled via the prototype state switch, default "Signal detected"): Signal detected, No significant signal, Insufficient evidence, Data error — each with copy that never overclaims. The uncertain state shows "Confidence: Needs investigation" and "Correlation, not a confirmed cause"; the error state suppresses the recommended business action.
3. **Control / current dashboard:** `/classic` — the same twelve metrics, no interpretation, no prioritisation, an Export button. The honest baseline the guided experience is measured against.
4. **Experiment brief:** `/experiment` (outside the product nav) — hypothesis, success measures, the 60% / 74% / 12-charts / 6-clicks baseline evidence, the three user quotes, and the kill switch.

## What's solid vs. what's duct tape

| Area | State | Notes |
|---|---|---|
| Feature-folder architecture. | solid | Routes are thin; each screen's display, data, and (where relevant) server functions live together in src/features/<feature>/. Clear separation of data, state, and presentation. |
| Static twelve-metric dataset + funnel | rough | One hardcoded week; growth_metrics/funnel_steps are seeded but the control & section screens still read the constants |

## Risks & assumptions for the team

The biggest assumption is the hypothesis itself. The prototype demonstrates the journey is possible in one screen and two clicks; it does not prove behaviour change. Bounce rate, action click rate, time-to-issue, and export rate are simulated readings, not measured outcomes — even though the experiment_measures table now holds their baselines/targets.
The "analysis" is not real. Because the reviewer manually selects the state, the honest-states logic is validated only as presentation, not as detection. A real system must derive "no significant signal" and "insufficient evidence" from data — the thresholds for both are undefined (see PRD open questions).
Kill switch is conceptual, not automatic. killSwitchFired is derived from a reviewer-set reading, not from observed behaviour. In production the exit condition must be wired to real bounce data with a defined window.
Partial backend coupling is a footgun. The overview and decision are DB-backed; the control dashboard, funnel, experiment page, and section pages are not. An engineer touching data must remember which surfaces read the DB and which read constants — the static fallback on the overview can also mask a broken query as "still working".
Anonymous sessions, no auth/multi-tenancy. One localStorage id scopes the recorded decision; there is no user, account, or permissions model. Any real version needs per-account scoping.
No mobile layout. The sidebar is desktop-only. Given that "mobile activation" is the story being told, the irony is noted.
recharts is an unused dependency in the product screens — it inflates the bundle for no current benefit. Remove it or commit to it; don't leave it ambiguous.
One hardcoded narrative. The insight is always "Trial conversion dropped 18%." The honest-states logic enforces not overclaiming it only because the reviewer flips the switch; a real detector must stop asserting it when the data no longer supports it.
The PRD's event list is only partially instrumented. investigation_flagged is logged; overview_viewed, recommended_action_clicked, time_to_key_issue, export_clicked, and the no-signal/uncertain/error state exposures are specified but emit nothing today.

## How to run it

```
# Install dependencies
npm install            # or bun install / pnpm install

# Start the dev server (Vite, http://localhost:8080)
npm run dev

# Production build
npm run build

# Development build (used by the Lovable preview / pre-render)
npm run build:dev

# Preview the production build
npm run preview

# Lint / format
npm run lint
npm run format
```
