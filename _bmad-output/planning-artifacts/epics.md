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

### Story 1.6: Policy Gate — Block New Entries When Trading Is Forbidden (FR7)
As a technical operator,
I want the system to block new position entries for instruments or sessions where policy forbids trading,
So that the system never opens new risk during blackout windows, observation-only windows, or configured restriction periods.

**Acceptance Criteria:**
1. **Given** the system is within a full trading blackout (no new positions), or an observation-only window, or a configured “no new positions” period
   **When** a strategy signal would otherwise produce an entry for an instrument/session
   **Then** the system suppresses the entry decision and does not place a new order for that instrument/session.
2. **Given** there are existing open positions when policy restriction becomes active
   **When** the system processes market updates during the restricted window
   **Then** it continues to manage exits/stops/position monitoring without forcing closes, while still blocking new entries.
3. **Given** an instrument is excluded by universe filters or otherwise not eligible by policy
   **When** an entry is evaluated
   **Then** the entry is blocked and the operator is shown an operator-facing “entries allowed” state consistent with policy.
4. **Given** the operator console UI is open
   **When** policy forbids trading for the current session/window
   **Then** the UI shows `Entries allowed: No` and includes a reason label (not only a backend state).
5. **Given** the policy window ends (e.g., observation-only time passes, blackout ends, or restriction period is over)
   **When** the system evaluates new signals
   **Then** entries become allowed again only after data readiness requirements (freshness/data-mode gates) are satisfied.
6. **Given** the system is in degraded mode (`REST_POLL`)
   **When** policy restrictions apply at the same time as degraded semantics
   **Then** degraded semantics remain correct (entries suppressed due to granularity), and policy suppression does not incorrectly “re-open” entries due to mode transition timing.

## Epic 2: Risk-Gated Trading Decisions & Operator Trust Loop
Epic goal: Convert feature/signal proposals into fail-closed act/block decisions, place/track orders (paper/live parity), enforce risk constraints, and keep the operator informed with stable reason codes and next actions.

### Story 2.1: Feature Snapshot + Recommendation Suppression (Data Mode + Freshness Gate) with Immutable Audit Trace
As a technical operator,
I want the system to compute features from available market data, persist the feature context, and emit (or suppress) trade recommendations based on data-mode/freshness rules,
So that the operator can trust act/block outcomes with auditable evidence.

**Acceptance Criteria:**
1. **Given** an instrument has a configured feature window and the active data mode is `WEBSOCKET` (or a mode where new risk is allowed)
   **When** the system computes features for a prediction cycle
   **Then** it computes features using only available market data (no disallowed future information) and persists the feature context used.
2. **Given** a feature freshness check is configured
   **When** any mandatory feature in the required set is more than the configured staleness limit
   **Then** the system suppresses the recommendation for that instrument.
3. **Given** the active data mode is `REST_POLL` (degraded semantics) or otherwise forbids new risk
   **When** a recommendation would be generated
   **Then** the system suppresses recommendations according to policy (entries/risk are not opened under forbidden mode).
4. **Given** a recommendation is not suppressed
   **When** the system emits the recommendation
   **Then** the recommendation includes direction, confidence, and regime context, and the persisted feature context is linked to that recommendation for later audit.
5. **Given** a “traceability chain” requirement is enabled
   **When** a recommendation is emitted or suppressed
   **Then** the system writes immutable audit evidence with a `correlation_id` that can link: feature context -> recommendation outcome -> downstream risk decision trace.
6. **Given** the same triggering event is retried (idempotent delivery scenario)
   **When** the system processes it again
   **Then** it does not create duplicate audit/recommendation records for the same `correlation_id` / idempotency key.
7. **Given** the operator later requests evidence for “what the model knew”
   **When** the operator inspects the stored feature context for a recommendation
   **Then** it can be retrieved and displayed as the exact feature snapshot used to reach the recommendation outcome.

### Story 2.2: Pre-trade Risk Checks + Stable Act/Block Decision Trace (No Order Submission Yet)
As a technical operator,
I want the system to evaluate each non-suppressed recommendation against enumerated risk checks and return a deterministic **ACT vs BLOCK** verdict with stable reason codes and drill-down evidence,
So that I can trust what the system will do next and what it refused to do (with an auditable causal chain).

**Acceptance Criteria:**
1. **Given** a recommendation is eligible for risk evaluation (not suppressed by data-mode freshness policy) and has a `correlation_id` / idempotency context
   **When** the system runs pre-trade risk checks
   **Then** it returns a verdict: `ACT` or `BLOCK` (before any order is placed).
