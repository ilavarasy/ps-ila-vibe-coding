# PRD — Northbeam Guided Growth Insight

**Status:** extracted from the clickable prototype (validation stage, no backend)
**Date:** 21 September 2026

---

## Problem

Growth teams get accurate dashboards and still fail to act on them. Northbeam's current dashboard shows twelve equally weighted metrics with no interpretation and no prioritisation, so the user has to find the signal themselves.

Research evidence from the current experience:

- 60% bounce rate — sessions under 15 seconds with no interaction
- 12 charts on the default landing view, all equally weighted
- 74% of users export to a spreadsheet to actually analyse
- 6 clicks (median ~2m 10s) to reach the metric that explains the week

User voice:

- "I open it, see twelve charts, and have no idea which one I'm supposed to act on. So I close it." — Marketing manager
- "It tells me *what* happened but never *what to do about it*. I still export to a spreadsheet to think." — Product lead
- "My exec just wants one slide. The dashboard gives me forty widgets instead." — PMM

**Hypothesis under test:** automatically surfacing the highest-signal insight, explaining it in plain language, and presenting a single recommended action will cause decision makers to act rather than abandon the dashboard.

**Validated in the prototype:** the journey from data to a recorded decision is achievable in one screen and two clicks, and the guided screen makes the recommended action the single unmistakable next step. What the prototype does *not* yet validate is behaviour change with real users — bounce rate, click rate, time-to-issue and export rate are simulated readings, not measured outcomes. The kill switch states the exit condition explicitly: if users still bounce despite a guided insight and a recommended action, stop optimising the interface and test whether the surfaced metric is the wrong metric.

---

## Users & jobs

**Primary user:** Product Manager or Growth Manager at a B2B SaaS company, reviewing weekly growth performance and accountable for deciding what the team works on next.

**Job to be done:** "When I open my weekly growth review, I want to know the single most important thing that changed and what to do about it, so I can make a decision without exporting to a spreadsheet or defending my reasoning from scratch."

**Secondary users:**
- Executive / exec sponsor — wants one headline and one implication, not forty widgets.
- PMM — needs to repeat the takeaway in plain language to others.

---

## Scope

### In scope

- Guided Overview that surfaces one highest-signal insight, a plain-language takeaway, supporting evidence and one recommended action.
- Four honest analysis states: signal detected, no significant signal, insufficient evidence (correlation without confirmed cause), data error. Plus a brief analysing state on load.
- Investigation screen: onboarding funnel with step-to-step conversion and the abnormal step highlighted, a "What changed?" evidence panel, and a clearly hedged possible-driver note.
- Decision screen: flagged confirmation, compact evidence summary, "Decision recorded — Today", return paths.
- Persistence of the flagged state across navigation within the session.
- Control experience: the existing twelve-metric dashboard with date-range filters and export, kept genuinely credible.
- Section pages: Acquisition, Activation, Retention, Revenue.
- Experiment brief page (reviewers only, outside product navigation): hypothesis, success measures, kill switch, research quotes.

### Out of scope

- Real data pipelines, anomaly detection models, statistical confidence calculation.
- Accounts, authentication, permissions, multi-tenant data.
- Assigning the flagged investigation to a person, due dates, ticket creation, Jira/Linear/Slack integration.
- Notifications, digests, scheduled reports.
- Custom dashboards, custom metric definitions, chart-level drilldowns beyond the onboarding funnel.
- Multiple concurrent insights or an insight inbox/history.
- Mobile-optimised layout (desktop analytics context assumed).

