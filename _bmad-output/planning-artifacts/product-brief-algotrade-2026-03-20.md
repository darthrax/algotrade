---
stepsCompleted: [1, 2, 3, 4, 5, 6]
briefCompletedAt: "2026-03-20"
inputDocuments:
  - _bmad-output/planning-artifacts/research/technical-ai-powered-nse-bse-intraday-trading-in-india-research-2026-03-20.md
  - _bmad-output/planning-artifacts/research/domain-ai-powered-nse-bse-intraday-trading-in-india-research-2026-03-20.md
  - _bmad-output/planning-artifacts/research/market-ai-powered-nse-bse-intraday-trading-in-india-research-2026-03-20.md
  - docs/AlgoTrade_India_Product_Brief_v1.1.md
date: 2026-03-20
author: Darthrax
---

# Product Brief: algotrade

## Executive Summary

AlgoTrade (AlgoTrade India) is a **personal, self-hosted** path into **Indian intraday equity trading** for someone who wants to begin **with process and safety**, not with scattered tips or opaque automation. The bet is **survivability over signal-chasing**: one integrated stack—from **ingest** through **features, models, risk-gated execution, and attribution**—so “where to begin” is a **defined progression** (**paper-first**, then **gated live**), with **hard risk controls** that **no model can override**.

**Year-one success** is explicit: a **trustworthy paper-to-live workflow**—reliable operation during market hours, **honest** metrics, and **capital not lost to chaos** while you learn—not a promise of immediate profitability. Profit is a **later** test **after** the process proves it won’t fool you. The **long-form spec** is the **north star**; **early delivery** is intentionally **modest**: a **small universe**, **paper mode**, and **reconciliation you trust** before ambition expands.

**Operating model:** a **dedicated machine** during **market hours** so trading is **infrastructure**, not a background tab—fewer split-attention errors, clearer incident response, and a cleaner boundary between everyday computing and **market risk**.

**Human clarity:** you should always be able to answer—**why did the system trade, why did it refuse, and does broker state match internal state?** Transparency there is the emotional and operational anchor, especially while you are still learning.

This is **not** a claim that building software replaces learning order flow, costs, and discipline. The stack **enforces** and **records** those lessons—it does not remove them.

---

## Core Vision

### Problem Statement

Retail intraday entry is overwhelming: fragmented tools, uneven education, real **India-specific** broker and regulatory constraints, and defaults that reward **activity** over **repeatable decision quality**. Someone can **want** to day trade and still lack a **credible architecture** to learn safely: a single path to simulate, measure, reconcile, and only then scale capital behind **non-negotiable guardrails**.

### Problem Impact

Without that architecture, the usual failure modes appear: **trial-and-error with real money**, weak feedback loops (no clean attribution), and automation that **amplifies** mistakes. In a **concentrated broker** environment with **evolving retail-algo expectations**, the cost of an unstructured start is **time, money, and trust**. A dedicated machine **reduces** attention risk; it does **not** remove behavioural or market risk.

### Why Existing Solutions Fall Short

**Broker apps** optimise for access, not an **end-to-end** governed pipeline. **Signal sellers** obscure edge, limits, and failure modes. **Generic automation** often skips **paper/live parity**, **replay**, **reconciliation**, and **risk-before-model** ordering—yet those are what make **personal** automation **legible and safe enough to live with**. Manual trading plus a journal is valid for learning **mechanics**; it typically does not give you **contract-grade** process, **India integration reality**, or **one place** where truth and controls live together.

### Proposed Solution

Build a **personal AlgoTrade system**: NSE/BSE intraday equity (per the technical brief), **Breeze-centric** integration, **Timescale-backed** market truth, **ML-assisted** signals only under **ensemble and regime discipline**, and **FastAPI-gated execution** so orders never bypass **pre-trade checks** and **kill-switch** semantics. Progress is **staged** and **complexity follows evidence**: prove **replay**, the **risk gateway**, and **broker-aligned truth** on **one coherent path** (modular monolith mindset) before widening scope—then **step** into live capital. “Getting started” means **passing gates**, not flipping a gamble switch.

