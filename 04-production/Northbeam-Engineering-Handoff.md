# Northbeam Guided Growth Dashboard — Engineering Handoff

## 60-second read

Northbeam is a **validation prototype**, not a product. It tests one hypothesis: if a growth dashboard leads with one highest-signal insight, a plain-language explanation, and a single recommended action, a Product Manager will act instead of abandoning the dashboard. The app is a TanStack Start (React 19) client-rendered SPA with a **thin Supabase (Lovable Cloud) backend** wired in for exactly two things — the guided overview's insight/evidence and the recorded decision — and everything else still hardcoded. What is genuinely built and reusable: nine-screen routing, a clean feature-folder architecture (data / state / presentation separated), a context-driven state machine that correctly distinguishes *observed fact → correlation → confirmed cause* and refuses to invent an insight when the data does not support one, the full journey from insight to a **persisted** decision (survives reload), four honest analysis states plus the analysing transition, and a credible twelve-chart control as the baseline to beat. What is duct tape: the "analysis" is a ~900 ms timer the reviewer manually flips between four canned states (not detection); the twelve-metric dataset, the funnel, the quotes and the experiment numbers are still static constants; date-range filters change selection, not numbers; Export logs a flag and produces no file. Treat the visual layer and the honest-states logic as the deliverable; treat the data and detection layers as placeholders you replace.

---

## Architecture in plain language

### Frontend

- **Framework:** TanStack Start v1 (file-based routing) on Vite 8, React 19, fully client-rendered SPA (`__root` is not `ssr: false`-gated because there is no auth; pages SSR their markup but the dashboard interactivity is client-side). TypeScript strict.
- **Styling:** Tailwind CSS v4 via `src/styles.css` (`@theme` tokens, no `tailwind.config.js`). shadcn/ui component library under `src/components/ui/*` (Radix primitives). Charts are **inline SVG / CSS** — `recharts` is installed but the product screens do not use it.
- **Routing:** `src/routes/*` are thin registrations only — each imports its screen component from `src/features/*` and defines a `head()` with per-page metadata. `routeTree.gen.ts` regenerates automatically. The root (`src/routes/__root.tsx`) wraps `QueryClientProvider → PrototypeProvider → NorthbeamAppShell → <Outlet/>` and registers 404 / error boundaries.
- **Feature folders:** each screen lives in `src/features/<feature>/` with its display component, a sibling `*-data.ts` of pure constants, and (where it touches the DB) a `*.functions.ts` server-function module. Routes import components, never data-fetching logic directly.

### Backend / data

A **Supabase (Lovable Cloud)** project is live. RLS is enabled on every table; reads use a server-local **publishable-key** client (respects RLS as `anon`), writes go through the same client against `anon`-write policies. The service-role client is intentionally **not** used anywhere yet.

| Table | Holds | Wired to the UI? |
|---|---|---|
| `growth_metrics` | the twelve fictional metrics | seeded, **not yet wired** (control + section pages still read static `METRICS`) |
| `funnel_steps` | the mobile-onboarding funnel, abnormal drop flagged | seeded, **not yet wired** (`/investigate` reads static funnel) |
| `insights` + `insight_evidence` + `investigation_findings` | the highest-signal insight, evidence, and findings tagged `observed_fact` / `correlation` / `context` | **wired** — guided overview renders headline, takeaway, evidence from here |
| `flagged_investigations` | the recorded decision per session (`session_id` + `insight_slug`, unique) | **wired** — "Flag for investigation" upserts a row; load restores it |
| `research_quotes` | the three user quotes | seeded, shown via static copy |
| `experiment_measures` | bounce / click / time-to-issue / export baselines + targets | seeded, experiment page reads static copy |
| `prototype_events` | append-only behaviour log | write-only wired (an `investigation_flagged` event is logged on flag) |

Server functions live in `src/features/growth-data/growth-data.functions.ts` (`createServerFn`, zod-validated inputs): `getGuidedInsight`, `getFlaggedInvestigation`, `flagInvestigation` (upserts + emits event), and `logPrototypeEvent`. They are consumed via `@tanstack/react-query` options in `growth-queries.ts`. A per-browser `session_id` (stable in `localStorage`, `src/features/prototype/session-id.ts`) scopes the recorded decision; reloading the page re-fetches it so the "Investigation flagged ✓" state persists.

