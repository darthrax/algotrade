---
stepsCompleted: [step-01-validate-prerequisites-extract-requirements, step-02-design-epics]
inputDocuments:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/ux-design-specification.md
  - _bmad-output/planning-artifacts/ux-design-directions.html
---

# algotrade - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for algotrade, decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

FR1: The system can ingest streaming market data for subscribed instruments during the trading session.
FR2: The system can retrieve and persist historical price series to backfill gaps and onboard new symbols.
FR3: The system can reject or reconcile duplicate and out-of-sequence ticks per defined rules.
FR4: The system can detect missing data beyond a defined threshold during market hours and notify the operator.
FR5: The system can operate in a degraded quote mode when streaming is unavailable and expose the active mode per instrument.
FR6: The operator can maintain a tradable universe using configurable liquidity, price, and event-proximity filters.
FR7: The system can block new entries for instruments or sessions where policy forbids trading.
FR8: The system can compute features from available market data without using disallowed future information.
FR9: The system can emit trade recommendations with direction, confidence, and regime context.
FR10: The system can suppress recommendations when required inputs are stale or the active data mode forbids new risk.
FR11: The system can persist the feature context used for each recommendation for later audit.
FR12: The operator can run the product in a simulated-fill mode or a broker-backed mode with shared decision logic.
FR13: The system can evaluate each recommendation against enumerated risk checks before placing orders.
FR14: The system can place, amend, and close orders with protective exit parameters according to policy.
FR15: The system can advance and terminate order lifecycle states including partial fills, rejects, and timeouts.
FR16: The operator can invoke an emergency stop that closes positions and halts new risk until cleared.
FR17: The system can enforce position count, concentration, spread, and daily loss constraints.
FR18: The system can simulate prior sessions through the same decision pipeline used during live operation.
FR19: The operator can compare strategy performance across time splits and benchmarks under a shared cost model.
FR20: The system can derive training labels from stored recommendations and subsequent price outcomes.
FR21: The system can run scheduled training jobs that produce candidate models without manual ad hoc steps.
FR22: The system can store and retrieve versioned model artifacts with training data span and configuration lineage.
FR23: The system can compare candidate models to the production champion using agreed performance gates.
FR24: The system can withhold promotion when gates fail and retain the prior champion.
FR25: The system can revert to a prior champion when live monitoring breaches rollback policy.
FR26: The operator can approve or reject a promotion decision outside market hours when human sign-off is required.
FR27: The operator can inspect positions, balances, and lifecycle state through a local console.
FR28: The system can deliver notifications for connectivity, risk, training, and health events through a configured channel.
FR29: The system can report readiness and liveness for independent watchdog processes.
FR30: The system can export trades, charges, and summaries for accounting and tax workflows.
FR31: The system can track cumulative turnover and surface proximity to statutory audit thresholds.
FR32: The system can record immutable audit trails for recommendations, orders, fills, configuration changes, and model promotions.
FR33: The system can produce day-end reconciliation artifacts aligning broker activity with internal records.
FR34: The operator can change risk, universe, and model parameters outside market hours with each change audited.
FR35: The system can apply blackout and session-window rules that restrict or throttle new risk.
FR36: The system can manage broker session authentication lifecycle without exposing secrets through operator-facing APIs.
FR37: The operator can view explainability summaries tied to individual recommendations.
FR38: The system can record a reason when performing a manual override or exceptional action.
FR39: The system can enforce stepped live-capital increases only after defined gates and explicit operator confirmation.

### NonFunctional Requirements

