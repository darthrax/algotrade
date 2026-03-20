---

## Architecture Validation Results

### Coherence Validation ✅
**Decision Compatibility:**
- `TimescaleDB on Postgres` aligns with authoritative tick/time-series persistence and retention/compression.
- Auth/security choices protect risk-increasing endpoints (`POST /kill`, `POST /mode`) without weakening the fail-closed execution spine.
- Hybrid control-plane (sync, deterministic, fail-closed) + async propagation (durable events) supports ML-first “no later integration surgery”.

**Pattern Consistency:**
- Naming/format/communication/process rules enforce the key invariants: no bypass paths, idempotent async consumers, and auditable privileged actions.

**Structure Alignment:**
- The physical tree includes `ml/`, `services/risk_execution/`, `adapters/breeze/`, `audit/`, `jobs/`, `operator/`, and DB repositories so agents have non-overlapping ownership boundaries.

### Requirements Coverage Validation ✅
**Functional Requirements Coverage:**
- PRD capability clusters are implementable within the proposed boundaries (ingestion/degraded modes, feature->signal suppression rules, risk/execution state machine + kill switch, deterministic replay/backtesting, training + champion/challenger promotion/rollback gates, operator visibility + watchdog, reconciliation/export/audit, config lockout semantics, explainability tie-ins).

### Implementation Readiness Validation ✅
**Decision Completeness:**
- Critical decisions, patterns, and boundaries are documented so multiple agents can implement compatibly.

### Gap Analysis Results
**Important (non-blocking) gaps to tighten later:**
- Add an explicit FR-category (or FR-bucket) to directory/module mapping table for stronger traceability.
- Add an “observability + deployment policy” section (what to measure, where dashboards live, how failure modes are surfaced) to make NFRs more operationally testable.

### Architecture Readiness Assessment
**Overall Status:** READY FOR IMPLEMENTATION

**Key Strengths:**
- Invariant-driven architecture: risk/execution is the only order path; ML cannot bypass it.
- Durable-source async model fits replay/recovery and retraining correctness.
- Clear agent ownership boundaries reduce cross-module conflicts.

stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
inputDocuments:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/ux-design-specification.md
  - _bmad-output/planning-artifacts/ux-design-directions.html
  - _bmad-output/planning-artifacts/research/technical-ai-powered-nse-bse-intraday-trading-in-india-research-2026-03-20.md
  - _bmad-output/planning-artifacts/research/domain-ai-powered-nse-bse-intraday-trading-in-india-research-2026-03-20.md
  - _bmad-output/planning-artifacts/research/market-ai-powered-nse-bse-intraday-trading-in-india-research-2026-03-20.md
  - docs/AlgoTrade_India_Product_Brief_v1.1.md
workflowType: 'architecture'
project_name: 'algotrade'
user_name: 'Darthrax'
date: '2026-03-20T13:41:49+05:30'
lastStep: 8
status: 'complete'
completedAt: '2026-03-20T14:38:44+05:30'
---

# Architecture Decision Document
_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis
### Requirements Overview
**Functional Requirements:**

From the PRD’s functional requirement list (`FR1`–`FR39`), the system must cover these capability clusters (39 total FRs):

- **Market data ingestion + persistence:** real-time ingestion, historical retrieval/backfill, duplicate/out-of-sequence handling, missing-data detection/alerts, degraded quote-mode behavior.
- **Feature engineering + inference:** compute only valid/available features; emit signals with confidence/regime context; suppress signals when freshness/data-mode policy forbids it; persist feature context for audit.
- **Risk + execution:** evaluate each recommendation against pre-trade checks; enforce limits (position/concentration/daily loss/etc.); manage an order lifecycle state machine (partial fills, timeouts, rejects); support emergency kill switch; fail-closed behavior when risk layer is unavailable.
- **Backtesting + replay:** deterministic simulation/replay through the same decision pipeline; compare performance across time splits/benchmarks.
- **Model training + governance:** derive labels from stored outcomes; run scheduled training jobs; store/retrieve versioned artifacts with lineage; compare candidate models vs champion under agreed gates; withhold promotion or rollback when gates fail; support human approval where policy requires it.
- **Operations + alerting:** operator can inspect positions/health; system sends notifications for connectivity/risk/training/health; readiness/liveness endpoints to support watchdog/dead-man’s-switch semantics.
- **Compliance + reporting:** export trade/charges/summaries for tax/accounting; track turnover against audit thresholds; record immutable audit trails for signal/order/fills/config/model promotions; produce day-end reconciliation artifacts.
- **Configuration, calendar, and explainability:** configuration changes only in allowed windows with audit; session/calendar blackout rules; operator override reason capture; explainability summaries tied to recommendations.

**Non-Functional Requirements:**

