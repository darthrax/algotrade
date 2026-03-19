---
stepsCompleted:
  - step-01-init
  - step-02-discovery
  - step-02b-vision
  - step-02c-executive-summary
  - step-03-success
  - step-04-journeys
  - step-05-domain
  - step-06-innovation
inputDocuments:
  - docs/AlgoTrade_India_Product_Brief_v1.1.md
  - _bmad-output/planning-artifacts/research/market-ai-powered-nse-bse-intraday-trading-in-india-research-2026-03-20.md
  - _bmad-output/planning-artifacts/research/domain-ai-powered-nse-bse-intraday-trading-in-india-research-2026-03-20.md
  - _bmad-output/planning-artifacts/research/technical-ai-powered-nse-bse-intraday-trading-in-india-research-2026-03-20.md
briefCount: 1
researchCount: 3
brainstormingCount: 0
projectDocsCount: 1
workflowType: 'prd'
classification:
  projectType: api_backend
  projectTypeLabel: API-first execution and risk control platform
  domain: fintech
  domainDetail: indian_listed_equities_intraday_single_account
  complexity: high
  projectContext: brownfield
  operatingModel: single_operator_self_hosted
  majorSubsystems:
    - operator_web_console
    - data_feature_ml_pipeline
    - retraining_model_registry
  prdSpineRationale: >-
    Path from market data to bounded broker actions with auditable decisions;
    other subsystems support or observe that path.
---

# Product Requirements Document - algotrade

**Author:** Darthrax
**Date:** 2026-03-20

## Executive Summary

**AlgoTrade India** is a **single-operator, self-hosted** system for **AI-assisted intraday equity trading** on **NSE/BSE**, integrated primarily with **ICICI Direct Breeze** (WebSocket + REST). It ingests and backfills market data into a **durable store**, engineers features, runs **ML-driven** buy/hold/sell **signals**, and routes every candidate action through a **risk and execution layer** that is the **only** path to **paper or live** orders. **Paper trading uses the same code paths as live**, with capital **staged** only after **long-run** paper proof and explicit gates. The system **retrains from real outcomes** (nightly cadence per brief), maintains **model/version lineage**, and treats **broker reconciliation, degraded data modes, and tamper-evident audit** as part of the product, not an afterthought.

**Target user:** one **technical operator** (“prosumer”) who owns capital, infrastructure, and compliance outcomes—not a multi-tenant SaaS customer base at this stage.

**Problem:** Retail algo stacks often optimize **signal sophistication** while under-investing in **execution safety**, **India-specific broker/session/static-IP reality**, and **evidence** under loss. That mismatch shows up as **silent drift**, **fragile live behavior**, and **unresolved broker-vs-internal truth**.

**Success looks like:** **90+ trading days** of **credible** paper (or gated live) behavior, **stable reconciliation**, controlled **model promotion/rollback**, and **predictable** behavior under **WebSocket loss** and **REST_POLL**—before capital scales.

### What Makes This Special

- **Risk owns the broker:** ML **proposes**; **pre-trade checks, limits, kill switch, and order state handling** **dispose**. No silent bypass.
- **Paper/live parity:** Same pipeline and risk logic; only the **final submission** differs—so proof in paper is **comparable** to live.
- **Survivability over novelty:** Differentiation is **governance + reliability under capital risk** (aligned with India **SEBI/NSE/broker** trajectory and research on **trust**), not a pitch for “more alpha.”
- **India-realistic operations:** **Static IP / session**, **gap fill**, **deduped** multi-source ticks, **REST fallback**, **no new entries in coarse poll mode** where the brief forbids it—designed as **constraints**, not bugs to fix later.

## Project Classification

| Dimension | Classification |
|-----------|----------------|
| **Primary product type** | **API-first execution and risk control platform** (`api_backend`); **FastAPI** internal service as **contract-critical** surface |
| **Major subsystems** | **Operator web console** (Streamlit); **data / feature / ML pipeline**; **retraining and model registry** |
| **Domain** | **Fintech** — **Indian listed equities**, **intraday**, **single-account** automation |
| **Complexity** | **High** (capital loss, regulatory/evidence burden, broker and feed fragility) |
| **Context** | **Brownfield** — **authoritative product brief + market/domain/technical research** already define the system; this PRD **refines and gates** implementation |
| **Operating model** | **Single-operator, self-hosted** (not multi-tenant SaaS) |

**PRD spine:** From market data to **bounded, auditable** broker actions; all other capabilities **observe, explain, or improve** that path—not replace it.

## Success Criteria

### User Success

