---
stepsCompleted:
  - step-01-init
  - step-02-discovery
  - step-02b-vision
  - step-02c-executive-summary
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
