---
stepsCompleted: [1, 2, 3, 4]
inputDocuments:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/product-brief-algotrade-2026-03-20.md
  - docs/AlgoTrade_India_Product_Brief_v1.1.md
---

# UX Design Specification {{project_name}}

**Author:** {{user_name}}
**Date:** {{date}}

---

<!-- UX design content will be appended sequentially through collaborative workflow steps -->

## Executive Summary

### Project Vision
A trust-first intraday trading experience for a single technical operator: a governed, risk-gated system where decisions are transparent (“act vs block”) and degraded behavior is an explicit contract (not a guess). The operator should feel, in seconds, what the system knew, what risk allowed, what the broker did, and why it refused—so paper-to-live progression stays emotionally and operationally safe.

### Target Users
Primary: one solo technical operator (“builder-trader”) running a dedicated machine during market hours. Intermediate skill level; comfortable with local services/logs; needs clear operational status, decision reasons, and evidence (reconciliation, audit trails, model/rule lineage) more than exploratory analytics.

Secondary: “downstream consumers” of exports (e.g., CA/accounting workflows). They are not in-product decision-makers, so the product should produce export-ready artifacts with the same traceability guarantees.

### Key Design Challenges
1) Make the order/risk lifecycle legible: the UI must explain outcomes with stable reason codes and traceable evidence (input freshness -> data mode -> risk gate -> resulting order state).
2) Degraded-mode semantics must be unmistakable: when the system enters `REST_POLL` / monitor-only behavior, the operator must clearly understand that “entries are suppressed by policy” and what is still actively managed (stops/exits).
3) Reduce attention load during stress: the console must support fast “next action” comprehension (kill switch drill confirmation, reconciliation checklist prompts, governance lockouts) without requiring forensic browsing.
4) Operationalize reconciliation confidence: “Did broker match us?” becomes a checklist-style pass/fail with explicit next steps (block next-day live vs waived-with-audit-note).

### Design Opportunities
1) “Trust rituals” that compress complexity into predictable moments: morning brief overview + during-session state cards + end-of-day reconciliation close-out.
2) Act/Block decision cards that always show the same causal chain: freshness status, data source mode, feature/signal snapshot, risk gate verdict, then broker/order state.
3) Governance UX that feels protective, not bureaucratic: explicit lock/disabled controls during market hours, plus calm review/approve/deny flow outside market hours with clear audit context.
4) Explainability that’s purposeful: SHAP/top-features surfacing framed as sanity checks tied to sampled decisions, not an invitation to override the risk gateway.

## Core User Experience

### Defining Experience
The operator’s core loop is: **glance → understand trust state → decide whether to act**.
In practice, this means the UI continuously answers three questions without requiring the operator to interpret logs or guess semantics:
1) **Are we in a normal trading contract or a degraded/locked contract?**
2) **If the system is trading, what decisions did it make and why (act vs block)?**
3) **If something is off, what is the next safe action (ack, wait, reconcile, or execute the kill switch with confirmation)?**

This keeps the operator’s attention on governed outcomes rather than internal complexity.

### Platform Strategy
Provide a **local operator console** (desktop-browser experience via Streamlit) that is optimized for:
- **Fast comprehension** in under ~10 seconds during market hours
- **Single-screen readability** with clear modes, reason codes, and actionable next steps
- **Keyboard/mouse efficiency** for an intermediate technical operator
- **Progressive disclosure**: summary first, details on demand

Pair it with **Telegram notifications** for out-of-band “must-not-miss” events:
- mode changes (e.g., WebSocket vs `REST_POLL`)
- kill switch events and confirmations
- reconciliation failure summaries
- training/retraining status and failures
- dead-man’s-switch alerts

### Effortless Interactions
The UI should make these actions nearly automatic:
- **Default dashboards that stay honest**: prominent mode/state labels that directly reflect the system contract
- **Act/Block decision cards** that always show the same causal chain: freshness/data mode → feature/signal snapshot → risk verdict → broker/order outcome (or explicit suppression)
- **Reason codes with “what this means”**: each block/reject explanation should include an operator-facing interpretation and next step
- **One-click acknowledgements where safe** (e.g., acknowledging alerts) while keeping risk-increasing actions gated
- **Reconciliation readiness cues**: clear “pass/failed/needs waiver” signals so EOD is a routine checklist, not an investigation
- **Kill switch ergonomics**: visible only as “dangerous but present,” with a deliberate two-step confirmation flow