- **Operator trusts the loop:** Can answer *“what did the model know, what did risk allow, what hit the broker, what filled?”* for any session from **logs + reconciliation**, without ad hoc SQL archaeology.
- **Paper proves live:** After **≥90 consecutive trading days** in paper, operator has **positive risk-adjusted performance** (brief: Sharpe-oriented framing), **max drawdown within guardrails** (brief: e.g. **<10%** before live scale-up), and **no unexplained broker–internal mismatches** blocking the next session.
- **Crisis clarity:** Kill switch and **daily loss limit** behave as specified; **Telegram / dashboard** make **degraded mode** (WebSocket vs **REST_POLL**) obvious within **one glance**.
- **Compliance peace of mind:** Running **turnover** and **tax-ready exports** stay usable for **CA / ITR**; **append-only audit** with **checksum discipline** is maintained.

### Business Success

*(Single-account product: “business” = capital preservation + evidence for scaling capital, not ARR.)*

- **Staged live works:** **25% → 50% → 100%** capital ramp (brief §10.4) only after **explicit human sign-off** at each gate, with **no regression** in reconciliation or drift alerts.
- **Cost of mistakes bounded:** **Incident recurrence** drops run-over-run; **postmortem actions** close before raising live risk.
- **Regulatory posture defensible:** Retail algo / broker obligations treated as **release gates** (static IP, session hygiene, traceability)—not a late scramble.

### Technical Success

- **Data integrity:** DB remains **source of truth**; **startup gap fill** and **reconnect backfill** restore **continuous series** before signals; **dedup / source precedence** rules enforced (incl. **poll vs websocket**).
- **Execution path:** **Pre-trade checks** + **order state machine** cover **partial fills**, **timeouts**, **session boundaries**; **risk layer failure ⇒ no new orders** (fail closed).
- **Model governance:** **Champion–challenger**, **shadow / canary**, **rollback SLA** (brief §11.5); **no schema / promotion / config that breaks market-hours lock** without breach protocol.
- **SLOs (measurable):** Meet brief-class targets for **tick freshness**, **signal latency**, **order ack latency**, and **session availability** during RTH; exact numeric thresholds carry from **§24** into build/test plans.

### Measurable Outcomes

| Area | Examples (tune numerically in implementation) |
|------|-----------------------------------------------|
| **Paper qualification** | **≥90** trading days; **Sharpe > 0** (or agreed baseline); **drawdown cap**; **reconciliation pass rate** |
| **Reliability** | **% session** in **WEBSOCKET** vs **REST_POLL**; **gap-fill completion** time; **incident MTTR** |
| **Model** | **PSI / accuracy drift** vs thresholds; **promotion pass rate**; **rollback events** |
| **Risk** | **Kill-switch drills**; **daily loss limit** trips; **manual intervention count** |

## Product Scope

### MVP — Minimum Viable Product

Aligned to **brief Phase 1** (months 1–2): **Breeze WebSocket + REST CRUD**, **historical bootstrap**, **startup gap fill**, **dedup/storage**, **feature engineering**, **backtesting engine (simulation clock)**, **LSTM signal path**, **paper trading end-to-end**, **basic Streamlit + alerting**. **Gate:** walk-forward validation **non-trivially positive** on risk-adjusted basis; **30 days** stable paper; **REST fallback** exercised under test.

### Growth Features (Post-MVP)

**Brief Phase 2:** ensemble (LSTM + XGBoost + baseline), **regime detection**, **full risk + pre-trade**, **audit trail completeness**, **WS disconnect → REST_POLL** behavior, **60 days** cumulative paper, **chaos / failure** testing.

**Brief Phase 3:** **staged live** capital, **RL** training in parallel (not necessarily prod), **attribution**, **full tax reporting** depth.

### Vision (Future)

**Brief Phase 4+:** **RL in production ensemble** if shadow beats champion; richer features; **external capital** data model **without** breaking v1 schema.

**Scope note:** Success is measured **first** on **survivability artifacts** (reconciliation, degraded modes, governance), **then** on **PnL metrics**—matching the Executive Summary differentiator.

## User Journeys

### Journey 1 — Primary operator: “Normal RTH day” (success path)

**Priya** runs AlgoTrade on a home PC with static IP. **08:45** she gets the **Telegram morning brief**: watchlist, events, model health, last tick ages. She glances at **Streamlit**: WebSocket **green**, positions flat. **09:15–09:30** she expects **observation-only**—no surprises. After **09:30**, signals may flow; when a **buy** clears the ensemble and regime filters, the **internal API** runs **pre-trade checks**, places a **bracket** order via Breeze, and the dashboard shows **order state** advancing to **POSITION_OPEN**. She checks **SHAP top features** once—not to override, but to sanity-check. **15:20** flat **or** stops/targets hit; **EOD reconciliation** runs. She closes the laptop feeling **bored in a good way**: the system did the boring parts and left an **audit trail**.