NFR1: For active watchlist symbols during RTH, ingest-to-durable-store latency meets p95/p99 targets in the operational SLO set.
NFR2: The feature-complete -> risk decision path meets SLO under normal streaming conditions; under degraded quote mode, behavior matches Functional Requirements (no new-entry path when forbidden).
NFR3: Submit -> broker acknowledgment meets SLO when broker health is normal; sustained breach triggers escalation/safe-close runbook.
NFR4: Operator console primary views (positions, health, mode, last signal time) load within 2 seconds on typical local hardware under normal load (smoke-testable).
NFR5: Scheduled train+evaluate completes inside the defined overnight window on the reference profile, or surfaces failure to the operator within 15 minutes of the scheduled end.
NFR6: After an unplanned restart during RTH, gap fill for the watchlist completes before new automated entries are accepted, or the system remains in blocked-entry with explicit reason.
NFR7: Broker credentials, session material, and notification tokens are encrypted at rest and never logged in plaintext.
NFR8: Operator-facing execution or mode APIs on non-loopback binds are authenticated; kill, mode change, and capital stage actions require stronger trust than read-only health.
NFR9: Audit and config-change trails follow tamper-evident discipline; backups use encrypted handling.
NFR10: Remote access beyond localhost uses TLS (or equivalent recommended default).
NFR11: The client obeys documented broker rate/burst/subscription constraints; sustained limit signals trigger backoff and degraded mode (not blind retry storms).
NFR12: Broker vs internal reconciliation meets accuracy targets (no unexplained material differences) before live is armed for the next session, unless waived with an audit reason.
NFR13: System clock stays within NTP tolerance required by pre-trade checks; drift breach blocks trading until cleared.

### Additional Requirements

- AR1: Starter approach is “from scratch (with ML baked in immediately)”, not a generic scaffold, to avoid architectural drift from the ML->feature->signal->risk/execution control spine.
- AR2: Initialize internal “decision spine” module boundaries so ML cannot bypass `services/risk_execution/`.
- AR3: Build persistence + schema evolution (migrations) aligned with auditability requirements.
- AR4: Include deterministic replay/backtest hooks as a parallel path to live/paper behavior from day one.
- AR5: Wire scheduled training orchestration plus model governance/promotion gates into the system lifecycle early (not as a later bolt-on).
- AR6: Use TimescaleDB on Postgres as the authoritative tick/time-series store; support retention/downsampling/compression so tick volume doesn’t degrade operational viability.
- AR7: Use privileged shared secret token(s) for risk-increasing internal API routes (notably `POST /kill` and `POST /mode`) provided via an `Authorization` header.
- AR8: Separate tokens for operator vs automation to enable scoping and auditing by actor.
- AR9: Ensure privileged tokens are not logged in plaintext and that all privileged actions create auditable entries tied into the correlation/log chain.
- AR10: Treat control-plane as synchronous and propagation as asynchronous: sync critical path triggers deterministic risk/execution; async consumers persist durable events for safe recovery and must not place orders.
- AR11: Require durable-source-of-truth for async consumers: consumers must derive work from durable records (DB/outbox-like durable events) to recover/replay correctly after restarts.
- AR12: Require idempotency + correlation_id for every async handler (duplicate deliveries must not re-trigger capital actions).
- AR13: Enforce fail-closed behavior: risk/execution errors produce “no new risk actions”; order submission is disabled if the risk service is unhealthy.
- AR14: Enforce stable error/risk rejection machine codes and “no silent success”: suppressed/block-by-policy outcomes must emit traceable events and operator-facing reason codes.
- AR15: Enforce architectural communication conventions: ISO-8601 UTC timestamps; `snake_case` event names/identifiers; consistent API response wrapper `{ ok, data }` vs `{ ok, error: { code, message, correlation_id } }`.
- AR16: Treat static IP and session token lifecycle as operational dependencies for Breeze connectivity (static IP mandatory starting April 2026).
- AR17: Enforce time/session correctness keyed to India market session windows, including blackout/lockout policy for configuration and/or new entries.
- AR18: Represent and enforce data-mode semantics across the stack (`WEBSOCKET` vs `REST_HISTORICAL` vs `REST_BACKFILL` vs `REST_POLL`), including `REST_POLL` entry suppression-by-policy behavior.
- AR19: Deterministic replay requirement: simulation clock/replay tools must run through the same decision path used during live trading to avoid paper/live divergence.