2. **Given** one or more enumerated risk checks fail (examples: spread/price constraints, concentration/position count limits, or daily loss constraints)
   **When** the verdict is computed
   **Then** the verdict is `BLOCK` and includes a stable machine reason code plus an operator-facing reason label and “next action” expectation.
3. **Given** all enumerated risk checks pass
   **When** the verdict is computed
   **Then** the verdict is `ACT` and the system records an order intent / next execution step marker (without submitting to the broker in this story).
4. **Given** the same incoming risk evaluation is retried with the same idempotency key / `correlation_id`
   **When** the system processes the retry
   **Then** it does not create duplicate risk-decision records or duplicate audit-evidence entries for the same evaluation.
5. **Given** the risk service is unavailable or errors during pre-trade evaluation
   **When** the system attempts risk evaluation
   **Then** it fails closed: returns `BLOCK` with reason code “risk_unavailable”, submits no order, and surfaces an operator-visible alert/notification.
6. **Given** the UI is showing the current decision for the operator
   **When** the verdict is `BLOCK` or `ACT`
   **Then** the operator sees an Act/Block decision card that includes:
   - the stable reason code
   - operator-language explanation
   - explicit next action
   - a “View full trace” drill-down entry point (trace may be loading/unavailable, but reason + next action remain visible).
7. **Given** “View full trace” is opened for a decision
   **When** the trace is rendered
   **Then** it shows the causal chain in order: freshness/data mode -> feature snapshot -> risk verdict -> (suppression if `BLOCK`, or “order intent created” if `ACT`).

### Story 2.3: Order Submission (Paper Fill vs Broker-Backed) via Risk-Execution State Machine
As a technical operator,
I want the system to submit orders only when the pre-trade risk verdict is `ACT`, using the configured fill mode (paper vs broker-backed) while tracking the order lifecycle state machine end-to-end,
So that executions (or simulated fills) are consistent with paper/live parity and auditable for later reconciliation.

**Acceptance Criteria:**
1. **Given** the system has an approved `ACT` pre-trade verdict for a recommendation and the policy/data-mode gates allow entries
   **When** the operator/inference cycle triggers execution
   **Then** the system creates an order intent and submits an order through the configured execution fill mode.
2. **Given** the fill mode is `paper/simulated`
   **When** an order intent is executed
   **Then** the system performs a simulated fill consistent with the same decision spine and persists order state transitions without contacting the broker.
3. **Given** the fill mode is `broker-backed/live`
   **When** an order is submitted
   **Then** the system uses the broker adapter to place the order and persists broker acknowledgment and correlation IDs linking to the original decision trace.
4. **Given** an order is submitted
   **When** broker responses arrive (including partial fills/rejects/timeouts)
   **Then** the system advances the order lifecycle through explicit states (e.g., `ORDER_SENT -> ORDER_CONFIRMED -> POSITION_OPEN`) and transitions for partial fills/rejects/timeouts per policy.
5. **Given** the system cannot receive broker acknowledgment within configured timeouts
   **When** the timeout triggers
   **Then** the system closes/terminates the order lifecycle safely according to policy and surfaces an operator-visible reason code (no silent success).
6. **Given** the risk execution service is unavailable at submission time
   **When** a submission attempt is made
   **Then** the system fails closed: it does not place a new order and records a risk-unavailable outcome tied to the correlation ID.
7. **Given** the same submission event is retried (same idempotency key / correlation_id)
   **When** execution runs again
   **Then** the system does not create duplicate orders; it reuses/updates the existing persisted lifecycle for that intent.

### Story 2.4: Emergency Stop (Kill Switch) with Two-Step Confirm, Fail-Closed Behavior, and Operator Lock State (FR16 + UX)
As a technical operator,
I want an emergency kill switch that requires a deliberate two-step confirmation to activate,
So that the system closes positions and halts all new risk immediately and remains locked until cleared, with clear operator messaging.

**Acceptance Criteria:**
1. **Given** the operator initiates the kill switch action in the operator console
   **When** they press the kill switch control
   **Then** the UI starts the two-step confirmation flow (step 1 description + step 2 explicit confirm requirement).
2. **Given** the operator is on kill switch step 1
   **When** they confirm in step 2 by typing `CONFIRM` (or equivalent explicit confirmation) / checking the confirm box
   **Then** the kill switch activation request is accepted and proceeds to activation.
