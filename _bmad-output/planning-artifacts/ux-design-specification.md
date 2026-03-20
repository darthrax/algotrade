---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
inputDocuments:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/product-brief-algotrade-2026-03-20.md
  - docs/AlgoTrade_India_Product_Brief_v1.1.md
---

# UX Design Specification {{project_name}}

**Author:** {{user_name}}
**Date:** {{date}}

---

<!-- UX design content will be appended sequentially through collaborative workflow steps -->

## Executive Summary

### Project Vision
A trust-first intraday trading experience for a single technical operator: a governed, risk-gated system where decisions are transparent (“act vs block”) and degraded behavior is an explicit contract (not a guess). The operator should feel, in seconds, what the system knew, what risk allowed, what the broker did, and why it refused—so paper-to-live progression stays emotionally and operationally safe.

### Target Users
Primary: one solo technical operator (“builder-trader”) running a dedicated machine during market hours. Intermediate skill level; comfortable with local services/logs; needs clear operational status, decision reasons, and evidence (reconciliation, audit trails, model/rule lineage) more than exploratory analytics.

Secondary: “downstream consumers” of exports (e.g., CA/accounting workflows). They are not in-product decision-makers, so the product should produce export-ready artifacts with the same traceability guarantees.

### Key Design Challenges
1) Make the order/risk lifecycle legible: the UI must explain outcomes with stable reason codes and traceable evidence (input freshness -> data mode -> risk gate -> resulting order state).
2) Degraded-mode semantics must be unmistakable: when the system enters `REST_POLL` / monitor-only behavior, the operator must clearly understand that “entries are suppressed by policy” and what is still actively managed (stops/exits).
3) Reduce attention load during stress: the console must support fast “next action” comprehension (kill switch drill confirmation, reconciliation checklist prompts, governance lockouts) without requiring forensic browsing.
4) Operationalize reconciliation confidence: “Did broker match us?” becomes a checklist-style pass/fail with explicit next steps (block next-day live vs waived-with-audit-note).

### Design Opportunities
1) “Trust rituals” that compress complexity into predictable moments: morning brief overview + during-session state cards + end-of-day reconciliation close-out.
2) Act/Block decision cards that always show the same causal chain: freshness status, data source mode, feature/signal snapshot, risk gate verdict, then broker/order state.
3) Governance UX that feels protective, not bureaucratic: explicit lock/disabled controls during market hours, plus calm review/approve/deny flow outside market hours with clear audit context.
4) Explainability that’s purposeful: SHAP/top-features surfacing framed as sanity checks tied to sampled decisions, not an invitation to override the risk gateway.

## Core User Experience

### Defining Experience
The operator’s core loop is: **glance → understand trust state → decide whether to act**.
In practice, this means the UI continuously answers three questions without requiring the operator to interpret logs or guess semantics:
1) **Are we in a normal trading contract or a degraded/locked contract?**
2) **If the system is trading, what decisions did it make and why (act vs block)?**
3) **If something is off, what is the next safe action (ack, wait, reconcile, or execute the kill switch with confirmation)?**

This keeps the operator’s attention on governed outcomes rather than internal complexity.

### Platform Strategy
Provide a **local operator console** (desktop-browser experience via Streamlit) that is optimized for:
- **Fast comprehension** in under ~10 seconds during market hours
- **Single-screen readability** with clear modes, reason codes, and actionable next steps
- **Keyboard/mouse efficiency** for an intermediate technical operator
- **Progressive disclosure**: summary first, details on demand

Pair it with **Telegram notifications** for out-of-band “must-not-miss” events:
- mode changes (e.g., WebSocket vs `REST_POLL`)
- kill switch events and confirmations
- reconciliation failure summaries
- training/retraining status and failures
- dead-man’s-switch alerts

### Effortless Interactions
The UI should make these actions nearly automatic:
- **Default dashboards that stay honest**: prominent mode/state labels that directly reflect the system contract
- **Act/Block decision cards** that always show the same causal chain: freshness/data mode → feature/signal snapshot → risk verdict → broker/order outcome (or explicit suppression)
- **Reason codes with “what this means”**: each block/reject explanation should include an operator-facing interpretation and next step
- **One-click acknowledgements where safe** (e.g., acknowledging alerts) while keeping risk-increasing actions gated
- **Reconciliation readiness cues**: clear “pass/failed/needs waiver” signals so EOD is a routine checklist, not an investigation
- **Kill switch ergonomics**: visible only as “dangerous but present,” with a deliberate two-step confirmation flow