The **dedicated machine** is part of reliability: predictable environment, market-hours focus, and serious **ops** habits (health, reconnections, incidents).

### Common failure modes (pre-mortem guardrails)

The vision explicitly guards against: **big-bang complexity** before basics; **paper success** without reconciliation; **bypassed** limits after drawdowns; **scope creep** before metrics are stable; and treating the build as **avoidance** of learning—mitigated by **staging**, **immutable logs**, **promotion rules** for models, and **human** sign-off at stage transitions.

### Key Differentiators

- **Personal ownership**: one operator; full transparency, rollback, and **no** black-box vendor narrative.
- **Governance-first automation**: risk layer **first**; audit trails; reconciliation and operational readiness as **features**.
- **India realism**: broker/API constraints, degraded **monitor-only** behaviour when data or integration is untrustworthy, traceability **designed in**.
- **Learning-system posture**: retraining and **champion–challenger** only under **explicit promotion rules**—aligned with **home-class** hardware and **intermediate** ambition without pretending ML fixes bad process.
- **Operational seriousness**: **separate machine** and **market-hours focus** as a commitment device, not cosplay.
- **Legible behaviour**: explainable **act / block** decisions and **reconciliation** as the default posture, not a debug-only afterthought.

## Target Users

### Primary Users

**Solo operator — “the builder-trader” (you)**  
Single primary user: **Darthrax**, using AlgoTrade **only for personal** capital and learning—not as a service to others.

- **Context:** Indian equity **intraday** focus (NSE/BSE per product direction), **ICICI Direct Breeze** as the integration surface, **dedicated machine** active during **market hours** so trading is a deliberate ops shift, not background multitasking.
- **Skill level:** **Intermediate** technical capability (per project config)—comfortable with APIs, databases, and running a local stack, still **building** markets literacy and **process** discipline in parallel.
- **Motivation:** Enter **day trading** without an ad hoc stack of tabs, opaque automation, or unstructured trial-and-error; prefer **staged** progress (**paper-first**, **gated live**) and **hard limits** that do not depend on model enthusiasm.
- **Problem experience today:** “I don’t know where to begin” — fragmented how-tos, unclear safe path from learning to live, and fear of **amplify-by-code** mistakes without traceability.
- **Success vision:** Year-one **trust** in the **workflow**—reliable runs, **honest** metrics, **explainable** act/block behaviour, **reconciliation** you believe—**without** losing money to chaos while learning. Profit is **not** the defining success metric for year one.
- **“This is exactly what I needed” moment:** When you can answer, without angst: **why** something traded or **didn’t**, whether **broker state matches** your system, and what gate you’re on toward live—**before** optimising for more alpha.

### Secondary Users

**N/A** — The product is **not** intended for other people to operate or consume as end users. No multi-user accounts, no delegated operators, and no external “client” persona in v1. Third parties (e.g. CA) may consume **exports** you generate; that remains **your** responsibility and **your** data handoff, not an in-product user role.

### User Journey

**Discovery**  
You encounter friction in starting **disciplined** intraday participation (education noise, broker/API reality, India-specific constraints). You choose **self-hosted** AlgoTrade as the **single spine**: data → features → (later) models → **risk-gated** execution → logs and attribution.

**Onboarding**  
Stand up the stack on the **trading machine**; connect broker/session reality; start in **paper** (or equivalent) with a **small universe**; verify **health**, **data quality modes**, and **kill switch** behaviour before caring about model sophistication.

**Core usage (market hours)**  
Monitor **dashboard / alerts**; observe **signals accepted vs suppressed** with reasons; respect **degraded / monitor-only** behaviour when data or broker is untrustworthy; end-of-day (or scheduled) **reconciliation** and review—**routine**, not heroic debugging.

**Success moment (“aha”)**  
Reconciliation and **explainability** feel **boring and reliable**; paper progression feels **evidence-based** (“I trust the numbers and the gates”), not lucky.

**Long term**  
**Staged live capital** only after sustained paper discipline; **model and scope** grow **after** replay, risk gateway, and broker truth are boring; the **dedicated machine** stays the boundary between normal life and **market risk**.

## Success Metrics

