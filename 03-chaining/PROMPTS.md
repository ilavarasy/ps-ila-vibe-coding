# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: dashboard improvement chain

### Step 1: Expand, build new screens in a strict sequence
```
Build the next phase of the Northbeam Growth Dashboard prototype in a strict sequence.

Use the existing Northbeam visual design system and the attached dashboard design reference as the visual North Star. Keep the current sidebar, typography, spacing, cards, colours and overall B2B SaaS analytics style consistent.

Do not redesign the existing dashboard. Expand the user journey with the following screens/states.

1. SCREEN A — Guided Overview

Use the existing Guided Overview as the starting screen.

The primary insight must remain:

“Trial conversion dropped 18% this week.”

Supporting takeaway:

“Traffic is up 13%, but trial users are converting at a lower rate. The decline is concentrated among mobile users arriving from Paid Social.”

Show supporting evidence:
• Mobile activation: 57% → 38%, down 19 pts
• Paid Social conversion: 13.7% → 8.1%, down 41%
• Paid Social traffic: 8,900 → 12,600, up 42%
• Desktop activation: 66% → 65%, broadly unchanged

Primary CTA:
“Investigate mobile onboarding”

Navigation:
Clicking “Investigate mobile onboarding” must take the user to Screen B.

2. SCREEN B — Investigation / Onboarding Funnel

Create a focused investigation screen matching the visual density and hierarchy of the attached reference.

Page title:
“Mobile onboarding”

Headline:
“Largest drop occurs at Connect your workspace”

Show this funnel:

Started trial — 4,150 — 100%
Started onboarding — 3,620 — 87%
Connected workspace — 1,580 — 38%
Reached first value — 1,370 — 33%

Visually highlight the abnormal drop between “Started onboarding” and “Connected workspace”.

Add an evidence panel:

“What changed?”

• Mobile activation fell from 57% to 38%.
• The decline started after release v4.8.
• 71% of the onboarding drop occurs at the Connect your workspace step.
• Desktop activation remained broadly stable.

Do NOT claim that release v4.8 definitely caused the decline.

Instead show:

“Possible driver: The decline began after release v4.8. This is a correlation and requires investigation before confirming causation.”

Primary CTA:
“Flag for investigation”

Secondary CTA:
“View breakdown”

Clicking “Flag for investigation” must take the user to Screen C.

3. SCREEN C — Decision / Action Completed

Create a clear action-completed state confirming that the user has acted on the insight.

Show:

“Investigation flagged”

Supporting copy:

“Mobile onboarding has been flagged as the highest-priority growth issue for investigation.”

Include a compact evidence summary:

Trial conversion: ↓18%
Mobile activation: ↓19 pts
Paid Social conversion: ↓41%
Potential correlation: Release v4.8

Show:

“Decision recorded”

and timestamp it as:
“Today”

Provide two actions:

Primary:
“Return to overview”

Secondary:
“Continue exploring funnel”

When the user returns to the Guided Overview, replace the original “Investigate mobile onboarding” CTA with a completed state:

“Investigation flagged ✓”

and supporting text:

“You acted on this week's highest-signal issue.”

This final state is important because the prototype must visibly demonstrate that the user moved from insight to action.

The complete journey must therefore be:

DATA → SIGNAL → INSIGHT → INVESTIGATION → DECISION → ACTION RECORDED

Do not add additional product features.

The purpose of these screens is to test whether prioritised insights and clear recommended actions help a Product Manager move from dashboard data to a decision faster than the current 12-chart dashboard.
```

### Step 2: Behavior, hard-code the states
```
Apply the following behavior constraints to the Northbeam Guided Growth Dashboard flow.

The purpose of these states is to make the prototype behave like a credible analytics product and prevent it from always presenting a confident insight when the data does not support one.

1. LOADING / ANALYSIS STATE

When the Guided Overview first loads, do not immediately display the highest-signal insight.

Show a skeleton version of the insight card and supporting metric cards.

Display this exact message:

“Analysing 12 growth signals…”

Supporting text:

“Comparing this week’s performance to previous periods to identify what needs your attention.”

After the loading state, transition to the existing Guided Overview:

“Trial conversion dropped 18% this week.”

Keep the loading state brief in the prototype so reviewers can observe it without slowing down the demo.

2. EMPTY / NO SIGNIFICANT SIGNAL STATE

Create an observable prototype state for when the data contains no meaningful anomaly.

Do not manufacture a recommended action when no significant signal exists.

Display:

“No significant issues detected this week”

Supporting text:

“Your key growth metrics are within their expected ranges. There is nothing that requires immediate action.”

Do not show the “Investigate mobile onboarding” CTA in this state.

Instead show:

“View all metrics”

Also show a subtle positive status:

“All 12 growth signals checked ✓”

3. INSUFFICIENT EVIDENCE / UNCERTAIN STATE

If the dashboard detects a correlation but does not have enough evidence to establish causation, never present the suspected driver as a confirmed cause.

For the current mobile onboarding scenario, display:

“Possible driver identified”

Supporting text:

“Mobile activation declined after release v4.8, but there is not enough evidence to confirm that the release caused the drop.”

Show:

“Confidence: Needs investigation”

Primary CTA:

“Investigate correlation”

Never change this language to:
“Release v4.8 caused the decline.”

The system must clearly distinguish between:
• Observed fact
• Correlation
• Confirmed cause

4. ERROR / DATA FAILURE STATE

If analytics data cannot be loaded, do not show stale insights as if they are current.

Display:

“We couldn’t analyse your growth signals”

Supporting text:

“Some analytics data could not be loaded, so we can’t reliably identify the highest-signal insight.”

Primary CTA:

“Try again”

Secondary CTA:

“View available metrics”

Do not display a recommended business action while the required data is unavailable.

5. ACTION COMPLETION STATE

When the user clicks “Flag for investigation”, persist that state throughout the prototype.

On returning to the Guided Overview, replace:

“Investigate mobile onboarding”

with:

“Investigation flagged ✓”

Supporting text:

“You acted on this week’s highest-signal issue.”

Do not reset the action when navigating between Overview, Activation, Acquisition, Retention or Revenue.

PROTOTYPE CONTROLS

Because this is a validation prototype with no backend, make all four dashboard conditions observable through a small prototype-only state switch:

• Signal detected
• No significant signal
• Insufficient evidence
• Data error

Keep this control visually separate from the real product experience and label it:

“Prototype state”

The normal default state must be “Signal detected”.

Maintain the existing Northbeam design language across all states.

Do not add new product features.

The dashboard must never invent an insight, recommendation or causal explanation when the available data does not support it.
```

### Step 3: Refine, one surgical polish
```
The Northbeam Guided Overview needs one surgical UI refinement.

First, audit the screen and identify the 3 biggest visual gaps in typography and spacing.

Then make ONLY ONE change:

Refine the primary “Investigate mobile onboarding” button so it is visually clear that this is the single recommended next action.

Make the button slightly more prominent through its size, spacing and visual emphasis, while keeping it consistent with the existing Northbeam design system.

Do not change the headline.
Do not change the insight text.
Do not change the evidence cards.
Do not change any other buttons.
Do not change navigation.
Do not change colours elsewhere.
Do not change any content, functionality, states, routes or underlying logic.

Only refine the “Investigate mobile onboarding” CTA.

The purpose of this single refinement is to make the recommended action easier to identify and click, because recommended-action click rate is a key measure of the prototype hypothesis.
```

## Reusable techniques learned

- _____
- _____

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

_____