### Critical Success Moments
1) **Morning briefing (first glance success)**: the operator instantly understands watchlist readiness, mode status, and whether entries are allowed yet.
2) **Observation-only window (09:15–09:30)**: the UI must make “no new entries by policy” feel intentional and obvious.
3) **Degraded mode transition (WebSocket drop → `REST_POLL`)**: within moments, the operator understands what is still actively managed vs what is intentionally suppressed.
4) **Risk gate rejection moment**: the operator sees the stable reason and knows whether anything changes next time or today requires intervention.
5) **EOD reconciliation**: the operator gets a clear “reconciled” outcome or a blocked-next-day outcome with a constrained path to clear it.

### Experience Principles
- **Contract-first semantics**: the UI mirrors system truth (including suppression-by-policy), never approximate it.
- **Attention compression**: summaries and reason codes are designed for fast “next action” understanding.
- **Safety through constrained affordances**: dangerous actions require deliberate user confirmation and explicit audit context.
- **Traceability by default**: decision context shown in the UI is backed by the same evidence the operator can reconcile later.
- **Progressive disclosure, not cognitive load**: reveal detail only when the operator asks for it.

## Desired Emotional Response

### Primary Emotional Goals
- **Calm control**: the operator feels steady and oriented, even when the system is degraded or locked.
- **Trust without blind faith**: “I can believe this because it’s explainable and reconcilable,” not “I hope it’s fine.”
- **Instant clarity**: the operator experiences quick comprehension of mode + act/block outcomes + next action.
- **Preparedness**: when something goes wrong (e.g., WebSocket loss), the operator feels supported by an explicit, predictable contract.
- **Closure and confidence at day end**: reconciliation feels like a checklist that resolves, not a mystery to investigate.

### Emotional Journey Mapping
- **First discovery / start-of-day**: curiosity becomes reassurance as the morning overview quickly communicates watchlist readiness and whether entries are allowed yet.
- **During the core loop (market hours)**: focused attentiveness—operator feels “in command” because reason codes and next actions reduce uncertainty.
- **Observation-only / locked windows**: tension is replaced with relief when the UI makes “no new entries by policy” intentional and unambiguous.
- **Degraded mode transition (`WEBSOCKET` → `REST_POLL`)**: anxiety is dampened by explicit mode labels and a clear contract: monitor/maintain positions, suppress new entries.
- **Risk gate rejections**: skepticism converts to confidence when the block reason is stable and paired with “what would change” expectations.
- **Governance moments (config/model approvals)**: seriousness without fear—controls feel locked when required and review feels safe when allowed.
- **EOD reconciliation**: satisfaction comes from evidence-based resolution (pass/failed/waived with explicit next steps).

### Micro-Emotions
- Confidence instead of confusion (fast “what state are we in?”)
- Trust instead of skepticism (reason codes tied to evidence)
- Relief instead of anxiety (degraded-mode semantics are explicit)
- Accomplishment instead of frustration (next actions are clear and constrained)
- Satisfaction (and occasional delight) from “boring reliability” that consistently delivers clarity

### Design Implications
- Always-visible **contract-first status** (normal vs degraded vs locked) with explicit meaning for each.
- **Act/Block decision cards** that present a consistent causal chain and never leave the operator guessing whether silence means success.
- **Reason codes + operator interpretation + next step** on every reject/block.
- **Reconciliation as a workflow** (pass/fail/needs waiver) rather than a buried dashboard detail.
- **Kill switch ergonomics**: prominent but not frantic; two-step confirmation reinforces safety without panic.
- **Governance lock UX**: disabled controls + visible “why locked” tied to market-hours policy.

### Emotional Design Principles
- **Truthfulness over aesthetics**: the UI must mirror system semantics precisely (including suppression-by-policy), never approximate it.
- **Attention compression**: show what the operator needs now; hide forensic detail unless asked.
- **Guided next actions**: every failure mode ends with “here’s what to do next,” not just an explanation.
- **Evidence-backed reassurance**: explanations should map to logs/reconciliation so trust is earned.
- **Constrained affordances for dangerous actions**: deliberate friction only where risk increases.

## UX Pattern Analysis & Inspiration

### Inspiring Products Analysis

**Grafana**
- Panel-based dashboard consistency helps operators stay oriented without hunting.
- Threshold accents communicate “what matters now” instantly.
- Alerting visuals stay consistent with dashboard semantics, reducing ambiguity during stress.
- Drill-down preserves mental map (you can go from “something is wrong” to “why”).

**Datadog**
- Event-driven incident UX: severity + narrative + actionable next steps.
- Timelines/correlation reduce “confused investigation” moments.
- Alerts are built to answer “what changed?” and often point to a path forward (ack/runbook/triage).

**TradingView**
- Chart annotations turn time-series into explainable moments.
- Strong progressive disclosure: scan quickly, then inspect details in-place.

### Transferable UX Patterns