The PRD + UX spec + product brief together impose these NFR drivers (all of which shape architectural decisions via contracts and guardrails):

- **Performance / timeliness:** market-data freshness SLOs, signal/decision latency targets, broker ack latency SLOs, operator console load targets, retraining completion within the scheduled window, and restart recovery gap-fill before accepting new entries.
- **Security / integrity:** secrets encrypted at rest and not logged in plaintext; authenticated/authorized access for risk-increasing actions; tamper-evident append-only audit discipline with integrity checksums; TLS for remote operations; least-privilege boundaries.
- **Reliability / fail-safety:** risk/execution must fail closed (unavailable risk path => no new orders); degraded data modes must have defined behavioral semantics (not “best effort”); watchdog/dead-man’s-switch alerting behavior.
- **Integration contracts:** strict treatment of broker limits (rate/burst/subscription constraints), data-provider payload quality expectations, and broker vs internal reconciliation accuracy requirements.
- **UX quality constraints (operator safety):** accessibility requirements (WCAG Level AA target), explicit contract-first semantics for Normal vs Degraded vs Locked behavior, and fast operator comprehension (“glance → trust state → next safe action”).
- **Regulatory / compliance posture:** SEBI retail algo expectations (track-and-trace metadata, registration/accountability constraints) and India privacy-awareness (data minimization/retention/access logging for any personal data processed by dashboards/notifications).

**Scale & Complexity:**

- **Complexity level:** high (capital loss risk + regulated constraints + real-time integration fragility + long-lived governance/audit obligations).
- **Primary technical domain(s):** API backend + ML/data pipeline + execution/risk operations + operator UX + governance tooling.
- **Estimated architectural components (at a planning level):**
  - Layered core for ingestion, features/inference, risk/execution, training, and operator-facing observability.
  - Cross-cutting components for: audit logging + reconciliation, configuration/calendar policy enforcement, degraded-mode/data-mode gating, and model governance/promotion controls.

### Technical Constraints & Dependencies
Key constraints and dependencies called out across the loaded docs:

- **Broker integration shape:** Breeze WebSocket for live incremental data and Breeze REST for bootstrap, startup gap fill, and fallback modes; session token lifecycle and retry/re-auth behavior must be handled without exposing secrets via operator APIs.
- **Static network requirement:** static IP is mandatory for Breeze API-based trading starting April 2026 (operational dependency that affects deployment/runbook decisions).
- **Time/session correctness:** NTP discipline, and behavior keyed to India market session windows; blackout/lockout rules restrict configuration and/or new entries by policy.
- **Data-mode semantics:** system must explicitly represent and enforce `WEBSOCKET` vs `REST_HISTORICAL` vs `REST_BACKFILL` vs `REST_POLL` differences; `REST_POLL` has defined “entries suppressed by policy” behavior that must be preserved across the stack.
- **Deterministic replay requirement:** simulation clock and replay tools must run through the same decision path used during live trading to avoid “paper vs live” divergence.
- **Auditability:** append-only event logging for signal/order/fills/config/model changes; UTC timestamps; daily checksums; 3-year retention requirement.
- **Model governance:** champion/challenger plus promotion/rollback gates; leakage prevention (timing/data availability constraints) and drift monitoring; human approval flows where required by policy.
- **Operational resilience:** graceful shutdown procedure and startup recovery (gap fill + reconciliation gating) before trading proceeds.
- **Operational automation:** scheduled jobs for training and recurring performance reporting, plus watchdog/dead-man’s-switch health semantics.

### Cross-Cutting Concerns Identified
These concerns span multiple capability clusters and will need consistent architectural contracts:

- **Fail-closed execution:** ML can propose; risk/execution is the only path to order submission; when the risk layer is unavailable, the system must not open new risk.
- **Policy-driven behavior under degradation:** degraded modes must not silently change behavior; entry suppression, monitoring-only semantics, and operator visibility must be consistent across backend + UI.
- **Paper/live parity:** identical decision logic until the final order submission switch; deterministic replay reduces the chance of untested divergences.
- **Traceability chain:** end-to-end correlation across “signal → risk decision → broker action → fills/outcomes → model version/lineage”.
- **Configuration and governance lockouts:** configuration is locked during market hours; governance/admin actions are versioned and audited; promotion decisions must be attributable and rollback-able.
- **Reconciliation as a gate:** broker vs internal reconciliation outcomes shape go/no-go for next-day live readiness; mismatches must block (or be waived with auditable notes).

This drafted context matches what your PRD/UX/research/product-brief describe as the project’s scope and constraints.

## Starter Template Evaluation
### Primary Technology Domain
FastAPI backend + Postgres persistence + migrations + local Dockerized development environment.

### Selected Starter Approach
**From scratch (with ML baked in immediately)**