### User success (what “working” means for you)

**North star (year one, aligned with your stated “1a” goal)**  
You have a **paper → gated live** path you **trust**: you can explain **act vs block**, **internal state matches broker** often enough to rely on, and **capital isn’t lost to ops chaos** (bugs, drift, bypassed limits, mystery fills) while you’re still learning **markets** and **your own stack**.

**90-day paper foundation (early “worth it” window)**  
Success isn’t profitability yet—it’s **boring reliability**:

- **Session ops:** trading machine runs through **full market sessions** you intend to trade, without ad-hoc “I’ll fix it live” patches becoming routine.
- **Data truth:** **WebSocket-primary** path works; when it degrades, the system behaves **as designed** (e.g. monitor-only / no new entries on coarse feeds)—not silently “best effort.”
- **Explainability:** for a **sampled set** of decisions (e.g. weekly review), you can answer **why trade / why no trade** from logs + feature/signal snapshot, without forensic archaeology.
- **Limits culture:** **no risk-parameter changes during market hours** except true emergency protocol—you’re not “tuning away” a bad day.

**Behavioural signals you’re getting value (not vanity)**  
These are **observable habits** the system supports:

- You complete a **short daily close-out**: positions flat or explained, reconciliation ticked, incidents noted.
- **Kill switch / degraded mode** are **tested** on a cadence you define (e.g. monthly drill)—you trust them before you need them.
- Paper trading is **long enough and honest enough** that you’d be embarrassed to promote to live if reconciliation were failing—**the gate is emotional + quantitative**.

---

### Business objectives

There is **no commercial business** for v1—**you are the only “shareholder.”** Objectives are **personal portfolio governance**, not revenue.

| Horizon | Objective |
|--------|-----------|
| **0–3 months** | **Prove the machine**: install → paper → **stable ingest + risk gateway + attribution** on a **small universe**; treat “complexity added” as a **risk** until gates pass. |
| **3–12 months** | **Prove the process under staged live rules**: only after paper discipline; **capital ramps** follow **human sign-off** and **missed-check consequences** (e.g. back to paper). |
| **Ongoing** | **Stay audit-ready for yourself**: exports and logs good enough that **future-you** (or a CA) can reconstruct **what happened and why**—aligned with your long brief’s tax/audit posture. |

**Strategic “moat” (personal)**  
Your advantage isn’t a secret indicator—it’s **execution realism + governance**: parity between paper/live paths, **reconciliation**, **replay**, and **promotion rules** for anything that emits orders.

---

### Key Performance Indicators

Metrics below are **examples** until you wire exact thresholds in implementation; they’re chosen so you can **measure** them from **logs, DB, broker statements**, not vibes.

#### Operations & trust (leading indicators)

| KPI | What it measures | Example target / review |
|-----|------------------|-------------------------|
| **Market-hours availability** | Stack healthy when you need it | ≥ **X%** of intended sessions without **unplanned** downtime (define X after you see baseline). |
| **Data freshness age** | Latency / gap detection | Tick/feature age within **SLO** you set in the long brief (e.g. “normal” vs “degraded”). |
| **Mode time split** | Time in `WEBSOCKET` vs fallback | Track % session in **REST_POLL** or degraded—**high fallback** → **signal quality** and **confidence** must **not** pretend nothing changed. |
| **Reconciliation pass rate** | Internal vs broker | **100%** Explained exceptions with **ticket/note**; **unexplained mismatch blocks next-day live** (per your spec philosophy). |
| **Block reason coverage** | Explainability | **100%** of blocks have **coded reason** + enough context to replay the decision. |

#### Risk & discipline (guardrail effectiveness)

| KPI | What it measures | Example target / review |
|-----|------------------|-------------------------|
| **Limit breaches** | Hard rules vs reality | **0** unintended breaches; any breach is **incident-class** with postmortem. |
| **Kill-switch activations** | Emergency use | Rare; each logged with **cause** and **follow-up** (test or rule change). |
| **Config changes in session** | Discipline | **0** “normal” changes during **locked window**; violations logged. |

#### Learning system / ML (lagging & quality—only after plumbing works)

