---
project_name: 'algotrade'
user_name: 'Darthrax'
date: '2026-03-20T14:51:05+05:30'
sections_completed: ['technology_stack', 'language_specific', 'framework_specific', 'testing_rules', 'code_quality_style', 'development_workflow_rules']
existing_patterns_found: 5
---

# Project Context for AI Agents

_This file contains critical rules and patterns that AI agents must follow when implementing code in this project. Focus on unobvious details that agents might otherwise miss._

---

## Technology Stack & Versions

_Intermediate mode: documented after discovery phase_

### Core Technologies (planned / architecture-specified)
- FastAPI backend (`TBD` exact version)
- Postgres with TimescaleDB for tick/time-series persistence (`TBD`)
- Alembic for DB migrations (`TBD`)
- APScheduler for scheduled jobs (`TBD`)
- Dockerized local development (planned `docker-compose.yml`, `Dockerfile.api`, `Dockerfile.worker`) (`TBD`)

### Key Dependencies (integration & operational constraints)
- ICICI Breeze integration via:
  - WebSocket for live incremental data (`breeze-connect` official SDK, `TBD` version)
  - REST for bootstrap, gap-fill, and fallback polling (`breeze-connect` REST methods, `TBD` version)
- Deployment constraint: static IP mandatory for Breeze API-based trading starting April 2026

### Version/compatibility note for agents
- Until lockfiles/manifests exist in-repo, treat “exact versions” as `TBD` and rely on the architecture contracts and module boundaries as the compatibility source of truth.

## Critical Implementation Rules

### Language-Specific Rules

- **Python naming (agents writing Python):**
  - `snake_case` for functions/variables
  - `PascalCase` for classes
  - `UPPER_SNAKE_CASE` for constants

- **Schema + event naming (applies to Python code that defines/uses them):**
  - DB schema: `snake_case` for tables/columns/indexes/constraints
  - Events (async/lifecycle): `snake_case` event names (e.g. `signal_emitted`, `risk_rejected`, `order_state_changed`)
  - Identifiers: suffix by role (`*_id`, `correlation_id`, `model_version_id`, `signal_id`, `order_id`)

- **Serialization / API contracts:**
  - Timestamps everywhere in APIs + audit/events are **ISO-8601 UTC** (example: `2026-03-20T08:45:00Z`)
  - JSON keys are **`snake_case`** for API payloads and event payloads
  - All API endpoints return the same wrapper shape:
    - Success: `{ "ok": true, "data": ..., "correlation_id": "..." }`
    - Failure: `{ "ok": false, "error": { "code": "...", "message": "...", "correlation_id": "..." } }`

- **Error handling + “no silent success”:**
  - Use **stable machine error codes** to distinguish validation errors vs risk rejections vs broker transient/permanent failures
  - “Suppressed/block-by-policy” decisions (e.g., `REST_POLL` entry suppression) must emit a traceable event and an operator-facing reason code

- **Async idempotency requirement (implemented in Python async consumers):**
  - Every async handler must tolerate duplicate deliveries using `correlation_id` plus a persisted idempotency key (e.g., `event_id` or equivalent persisted key)

### Framework-Specific Rules (FastAPI + APScheduler)

- **FastAPI control-plane organization:**
  - Control-plane endpoints live under `src/algotrade/api/routers/` and are composed via `APIRouter`.
  - “Signal -> deterministic risk/execution -> order submission” must route through `services/risk_execution/` (no bypass paths in routers).

- **Security enforcement for privileged endpoints:**
  - Privileged routes (`POST /kill`, `POST /mode`) require the shared secret token via `Authorization` header.
  - Privileged token checks must be implemented as FastAPI dependencies (e.g., via `Depends`) in `security/` modules.
  - Never log raw tokens or the full `Authorization` header; redact any secret-bearing values.

- **Consistent response/error contracts (FastAPI):**
  - All endpoints must return the standard wrapper:
    - Success: `{ "ok": true, "data": ..., "correlation_id": "..." }`
    - Failure: `{ "ok": false, "error": { "code": "...", "message": "...", "correlation_id": "..." } }`
  - Use stable machine error `code`s to distinguish validation errors vs risk rejections vs broker transient/permanent failures (even if HTTP status varies).

- **Fail-closed runtime behavior:**
  - If risk/execution cannot complete (exceptions, downstream unhealthy, etc.), endpoints must return `ok: false` and must not advance order state into “submitted” / “new risk action” pathways.