### Rationale for Selection
You want the ML pipeline to be a first-class capability of the architecture from day one, not a later integration step. A generic starter is usually scaffolding for only the API/UI skeleton; to avoid architectural drift, we will build the repo structure and module boundaries to enforce the ML -> feature engineering -> signal generation -> risk/execution -> outcomes labeling -> retraining -> promotion/rollback gates as an integrated control spine from the first implementation story.

### Initialization Scaffolding (what we build first)
- Internal “decision spine” module boundaries so ML cannot bypass risk/execution.
- Persistence + schema evolution support (migrations) aligned with auditability requirements.
- Deterministic replay/backtest hooks as a parallel path to live/paper behavior.
- Scheduled training orchestration + model governance/promotion gates wired into the system lifecycle.

---

## Core Architectural Decisions
### Data Architecture
- **Database choice:** TimescaleDB on Postgres (authoritative tick/time-series store)
- **Why it fits your requirements:**
  - Supports tick/time-series ingestion with relational integrity for state and reconciliation.
  - Enables time-series lifecycle management (retention/downsampling/compression) so tick volume doesn’t degrade operational viability.
  - Keeps the data contracts aligned with your PRD: ticks persist with `data_source` lineage, while feature snapshots, signals, labels, and trade outcomes remain reproducible for deterministic replay and audit.

### Authentication & Security
- **Privileged shared secret token** for risk-increasing internal API routes (notably `POST /kill` and `POST /mode`), sent via an `Authorization` header.
- **Token scoping:** separate tokens for the operator versus automation, so you can audit intent and limit authority per actor.
- **Security requirements:** tokens must not be logged in plaintext; strong authentication is required for risk/session changes; all privileged actions must create auditable entries tied to the same correlation_id/log chain used for the order/risk lifecycle.

### API & Communication Patterns
- **Control-plane is synchronous; propagation is asynchronous**:
  - ML/feature engineering proposes signals and triggers deterministic risk/execution via internal FastAPI endpoints (critical path).
  - Lifecycle changes are also persisted as durable events so background consumers can safely trigger retraining, dashboards, and notifications.
- **Durable-source-of-truth async**: event consumers must read from the DB (or an outbox-like durable record) so they can recover and replay safely after restarts.
- **Idempotency + correlation_id** are mandatory: every async consumer must tolerate duplicate deliveries without re-triggering capital actions.


## Implementation Patterns & Consistency Rules
### Naming Patterns
- **Python:** `snake_case` functions/variables, `PascalCase` classes, `UPPER_SNAKE_CASE` constants.
- **DB schema:** `snake_case` for tables/columns/indexes/constraints.
- **Events (async/lifecycle):** `snake_case` event names (e.g. `signal_emitted`, `risk_rejected`, `order_state_changed`).
- **Identifiers:** suffix by role (`*_id`, `correlation_id`, `model_version_id`, `signal_id`, `order_id`).

### Structure Patterns (module/package boundaries)
Maintain a single “decision spine” oriented layout so agents cannot accidentally create bypass paths:
- `api/` (FastAPI routes: `signal`, `order`, `kill`, `mode`, `health`, etc.)
- `security/` (auth + privilege checks for risk-increasing calls)
- `domain/` (typed entities: `Signal`, `OrderIntent`, `OrderState`, `FillOutcome`, `AuditEvent`)
- `adapters/` (ICICI Breeze WS/REST translation into domain shapes only)
- `services/`
  - `risk_execution/` (pre-trade checks, state machine transitions, fail-closed behavior)
  - `reconciliation/` (broker vs internal alignment logic)
- `ml/`
  - `features/`, `inference/`, `label_generation/`
  - `training/`, `model_governance/` (champion/challenger + promotion/rollback gates)
- `jobs/` (APScheduler tasks: retraining, health polling, reconciliation runs)
- `audit/` (shared append-only audit writer utilities)

### Format Patterns
- **API response wrapper:** all endpoints return the same shape.
  - Success: `{ "ok": true, "data": ..., "correlation_id": "..." }`
  - Failure: `{ "ok": false, "error": { "code": "...", "message": "...", "correlation_id": "..." } }`
- **Timestamps:** APIs and audit/events use ISO-8601 UTC timestamps (e.g. `2026-03-20T08:45:00Z`).
- **JSON keys:** `snake_case` for API payloads and event payloads.

### Communication Patterns (hybrid control-plane + async propagation)
- **Control-plane is synchronous; propagation is asynchronous.**
  - Critical path (feature/signal -> deterministic risk/execution) is synchronous and fail-closed.
  - Async consumers (retraining, dashboards, notifications) may be eventually consistent but must not create/submit orders.