### UX Design Requirements

UX-DR1: Implement a tokens-first dark Streamlit operator console design system with semantic tokens: `--bg`, `--panel`, `--text`, `--muted`, `--accent`, and semantic accents `--status-ok`, `--status-warn`, `--status-block`.
UX-DR2: Provide exactly three selectable visual themes (Theme 1 “Nebula Teal”, Theme 2 “Graphite Lime”, Theme 3 “Charcoal Violet”) with explicit color assignments for OK/Warning/Block accents and state accents (Normal, Degraded `REST_POLL`, Locked).
UX-DR3: Theme switching is visual-only: it must not change semantics, labels, reason codes, or permission/risk logic.
UX-DR4: Enforce non-color cues across UI: every “Block / Degraded / Locked” must include label + icon + operator-facing reason code (no color-only meaning).
UX-DR5: Ensure contrast and readability for every theme (target WCAG Level AA; explicitly validate contrast of state/reason text).
UX-DR6: Make keyboard focus states clearly visible on all interactive elements, and ensure tab order matches visible hierarchy and “next action” flow.
UX-DR7: Implement a fixed, always-visible “State/Contract Header” at a stable screen location across views.
UX-DR8: The State/Contract Header must display contract/mode (Normal (RTH), Degraded `REST_POLL`, Locked), an explicit “Entries allowed” yes/no per policy, and next window guidance (what changes next).
UX-DR9: State/Contract Header must show must-not-miss alerts (top 1–3) and provide a focusable “Why?” affordance for contract/mode changes.
UX-DR10: The State/Contract Header must use `aria-live="polite"` for contract/mode changes and handle Unknown/Error by showing safe fallback messaging derived from last-known safe contract.
UX-DR11: Implement an “Act/Block Decision Card” that always renders the act/block outcome plus a stable reason code, operator-language explanation, and an explicit next action (wait/observe/reconcile/kill-switch confirmation).
UX-DR12: Decision Cards must include a drill-down entry point (“View full trace”) and must never rely on “silence = success.”
UX-DR13: Decision Cards must support trace-loading states (“Loading trace”, “Trace unavailable”) without losing the act/block reason and next action.
UX-DR14: Implement a “Decision Trace / Drill-down” with progressive disclosure showing the causal chain: freshness/data mode -> feature/signal snapshot -> risk checks -> broker outcome or explicit suppression, plus evidence blocks for missing/unknown parts.
UX-DR15: Decision Trace must be implemented with accessible accordion semantics (keyboard navigable, `aria-expanded` / role semantics) and show “what is unknown” when evidence is incomplete.
UX-DR16: Implement a “Reconciliation Status Panel” that supports statuses: Pass, Fail, Needs waiver, and Block next-day live; it must include broker vs internal alignment highlights and a clear next action (resolve mismatch vs waive with audit note).
UX-DR17: Reconciliation Status Panel must surface pending/running states and emit an `aria-live` summary for operator reassurance.
UX-DR18: Implement a “Governance Lockout / Approval Card” that (1) shows why locked + next allowed time, and (2) provides an outside-lock review card with requested change + version lineage + candidate metrics summary.
UX-DR19: Governance Lockout / Approval Card must disable risk-increasing controls during lock windows and include an attached explanation via `aria-describedby` (never silent disable).
UX-DR20: Approval/denial must require an audit note entry and be captured in UI via a modal/dialog flow with confirmation/audit context.
UX-DR21: Implement waiver flow integration in the UI as part of reconciliation/governance, including capture of waiver intent and audit note entry.
UX-DR22: Implement a “Kill Switch Two-Step Confirm” that uses a deliberate two-step flow (step 1: high-level impact; step 2: explicit confirm via typing `CONFIRM` or checkbox).
UX-DR23: Kill Switch Two-Step Confirm must include accessibility: dialog role, focus trap, and safe ESC/cancel behavior (ESC cancels only when appropriate, per step).
UX-DR24: After kill-switch activation, the UI must reflect a Locked contract state and surface required recovery steps.
UX-DR25: Implement an “Alert Stream” as an event feed with severity accents + stable event labels, “what changed” summaries, and next actions (ack/wait/open trace).
UX-DR26: Alert Stream must group alerts into a single accessible region, provide focusable action buttons, and require explicit acknowledgment for critical alerts (with evidence persistence).
UX-DR27: Implement a consistent “button hierarchy” pattern: primary/safe guided actions, secondary/inspect actions, and danger actions; disabled controls must remain readable and always include a tied explanation.
UX-DR28: Implement a consistent “Feedback pattern” across the UI: Success includes completion detail; Warning includes what is limited + next safe action; Error/Block includes stable reason code + plain-language interpretation + next action expectation.
UX-DR29: Implement navigation patterns that preserve the operator mental model: contract header always visible; drill-down views use progressive disclosure (overview remains visible while details expand in-place or in a stable side panel).
UX-DR30: Tabs/left-nav must be keyboard navigable with accessible names for each view (e.g., “Decisions”, “Reconciliation”, “Governance”, “Alerts”).
UX-DR31: Responsive strategy must be implemented for Desktop (1024px+), Tablet (768–1023px), and Mobile (320–767px) with explicit rules: on mobile the State/Contract Header becomes sticky and the latest decision/next action is in the first viewport.
UX-DR32: Breakpoints are fixed at Mobile 320–767px, Tablet 768–1023px, Desktop 1024px+; no custom breakpoints unless Streamlit constraints require it.
UX-DR33: Responsive testing must be done at representative widths (~375px, ~768px, ~1024px) ensuring the State/Contract Header remains visible and the next-action affordance remains accessible on mobile.
UX-DR34: Accessibility testing must include automated `axe`/Lighthouse checks for each theme and keyboard-only walkthroughs for critical flows: Decisions->trace drill-down, Reconciliation checklist+waiver capture, Governance approval/denial, and Kill switch two-step confirm.
UX-DR35: Screen reader checks must validate the contract header and reason-code reading order for each theme.
UX-DR36: Implement reconciliation checklist-style UI so EOD reconciliation is pass/fail/waiver resolved as a routine checklist rather than investigation.
UX-DR37: Implement SHAP/top-features surfacing as explainability tied to individual recommendations for operator sanity-checking (must not bypass the risk gateway).
UX-DR38: Ensure governance lockout window messaging is operator-facing and aligned to the UX direction’s market-hours lock semantics (controls disabled during the locked window; “why locked” and “when allowed” must be explicit).
UX-DR39: Implement layout Direction 2 as a two-panel structure: Panel A fixed State/Contract region; Panel B decision trace card plus reconciliation status block for scan-friendly closure.
UX-DR40: Implement UI defaults that keep contract honesty: mode/state labels must reflect true semantics (including suppression-by-policy), never approximate it or hide it in logs.