**Navigation / hierarchy**
- A contract-first “state header” (Normal vs Degraded `REST_POLL` vs Locked) that remains in a fixed location and color language.
- Airy, grid-stable layout: essential decision context stays in the same positions across screens to reduce cognitive load.

**Interaction**
- Act/Block decision cards styled like alert severities: consistent accents and always ending in a next action.
- Decision trace view (drill-down) that shows the causal chain: data freshness/data mode -> feature/signal snapshot -> risk verdict -> broker/order outcome (or explicit suppression).

**Visual language (dark + accents + airy)**
- Use accent colors consistently for mode, freshness health, risk verdict, and reconciliation status (with legend/labels, not color-only meaning).
- Use empty states intentionally (“suppressed by policy”) instead of silent failure.

### Anti-Patterns to Avoid
- “Green/red only” semantics that don’t explain contract boundaries (especially for `REST_POLL` and locked governance windows).
- Alerts that disappear or lack a stable operator-facing reason code.
- Dense dashboards that force scrolling during time-critical moments.
- Suppression-by-policy hidden behind technical logs instead of explicitly communicated in the UI.

### Design Inspiration Strategy

**What to Adopt**
- Grafana-like threshold accents and stable panel semantics for monitoring/trust states.
- Datadog-like incident/event framing for reconciliation failures and degraded-mode transitions.
- TradingView-like annotation/timeline thinking for decision traces and “what happened when”.

**What to Adapt**
- Convert monitoring severity into trading-specific “risk verdict severity + reason codes”.
- Convert incident timelines into decision traces with explicit next actions (“ack/wait/reconcile/kill switch”).

**What to Avoid**
- Avoid ambiguity in suppressed states: “silence” must never be interpretable as success.

## Design System Foundation

### 1.1 Design System Choice
Custom design system (tokens-first) tailored for a dark Streamlit operator console.

### Rationale for Selection
- Streamlit-first UI favors a tokens approach (colors, spacing, typography) that can be applied consistently across existing Streamlit components.
- “Airy layout” + accent-led states (Normal vs Degraded `REST_POLL` vs Locked) is easiest to guarantee when you control the visual language directly.
- You only need a small, repeatable set of components (state header, act/block cards, alert severity banners, reconciliation checklist, decision trace drill-down), so a full external component library is optional.

### Implementation Approach
- Define design tokens: `--bg`, `--panel`, `--text`, `--muted`, `--accent`, and semantic accents like `--status-ok`, `--status-warn`, `--status-block`.
- Define a spacing and layout grid that preserves “airiness” (consistent margins/padding, stable card sizes, predictable panel locations).
- Implement a small component pattern library (even if internally “just functions/templates”): state header, decision card, timeline/trace, reconciliation checklist, and two-step confirm modal.

### Customization Strategy
- Start with one dark theme; keep secondary palettes optional for future diagnostics views.
- Maintain consistent state semantics across the whole UI (colors + labels + icons), so meaning does not depend on color alone.

## 2. Core User Experience

### 2.1 Defining Experience
The operator’s defining daily interaction is the **Morning Brief Glance**: opening the local console and instantly understanding the day’s operating contract.

In under ~10 seconds, the operator should be able to answer:
- What data mode/contract are we in (Normal vs Degraded `REST_POLL` vs Locked)?
- Are entries allowed yet (and if not, when they begin)?
- What are the top “trust signals” (freshness/readiness indicators) and any must-not-miss alerts?

This turns the start of the day into a predictable ritual: **glance → confidence → action policy**.

### 2.2 User Mental Model
The operator mentally treats the system like a governed control-plane:
- The model may *suggest*, but the system’s **risk gates** decide act vs block.
- **Mode is a contract**, not a status label: degraded mode (`REST_POLL`) changes what the system is permitted to do (not just how “fast” it is).
- The operator’s job at start-of-day is to confirm **readiness** and **policy constraints**, not to debug internals.

Expected mental questions the UI answers directly:
- “Can the system place entries right now, yes or no?”
- “If it can’t, what is the stable reason and what happens next?”
- “What is being actively managed anyway (e.g., positions/stops), even if entries are suppressed?”

### 2.3 Success Criteria
The Morning Brief Glance is “successful” when the operator feels:
- **Comprehension**: mode/contract + entries-allowed policy are understood immediately.
- **Trust**: the UI provides stable, coded reasons and reflects true system semantics (including suppression-by-policy).
- **Agency**: there is an obvious next step (e.g., “observe only until 09:30” or “review today’s reconciliation gate”).

Success indicators:
- The operator can state the day contract and entry permission without opening logs.
- Degraded/locked states are unambiguously labeled with their behavioral implications.
- Trading-relevant alerts have a clear operator-next-action (not just a message).