3. **Given** the kill switch two-step dialog is open
   **When** the operator attempts to cancel/escape
   **Then** ESC/cancel only dismisses safely per the two-step rules (does not activate on accidental dismiss).
4. **Given** kill switch is activated
   **When** the system processes the activation
   **Then** it closes/terminates open positions and halts new risk immediately (no new order placement).
5. **Given** kill switch is active
   **When** the inference/risk pipeline attempts to process a new recommendation
   **Then** it returns a fail-closed blocked outcome (no new orders) with an operator-visible reason code and “next action” expectation.
6. **Given** kill switch is activated
   **When** audit/traceability is enabled
   **Then** the system writes immutable audit evidence linked to a `correlation_id` (showing who/what initiated the kill switch and the resulting state change).
7. **Given** the kill switch is active
   **When** the operator views current state
   **Then** the UI reflects a `Locked` contract state and provides required recovery steps for when/ how normal trading can resume.
8. **Given** the operator console requires strong auth for risk-increasing actions
   **When** kill switch activation is requested
   **Then** unauthenticated/invalid requests are rejected and no position-closing/risk-halt actions occur (fail closed).

### Story 2.5: Broker Adapter Order Acknowledgement + Partial Fills / Rejects / Timeouts State Handling (FR15)
As a technical operator,
I want the system to track the full order lifecycle state machine including partial fills, rejects, and timeouts,
So that the operator always has accurate order/position state and safe reconciliation outcomes.

**Acceptance Criteria:**
1. **Given** an order is submitted (paper fill or broker-backed)
   **When** the system records the initial order lifecycle entry
   **Then** it persists an order intent with a correlation_id and timestamps.
2. **Given** a broker-backed order acknowledgement is received (or a simulated ack in paper mode)
   **When** the acknowledgement is processed
   **Then** the order lifecycle transitions to `ORDER_SENT` and then `ORDER_CONFIRMED` with persisted timestamps.
3. **Given** a broker-backed order can be partially filled
   **When** partial fill reports arrive
   **Then** the system advances lifecycle state through explicit partial-fill transitions and persists filled quantity and remaining quantity.
4. **Given** the broker returns a reject for an order
   **When** the reject response is processed
   **Then** the lifecycle transitions to a rejected/terminated state and the operator is shown a stable reason code (no silent failure).
5. **Given** broker acknowledgement or fill completion does not arrive within configured timeouts
   **When** the timeout triggers
   **Then** the system terminates the order lifecycle safely according to policy and records an operator-visible reason code.
6. **Given** duplicate broker responses or retried delivery occurs (same correlation_id / idempotency context)
   **When** the lifecycle updater processes the response again
   **Then** state updates are idempotent and do not regress or create duplicate fill events.
7. **Given** the lifecycle transitions to a completion point (position open or terminal close)
   **When** the system updates positions
   **Then** the positions view model is consistent with the lifecycle state (no mismatch between order state and displayed position state).
8. **Given** the traceability chain is enabled
   **When** lifecycle state changes occur
   **Then** immutable audit evidence is written linking: decision trace -> order intent -> lifecycle transitions -> fills/rejects/timeouts via correlation_id.

### Story 2.6: Enforce Position/Concentration/Spread/Daily Loss Constraints (FR17)
As a technical operator,
I want the system to enforce position count, concentration, spread, and daily loss constraints on each risk evaluation,
So that capital loss remains bounded within guardrails and the system fails safely when limits are hit.

**Acceptance Criteria:**
1. **Given** the system is evaluating a recommendation that would create a new position entry (or increase exposure)
   **When** it enforces position count, concentration, spread, and daily loss constraints
   **Then** it either permits `ACT` or returns `BLOCK` with stable reason codes for any violated constraint.
2. **Given** at least one constraint is violated (max positions reached, concentration limit exceeded, spread too wide, or daily loss limit reached)
   **When** the verdict is computed
   **Then** the outcome is deterministic `BLOCK` with operator-facing explanation and an explicit next action expectation.
3. **Given** the daily loss limit is reached
   **When** subsequent eligible recommendations are evaluated during the same trading day
   **Then** all new risk is suppressed/blocked according to policy until the next allowed time window.
4. **Given** constraints depend on current positions/fills state
   **When** risk evaluation runs concurrently with order lifecycle updates
   **Then** constraint checks operate on a consistent snapshot so decisions are not based on partial intermediate state.