### Key flows

1. **Guided journey (the hypothesis under test):** `/` (Guided Overview) → "Investigate mobile onboarding" CTA → `/investigate` (Mobile onboarding funnel, abnormal drop highlighted, hedged v4.8 driver) → "Flag for investigation" → `/decision` (Action completed) → "Return to overview" → `/` now shows "Investigation flagged ✓" and "You acted on this week's highest-signal issue." The decision is persisted to `flagged_investigations`, so it survives a reload and cannot be reached on `/decision` without a real prior flag (the screen prompts back to the investigation if `investigated` is false).
2. **Four honest overview states** (reviewer-controlled via the prototype state switch, default "Signal detected"): Signal detected, No significant signal, Insufficient evidence, Data error — each with copy that never overclaims. The uncertain state shows "Confidence: Needs investigation" and "Correlation, not a confirmed cause"; the error state suppresses the recommended business action.
3. **Control / current dashboard:** `/classic` — the same twelve metrics, no interpretation, no prioritisation, an Export button. The honest baseline the guided experience is measured against.
4. **Experiment brief:** `/experiment` (outside the product nav) — hypothesis, success measures, the 60% / 74% / 12-charts / 6-clicks baseline evidence, the three user quotes, and the kill switch.

### State ownership

`src/features/prototype/PrototypeStateProvider.tsx` is the single client-side source of truth. It exposes: `investigated` / `markInvestigated` (now a DB write via `flagInvestigation`), `flaggedAt`, `exported` / `markExported`, `reading` / `setReading`, `dashboardState` / `setDashboardState`, `analysing` / `reanalyse`, and the derived `killSwitchFired`. On mount it fetches any existing flagged decision for this session. The context is **pinned to `globalThis.__prototypeCtx`** so Fast Refresh reuses the same context instance. Pure data modules (`growth-data.ts`, `experiment-data.ts`, and per-feature `*-data.ts`) hold constants only.

---

## What's solid vs. what's duct tape

### Solid (carry this forward)

- **Feature-folder architecture.** Routes are thin; each screen's display, data, and (where relevant) server functions live together in `src/features/<feature>/`. Clear separation of data, state, and presentation.
- **The honest-states logic.** The state machine that refuses to invent an insight, distinguishes correlation from cause, and hides the recommended action during a data failure is the conceptual core. This is the part worth preserving.
- **The guided journey, now persisted.** DATA → SIGNAL → INSIGHT → INVESTIGATION → DECISION → ACTION RECORDED is fully wired, and the decision survives a reload because it lives in `flagged_investigations`, not memory.
- **DB-backed insight + decision.** The headline, takeaway, and evidence render from `insights` / `insight_evidence` (with safe static fallback while loading); wording can be changed in the database without touching the design.
- **Routing + metadata.** Nine routes, each with its own `head()`, 404 and error boundaries on the root.
- **Inline-SVG charts.** Lightweight, no chart-library coupling — easy to rewire to real data later.
- **Design system consistency.** One semantic token set, Tailwind v4 `@theme`, shadcn primitives. The "Investigate mobile onboarding" CTA is intentionally the single most visually dominant action.

### Duct tape (replace before production)