### 2.4 Novel UX Patterns
The “novel” element is the interaction contract, not visual flair:
- **Contract-first briefing ritual**: a monitoring-style overview that translates system semantics into “permission to act” rather than raw technical status.
- **Severity accents repurposed for trading decisions**: Grafana/Datadog-like accent language, but grounded in act/block and policy meaning.
- **Decision trace readiness**: the UI makes it easy to drill from “what’s happening now” into “why” while preserving context.

### 2.5 Experience Mechanics
1. **Initiation**
   - Operator opens the console (or it auto-loads on refresh).
   - The UI shows a fixed **State Header** at the top (same location every time).
2. **Interaction**
   - Operator scans in a consistent order:
     1) **Contract/Mode**
     2) **Entries Allowed** (explicit yes/no tied to market-hours policy)
     3) **Trust Signals** (freshness/readiness + must-not-miss alerts)
     4) **Next Window Guidance** (what the operator should expect next)
3. **Feedback**
   - If degraded/locked: UI communicates “entries suppressed by policy” plus what remains actively managed.
   - If readiness gaps: UI explains the reason code and provides the next safe action (usually “wait for gap-fill / readiness window”).
   - The UI never relies on “silence” to imply success.
4. **Completion**
   - Completion is defined as: operator confidence about *today’s permission to act*.
   - After the brief glance, the UI reduces clutter and directs attention to during-session trust rituals (act/block outcomes + reconciliation cues as they appear).

## Visual Design Foundation

### Color System
Default theme (selectable): **Theme 1 — Nebula Teal**
- Background: `#0B1220`
- Panel: `#121C2F`
- Text: `#E6EDF6`
- Muted: `#8AA2BF`
- OK accent: `#2FE6A6`
- Warning accent: `#FFCC66`
- Block accent: `#FF5C7A`
- State accents:
  - Normal: `#45B7FF`
  - Degraded (`REST_POLL`): `#FFCC66`
  - Locked: `#9B8CFF`

Alternate selectable themes:
**Theme 2 — Graphite Lime**
- Background: `#0A0D12`
- Panel: `#141A22`
- Text: `#F1F5FF`
- Muted: `#93A4B8`
- OK accent: `#8CFF2A`
- Warning accent: `#FFC857`
- Block accent: `#FF3B5C`
- State accents:
  - Normal: `#6BE6FF`
  - Degraded (`REST_POLL`): `#FFC857`
  - Locked: `#B78BFF`

**Theme 3 — Charcoal Violet**
- Background: `#0E0F14`
- Panel: `#161823`
- Text: `#F2F4FF`
- Muted: `#A2AAC2`
- OK accent: `#46E6B6`
- Warning accent: `#FFB020`
- Block accent: `#FF4D6D`
- State accents:
  - Normal: `#59C2FF`
  - Degraded (`REST_POLL`): `#FFB020`
  - Locked: `#C69CFF`

Color usage rules (to keep meaning unambiguous):
- Never use color alone for semantic meaning (always pair with stable labels/icons and reason codes).
- Use accents for “contract” states (Normal / Degraded `REST_POLL` / Locked) and for act/block outcome severity (OK / Warning / Block).
- Reserve strong accent fills/borders for state headers and act/block decision cards.

Theme selection UX requirement:
- Provide an “Appearance / Theme” selector (Theme 1 / 2 / 3) in the operator settings panel.
- Theme changes must not alter any risk/permission semantics—only visual presentation.

### Typography System
- Typography style: **sans-serif**
- Goals: crisp readability for fast scanning under stress (morning brief + alert review).
- Typography tokens (initial targets):
  - Headings: semibold
  - Body: regular
  - Line height: comfortable (prioritize scan speed; avoid cramped tables)

All state and reason-code text must remain readable at the console’s default font size; contrast is treated as a non-negotiable constraint.

### Spacing & Layout Foundation
- Layout feel: **airy but stable** (avoid reflow surprises as data updates).
- Spacing rhythm:
  - Base unit: `8px`
  - Primary panels: consistent padding and gaps (use 16/24/32 derived from 8px only)
- Grid rule:
  - Keep key elements in stable positions (fixed State Header region + consistent panel layout) to reduce cognitive load and “where did it go?” confusion.
- Component rhythm:
  - Decision cards and checklists should have consistent padding and predictable vertical flow so the operator can scan them quickly.

### Accessibility Considerations
- Contrast: ensure minimum contrast for text over background for every theme (target ~4.5:1 for normal text; 3:1 for larger text).
- Non-color cues: every “Block / Degraded / Locked” must include label + icon + operator-facing reason code.
- Focus visibility: keyboard focus states must be clearly visible on interactive elements (alerts, confirmation flows, settings).