| KPI | What it measures | Notes |
|-----|------------------|------|
| **Paper performance (risk-adjusted)** | Is there *any* plausible edge in simulation | Use **Sharpe / max DD / reconciliation-clean periods**—not raw PnL alone. |
| **Promotion success rate** | Model/challenger hygiene | % candidates promoted; **rollback count** after promotion; tie to **champion–challenger** policy in long brief. |
| **Signal disagreement / regime flags** | Early warning | Rising disagreement → **downgrade path** (rules / wider thresholds), not “trade more.” |

#### Personal “capital” objectives (live phase only, staged)

| KPI | What it measures |
|-----|------------------|
| **Live capital stage adherence** | No skipping stages without explicit signed decision |
| **Live max drawdown vs plan** | Breach → **automatic** step-down to paper or lower stage |

---

### What we are deliberately *not* optimizing for in year one

- **Maximizing trade count** or “always in the market.”  
- **Short-term profit** as the **primary** KPI before **reconciliation + explainability** are boringly solid.  
- **Model complexity** ahead of **risk gateway + data truth**.

### Strategic alignment (vision ↔ metrics)

- **Process-first (“1a”)** → ops + reconciliation KPIs **precede** performance KPIs.  
- **Dedicated machine** → availability and **session ritual** metrics matter—they’re how you buy **attention** and **fewer mistakes**.  
- **Solo operator** → every KPI must be **reducible to a checklist** one person can run without a NOC team.

## MVP Scope

MVP is aligned to **Phase 1 + Phase 2** of the long product brief (`docs/AlgoTrade_India_Product_Brief_v1.1.md`, §19): a **paper-trading-complete** loop—**truthful data**, **features**, **models (through ensemble + regime)**, **risk-gated execution API**, **retraining**, **auditability**, and **resilience** under **WebSocket + REST** modes—**without** **live capital**, **production RL**, or **Phase 3+ live ramp** to call the MVP “done.”

### Delivery sequencing (recommended for solo operator)

Phase 1–2 in the long brief is **large** for one person. **Ship in two internal cut lines** while still matching the same end state:

**MVP-A — Truth path (build first)**  
Ingest + storage + gap fill + dedup + **minimal** feature set + **manual or rule-based** signals + **full risk gateway** + **paper execution** + **logs** + **dashboard** + **alerts**. Goal: **session-stable**, **explainable blocks**, **no mystery state**, broker/session/static-IP **reality** verified on the dedicated machine.

**MVP-B — Learning path (build second)**  
Backtest / walk-forward + **LSTM** + **ensemble** + **regime** + **retraining** + **promotion rules** + richer feature depth per long brief.

You are still “Phase 1–2 complete” only when **both** cut lines are met; this ordering reduces simultaneous subsystem debugging.

### Core Features

**Phase 1 (Months 1–2) — “data → signal → paper”**

- **Ingestion:** Breeze **WebSocket** + **REST (CRUD)**; **historical bootstrap**, **startup gap fill**, **dedup**/merge, **TimescaleDB** (and archival direction per long brief).
- **Storage & lineage:** Authoritative **DB**; `data_source` (or equivalent) for **provenance**; schema/migration discipline.
- **Feature engineering:** Pipeline per brief (indicators, leakage controls, availability classes as applicable)—may be **trimmed** in MVP-A, **expanded** for MVP-B.
- **Backtesting / simulation clock:** Engine with **realistic costs/slippage** and **no look-ahead**; supports walk-forward style evaluation (MVP-B).
- **ML Phase 1:** **LSTM** (or equivalent Phase-1 model) with **confidence**; reaches execution only **through** the risk layer.
- **Paper trading:** **Same code path** as live through risk/execution; **simulated fills** per brief.
- **Operator surface:** **Basic dashboard** (e.g. Streamlit) + **alerting** (e.g. Telegram).

**Phase 2 (Months 2–3) — “governance + resilience + learning loop”**