5. **Given** the system is in degraded mode (`REST_POLL`)
   **When** spread/concentration/daily-loss checks run
   **Then** degraded semantics remain correct and constraint logic does not re-enable entries forbidden by data-mode policy.
6. **Given** constraint enforcement outcomes are produced
   **When** traceability is enabled
   **Then** immutable audit evidence is written with `correlation_id` linking: recommendation -> constraint checks -> verdict reason code.
7. **Given** the same evaluation is retried (same idempotency key / `correlation_id`)
   **When** risk evaluation re-runs
   **Then** the constraint outcome and reason code remain identical (no oscillation for the same inputs/context).

### Story 2.7: Operator Console Positions & Lifecycle Inspection (FR27 + UX State Visibility)
As a technical operator,
I want the local console to display open positions, balances, and order lifecycle state clearly,
So that I can verify the system is tracking capital safely and reconciling actions correctly during the session.

**Acceptance Criteria:**
1. **Given** the operator is viewing the console during market hours
   **When** the console loads (normal refresh)
   **Then** it displays a “primary views” panel for positions, health/mode, and last state update, and those views load within the operator performance expectation (NFR4: ~2 seconds on normal local load).
2. **Given** there are open positions
   **When** the positions view is rendered
   **Then** it lists each open position with: symbol, entry price, current price, unrealized P&L, stop-loss, and time in position.
3. **Given** there are no open positions
   **When** the positions view is rendered
   **Then** it shows an explicit empty state (not a blank area) indicating the system is currently flat.
4. **Given** an order lifecycle transitions (e.g., partial fill, confirmed open, reject/timeout termination leading to no new position)
   **When** the corresponding position state changes
   **Then** the positions view reflects the new state consistently (no mismatch between order lifecycle status and displayed position state).
5. **Given** the operator views the active order lifecycle state for a symbol
   **When** they open lifecycle details for that symbol (or expand the row)
   **Then** the UI shows the current lifecycle state and at least the most recent transition reason/code.
6. **Given** the system is in degraded/policy states
   **When** the operator opens the console
   **Then** the UI still shows the active contract/mode and the “entries allowed” interpretation alongside the positions view (contract-first semantics, per UX).
7. **Given** console data retrieval fails temporarily (e.g., transient internal API error)
   **When** the console refresh runs
   **Then** it surfaces a user-facing error with a stable reason label and “next action” guidance (retry/wait), rather than silently hiding data.

### Story 2.8: Operator Notifications for Connectivity, Risk, Training, and Health (FR28 + UX)
As a technical operator,
I want the system to deliver notifications for connectivity, risk, training, and health events through a configured channel (Telegram/dashboard),
So that must-not-miss operational and governance signals are obvious immediately.

**Acceptance Criteria:**
1. **Given** WebSocket connectivity changes occur (disconnect/reconnect)
   **When** the system detects the event
   **Then** it sends an operator notification including the new data-mode contract (`WEBSOCKET` ↔ `REST_POLL`) and what it means for entry permission.
2. **Given** the risk layer blocks an action or enforces a constraint (e.g., risk rejection, daily loss block)
   **When** the rejection/block occurs
   **Then** the system sends a notification including stable reason code(s), operator-facing interpretation, and an explicit next action expectation.
3. **Given** scheduled retraining runs and completes
   **When** retraining succeeds
   **Then** a notification is sent indicating success and key identifiers (run id / model candidate id) without leaking secrets.
4. **Given** scheduled retraining runs and fails
   **When** retraining fails
   **Then** a notification is sent including failure reason codes / summary for operator action and debugging.
5. **Given** health/dependency readiness changes (e.g., dead-man’s-switch status, DB/broker session readiness)
   **When** the health state updates
   **Then** the system sends a notification summarizing “what changed” and the operator next safe action (ack/wait/open trace).
6. **Given** notifications can be delivered multiple times due to transient errors
   **When** the same notification is retried (same event correlation_id / event_id)
   **Then** the system deduplicates notification delivery and does not spam duplicate messages for the same event.
7. **Given** the system is in degraded mode (`REST_POLL`)
   **When** a new notification is generated
   **Then** it reflects degraded-mode semantics correctly and does not falsely imply new entries are allowed.

### Story 2.9: Readiness & Liveness Endpoints (FR29 + Dead-Man’s-Switch Semantics)
As a technical operator,
I want the system to expose readiness and liveness endpoints for independent watchdog processes,
So that “dead man’s switch” alerts and health gating behavior are reliable and operator-trustworthy.