### Critical Success Moments
1) **Morning briefing (first glance success)**: the operator instantly understands watchlist readiness, mode status, and whether entries are allowed yet.
2) **Observation-only window (09:15–09:30)**: the UI must make “no new entries by policy” feel intentional and obvious.
3) **Degraded mode transition (WebSocket drop → `REST_POLL`)**: within moments, the operator understands what is still actively managed vs what is intentionally suppressed.
4) **Risk gate rejection moment**: the operator sees the stable reason and knows whether anything changes next time or today requires intervention.
5) **EOD reconciliation**: the operator gets a clear “reconciled” outcome or a blocked-next-day outcome with a constrained path to clear it.

### Experience Principles
- **Contract-first semantics**: the UI mirrors system truth (including suppression-by-policy), never approximate it.
- **Attention compression**: summaries and reason codes are designed for fast “next action” understanding.
- **Safety through constrained affordances**: dangerous actions require deliberate user confirmation and explicit audit context.
- **Traceability by default**: decision context shown in the UI is backed by the same evidence the operator can reconcile later.
- **Progressive disclosure, not cognitive load**: reveal detail only when the operator asks for it.

## Desired Emotional Response

### Primary Emotional Goals
- **Calm control**: the operator feels steady and oriented, even when the system is degraded or locked.
- **Trust without blind faith**: “I can believe this because it’s explainable and reconcilable,” not “I hope it’s fine.”
- **Instant clarity**: the operator experiences quick comprehension of mode + act/block outcomes + next action.
- **Preparedness**: when something goes wrong (e.g., WebSocket loss), the operator feels supported by an explicit, predictable contract.
- **Closure and confidence at day end**: reconciliation feels like a checklist that resolves, not a mystery to investigate.

### Emotional Journey Mapping
- **First discovery / start-of-day**: curiosity becomes reassurance as the morning overview quickly communicates watchlist readiness and whether entries are allowed yet.
- **During the core loop (market hours)**: focused attentiveness—operator feels “in command” because reason codes and next actions reduce uncertainty.
- **Observation-only / locked windows**: tension is replaced with relief when the UI makes “no new entries by policy” intentional and unambiguous.
- **Degraded mode transition (`WEBSOCKET` → `REST_POLL`)**: anxiety is dampened by explicit mode labels and a clear contract: monitor/maintain positions, suppress new entries.
- **Risk gate rejections**: skepticism converts to confidence when the block reason is stable and paired with “what would change” expectations.
- **Governance moments (config/model approvals)**: seriousness without fear—controls feel locked when required and review feels safe when allowed.
- **EOD reconciliation**: satisfaction comes from evidence-based resolution (pass/failed/waived with explicit next steps).

### Micro-Emotions
- Confidence instead of confusion (fast “what state are we in?”)
- Trust instead of skepticism (reason codes tied to evidence)
- Relief instead of anxiety (degraded-mode semantics are explicit)
- Accomplishment instead of frustration (next actions are clear and constrained)
- Satisfaction (and occasional delight) from “boring reliability” that consistently delivers clarity

### Design Implications
- Always-visible **contract-first status** (normal vs degraded vs locked) with explicit meaning for each.
- **Act/Block decision cards** that present a consistent causal chain and never leave the operator guessing whether silence means success.
- **Reason codes + operator interpretation + next step** on every reject/block.
- **Reconciliation as a workflow** (pass/fail/needs waiver) rather than a buried dashboard detail.
- **Kill switch ergonomics**: prominent but not frantic; two-step confirmation reinforces safety without panic.
- **Governance lock UX**: disabled controls + visible “why locked” tied to market-hours policy.

### Emotional Design Principles
- **Truthfulness over aesthetics**: the UI must mirror system semantics precisely (including suppression-by-policy), never approximate it.
- **Attention compression**: show what the operator needs now; hide forensic detail unless asked.
- **Guided next actions**: every failure mode ends with “here’s what to do next,” not just an explanation.
- **Evidence-backed reassurance**: explanations should map to logs/reconciliation so trust is earned.
- **Constrained affordances for dangerous actions**: deliberate friction only where risk increases.