- **Ensemble:** **LSTM + XGBoost + rule baseline** (or brief’s three-way agreement pattern); **disagreement** logged/monitored.
- **Regime detection:** **Trend / chop / high-vol** handling with **threshold / sizing** adjustments per brief.
- **Risk management:** **Full** pre-trade checks, **position/portfolio limits**, **kill switch**, **order state machine**, **bracket/default order** behaviour; **ML never bypasses** risk.
- **Execution API:** **FastAPI** internal service as **sole** path from signal to **paper** submission adapter.
- **Retraining pipeline:** Scheduled retrain with **promotion rules** vs incumbent; **model registry / versioning** (e.g. MLflow local) per brief.
- **Audit trail:** Append-style logging of signals, orders (paper), model version, config changes, key risk decisions.
- **Operational resilience:** WebSocket disconnect, **REST polling**, gap backfill, **degraded / monitor-only** when brief forbids new entries on coarse data.

**Engineering enablers (high leverage, not scope creep)**

- **Broker adapter:** **Thin** Breeze boundary with **contract tests**; at least one **mocked** integration path so CI is not market-hours dependent.
- **Failure-injection drills:** Short **repeatable** checklist (disconnect WS, force poll mode, session expiry/stress, optional clock-sync check)—**pass/fail + notes**—before claiming Phase 2 operational.

### Out of Scope for MVP

**Deferred to Phase 3+ (per long brief §19 and v1.0 out-of-scope where applicable)**

- **Live trading** and **staged live capital** (Phase 3); MVP is **paper-complete**, not live-complete.
- **RL agent in production** (Phase 4); optional **parallel RL training** is **not** MVP-required if it competes with Phase 1–2 gates.
- **Multi-user / multi-account**, **mobile app**, **F&O**, **HFT**, **SEBI AIF / third-party fund** — unchanged from long brief.
- **Full** institutional DR / multi-region ops beyond what Phase 2 implies — **not** MVP unless explicitly promoted later.

**Tax / reporting:** MVP includes **exportable** logs and foundations for **running turnover / P&amp;L**; **full** CA-ready automation (long brief §17 depth) can deepen in **Phase 3** unless you later promote a **minimal** CSV/PDF export into MVP explicitly.

### Internal reconciliation (paper)

**Definition:** On paper, “reconciliation” means **internal ledger coherence**: orders, positions, fills (simulated), and P&amp;L **tie out** from a **single closed-form rule set** (e.g. strict order state machine + event trail), with **zero unexplained transitions** over **N** consecutive sessions (set N when implementing). **Stretch (optional):** read-only **broker** sanity checks where API permits, without relying on live order placement for proof.

### MVP Success Criteria

**Phase 1 gate (long brief §19)**  

- **Walk-forward** (or equivalent) shows **positive risk-adjusted** results **or** a **documented** negative outcome that **blocks** naïve promotion—no silent optimism.
- **Paper trading** stable **30 days** (milestone within the longer paper runway).
- **REST fallback** **tested**, not only specified.

**Phase 2 gate (long brief §19)**  

- **Must-have** Phase 2 capabilities **operational** (ensemble, regime, risk, retraining, audit, disconnect/poll modes).
- **60 days** **cumulative** paper trading **total** (Phase 2 sign-off). **Clarification:** the **30-day** Phase 1 checkpoint is an **early** stability milestone inside the same paper journey—the **60-day** count is **total paper days** from first stable paper session through Phase 2 completion, not “reset at Phase 2.”
- **Simulated failure** exercises completed per drill checklist; outcomes **logged**.

**Environment readiness (pre–Phase 1 “stable”)**  

- **Static IP**, **session/auth**, and **key** handling match broker/exchange reality per long brief (including noted **April 2026** static-IP / key-rotation constraints)—verified on the **dedicated** machine.

**Cross-phase bars (BMM Success Metrics)**  

- **Block/act** explainability; **kill-switch** drills passed; **no routine** risk-limit edits **during** the locked market-hours window.

### Future Vision

- **Phase 3:** Staged **live** capital, **90**-trading-day-class paper history gate where applicable, deeper **attribution**, fuller **tax reporting**.
- **Phase 4+:** **Production RL**, broader features, **external capital / legal structure** only if requirements change.
- **Personal arc:** MVP proves a **serious paper system with governance**; later phases prove **capital under stress** without self-deception.