**Acceptance Criteria:**
1. **Given** the system is running normally
   **When** `GET /health` is called
   **Then** it returns a response indicating main system liveness, and includes enough detail for watchdog/dead-man’s-switch logic.
2. **Given** critical dependencies are not ready (e.g., DB unreachable, broker session unhealthy, clock sync failure)
   **When** the operator or watchdog calls `/health`
   **Then** it still reports liveness, but indicates dependency readiness is insufficient to trade, using explicit alive-but-not-trade-ready semantics.
3. **Given** readiness is insufficient during market hours
   **When** the inference/risk pipeline attempts to process eligible recommendations
   **Then** the system blocks new entries (fail-closed) and surfaces an operator-visible reason consistent with dependency readiness.
4. **Given** `/health` polling runs repeatedly (watchdog behavior)
   **When** the system recovers dependencies
   **Then** readiness transitions to ready within a bounded timeframe, and subsequent `/health` reads reflect the correct state.
5. **Given** the system experiences transient failures during health evaluation
   **When** `/health` is requested again
   **Then** results are consistent and do not oscillate rapidly (no misleading “ready” blips while dependencies remain unhealthy).
6. **Given** dead-man’s-switch monitoring is enabled
   **When** the main system fails to respond within the watchdog timeout window during market hours
   **Then** the system triggers an operator notification/alert and records evidence that the watchdog fired.
7. **Given** `/health` output is used for operational automation
   **When** health/dead-man’s-switch incidents include stable identifiers
   **Then** incidents include stable IDs linking: alert -> incident -> operator trace via correlation_id (or run_id).

### Story 2.10: Blackout and Session-Window Rules (FR35) — Restrict/Throttle New Risk (with UX Contract Display)
As a technical operator,
I want the system to enforce blackout and session-window rules that restrict or throttle new risk,
So that the system never opens new risk outside allowed times and the UI explains why and when entries resume.

**Acceptance Criteria:**
1. **Given** blackout/calendar rules are configured (full trading blackouts, partial restrictions, observation-only windows, last-N-minutes no new positions)
   **When** the system evaluates the current market time for a symbol
   **Then** the system produces an operator-facing policy status indicating whether new risk is allowed.
2. **Given** the current time falls into a full trading blackout or an observation-only window
   **When** the system receives a recommendation that would create a new position
   **Then** the system blocks/suppresses the entry decision for that window (no new order submission).
3. **Given** partial restrictions apply (e.g., no new positions after 14:00 IST; no new positions in last 10 minutes with existing positions managed)
   **When** the system processes a recommendation during the restricted sub-window
   **Then** the system blocks only the entry portion while still allowing managed exits/stops for existing positions.
4. **Given** the last window for “no new positions” ends
   **When** the system evaluates new signals
   **Then** entries become allowed again only after data readiness requirements (freshness/data-mode gates) are satisfied.
5. **Given** blackout rules interact with degraded mode (`REST_POLL`)
   **When** both “entries suppressed by policy” and “entries suppressed by degraded semantics” apply
   **Then** the system maintains correct combined behavior: no accidental re-enabling of entries due to mode transitions.
6. **Given** the operator console is open
   **When** a policy restriction becomes active or ends
   **Then** the State/Contract Header reflects the updated contract and includes “entries allowed” interpretation and reason (contract-first UX).
7. **Given** a policy/calendar configuration update occurs outside market hours
   **When** the new configuration is loaded
   **Then** it is applied on the next allowed cycle (not retroactively during market hours) and the change is audit-logged.

### Story 2.11: Broker Session Authentication Lifecycle without Exposing Secrets (FR36)
As a technical operator,
I want the system to manage broker session authentication lifecycle (token/session refresh, expiry/re-auth, retry/recovery)
without ever exposing broker secrets through operator-facing APIs or logs,
So that broker connectivity stays reliable and risk actions remain secure.

**Acceptance Criteria:**
1. **Given** broker credentials/session material are stored in encrypted storage
   **When** the system needs to establish Breeze connectivity
   **Then** it authenticates successfully while ensuring secrets are never included in operator-facing API responses.
2. **Given** a broker session token has an expiry time
   **When** token expiry approaches or token becomes invalid
   **Then** the system refreshes/re-authenticates automatically according to policy (or triggers recovery flow if refresh fails).
3. **Given** re-authentication fails during market hours
   **When** the recovery window is exceeded
   **Then** the system enters a safe mode (e.g., position monitoring only / degraded semantics) and blocks new entries, and notifies the operator.