## FR Coverage Map
FR1: Epic 1 - ingest streaming market data
FR2: Epic 1 - retrieve/persist historical backfill data
FR3: Epic 1 - reject/reconcile duplicate and out-of-sequence ticks
FR4: Epic 1 - detect missing data + notify operator
FR5: Epic 1 - degraded quote mode + expose active mode per instrument
FR6: Epic 1 - maintain tradable universe via liquidity/price/event-proximity filters
FR7: Epic 1 - block new entries when policy forbids trading
FR8: Epic 2 - compute features from available data only
FR9: Epic 2 - emit trade recommendations w/ confidence + regime context
FR10: Epic 2 - suppress recommendations when inputs stale or mode forbids new risk
FR11: Epic 2 - persist feature context for audit
FR12: Epic 2 - simulate vs broker-backed fill modes using shared decision logic
FR13: Epic 2 - evaluate recommendations against enumerated risk checks
FR14: Epic 2 - place/amend/close orders with protective exits
FR15: Epic 2 - manage order lifecycle states (partial fills/rejects/timeouts)
FR16: Epic 2 - emergency stop closes positions + halts new risk
FR17: Epic 2 - enforce position count/concentration/spread/daily loss constraints
FR18: Epic 3 - simulate prior sessions via the same decision pipeline
FR19: Epic 3 - compare performance across time splits/benchmarks under shared cost model
FR20: Epic 4 - derive training labels from stored recommendations + outcomes
FR21: Epic 4 - run scheduled training jobs without ad hoc steps
FR22: Epic 4 - store/retrieve versioned model artifacts + training span/config lineage
FR23: Epic 4 - compare candidates to champion using agreed performance gates
FR24: Epic 4 - withhold promotion and retain champion when gates fail
FR25: Epic 4 - revert to prior champion when rollback policy is breached
FR26: Epic 4 - operator approve/reject promotion outside market hours when required
FR27: Epic 2 - operator inspects positions/balances/lifecycle state in console
FR28: Epic 2 - notifications for connectivity/risk/training/health events
FR29: Epic 2 - readiness/liveness endpoints for watchdog processes
FR30: Epic 5 - export trades/charges/summaries for accounting/tax
FR31: Epic 5 - track cumulative turnover + proximity to audit thresholds
FR32: Epic 2 - record immutable audit trails for recommendations/orders/fills/config/promotions
FR33: Epic 5 - produce day-end reconciliation artifacts aligning broker vs internal
FR34: Epic 4 - change risk/universe/model parameters outside market hours (audited)
FR35: Epic 2 - apply blackout/session-window rules restricting/throttling new risk
FR36: Epic 2 - manage broker session auth lifecycle without exposing secrets via API
FR37: Epic 2 - view explainability summaries tied to recommendations
FR38: Epic 2 - record reason for manual override/exceptional action
FR39: Epic 4 - stepped live-capital increases only after defined gates + explicit confirmation

