# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem

_What user problem does this solve? Tie to the validated hypothesis._

Growth teams get accurate dashboards and still fail to act on them. Northbeam's current dashboard shows twelve equally weighted metrics with no interpretation and no prioritisation, so the user has to find the signal themselves.

Research evidence from the current experience:

60% bounce rate — sessions under 15 seconds with no interaction
12 charts on the default landing view, all equally weighted
74% of users export to a spreadsheet to actually analyse
6 clicks (median ~2m 10s) to reach the metric that explains the week
User voice:

"I open it, see twelve charts, and have no idea which one I'm supposed to act on. So I close it." — Marketing manager
"It tells me what happened but never what to do about it. I still export to a spreadsheet to think." — Product lead
"My exec just wants one slide. The dashboard gives me forty widgets instead." — PMM
Hypothesis under test: automatically surfacing the highest-signal insight, explaining it in plain language, and presenting a single recommended action will cause decision makers to act rather than abandon the dashboard.

Validated in the prototype: the journey from data to a recorded decision is achievable in one screen and two clicks, and the guided screen makes the recommended action the single unmistakable next step. What the prototype does not yet validate is behaviour change with real users — bounce rate, click rate, time-to-issue and export rate are simulated readings, not measured outcomes. The kill switch states the exit condition explicitly: if users still bounce despite a guided insight and a recommended action, stop optimising the interface and test whether the surfaced metric is the wrong metric.

## Users & jobs

- **Primary user:** Product Manager or Growth Manager at a B2B SaaS company, reviewing weekly growth performance and accountable for deciding what the team works on next.
- **Job to be done:** When I open my weekly growth review, I want to know the single most important thing that changed and what to do about it, so I can make a decision without exporting to a spreadsheet or defending my reasoning from scratch.

## Scope

- **In:** Guided Overview that surfaces one highest-signal insight, a plain-language takeaway, supporting evidence and one recommended action.
Four honest analysis states: signal detected, no significant signal, insufficient evidence (correlation without confirmed cause), data error. Plus a brief analysing state on load.
Investigation screen: onboarding funnel with step-to-step conversion and the abnormal step highlighted, a "What changed?" evidence panel, and a clearly hedged possible-driver note.
Decision screen: flagged confirmation, compact evidence summary, "Decision recorded — Today", return paths.
Persistence of the flagged state across navigation within the session.
Control experience: the existing twelve-metric dashboard with date-range filters and export, kept genuinely credible.
Section pages: Acquisition, Activation, Retention, Revenue.
Experiment brief page (reviewers only, outside product navigation): hypothesis, success measures, kill switch, research quotes.
- **Out (explicitly):** Real data pipelines, anomaly detection models, statistical confidence calculation.
Accounts, authentication, permissions, multi-tenant data.
Assigning the flagged investigation to a person, due dates, ticket creation, Jira/Linear/Slack integration.
Notifications, digests, scheduled reports.
Custom dashboards, custom metric definitions, chart-level drilldowns beyond the onboarding funnel.
Multiple concurrent insights or an insight inbox/history.
Mobile-optimised layout (desktop analytics context assumed).

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| 1 | Overview surfaces exactly one highest-signal insight with headline, plain-language takeaway and supporting evidence | Must | Headline reads "Trial conversion dropped 18% this week"; takeaway names traffic up 13% and concentration in mobile / Paid Social; evidence lists mobile activation 57%→38%, Paid Social conversion 13.7%→8.1%, Paid Social traffic 8,900→12,600, desktop activation broadly unchanged. No second competing insight above the fold. |
| 2 | Exactly one recommended action, visually dominant | Must | "Investigate mobile onboarding" is the only primary button in the insight card, larger and higher-emphasis than every other control on the page; keyboard-focusable with a visible focus ring; navigates to the investigation screen. |

## Data & events

_What gets stored, what gets tracked._

Real in the prototype: all navigation and routing; the four analysis states and the analysing transition; the insight → investigation → decision → recorded journey; persistence of the flagged action across pages within a session; the control dashboard's filters and export interaction; the kill-switch simulator.

Mocked: the entire dataset (static, one fixed week); the "analysis" (a ~900ms timer, not detection — which state appears is chosen by the reviewer, not derived from data); sparklines (generated shapes, not series data); date-range filters (the toggle changes selection, not the numbers); Export (records that the user left for a spreadsheet, produces no file); "Decision recorded — Today" (no stored record, no timestamp persisted, no assignee); every success-measure figure on the Experiment page (simulated 14-day reads, two preset outcomes); the user quotes (research inputs written into the prototype, not collected in-product).

Not present at all: database, API, authentication, analytics SDK. State lives in memory and resets on reload.

## Open questions

Is trial conversion the right signal to surface? The kill switch assumes it might not be. If users still bounce, what is the candidate replacement — revenue at risk, or something role-specific?
How is "highest signal" actually determined? The prototype picks it by hand. Real ranking needs anomaly detection plus a business-impact weighting, and we have not defined either.
What is the confidence threshold that separates "signal detected" from "insufficient evidence", and who owns that rule?
What does flagging actually do downstream? Create a ticket, notify a team, appear in a queue? Today it only changes the UI.
One insight or a ranked few? Executives want one. Growth managers may feel blind if a second real issue is suppressed.
Does the takeaway need to be role-aware? The same drop reads differently to a PMM and a PM.
How often should the insight change? Weekly cadence is implied; a stale insight after acting may erode trust.
Who validates the recommendation was right? There is currently no follow-up loop showing whether the flagged investigation resolved the drop.
How do we avoid the opposite failure — users trusting a recommendation uncritically because the interface sounds confident?
Do the no-signal weeks hold attention? If the product says "nothing to do" often, does the user stop opening it at all?