---

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| R1 | Overview surfaces exactly one highest-signal insight with headline, plain-language takeaway and supporting evidence | P0 | Headline reads "Trial conversion dropped 18% this week"; takeaway names traffic up 13% and concentration in mobile / Paid Social; evidence lists mobile activation 57%→38%, Paid Social conversion 13.7%→8.1%, Paid Social traffic 8,900→12,600, desktop activation broadly unchanged. No second competing insight above the fold. |
| R2 | Exactly one recommended action, visually dominant | P0 | "Investigate mobile onboarding" is the only primary button in the insight card, larger and higher-emphasis than every other control on the page; keyboard-focusable with a visible focus ring; navigates to the investigation screen. |
| R3 | Analysis state before any claim | P0 | On first load the insight and metric cards render as skeletons with "Analysing 12 growth signals…" and the comparison sub-text; no metric, insight or action is visible while analysing; resolves within ~1s. |
| R4 | No manufactured insight when no anomaly exists | P0 | No-signal state shows "No significant issues detected this week", the expected-ranges sub-text, an "All 12 growth signals checked ✓" status, and "View all metrics" as the only action. No recommended action rendered. |
| R5 | Correlation is never stated as cause | P0 | Uncertain state shows "Possible driver identified" in the caution colour, "Confidence: Needs investigation", and copy stating there is not enough evidence to confirm the release caused the drop. The string "Release v4.8 caused the decline" appears nowhere. Evidence panel is labelled as observed facts. |
| R6 | Data failure never shows stale insight as current | P0 | Error state shows "We couldn't analyse your growth signals", the sub-text, "Try again" (re-runs analysis) and "View available metrics". No headline metric, takeaway, context cards or secondary signals rendered. |
| R7 | Investigation screen localises the drop | P0 | Title "Mobile onboarding"; headline "Largest drop occurs at Connect your workspace"; funnel shows 4,150 / 100%, 3,620 / 87%, 1,580 / 38%, 1,370 / 33%; the Started onboarding → Connected workspace fall is visually distinguished from the other steps. |
| R8 | Investigation evidence panel with hedged driver | P0 | "What changed?" lists the four findings; the possible-driver note is a separate, visually distinct block using correlation language. Primary "Flag for investigation" → decision screen; secondary "View breakdown" → Activation. |
| R9 | Decision recorded state | P0 | Shows "Investigation flagged", the highest-priority supporting copy, the four-item evidence summary, "Decision recorded — Today", primary "Return to overview", secondary "Continue exploring funnel". |
| R10 | Action completion persists | P0 | After flagging, the Overview replaces the CTA with "Investigation flagged ✓" and "You acted on this week's highest-signal issue." State survives navigation to Acquisition, Activation, Retention, Revenue, the control view and back. |
| R11 | Decision screen cannot be faked | P1 | Reaching the decision screen without having flagged shows a prompt back to the investigation rather than a confirmation. |
| R12 | Control experience remains credible | P1 | Twelve metrics at equal visual weight with real values and deltas, date-range filter (Last 7 days / Last 30 days / Quarter) and an Export button. No interpretation, prioritisation or recommended action. Not degraded to make the treatment look better. |
| R13 | Experiment layer separated from product | P1 | Hypothesis, success measures, kill switch and quotes live only on the Experiment brief page, reached from a clearly labelled "Prototype only" area, never inside Overview / Acquisition / Activation / Retention / Revenue. |
| R14 | Prototype state switch observable but separate | P1 | A control labelled "Prototype state" with the four conditions, visually separated (dashed border, out-of-product styling) and annotated as prototype only. Default is Signal detected. Switching restarts the analysing state. |
| R15 | Kill switch observable | P1 | Experiment page shows the bounce reading against the 40% pivot threshold with a HOLDING / TRIGGERED status and the pivot instruction; a reviewer can toggle the simulated 14-day read and watch it fire. |
| R16 | Consistent visual language and per-screen metadata | P2 | All new screens reuse the existing sidebar, type scale, card, badge and colour tokens; no hardcoded colours. Each route has its own title, description and og tags. |

---

## Data & events

### Dataset (mocked — a fixed fictional week, hardcoded in the prototype)

Twelve metrics, each with current value, previous value, delta, direction, definition and group:

| Metric | This week | Previous | Change |
|---|---|---|---|
| Website visitors | 48,200 | 42,800 | +13% |
| Sign ups | 4,150 | 4,020 | +3% |
| Paid social traffic | 12,600 | 8,900 | +42% |
| Paid social conversion | 8.1% | 13.7% | −41% |
| Trial activation | 51% | 62% | −11 pts |
| Trial to paid conversion | 14.2% | 17.4% | −18% |
| Mobile activation | 38% | 57% | −19 pts |
| Desktop activation | 65% | 66% | −1 pt |
| MRR | $428K | $421K | +1.7% |
| Churn | 3.8% | 3.5% | +0.3 pts |
| NPS | 41 | 44 | −3 |
| Support tickets | 328 | 214 | +53% |

Onboarding funnel: Started trial 4,150 → Started onboarding 3,620 → Connected workspace 1,580 (abnormal) → Reached first value 1,370.

### What is real vs mocked

**Real in the prototype:** all navigation and routing; the four analysis states and the analysing transition; the insight → investigation → decision → recorded journey; persistence of the flagged action across pages within a session; the control dashboard's filters and export interaction; the kill-switch simulator.

**Mocked:** the entire dataset (static, one fixed week); the "analysis" (a ~900ms timer, not detection — which state appears is chosen by the reviewer, not derived from data); sparklines (generated shapes, not series data); date-range filters (the toggle changes selection, not the numbers); Export (records that the user left for a spreadsheet, produces no file); "Decision recorded — Today" (no stored record, no timestamp persisted, no assignee); every success-measure figure on the Experiment page (simulated 14-day reads, two preset outcomes); the user quotes (research inputs written into the prototype, not collected in-product).

**Not present at all:** database, API, authentication, analytics SDK. State lives in memory and resets on reload.

### Events to instrument for the real test

| Event | Properties | Why |
|---|---|---|
| `overview_viewed` | state (signal / no-signal / uncertain / error), analysing_ms | Denominator for bounce and click rate |
| `overview_bounced` | dwell_ms, interacted (false) | Primary success measure — target below 40% from a 60% baseline |
| `recommended_action_clicked` | insight_id, action_label, ms_since_view | Primary success measure — click rate on the one action |
| `investigation_viewed` | source (overview / direct), insight_id | Confirms the drill-down is reached through the insight |
| `investigation_flagged` | insight_id, ms_since_overview_view | Conversion from insight to decision |
| `decision_viewed` | flagged (true / false) | Detects arrivals without a real decision |
| `breakdown_opened` | target_page | Exploration depth beyond the recommended path |
| `time_to_key_issue` | ms, clicks, view (guided / control) | Control baseline 6 clicks / ~2m 10s |
| `export_clicked` | view, range | Export rate, 74% baseline; lower means thinking happens in-product |
| `no_signal_state_shown` | signals_checked | Verifies we are not manufacturing actions |
| `uncertain_state_shown` | insight_id, confidence | Verifies hedged language is being served, not suppressed |
| `analysis_error_shown` | reason, retried | Error rate and recovery |

---

## Open questions

1. **Is trial conversion the right signal to surface?** The kill switch assumes it might not be. If users still bounce, what is the candidate replacement — revenue at risk, or something role-specific?
2. **How is "highest signal" actually determined?** The prototype picks it by hand. Real ranking needs anomaly detection plus a business-impact weighting, and we have not defined either.
3. **What is the confidence threshold** that separates "signal detected" from "insufficient evidence", and who owns that rule?
4. **What does flagging actually do downstream?** Create a ticket, notify a team, appear in a queue? Today it only changes the UI.
5. **One insight or a ranked few?** Executives want one. Growth managers may feel blind if a second real issue is suppressed.
6. **Does the takeaway need to be role-aware?** The same drop reads differently to a PMM and a PM.
7. **How often should the insight change?** Weekly cadence is implied; a stale insight after acting may erode trust.
8. **Who validates the recommendation was right?** There is currently no follow-up loop showing whether the flagged investigation resolved the drop.
9. **How do we avoid the opposite failure** — users trusting a recommendation uncritically because the interface sounds confident?
10. **Do the no-signal weeks hold attention?** If the product says "nothing to do" often, does the user stop opening it at all?