**Climax:** First day she trusts **reconciliation** without opening the broker app for every fill.  
**Resolution:** Operator shifts from **watching ticks** to **watching SLOs and drift**.

### Journey 2 — Primary operator: “Feed dies, entries must stop” (edge / failure)

**11:20** WebSocket drops. Priya’s phone buzzes: **disconnect alert**, then **REST_POLL** mode. Dashboard shows **amber**: positions still monitored, **new entries suppressed** (brief policy). She does **not** flip configs mid-session—they’re **locked**. She confirms **gap-fill** is queued for reconnect. **Climax:** She verifies **no rogue entries** were attempted while granularity was coarse. **Resolution:** Back on WebSocket; backfill completes; next signal only when **feature freshness** and **data source** gates pass.

**Reveals:** Degraded-mode UX, poll vs WS semantics, pre-trade **REST_POLL** block, alerting, recovery ordering.

### Journey 3 — Operator as “governance admin”: Sunday night change window

After market close, Priya unlocks **config**: adjusts a **confidence** floor or **watchlist** rule. Change is **logged**, **versioned**, and effective **next** session per policy. She reviews **challenger** metrics from the week and approves or **rejects** promotion in **MLflow / registry** workflow. **Climax:** Saying **no** to a model that wins accuracy but fails **drawdown** on hold-out—documented decision. **Resolution:** Monday runs on a **known** policy surface; no “what did we ship Friday?” doubt.

**Reveals:** Market-hours lock, human sign-off for promotion, audit of config/model changes.

### Journey 4 — Trusted person / incident (support & DR narrative)

Mains fail; UPS countdown starts. **Priya’s partner** (documented in **printed DR**, brief §14) uses **broker web** to flatten if automation cannot. Separately, for a **tax audit**, **CA** receives **CSV/PDF exports** and **intervention log**—no hand-wavy screenshots. **Climax:** Capital safe **or** books defensible under scrutiny. **Resolution:** DR and compliance artifacts are **first-class outputs**, not side quests.

**Reveals:** Export schemas, DR runbook, kill/flatten instructions, retention.

### Journey 5 — Technical / “API consumer”: batch & automation

**Nightly 22:00 IST**, **scheduler** triggers **retraining**: labels from DB, **train**, **evaluate vs champion**, **register** artifact or rollback. Another job hits **`GET /health`** for **dead man’s switch**. **Climax:** Promotion blocked when **leakage audit** or **metric gates** fail—pipeline exits **non-zero**, alert fires. **Resolution:** Headless operation matches **human** expectations: no silent deploy.

**Reveals:** Internal API for health/mode, job idempotency, training gates, observability for batch paths.

### Journey Requirements Summary

| Journey | Capability areas surfaced |
|---------|---------------------------|
| **Normal RTH** | Ingestion health, feature + regime gates, ensemble, risk API, order state machine, bracket orders, dashboard, Telegram brief, SHAP surfacing |
| **Feed failure** | WS reconnect, REST poll, entry suppression, gap backfill, staleness gates, operator alerts |
| **Governance window** | Config lockout, change audit, model registry, champion–challenger, promotion/rollback |
| **DR / compliance** | Exports, turnover, intervention journal, third-party broker access, DR documentation |
| **Automation** | APScheduler jobs, training orchestration, health endpoints, failure alerts, CI-friendly gates |

## Domain-Specific Requirements

### Compliance & Regulatory

- **SEBI / exchange retail algo direction:** Design for **traceable orders**, **broker API discipline**, and **human-in-the-loop gates** consistent with evolving **retail algorithmic trading** norms (treat exchange circulars and broker implementation as **hard inputs**, not blog opinion).
- **Algo registration / Algo-ID:** System must support **attribution** of live orders to **strategy/version** metadata where the broker or exchange requires it; exact registration steps remain **broker-confirmed** before live.
- **Order-rate / surveillance posture:** Architecture stays **well below** thresholds that trigger **mandatory exchange algo registration** unless you **explicitly** choose to register—per brief, avoid >10 orders/sec class behavior by design.
- **Tax / audit:** **Intraday as business income** framing; **running turnover**; **3-year retention**; **exportable** trade and charge breakdown suitable for **CA / ITR**; awareness of **audit-trap thresholds** (turnover / profit per brief).
- **Collective investment boundary:** Product positioning stays **single-account / own capital** unless legal structure exists for external money (brief §2.1).