4. **Given** operator-facing endpoints are called (health, mode, positions, etc.)
   **When** endpoints render responses or UI refresh
   **Then** they never expose broker tokens, credentials, or raw session payloads in any response body.
5. **Given** transient failures occur during token refresh or re-auth attempts
   **When** retries are executed
   **Then** operator-facing notifications remain stable and non-spammy (idempotent per incident) and reason codes are consistent.
6. **Given** structured logs are enabled with correlation IDs
   **When** session auth success/failure events occur
   **Then** logs include correlation_id and stable reason codes, but redact secrets (no plaintext secrets in log lines).
7. **Given** the system transitions between connectivity modes (WS drop -> REST_POLL fallback -> reconnect/backfill)
   **When** broker session auth state changes during those transitions
   **Then** mode semantics remain correct and secrets are not leaked through any code path.

### Story 2.12: Explainability Summaries Tied to Each Recommendation (FR37)
As a technical operator,
I want the system to provide explainability summaries tied to individual recommendations (e.g., SHAP top features),
So that I can sanity-check decisions without bypassing the risk gateway.

**Acceptance Criteria:**
1. **Given** the system has generated a non-suppressed recommendation for a symbol
   **When** explainability is requested or auto-rendered in the UI
   **Then** the UI displays an explainability summary tied to that same `correlation_id` (or recommendation id).
2. **Given** explainability is computed from model features available at inference time
   **When** the system renders explainability
   **Then** the summary corresponds to the stored feature snapshot used to create the recommendation (no mismatch).
3. **Given** the system is in degraded mode (`REST_POLL`) or policy suppresses entries
   **When** a recommendation is suppressed or entries are blocked
   **Then** the UI must not present explainability in a way that implies actability; it either hides it or labels it “not actionable” while still showing the causal policy reason.
4. **Given** the operator opens a drill-down/trace context
   **When** they navigate to the explainability portion of the trace
   **Then** the explainability section is visible within that trace context and matches the same ids.
5. **Given** explainability is unavailable or still loading
   **When** the UI requests explainability
   **Then** the operator still sees stable “what is known” information and a reason if explainability cannot load, without losing the act/block reason and next action.
6. **Given** explainability is presented for sanity-checking
   **When** the operator attempts to override the decision
   **Then** the system enforces the risk gateway (explainability does not grant bypass permissions).
7. **Given** explainability assets are persisted for audit
   **When** the operator requests evidence after-the-fact
   **Then** the system can retrieve the explainability summary corresponding to that original recommendation.

### Story 2.13: Manual Override Reason Capture (FR38) — Auditable Operator Intent
As a technical operator,
I want the system to require a reason when I perform a manual override or exceptional action,
So that the override is auditable and traceable with operator intent in the evidence chain.

**Acceptance Criteria:**
1. **Given** the operator selects a manual override action in the console
   **When** the action is submitted
   **Then** the UI requires a non-empty reason input before the action can proceed.
2. **Given** an operator attempts to submit without providing a reason
   **When** they click confirm
   **Then** the system blocks the action and shows a user-facing validation error with a stable reason label.
3. **Given** a reason is provided
   **When** the manual override is executed (exceptional action path)
   **Then** the system persists the operator-provided reason text/metadata into the immutable audit trail linked to the same `correlation_id`.
4. **Given** the override affects a decision (act/block/order behavior)
   **When** the decision trace is later viewed
   **Then** the trace includes the operator-provided reason, tied to the same ids/correlation_id.
5. **Given** the operator retries the same override request (same idempotency key)
   **When** the retry is processed
   **Then** the system does not create duplicate override audit entries; it preserves the reason associated with that idempotency key.
6. **Given** the system is in degraded mode or locked policy state
   **When** the operator attempts a manual override
   **Then** the system still enforces reason capture and does not allow override to bypass risk gateway/contract semantics.
7. **Given** the UI is open and the operator is acting under stress
   **When** the reason flow is displayed
   **Then** it provides a clear operator “next action” expectation and does not allow silent state changes.

### Story 2.14: Bracket Orders + Protective Exit Parameters (FR14)
As a technical operator,
I want the system to place, amend, and close orders using protective exit parameters (bracket: stop-loss + target),
So that every open position has guaranteed exits on the broker and exit orders can be safely updated/cancelled/terminated per policy.

