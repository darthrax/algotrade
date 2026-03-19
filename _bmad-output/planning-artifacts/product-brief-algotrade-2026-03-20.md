---
stepsCompleted: [1, 2, 3]
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