### Technical Constraints

- **Network:** **Static IP** and **session token** lifecycle treated as **availability and compliance prerequisites** for Breeze-class connectivity (brief).
- **Clock / time:** **NTP discipline**; trading behaviors **keyed to IST exchange session** boundaries (pre-open, RTH, auction, close).
- **Secrets:** **Encrypted at rest**, **no plaintext** in repo/logs; **rotation** cadence for API, session, Telegram tokens.
- **Tamper evidence:** **Append-only** trading and config events; **daily checksums** on audit surfaces (brief).
- **Fail-closed execution:** Risk/execution path **unavailable ⇒ no new risk**; degraded data modes **suppress entries** per policy (e.g. **REST_POLL**).
- **Privacy (India DPDP-aware):** **Data minimization**, retention limits, access logging for any **personal** data processed by dashboards or notifications; exact legal interpretation **verified** before externalization beyond self-hosted use.

### Integration Requirements

- **ICICI Breeze (primary):** **WebSocket** live quotes + **REST** historical, gap fill, poll fallback; **rate-limit** and **burst** handling per endpoint **as configuration**.
- **Broker book of record:** **Positions, fills, charges** from broker must **reconcile** to internal state **daily** before **next-day live** activation (brief go/no-go).
- **Supplementary data:** NSE announcements / indices / optional Yahoo macro—each with **latency class** and **contract** per brief feature matrix.
- **Notifications:** **Telegram** (or equivalent) for **health, risk, reconciliation** without turning into an unauthenticated share of sensitive data.

### Risk Mitigations

| Risk | Mitigation (product-level) |
|------|----------------------------|
| **Regulatory change / interpretation drift** | Versioned **compliance checklist** in release process; **paper-first** default; no silent live |
| **Broker or API policy change** | **Adapter isolation**; contract tests; **monitor-only** mode when payloads degrade |
| **Model / data leakage** | **Leakage audit** gate before train/promote; **feature availability** classes enforced in live |
| **Operator behavioral risk** | **Config lockout** RTH; **intervention journal**; **cooling-off** after loss limits |
| **Operational catastrophe** | **Kill switch**, **DR print kit**, **trusted human** broker-web path; quarterly **DR evidence** |

## Innovation & Novel Patterns

### Detected Innovation Areas

- **Governance-first composition:** **ML + ensemble + regime filters** are **subordinate** to a **deterministic risk and execution API**—the “product shape” is **control-plane-led**, not model-led. That ordering is **uncommon** in retail signal tools that lead with alpha.
- **Operational truth in the loop:** **Continuous retraining** and **promotion** are tied to **broker-aligned labels**, **leakage audits**, and **rollback SLAs**—treating **model lifecycle** as part of **capital safety**, not a separate notebook step.
- **India-shaped degradation contract:** **REST_POLL** and **entry suppression** are **specified behavior**, not outages—reducing the class of bugs where automation **trades blind** on bad granularity.
- **Paper/live as one spine:** **Identical** decision path up to the **final submission** switch makes “paper proof” **evidentially comparable** to live—a **discipline choice** that many stacks shortcut.

### Market Context & Competitive Landscape

- **Brokers and platforms** own distribution; **differentiation** for this build is **not** “secret indicator” but **survivability**: reconciliation, traceability, staged capital, and **predictable** behavior under **SEBI/NSE/broker** evolution (aligned with research synthesis).
- **Commoditized pieces** (LSTM, XGBoost, Streamlit, Timescale) are **intentionally boring**; novelty is in **how tightly** they’re coupled to **risk and evidence**.

### Validation Approach

- **Paper and shadow:** **Champion vs challenger** on **live data** with **no capital** (or capped canary) until **metric + ops** gates pass.
- **Deterministic replay:** **Historical session replay** through the **same** pipeline as live to validate **state machine** and **degraded modes**.
- **Fault injection:** **WebSocket loss**, **stale features**, **partial fills**, **reject storms**—**release-blocking** scenarios, not ad hoc fire drills.
- **Regression suite** on **walk-forward** and **stress windows** (brief-style) before any promotion.

### Risk Mitigation

| Innovation risk | Fallback |
|-----------------|----------|
| **“Composition” becomes unmaintainable** | **Modular monolith** boundaries, **contract tests** at adapter seams, **feature flags** for risky paths |
| **Retraining introduces silent harm** | **Promotion veto** by **drawdown / PSI / reconciliation**; **automatic rollback**; **paper-only** on breach |
| **Overclaiming novelty** | Position as **excellent execution of known practices**, not magic alpha |
