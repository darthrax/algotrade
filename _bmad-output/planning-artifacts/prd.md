---
stepsCompleted:
  - step-01-init
  - step-02-discovery
  - step-02b-vision
  - step-02c-executive-summary
  - step-03-success
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