- **APScheduler job framework rules (background work):**
  - Scheduled jobs in `src/algotrade/jobs/` derive work from durable records/events in the DB.
  - Jobs must be idempotent: they must tolerate re-runs and duplicate deliveries using persisted idempotency keys (`event_id`/equivalent) and/or `correlation_id`.
  - Background jobs must not submit orders directly; they may only trigger/prepare durable events that the decision spine consumes.

- **Startup/readiness gating:**
  - The FastAPI application lifecycle must ensure broker connectivity health checks and required “gap fill + reconciliation gating” happens before accepting new trading actions.
  - Health/readiness endpoints in the API must expose enough operator signal for watchdog/dead-man’s-switch semantics.

### Testing Rules

- **Unit vs integration boundaries:**
  - Unit tests cover pure/domain logic (risk state machine transitions, naming/serialization helpers, idempotency key logic).
  - Integration tests cover DB persistence, outbox/durable-event reads, and broker adapter translation.

- **Idempotency correctness tests:**
  - Tests must include duplicate-delivery scenarios for async consumers, verifying idempotency keys prevent re-triggering capital actions.

- **Fail-closed behavior tests:**
  - When risk/execution dependencies are unhealthy or raise, tests must assert:
    - endpoints return `ok: false`
    - no state transitions into “submitted/new risk action” pathways

- **Decision-pipeline parity tests (replay vs live):**
  - Deterministic replay/backtest must run through the same decision pipeline as live/paper (except the final broker submission switch).
  - Tests should validate equivalence of outputs across replay runs for the same inputs.

- **Contract/shape tests for API responses:**
  - Tests must validate the standard wrapper shape:
    - Success: `{ "ok": true, "data": ..., "correlation_id": "..." }`
    - Failure: `{ "ok": false, "error": { "code": "...", "message": "...", "correlation_id": "..." } }`
  - Ensure timestamps are UTC/ISO-8601.

- **Coverage expectation (lean but essential):**
  - Prioritize tests for:
    - ordering/risk state machine edges
    - suppressed/block-by-policy cases producing traceable events
    - broker-vs-internal reconciliation gating logic

### Code Quality & Style Rules

- **Respect the decision-spine boundaries in code organization:**
  - `ml/` must only produce signals/candidate models.
  - `services/risk_execution/` is the only path from “signal proposal” to “order submission”.
  - `adapters/` must translate broker payloads to domain shapes without embedding decision logic.

- **Deterministic/pure code where possible:**
  - Domain logic should be written to be deterministic for a fixed input (especially replay/backtest decision pipeline).
  - Avoid hidden global state in decision code.

- **Type hints and explicit contracts:**
  - Add type annotations for public boundaries (API request/response shapes, domain entities, repository interfaces).
  - Prefer explicit data shapes over implicit dict-shaped payloads in critical paths.

- **Naming & structure consistency (reinforced):**
  - Continue enforcing `snake_case`/`PascalCase`/`UPPER_SNAKE_CASE`.
  - Ensure correlation/linkage fields are consistently named (`correlation_id`, `*_id`, etc.).

- **Minimal, high-signal documentation:**
  - Comments should explain “why” for non-obvious invariants (fail-closed, idempotency, suppression semantics).
  - Do not document obvious behavior; keep the agent context lean.

- **Security hygiene in code:**
  - Never log raw secrets or full `Authorization` header values.
  - Ensure token handling and privileged action checks are centralized (dependency/module), not scattered.

### Development Workflow Rules (repo/implementation)

- **Decision-spine boundary enforcement (conflict prevention):**
  - Code changes must preserve that `services/risk_execution/` is the only route to order submission.
  - When adding new modules, ensure adapters/services do not introduce bypass paths.

- **Commit message discipline (Conventional Commits):**
  - Use `<type>(<scope>): <subject>` format.
  - Keep subject concise and in imperative mood (per repo rules).

- **Change grouping discipline:**
  - Don’t batch unrelated meaningful changes into a single commit.
  - Treat security/risk-behavior changes as separate, reviewable increments.

- **PR review expectations (behavioral focus):**
  - Reviews should prioritize:
    - fail-closed correctness under risk/execution failures
    - idempotency under duplicate async deliveries
    - audit/traceability integrity (`correlation_id` chain)

- **Operational parity checks as part of implementation:**
  - Ensure deterministic replay/backtest uses the same decision pipeline code paths as live/paper up to the final broker submission switch.