- **Durable-source-of-truth:** async consumers must read from durable records (DB/outbox-like durable events) so recovery/replay is correct after restarts.
- **Idempotency:** every async handler must tolerate duplicate deliveries using `correlation_id` plus a persisted `event_id` (or an equivalent persisted idempotency key).
- **Traceability chain:** `signal -> risk decision -> order state changes -> fills/outcomes -> model lineage`, all linked by `correlation_id`.

### Process Patterns (agent conflict prevention)
- **Fail-closed rule:** any risk/execution error results in “no new risk actions”; order submission is disabled if the risk service is unhealthy.
- **Stable error codes:** define machine codes that distinguish validation errors vs risk rejections vs broker transient/permanent failures.
- **No silent success:** suppressed/block-by-policy decisions (e.g. `REST_POLL` entry suppression) must emit a traceable event and operator-facing reason code.
- **Audit-first privileged actions:** privileged endpoints (`POST /kill`, `POST /mode`) must write auditable events tied to the same correlation/log chain.

---
## Project Structure & Boundaries (Step 6)

### Complete Proposed Project Directory Structure
```text
.
├── pyproject.toml
├── README.md
├── .env.example
├── docker-compose.yml
├── Dockerfile.api
├── Dockerfile.worker
├── alembic.ini
├── alembic/
│   ├── env.py
│   └── versions/
├── scripts/
│   ├── dev_up.sh
│   ├── db_migrate.sh
│   ├── db_seed.sh
│   ├── run_api.sh
│   ├── run_worker.sh
│   └── run_operator.sh
├── src/
│   └── algotrade/
│       ├── __init__.py
│       ├── config/
│       │   ├── __init__.py
│       │   └── settings.py
│       ├── audit/
│       │   ├── __init__.py
│       │   └── event_writer.py
│       ├── domain/
│       │   ├── __init__.py
│       │   ├── signal.py
│       │   ├── order_intent.py
│       │   ├── order_state.py
│       │   ├── fill_outcome.py
│       │   ├── audit_event.py
│       │   └── value_objects.py
│       ├── security/
│       │   ├── __init__.py
│       │   ├── auth.py
│       │   └── privileged_tokens.py
│       ├── adapters/
│       │   ├── __init__.py
│       │   └── breeze/
│       │       ├── __init__.py
│       │       ├── ws_client.py
│       │       └── rest_client.py
│       ├── services/
│       │   ├── __init__.py
│       │   ├── risk_execution/
│       │   │   ├── __init__.py
│       │   │   ├── pre_trade_checks.py
│       │   │   ├── order_state_machine.py
│       │   │   ├── fail_closed.py
│       │   │   └── execution_api.py
│       │   └── reconciliation/
│       │       ├── __init__.py
│       │       └── broker_vs_internal.py
│       ├── ml/
│       │   ├── __init__.py
│       │   ├── features/
│       │   │   ├── __init__.py
│       │   │   └── feature_builder.py
│       │   ├── inference/
│       │   │   ├── __init__.py
│       │   │   └── signal_inference.py
│       │   ├── label_generation/
│       │   │   ├── __init__.py
│       │   │   └── label_generator.py
│       │   ├── training/
│       │   │   ├── __init__.py
│       │   │   └── train.py
│       │   ├── model_governance/
│       │   │   ├── __init__.py
│       │   │   ├── promotion_gates.py
│       │   │   └── rollback_policy.py
│       │   └── backtesting_replay/
│       │       ├── __init__.py
│       │       └── replay_engine.py
│       ├── api/
│       │   ├── __init__.py
│       │   ├── main.py
│       │   └── routers/
│       │       ├── __init__.py
│       │       ├── signal.py
│       │       ├── order.py
│       │       ├── positions.py
│       │       ├── pnl.py
│       │       ├── health.py
│       │       ├── kill.py
│       │       ├── mode.py
│       │       └── data_status.py
│       ├── jobs/
│       │   ├── __init__.py
│       │   ├── scheduler.py
│       │   ├── retraining_job.py
│       │   ├── health_watchdog_job.py
│       │   └── reconciliation_job.py
│       ├── operator/
│       │   ├── __init__.py
│       │   └── app.py
│       └── db/
│           ├── __init__.py
│           ├── session.py
│           ├── models.py
│           └── repositories/
│               ├── __init__.py
│               ├── tick_repo.py
│               ├── lifecycle_repo.py
│               └── audit_repo.py
└── tests/
    ├── unit/
    ├── integration/
    └── e2e/
```

### Architectural Boundaries (where decisions must not bypass)
- `ml/` produces signals and candidate models only.
- `services/risk_execution/` is the only component allowed to move from “signal proposal” to “order submission”.
- `adapters/breeze/` isolates broker payload/connection mechanics.
- Async consumers (retraining/dashboard/notifications) must derive work from durable records (DB/outbox-like durable events) and must not place orders.

---