## Epic List
### Epic 1: Data Continuity & Tradable Universe Policy
Epic goal: Keep market data continuous, maintain the tradable universe, and enforce “entries allowed vs suppressed” via degraded/policy modes.
**FRs covered:** FR1, FR2, FR3, FR4, FR5, FR6, FR7

### Epic 2: Risk-Gated Trading Decisions & Operator Trust Loop
Epic goal: Convert feature/signal proposals into fail-closed act/block decisions, place/track orders (paper/live parity), enforce risk constraints, and keep the operator informed with stable reason codes and next actions.
**FRs covered:** FR8, FR9, FR10, FR11, FR12, FR13, FR14, FR15, FR16, FR17, FR27, FR28, FR29, FR32, FR35, FR36, FR37, FR38

### Epic 3: Deterministic Replay & Backtest Evidence
Epic goal: Simulate prior sessions through the same decision pipeline and compare performance across time splits/benchmarks under a shared cost model.
**FRs covered:** FR18, FR19

### Epic 4: Model Training, Governance & Staged Live Capital
Epic goal: Run scheduled training, manage versioned artifacts, compare challenger vs champion with gates, support promotion/rollback approval outside market hours, and stage live capital increases only after explicit confirmation.
**FRs covered:** FR20, FR21, FR22, FR23, FR24, FR25, FR26, FR34, FR39

### Epic 5: Reconciliation, Day-End Packaging, and Tax-Ready Exports
Epic goal: Produce day-end reconciliation artifacts and export trade/charge/tax-ready records with turnover tracking.
**FRs covered:** FR30, FR31, FR33


## Epic 1: Data Continuity & Tradable Universe Policy
Epic goal: Keep market data continuous, maintain the tradable universe, and enforce "entries allowed vs suppressed" via degraded/policy modes.

### Story 1.1: WebSocket Tick Ingestion + Dedup Persistence
As a technical operator,
I want the system to validate and persist incoming Breeze WebSocket ticks with dedup and monotonic timestamp guarantees,
So that the downstream feature/prediction pipeline has a clean, auditable, continuously growing time series.