**Acceptance Criteria:**
1. **Given** an eligible recommendation reaches an `ACT` verdict and entries are allowed by contract/data-mode policy
   **When** the system executes the entry
   **Then** it places a **bracket order** via the internal execution/broker adapter using protective exit parameters (stop-loss + target) per policy/config.
2. **Given** bracket protective exit parameters are configured (defaults or policy values)
   **When** a bracket entry is placed
   **Then** the stop-loss and target values are persisted in the order intent/order record for audit.
3. **Given** bracket orders are placed and the system is tracking lifecycle state
   **When** the broker confirms entry acknowledgement and later confirms an exit via target or stop-loss
   **Then** the order lifecycle transitions through appropriate exit states (e.g., `EXIT_SENT -> EXIT_CONFIRMED -> FLAT`) and positions update consistently.
4. **Given** protective exit orders may need to be amended within configured limits
   **When** a policy-allowed amendment is requested (cancel/replace or amend exit orders)
   **Then** the system amends the protective exit orders, persists updated exit parameters, and maintains idempotency for the same `correlation_id`.
5. **Given** a protective exit order must be terminated because the position is being flattened (e.g., cancel/close due to policy/kill/termination path)
   **When** the close is processed
   **Then** the system ensures protective exits are not left dangling: it cancels/terminates exit orders per broker semantics and records terminal outcomes.
6. **Given** execution is in paper mode
   **When** a bracket entry is executed
   **Then** the system simulates protective exits consistently with the same policy parameters and records lifecycle transitions (marked as simulated) for later reconciliation.
7. **Given** the same bracket placement/amend request is retried (same `correlation_id` / idempotency key)
   **When** the retry is handled
   **Then** the system does not create duplicate entry/exit orders or duplicate lifecycle events; it reuses/updates the existing persisted lifecycle.

## Epic 3: Deterministic Replay & Backtest Evidence
Epic goal: Simulate prior sessions through the same decision pipeline and compare performance across time splits/benchmarks under a shared cost model.

### Story 3.1: Historical Replay Tool with Simulation Clock + Data Source Hierarchy (FR18)
As a technical operator,
I want the system to replay any past trading date through the production model using a simulation clock and the same decision pipeline as live,
So that I can produce a complete record of signals, trades, and P&L with no look-ahead bias and realistic fills.

**Acceptance Criteria:**
1. **Given** a past trading date is selected for replay
   **When** the replay tool is executed
   **Then** it replays that full day through the current production model and produces an output dataset containing every signal, every trade, and resulting P&L.
2. **Given** the replay requires market data for the full day
   **When** the replay fetches data for each symbol/timestamp range
   **Then** it uses the same data source hierarchy as live operation: stored database records first, and REST API only to fill gaps.
3. **Given** the replay is running with simulation clock mode enabled
   **When** the model computes a simulated signal at timestamp `T`
   **Then** the model receives only data that would have been available at exactly `T` (no look-ahead bias).
4. **Given** a signal is generated at timestamp `T`
   **When** the system simulates order execution
   **Then** fills occur at the next available price after `T` (not at the signal bar close).
5. **Given** transaction cost and slippage models are configured
   **When** each simulated trade is priced
   **Then** the replay applies transaction costs (brokerage/STT/exchange charges/GST on brokerage/stamp duty) and applies a slippage model consistent with order size and stock liquidity.
6. **Given** replay runs for multiple symbols in the same session
   **When** the replay completes
   **Then** the output is complete for all replayed symbols and includes data lineage identifiers needed to explain gap-filled behavior.
7. **Given** replay is retried (same replay configuration/version)
   **When** the same date replay is executed again with the same version
   **Then** results remain consistent and the replay output does not produce duplicate/contradictory records for that replay version.

### Story 3.2: Performance Comparison (Time Splits / Benchmarks) under Shared Cost Model (FR19)
As a technical operator,
I want the system to compare strategy performance across time splits and benchmarks using the same cost model,
So that I can validate robustness (and avoid overfitting) before promotion.

**Acceptance Criteria:**
1. **Given** a replay/backtest run has produced complete records for a date/split
   **When** the performance comparison tool runs
   **Then** it aggregates strategy metrics across time splits (e.g., walk-forward windows) and reports risk-adjusted performance.
2. **Given** benchmarks are configured (random generator, EMA crossover, Nifty 50 buy-and-hold)
   **When** comparisons are generated for the same time splits
   **Then** each benchmark is evaluated with the same position sizing and the same transaction cost + slippage model as the strategy.
