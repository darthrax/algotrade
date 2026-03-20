# Story 1.1: WebSocket Tick Ingestion + Dedup Persistence
Status: ready-for-dev

## Story
As a technical operator,
I want the system to validate and persist incoming Breeze WebSocket ticks with dedup and monotonic timestamp guarantees,
So that the downstream feature/prediction pipeline has a clean, auditable, continuously growing time series.

## Acceptance Criteria
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

## Tasks / Subtasks
- [ ] Implement Breeze WebSocket tick ingestion path for `data_source = 'WEBSOCKET'`
  - [ ] Subscribe to active watchlist symbols (from configured watchlist state)
  - [ ] Map incoming Breeze tick payloads into a domain tick model with explicit `stock_token`, `timestamp`, `resolution`, `price`, `volume`, and optional circuit-limit metadata
  - [ ] Ensure ingestion path is safe under duplicate and bursty deliveries (idempotent handling + bounded retries)
- [ ] Enforce monotonic timestamp + out-of-order rejection
  - [ ] For each `(stock_token, resolution)` maintain a durable “last stored timestamp” check (query or cache backed by durable truth)
  - [ ] If `incoming.timestamp <= last_timestamp`, discard and emit a warning/debug log (no crash, no insert attempt)
- [ ] Enforce dedup persistence with deterministic primary key behavior
  - [ ] Define/ensure ticks table primary key `(stock_token, timestamp, resolution)`
  - [ ] Upsert/dedup behavior must result in: duplicates do not create new rows and are discard-logged
- [ ] Enforce data quality guards (zero price/volume + tolerance band)
  - [ ] Discard ticks where `price == 0` or `volume == 0` (no insert)
  - [ ] Compute single-tick move versus previous tick for `(stock_token, resolution)` and validate tolerance band (default rule: reject moves > 5%)
- [ ] Persist circuit-limit indicator without disrupting ingestion
  - [ ] Detect whether tick is at/beyond NSE circuit limit (use configured/reference circuit-limit logic)
  - [ ] Store tick record with `circuit_limit_flag = true` (or equivalent) when detected; ensure the pipeline continues
- [ ] Add SLO measurement for ingest-to-durable-store latency
  - [ ] Measure end-to-end latency from tick receipt to successful persistence commit
  - [ ] Export metrics needed for p95/p99 evaluation for active watchlist symbols
- [ ] Integrate operator-facing “last tick” metadata path
  - [ ] Ensure `/data/status` can read “last stored tick metadata per symbol” from the durable store
- [ ] Tests (must cover rejection paths, not just happy path)
  - [ ] Unit tests for out-of-order rejection
  - [ ] Unit tests for primary-key dedup discard
  - [ ] Unit tests for zero price/volume discard
  - [ ] Unit tests for tolerance-band rejection
  - [ ] Integration test to verify “stored tick stream remains gap-free and continuously growing” under duplicate deliveries
  - [ ] Integration test to verify circuit-limit ticks persist with a flag and do not crash the ingestion loop

## Dev Notes
### Implementation Guardrails (what MUST NOT be wrong)
- **Durable store is source of truth**: monotonic checks and dedup behavior must be anchored to persistent data, not only in-memory state.
- **Idempotency is mandatory**: duplicate WS deliveries must not create duplicate rows or duplicate side effects (metrics/logs are fine; extra inserts are not).
- **No silent success on data rejection**: every discard reason should be observable via structured logging and metrics counters (at least: out_of_order, duplicate_pk, zero_price_or_volume, tolerance_band_violation).
- **No crash on circuit-limit conditions**: circuit-limit detection is a classification flag, not a failure mode.
- **Timestamp semantics**: monotonic ordering uses `<=` as the rejection boundary (timestamps equal to last stored are rejected as out-of-order per story AC #2).

### Data Model Expectations
- **Ticks table** (TimescaleDB hypertable recommended):
  - Primary key: `(stock_token, timestamp, resolution)` (story AC #3)
  - Columns should include: `data_source`, `price`, `volume`, `circuit_limit_flag` (or `nse_circuit_limit_flag`), plus appropriate audit timestamps (e.g., `ingested_at_utc`)
- **data_source**: store `'WEBSOCKET'` for these records (story AC #1)

### Required Architectural Alignment
- TimescaleDB on Postgres is the authoritative tick/time-series store, aligned with the PRD’s “continuous series” and dedup/write discipline requirements.  
  [Source: `_bmad-output/planning-artifacts/architecture.md` (Data architecture + DB choice)]
- Adapter boundary: Breeze WebSocket payloads should be isolated behind the Breeze adapter; persistence should operate on domain shapes, not raw vendor JSON.  
  [Source: `_bmad-output/planning-artifacts/architecture.md` (Adapter/API boundaries + consistency rules)]
- The system must expose per-symbol “last tick” metadata through `/data/status` so operator trust signals reflect durable ingestion state.  
  [Source: `_bmad-output/planning-artifacts/prd.md` (Endpoint spec: `/data/status`)]

### Observability / Metrics
- Required logs/metrics should include:
  - discard_reason counters (out_of_order, duplicate_pk, zero_price_or_volume, tolerance_band_violation)
  - ingest_to_durable_store_latency_ms with p95/p99 computation on active watchlist
  - circuit_limit_flag counts
- Include correlation fields where available, but do not require them for monotonic/dedup correctness.

### File/Module Touchpoints (expected)
Implementations should follow the directory structure described in `architecture.md`:
- `src/algotrade/adapters/breeze/ws_client.py` (WS ingestion + payload normalization)
- `src/algotrade/db/repositories/tick_repo.py` (durable persistence + dedup/monotonic enforcement)
- `src/algotrade/jobs/` or a dedicated ingestion service module under `src/algotrade/services/` to run the WS ingestion loop safely
- `src/algotrade/db/session.py`, `src/algotrade/db/models.py`, and Alembic migration for ticks table
- `src/algotrade/api/routers/data_status.py` (or equivalent) to back `/data/status`
- `tests/unit/` + `tests/integration/` (rejection-path coverage + persistence assertions)

## References
- Epic & story requirements:
  - [Source: `_bmad-output/planning-artifacts/epics.md` (Epic 1 / Story 1.1)]
- Architectural constraints and project structure patterns:
  - [Source: `_bmad-output/planning-artifacts/architecture.md` (DB choice + consistency + module boundaries + project structure)]
- Functional + endpoint expectations:
  - [Source: `_bmad-output/planning-artifacts/prd.md` (`/data/status` + data integrity + FR1/FR3)]

## Dev Agent Record
### Agent Model Used
TBD (generated by workflow execution in this environment)

### Debug Log References
- Discard logs should include enough fields to debug without leaking secrets: `stock_token`, `timestamp`, `resolution`, and `discard_reason`.

### Completion Notes List
- TimescaleDB persistence works with enforced primary key dedup.
- Out-of-order ticks are rejected (<= last timestamp).
- Zero-price/volume ticks are rejected.
- Tolerance-band violations are rejected and logged.
- Circuit-limit ticks persist with a flag and do not crash ingestion.
- Ingest-to-durable-store latency metrics exist and meet configured p95/p99 thresholds under normal load.

### File List
TBD (implementation should create/update the modules referenced in “File/Module Touchpoints”).