**Acceptance Criteria:**
1. **Given** the system is configured with an active watchlist containing a symbol and the Breeze WebSocket is connected (data source mode = `WEBSOCKET`)
   **When** a tick arrives for that symbol
   **Then** the tick is validated and persisted to TimescaleDB with `data_source = 'WEBSOCKET'`.
2. **Given** a previous stored tick exists for the same `stock_token` at the same `resolution`
   **When** a new incoming tick has a timestamp `<=` the last stored timestamp
   **Then** the incoming record is discarded as out-of-order, and a logged warning/debug entry is written.
3. **Given** the dedup primary key for ticks is `(stock_token, timestamp, resolution)`
   **When** an incoming tick duplicates an existing stored record on that primary key
   **Then** no duplicate row is created (row count remains unchanged) and the duplicate is discarded (with debug logging).
4. **Given** an incoming tick has `price` or `volume` equal to zero
   **When** the tick is processed
   **Then** the record is discarded and not written to the tick table.
5. **Given** a configurable tolerance band is set (max single-tick move exceeding 5% is invalid per data quality rules)
   **When** a tick violates the tolerance band vs the previous tick
   **Then** the record is discarded and logged.
6. **Given** ingest-to-durable-store SLO thresholds are configured for the active watchlist
   **When** ticks are ingested under normal load
   **Then** measured ingest-to-durable-store latency meets the configured `p95/p99` thresholds in the operational SLO set.
7. **Given** a tick is flagged as at/beyond the stock's NSE circuit limit
   **When** the tick is processed
   **Then** it is stored with an indicator/flag for the circuit-limit condition (without causing the ingestion flow to crash).

### Story 1.2: Startup Gap Fill & Historical Bootstrap (Breeze REST)
As a technical operator,
I want the system to fill any missing tick history at startup using Breeze REST before opening the WebSocket,
So that the feature engineering pipeline always runs on a complete, gap-free time series after restarts.

**Acceptance Criteria:**
1. **Given** the system is starting during a period where watchlist symbols are configured
   **When** startup gap fill runs
   **Then** for each configured symbol it queries Breeze REST historical data from the last stored timestamp (or epoch zero if new) up to “now”.
2. **Given** Breeze REST bootstrap/gap-fill records are retrieved
   **When** each record is validated
   **Then** it passes the same data quality pipeline as WebSocket ticks (discard zero price/volume, discard out-of-order, apply tolerance-band checks).
3. **Given** startup gap fill completes successfully for all watchlist symbols
   **When** readiness is evaluated for normal ingestion
   **Then** the system proceeds to open the WebSocket connection only after gap fill completion is confirmed.
4. **Given** the tick deduplication primary key is `(stock_token, timestamp, resolution)`
   **When** bootstrap re-runs for already-stored time ranges after a restart
   **Then** deduplication prevents duplicate rows from being created.
5. **Given** historical bootstrap/gap-fill writes produce records
   **When** historical persistence occurs
   **Then** records are stored with `data_source = 'REST_HISTORICAL'` during bootstrap and `data_source = 'REST_BACKFILL'` for reconnection backfill where applicable.
6. **Given** startup gap fill cannot complete for one or more symbols
   **When** the system evaluates readiness
   **Then** normal signal generation for that symbol is blocked and the operator receives an explicit readiness reason (no silent failure).

### Story 1.3: Missing Data Detection + Operator Notification
As a technical operator,
I want the system to detect missing market data beyond a configured threshold during market hours and notify me,
So that the feature window is flagged as unreliable and entries are handled safely.

**Acceptance Criteria:**
1. **Given** the system is running during market hours and ingesting ticks/quotes for subscribed instruments
   **When** it detects a gap in stored tick continuity exceeds the configured missing-data threshold
   **Then** it flags the affected symbol’s feature window as unreliable.
2. **Given** the configured missing-data threshold is exceeded
   **When** a gap is detected
   **Then** the system triggers an operator notification (Telegram and/or dashboard alert).
