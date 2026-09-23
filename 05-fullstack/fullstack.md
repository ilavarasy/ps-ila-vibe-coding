# Full-Stack: Data, Access Rules, Edge Cases, Deploy

> Module 5 · Full-Stack. Add data schemas, access rules, and edge cases; stress-test and deploy.

## Deployed link

_The working, shareable link that survives real users._

https://action-first-dash.lovable.app

## Data schema

| Entity | Key fields | Notes |
|---|---|---|
| Profiles | id | one row per signed-in person: display name, role (PM, Growth, Exec, PMM), created date. |
| decision_events | id | an audit trail of what a person did on a decision: flagged, reopened, closed, with a note. |

## Access rules

_Who can see / do what? Where are the auth boundaries?_

Reference content — growth metrics, readings, periods, funnel steps, insights, evidence, findings, quotes, experiment measures — stays readable by everyone, writable by nobody from the app.

profiles — a person can read and edit only their own.

flagged_investigations — a signed-in person reads and writes only rows they own; anonymous prototype sessions keep working, scoped to their own session id, with no ability to read anyone else's.

decision_events and saved_exports — readable and insertable only by the person they belong to; never deletable from the app.

prototype_events — insert only; reads limited to the owner. No open read access to the whole event log.

Every new table gets explicit table-level permissions in the same change, otherwise the app cannot reach it

## Edge cases hardened

| Case | Before | After |
|---|---|---|
| Empty / first-run state | Partial data (metrics load, funnel doesn't) | show what loaded, disable the action that depends on what didn't. |
| Bad / malicious input | visitors who aren't signed in can read or change them or add records under someone else's session | recorded decisions, exports and activity are now tied to a signed-in person only — visitors who aren't signed in can no longer read or change them or add records under someone else's session — and the special database function that could be called by anyone was removed. |
| Failure / offline | Any required read fails | the failure screen, never a stale insight presented as current. |

## Stress test results

_What you threw at it, and what held / broke._

Spam Click. I got the error message saying "For security purposes, you can only request this after 59 seconds."