3. **Given** transaction cost and slippage models are configured
   **When** simulated trades are priced in each split
   **Then** the resulting metrics include brokerage, STT, NSE exchange charges, GST on brokerage, stamp duty, and modeled slippage consistently.
4. **Given** the replay tool outputs per-trade results with P&L linked to signals
   **When** benchmark comparisons are computed
   **Then** the system produces a complete record of every signal and every trade impact used for the comparison.
5. **Given** the system applies the anti-overfitting mandate (permanent validation set used only for final evaluation)
   **When** performance is reported for a candidate model
   **Then** the tool indicates which dataset/period each metric came from and that metrics correspond to out-of-sample evaluation.
6. **Given** stress-test periods are configured monthly (e.g., worst drawdowns / known crisis windows)
   **When** stress comparisons run
   **Then** the tool reports performance across those stress periods in a way that supports operator review and decision gates.
7. **Given** comparisons are run repeatedly with the same configuration
   **When** rerun occurs
   **Then** results are stable (idempotent) and do not drift due to non-deterministic components when configuration is unchanged.

### Story 4.1: Training Label Generation from Stored Recommendations + Outcomes (FR20)
As a technical operator,
I want the system to derive training labels from stored recommendations and subsequent price outcomes,
So that retraining uses correctly labeled historical evidence (including reliability differences by data source).

**Acceptance Criteria:**
1. **Given** a stored recommendation exists with a signal direction (Buy/Sell/Hold context as defined by the pipeline) and a configured target window + threshold
   **When** label generation runs
   **Then** the system computes the subsequent price movement within the target window and assigns exactly one label: `Correct Up`, `Correct Down`, `Incorrect`, or `Inconclusive`.
2. **Given** the recommendation’s signal direction is `Buy`
   **When** the price rises more than the configured threshold within the target window
   **Then** the assigned label is `Correct Up`.
3. **Given** the recommendation’s signal direction is `Sell`
   **When** the price falls more than the configured threshold within the target window
   **Then** the assigned label is `Correct Down`.
4. **Given** the recommendation’s signal direction is `Buy`
   **When** the price does not rise beyond the configured threshold within the target window
   **Then** the assigned label is `Incorrect` if direction is wrong per label rules, otherwise `Inconclusive` if price remained within the threshold range.
5. **Given** the ticks used to generate the label come from multiple data sources
   **When** the label record is written
   **Then** the system stores the tick `data_source` used for that label, and flags `REST_POLL`-sourced labels as lower reliability.
6. **Given** the label-generation job is retried for the same recommendation set (same recommendation ids)
   **When** label generation re-runs
   **Then** it does not create duplicate label records for the same recommendation (idempotent label write keyed by recommendation id).
7. **Given** label generation cannot complete for some recommendations due to missing required post-window price data
   **When** label generation runs
   **Then** those recommendations are left in a “pending/not labeled” state (not incorrectly labeled), and the operator/logs can identify which recommendations were not labeled and why.

### Story 4.2: Nightly Retraining Procedure + Model Version Archival (FR21)
As a technical operator,
I want the system to run a scheduled nightly retraining job at the configured time and archive every model version with training metadata,
So that model promotion/rollback is based on complete evidence without ad hoc manual steps.

**Acceptance Criteria:**
1. **Given** the scheduled retraining job is enabled
   **When** the job reaches the configured nightly time (22:00 IST)
   **Then** it starts automatically and creates a retraining run id.
2. **Given** labeled recommendations exist
   **When** retraining runs
   **Then** the training pipeline uses accumulated data for training and excludes the permanent validation set from training/hyperparameter tuning.
3. **Given** retraining runs
   **When** feature engineering and model training completes
   **Then** it evaluates the candidate model and produces validation metrics (accuracy, Sharpe ratio, maximum drawdown, precision, recall).
4. **Given** retraining completes successfully
   **When** the system archives the new model version
   **Then** it stores training date, data date range, validation metrics, and the git commit hash for lineage.
5. **Given** retraining runs at night
   **When** the new model is not approved for deployment by gates
   **Then** the current production model is retained and the failure/decision is logged (no silent deployment).
6. **Given** the scheduled job fails due to an intermediate error
   **When** the failure occurs
   **Then** the run is marked as failed with an operator-visible reason code and no partial/invalid model version is promoted.
7. **Given** the scheduled job is triggered again with the same run configuration (idempotent retry)
   **When** the second trigger is handled
   **Then** it does not create duplicate/contradictory archived model versions for the same run id/config.