3. **Given** missing-data state is active for an instrument
   **When** a prediction/recommendation attempt is made for that instrument
   **Then** the recommendation is suppressed (or downgraded according to policy) and the operator receives a reason code tied to missingness.
4. **Given** the missing-data gap is resolved via backfill/continued ingestion
   **When** freshness/continuity is restored beyond the configured thresholds
   **Then** the unreliable flag is cleared for subsequent feature generation.
5. **Given** a restart occurs during market hours
   **When** startup recovery runs (including gap fill)
   **Then** missing-data detection still functions after gap fill completion, and unresolved gaps do not trigger duplicate notifications silently.
6. **Given** the system’s data mode is degraded (`REST_POLL`)
   **When** missingness detection logic runs
   **Then** it follows the intended degraded-mode semantics: monitoring/alerting stays accurate while new entries remain suppressed by policy.

### Story 1.4: Degraded Quote Mode (`REST_POLL`) + Entries Suppressed by Policy
As a technical operator,
I want the system to switch into degraded quote mode (`REST_POLL`) when streaming is unavailable and suppress new entries by policy,
So that exits/stops remain managed but the system never opens new risk on insufficient granularity.

**Acceptance Criteria:**
1. **Given** the system is trading in normal contract mode and WebSocket connectivity drops
   **When** streaming cannot be restored within the configured reconnect window
   **Then** it switches the per-instrument data/mode contract to `REST_POLL`.
2. **Given** the system enters `REST_POLL`
   **When** prediction/recommendation logic runs for a symbol
   **Then** the system does not allow new position entries (entries suppressed by policy) and emits a stable reason code tied to data mode/granularity.
3. **Given** positions already exist during the mode transition
   **When** in `REST_POLL`
   **Then** the system continues to manage exits/stops/position monitoring, but does not create new entries.
4. **Given** the system remains in `REST_POLL`
   **When** the operator views current state in the console/dashboard
   **Then** the active data mode for the instrument is displayed correctly and is consistent with the backend contract.
5. **Given** the system later reconnects WebSocket successfully
   **When** gap backfill completes
   **Then** the system returns from `REST_POLL` to normal ingestion for that symbol, and new entries are allowed only after freshness/data-mode gates pass.
6. **Given** `REST_POLL` is active and a prediction would otherwise trigger a BUY/SELL recommendation
   **When** the policy suppresses the entry
   **Then** the suppression is traceable in logs/audit evidence (no silent success), and the operator can see the “what changed / why blocked” reason.

### Story 1.5: Tradable Universe Filters (Liquidity / Price / Event Proximity)
As a technical operator,
I want to maintain a tradable universe using configurable liquidity, price, and event-proximity filters,
So that the system trades only eligible instruments per policy.

**Acceptance Criteria:**
1. **Given** a configured universe filter set exists (liquidity threshold, min/max price, and event-proximity rules)
   **When** the universe refresh runs (e.g., daily/weekly cadence per configuration)
   **Then** the system outputs an updated watchlist of eligible symbols.
2. **Given** an instrument fails the mandatory exclusion filters
   **When** the universe refresh evaluates that instrument
   **Then** it is excluded from the watchlist and is not eligible for signal generation or order placement.
3. **Given** an instrument passes inclusion criteria
   **When** the universe refresh runs
   **Then** the instrument is included in the watchlist, and its data subscription eligibility becomes active.
4. **Given** the system has an existing watchlist already
   **When** a symbol transitions from included → excluded after filters change
   **Then** the system stops applying new signal generation/risk evaluation for that symbol (without breaking existing open positions if any).
5. **Given** event-proximity inputs (e.g., corporate actions, results calendar) are loaded into the event/blackout calendar
   **When** an instrument has a pending event within the configured proximity window
   **Then** it is excluded from the universe according to policy.
6. **Given** the operator updates universe filter configuration
   **When** the update occurs outside market hours
   **Then** the change is applied to the next refresh cycle and the update is audit-logged.