| Duct tape | Reality | What it would take |
|---|---|---|
| `dashboardState` is reviewer-chosen | "Analysis" = a 900 ms timer + a manual switch | Real anomaly detection deriving state from data |
| Static twelve-metric dataset + funnel | One hardcoded week; `growth_metrics`/`funnel_steps` are seeded but the control & section screens still read the constants | Wire the control dashboard and section pages to `growth_metrics`; feed a real metrics pipeline into that table |
| Investigate screen reads static funnel | `/investigate` ignores `funnel_steps` | Wire it to `funnel_steps` (the abnormal-drop flag already lives there) |
| Experiment success numbers | Simulated 14-day reads, two preset outcomes | Live experiment instrumentation |
| Date-range filter | Changes selection, not numbers | Real time-series keyed by range |
| Export button | Sets an `exported` flag, no file | Real export + the event remains a valid signal |
| "Decision recorded — Today" | Timestamp is real (from `flagged_at`), but no assignee, priority, or downstream routing | Persisted investigation record with ownership + lifecycle |
| Session id in `localStorage` | Anonymous, per-browser; no real identity | Auth-backed user / account scoping |
| User quotes | Written into the code / seeded | Collected in-product |
| Bounce / click metrics | Not measured anywhere | Analytics SDK + the event list in the PRD |
| `prototype_events` | Write-only; nothing reads it back | Reporting on the hypothesis measures |

---

## Risks and assumptions

1. **The biggest assumption is the hypothesis itself.** The prototype demonstrates the journey is *possible* in one screen and two clicks; it does not prove behaviour change. Bounce rate, action click rate, time-to-issue, and export rate are simulated readings, not measured outcomes — even though the `experiment_measures` table now holds their baselines/targets.
2. **The "analysis" is not real.** Because the reviewer manually selects the state, the honest-states logic is validated only as *presentation*, not as *detection*. A real system must derive "no significant signal" and "insufficient evidence" from data — the thresholds for both are undefined (see PRD open questions).
3. **Kill switch is conceptual, not automatic.** `killSwitchFired` is derived from a reviewer-set reading, not from observed behaviour. In production the exit condition must be wired to real bounce data with a defined window.
4. **Partial backend coupling is a footgun.** The overview and decision are DB-backed; the control dashboard, funnel, experiment page, and section pages are not. An engineer touching data must remember which surfaces read the DB and which read constants — the static fallback on the overview can also mask a broken query as "still working".
5. **Anonymous sessions, no auth/multi-tenancy.** One `localStorage` id scopes the recorded decision; there is no user, account, or permissions model. Any real version needs per-account scoping.
6. **No mobile layout.** The sidebar is desktop-only. Given that "mobile activation" is the story being told, the irony is noted.
7. **`recharts` is an unused dependency** in the product screens — it inflates the bundle for no current benefit. Remove it or commit to it; don't leave it ambiguous.
8. **One hardcoded narrative.** The insight is always "Trial conversion dropped 18%." The honest-states logic enforces not overclaiming it only because the reviewer flips the switch; a real detector must stop asserting it when the data no longer supports it.
9. **The PRD's event list is only partially instrumented.** `investigation_flagged` is logged; `overview_viewed`, `recommended_action_clicked`, `time_to_key_issue`, `export_clicked`, and the no-signal/uncertain/error state exposures are specified but emit nothing today.

---

## How to run it

```bash
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

The backend is a managed Supabase project (Lovable Cloud). No local database to run — the publishable key and URL are already configured as environment variables. RLS is on; the overview and decision read/write the live tables, so flagging a decision will actually persist.

**To walk the prototype:**

1. Open `/` — after ~900 ms the Guided Overview (Signal detected) shows the insight and the "Investigate mobile onboarding" CTA. Headline, takeaway, and evidence are read from the `insights` table.
2. Click the CTA → `/investigate` (mobile onboarding funnel, abnormal drop highlighted, hedged v4.8 driver).
3. Click "Flag for investigation" → `/decision` (action completed; the decision is written to `flagged_investigations` and "Decision recorded — Today · HH:MM" shows the real timestamp).
4. Click "Return to overview" → `/` now shows "Investigation flagged ✓". Reload the page — the flagged state persists.
5. Use the **Prototype state** switch (overview only) to observe No significant signal, Insufficient evidence, and Data error.
6. Open `/classic` for the credible twelve-chart control. Try the Export button (it logs "left for a spreadsheet").
7. Open `/experiment` for the hypothesis, success measures, baseline evidence, and kill switch.

**Tech versions:** Node 20+, TanStack Start 1.168 / Router 1.170 / React Query 5.101 / React 19.2 / Vite 8 / Tailwind v4 / Supabase JS 2.116. TypeScript strict mode.
