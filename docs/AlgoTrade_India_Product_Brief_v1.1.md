# AlgoTrade India — Product Brief v1.2
**AI-Powered NSE/BSE Intraday Trading System**

| Field | Value |
|---|---|
| Version | 1.2 — Governance, Execution Realism, and Reliability Enhancements |
| Status | Draft — For BMAD Processing |
| Audience | Development Team / AI Agent |
| Domain | Algorithmic Day Trading, India (NSE/BSE) |
| Broker API | ICICI Direct Breeze (WebSocket + REST) |
| Target Hardware | Home PC — RTX 5060 Ti 16GB |

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Regulatory and Compliance Context](#2-regulatory-and-compliance-context)
3. [System Architecture Overview](#3-system-architecture-overview)
4. [Data Ingestion and Storage](#4-data-ingestion-and-storage)
5. [Watchlist and Universe Management](#5-watchlist-and-universe-management)
6. [Feature Engineering](#6-feature-engineering)
7. [ML Prediction Engine](#7-ml-prediction-engine)
8. [Backtesting and Validation](#8-backtesting-and-validation)
9. [Risk Management](#9-risk-management)
10. [Execution Engine](#10-execution-engine)
11. [Continuous Retraining Pipeline](#11-continuous-retraining-pipeline)
12. [Blackout and Calendar Management](#12-blackout-and-calendar-management)
13. [Monitoring and Alerting](#13-monitoring-and-alerting)
14. [Operational Resilience](#14-operational-resilience)
15. [Performance Tracking and Attribution](#15-performance-tracking-and-attribution)
16. [Behavioural and Psychological Safeguards](#16-behavioural-and-psychological-safeguards)
17. [Tax and Financial Record-Keeping](#17-tax-and-financial-record-keeping)
18. [Recommended Technology Stack](#18-recommended-technology-stack)
19. [Phased Delivery Plan](#19-phased-delivery-plan)
20. [Open Questions and Decisions Required](#20-open-questions-and-decisions-required)
21. [Data Contract and Feature Availability Matrix](#21-data-contract-and-feature-availability-matrix)
22. [Execution Policy and Broker API Constraints](#22-execution-policy-and-broker-api-constraints)
23. [Model Governance and Deployment Controls](#23-model-governance-and-deployment-controls)
24. [SLOs, Reconciliation, and Operational Readiness](#24-slos-reconciliation-and-operational-readiness)

---

## 1. Executive Summary

AlgoTrade India is a self-hosted, AI-powered intraday trading system designed for the Indian equity markets (NSE/BSE). It connects to live market data via the ICICI Direct Breeze WebSocket API, applies machine learning models to identify high-probability intraday trade opportunities, executes or simulates orders through an internal API, and continuously retrains on the outcomes of its own predictions.

The system uses a two-tier data acquisition strategy: the Breeze REST (CRUD) API provides historical data bootstrap, startup gap fill, and WebSocket failure recovery, while the WebSocket channel handles real-time incremental updates only. The database is always the authoritative source of truth.

> **Primary Goal:** Build a system that learns to trade profitably over time, starting in paper trading mode and graduating to live trading only after demonstrable, consistent, risk-adjusted performance across a minimum of 90 trading days.

### 1.1 Core Objectives

- Ingest real-time NSE/BSE tick data via Breeze WebSocket and store it in a structured, retraining-ready format
- Use Breeze REST API for historical data bootstrap, startup gap fill, and WebSocket failure fallback
- Engineer a rich feature set from raw price and volume data, supplemented by cross-asset and macro context
- Train, validate, and deploy ML models (LSTM Phase 1, Reinforcement Learning Phase 2) to generate Buy / Hold / Sell signals
- Execute signals in paper trading mode by default, with a gated transition to live trading
- Continuously retrain models on real outcomes to improve accuracy over time
- Enforce hard risk limits, circuit breakers, and kill switches independent of the ML layer
- Maintain a complete, tamper-evident audit trail for compliance, tax, and debugging

### 1.2 Out of Scope (v1.0)

- Futures and options trading
- Multi-user or multi-account support
- SEBI AIF registration or third-party fund management
- High-frequency trading (sub-second execution)
- Mobile application interface

---

## 2. Regulatory and Compliance Context

> **Critical Regulatory Notice:** A static IP address is mandatory for Breeze API-based trading from 1st April 2026. All existing API keys were invalidated at this date. A new API key must be generated. Ensure your network configuration provides a static IP before any live trading activity.

### 2.1 SEBI Algo Trading Rules

- Any strategy sending more than 10 orders per second must be registered as an algo with the exchange. This system is designed to stay well below this threshold.
- Every algorithm must carry an exchange-assigned Algo-ID. Confirm registration requirements with ICICI Direct before live deployment.
- The system must never operate as an unregistered collective investment scheme. If external capital is accepted, a formal legal structure (Partnership Firm or LLP) must be established before funds are received.

### 2.2 Tax Compliance Requirements

- Intraday equity trading profits are classified as business income in India, not capital gains.
- Turnover for intraday equity is the absolute sum of all settlement amounts. Turnover exceeding Rs. 10 crore or profit exceeding Rs. 1 crore triggers mandatory tax audit requirements.
- The system must maintain a running turnover figure updated after every trade.
- All trade logs must be exportable in a format suitable for CA/ITR filing.

### 2.3 Audit Trail Requirements

- Every signal, order, fill, model version, and configuration change must be logged with a UTC timestamp in an append-only format.
- Daily checksums must be maintained to demonstrate the audit trail has not been tampered with.
- All records must be retained for a minimum of three years.

---

## 3. System Architecture Overview

The system is structured as five distinct, independently deployable layers. Each layer has a single clear responsibility. The ML layer must never have direct access to order submission — all orders pass through the risk management layer.

| Layer | Responsibility | Key Components |
|---|---|---|
| 1 — Data Ingestion | Real-time and historical market data acquisition | Breeze WebSocket, Breeze REST API, CRUD fallback, dedup layer, TimescaleDB / Parquet |
| 2 — Feature Engineering | Transform raw data into model-ready features | Technical indicators, cross-asset context, watchlist filter |
| 3 — ML Prediction Engine | Generate directional signals with confidence scores | LSTM (Phase 1), RL Agent (Phase 2), ensemble layer |
| 4 — Risk and Execution | Validate signals, enforce limits, place orders | Pre-trade checks, order state machine, broker API adapter |
| 5 — Retraining Pipeline | Continuous model improvement from live outcomes | Label generator, training orchestrator, model registry |

### 3.1 Guiding Architectural Principles

- The risk management layer is the only path to order submission. The ML layer cannot bypass it under any circumstances.
- Paper trading mode and live trading mode must share identical code paths. Only the final order submission step differs.
- Every component must fail safe. An unhandled exception defaults to closing open positions, not opening new ones.
- Configuration is locked during market hours (09:00 to 15:45 IST). Parameter changes take effect only after market close.
- The system must support a simulation clock mode, enabling replay of any historical period through the full stack for backtesting and debugging.
- The database is the single source of truth for all market data. No component assumes the WebSocket is always available.

---

## 4. Data Ingestion and Storage

### 4.1 Market Data Sources

- **Primary real-time:** ICICI Direct Breeze WebSocket API — streaming OHLCV tick data for all subscribed NSE/BSE instruments during market hours
- **Historical and fallback:** ICICI Direct Breeze REST API — up to 3 years of second-level LTP data at no charge; also used for gap fill and disconnection recovery
- **Supplementary:** NSE announcements feed — corporate actions, board meeting dates, results calendar
- **Cross-asset:** Yahoo Finance API (free) — US S&P 500 futures, USD/INR, crude oil price, gold price
- **Index data:** Nifty 50 and Sensex real-time data via Breeze — mandatory model features for all individual stock predictions

### 4.2 Data Quality and Validation

All incoming data — regardless of source (WebSocket or REST) — must pass the same validation pipeline before being stored or used as model input:

- Price within a configurable tolerance band of the previous tick (no single-tick move exceeding 5%)
- Timestamp strictly monotonically increasing — out-of-order records are discarded with a logged warning
- No duplicate records — deduplication by `(stock_token, timestamp, resolution)` primary key
- Gap detection — if more than 60 seconds of data is missing during market hours, an alert is triggered and the affected feature window is flagged as unreliable
- Zero-price and zero-volume records are discarded
- Price band awareness — records at or beyond the stock's NSE circuit limit are flagged

### 4.3 Storage Architecture

- Tick data: TimescaleDB (PostgreSQL time-series extension) for recent data, with automated archival to Parquet files for long-term storage
- Every stored record includes a `data_source` field: `WEBSOCKET | REST_HISTORICAL | REST_BACKFILL | REST_POLL`
- Feature snapshots: each signal event records the complete feature vector at the moment of signal generation
- Model outputs: every prediction stored regardless of whether a trade was placed
- Trade outcomes: actual fill prices, quantities, timestamps, and P&L linked back to the originating signal
- Schema is immutable once data begins flowing — structural changes require a migration with version tagging

### 4.4 WebSocket Resilience

- Automatic reconnection with exponential backoff on connection loss
- Fallback to Breeze REST API polling if WebSocket cannot be re-established within 90 seconds
- Mobile push notification (Telegram bot) if connectivity cannot be restored within 120 seconds during market hours
- All reconnection attempts and outcomes are logged

### 4.5 CRUD Fallback and Historical Bootstrap Strategy

The WebSocket connection is the primary data channel, but it is not always available — it requires market hours, an active session token, and a stable network. The system therefore implements a two-tier data acquisition strategy: CRUD-based REST API calls for historical and recovery data, and WebSocket for real-time incremental updates only.

> **Design Principle:** WebSocket is a live stream, not a database. The system never depends on WebSocket availability for data completeness. All gaps — from startup, disconnection, or overnight periods — are filled using the Breeze REST (CRUD) API. The WebSocket layer only saves data that is genuinely new relative to what is already stored.

#### 4.5.1 Historical Data Bootstrap

When the system is first installed, or when a new stock is added to the watchlist, the bootstrap procedure fills local history using the Breeze historical CRUD endpoint:

- Query the Breeze historical data API (REST) for up to 3 years of OHLCV data at 1-minute resolution for each watchlist stock
- Determine the most recent timestamp already stored locally for that stock (or epoch zero if none exists)
- Request only the date range from that timestamp to the present — never re-download data that is already stored
- Validate all retrieved records through the standard data quality pipeline before writing to the database
- Mark all historically-sourced records with `data_source = 'REST_HISTORICAL'`
- Bootstrap runs sequentially per stock with a configurable inter-request delay (default: 500ms) to avoid API rate limits

#### 4.5.2 Startup Gap Fill

Every time the system starts, there will be a gap between the last stored tick and the current moment. This gap is filled before the WebSocket connection is opened:

- On startup, query the last stored timestamp per stock from the local database
- For each stock, call the Breeze REST historical endpoint to retrieve all data from that timestamp to now
- Write any new records to the database, marked as `REST_HISTORICAL`
- Only after gap fill is confirmed complete for all watchlist stocks does the system proceed to open the WebSocket connection
- This guarantees the feature engineering pipeline always has a complete, unbroken time series regardless of how long the system was offline

#### 4.5.3 WebSocket Disconnection Recovery

When the WebSocket connection drops during market hours and cannot be re-established within 90 seconds, the system switches to REST polling mode:

- Switch to Breeze REST quote API, polling at a configurable interval (default: every 60 seconds) for all watchlist stocks
- Each poll result is stored with `data_source = 'REST_POLL'` and the poll timestamp
- REST poll data is used for position monitoring and stop-loss evaluation only — **new position signals are suppressed during REST polling mode**, since data granularity is insufficient for reliable signal generation
- When WebSocket reconnects, issue a REST historical request for the gap period (last WebSocket tick to reconnection time) to backfill missed ticks, stored as `REST_BACKFILL`
- The system resumes normal WebSocket-driven signal generation after backfill is confirmed complete

#### 4.5.4 Deduplication and Merge Logic

Because multiple data sources may provide overlapping records for the same instrument and timestamp, a merge and deduplication layer sits between all data ingestion paths and the database:

- Primary key for deduplication: `(stock_token, timestamp, resolution)` — matching primary key = duplicate
- On conflict, the existing record is retained — incoming duplicates are discarded with a debug log entry
- **Exception:** if the existing record has `data_source = 'REST_POLL'` (low granularity) and the incoming record has source `WEBSOCKET` or `REST_HISTORICAL` (higher quality), the higher-quality record overwrites the poll record
- The `data_source` field on every stored record provides a complete data lineage trail

#### 4.5.5 Data Source Priority Hierarchy

| Data Source | Priority and Use |
|---|---|
| `WEBSOCKET` | Highest quality. Real-time tick data during normal operation. Used for all signal generation. |
| `REST_HISTORICAL` | High quality. Used for initial bootstrap and startup gap fill. Identical resolution to WebSocket data. |
| `REST_BACKFILL` | High quality. Used to fill specific gaps after WebSocket reconnection. |
| `REST_POLL` | Lower quality (60-second granularity). Used for position monitoring only during extended disconnection. Overwritten by higher-quality data when available. |

#### 4.5.6 Incremental Save — WebSocket Only Writes New Data

The WebSocket handler checks the last stored timestamp before writing any incoming tick. If the tick timestamp is not strictly greater than the last stored timestamp for that instrument, the tick is discarded as a duplicate. This means:

- The database is the single source of truth — the WebSocket handler is stateless with respect to history
- Restarting the WebSocket connection never causes duplicate data in the database
- The system can be restarted or redeployed without data integrity risk
- Storage growth is strictly monotonic — each new WebSocket tick either adds a new record or is discarded

#### 4.5.7 Session Token Management for CRUD Operations

- Session token is stored encrypted at rest and in memory only
- Token validity is checked before every REST API call — an expired token triggers a re-authentication flow
- Re-authentication is fully automated using stored credentials during market hours
- If re-authentication fails, the system alerts the operator, switches to position-monitoring-only mode, and retries every 5 minutes
- Session token rotation is logged with timestamps for audit purposes

---

## 5. Watchlist and Universe Management

The model does not observe all listed stocks. A dynamic watchlist defines the universe of eligible instruments on any given day, based on configurable filters evaluated weekly.

### 5.1 Mandatory Exclusion Filters

- Average daily traded value below a configurable minimum (default: Rs. 20 crore per day)
- Stocks where the system's intended order size exceeds 1% of the stock's average daily volume
- Stocks within 3% of their NSE upper or lower price circuit limit
- Stocks with a pending corporate action within the next 5 trading days
- Stocks with scheduled results announcements within 2 trading days

### 5.2 Configurable Inclusion Filters

- Minimum share price (default: Rs. 100)
- Maximum share price (default: Rs. 5,000)
- Sector inclusion/exclusion list
- Index membership (e.g., Nifty 100 only)
- Maximum sector concentration — no more than 2 stocks from the same NSE sector simultaneously

### 5.3 Watchlist Refresh

- Full filter evaluation runs every Saturday night on fresh weekly data
- The resulting watchlist takes effect at Monday market open
- Mid-week emergency removals are permitted and take effect immediately
- Any change to the watchlist is logged with the reason and timestamp
- When a stock is added to the watchlist, a historical data bootstrap (section 4.5.1) is triggered automatically before the stock becomes eligible for signals

---

## 6. Feature Engineering

### 6.1 Price and Volume Features

- OHLCV at 1-minute, 5-minute, and 15-minute bar resolutions
- RSI (14-period), MACD (12/26/9), Bollinger Bands (20, 2 standard deviations), ATR
- VWAP and deviation from VWAP
- EMA crossovers (9/21 and 50/200)
- Volume Z-score relative to rolling 20-day average volume at the same time of day
- Volume delta (buy volume minus sell volume, if available via market depth)
- On-Balance Volume (OBV)
- Delivery percentage (from NSE EOD data)
- Rolling momentum: price change over 1, 5, and 15 minute lookback windows

### 6.2 Cross-Asset and Macro Features

- Nifty 50 and Sensex: current day percentage change, 5-minute momentum, distance from day high/low
- Put-Call Ratio from NSE options chain data
- India VIX — current level and change from previous close
- FII and DII net buying/selling for the current day
- USD/INR exchange rate — current level and intraday change
- Crude oil price — current level and percentage change from previous close
- US S&P 500 futures — overnight change, used primarily for pre-market context

### 6.2.1 Feature Availability and Leakage Controls

To prevent look-ahead bias and hidden data leakage, every feature must be tagged with an availability class and enforced both in training and live inference:

- `REAL_TIME`: can be consumed immediately during market hours
- `DELAYED_INTRADAY`: available intraday but with known delay/latency
- `EOD_ONLY`: available only after market close; never usable for same-day intraday signal generation

Rules:

- Any feature tagged `EOD_ONLY` is excluded from live intraday inference and from backtest bars before its publication time
- Training pipelines must simulate actual publication/arrival latency for `DELAYED_INTRADAY` features
- If an external feature source is unavailable intraday, feature value is set to `MISSING` with explicit missingness flags; no forward-fill beyond configured TTL
- A pre-training leakage audit runs on every dataset version and blocks model training if timing violations are detected

### 6.3 Temporal Features

- Time of day encoded cyclically (sine and cosine of the minute within trading session)
- Day of week (Monday through Friday, one-hot encoded)
- Days until next scheduled F&O expiry (monthly, last Thursday)
- Days until next scheduled RBI policy announcement
- Is this an F&O expiry day (binary)
- Is this the first or last 15 minutes of the trading session (binary flags)

### 6.4 Feature Pipeline Requirements

- All features normalised using rolling Z-scores over a trailing window — no look-ahead bias
- Feature computation uses only data available at the moment of signal generation
- Feature importance measured and logged weekly using SHAP values
- A feature freshness check runs before each prediction: if any mandatory feature is more than 2 minutes stale, the prediction is suppressed
- If the `data_source` for any tick in the current feature window is `REST_POLL`, a low-confidence flag is attached to the resulting signal and the confidence threshold is raised by 10 percentage points

---

## 7. ML Prediction Engine

### 7.1 Phase 1 — LSTM Model

- **Input:** rolling window of 60 one-minute timesteps, each containing the full feature vector
- **Output:** three-class softmax — probability of price direction being Up, Flat, or Down over the next 5 to 15 minutes
- **Architecture:** two LSTM layers (128 units, 64 units) with dropout regularisation, followed by a dense classification head
- **Framework:** PyTorch with CUDA 12.8 (required for Blackwell / RTX 5060 Ti architecture)
- **Training data:** 3 years of historical Breeze data, with the most recent 6 months held out as a permanent validation set — never used for training
- **Signal generation:** a trade signal is only emitted when predicted probability exceeds a configurable confidence threshold (default: 65%)

### 7.2 Phase 2 — Reinforcement Learning Agent

- **Framework:** stable-baselines3 (PPO or DQN) in a custom OpenAI Gym environment
- **Inputs:** LSTM signal and confidence, current position state, unrealised P&L, time of day, current drawdown, recent win/loss streak
- **Outputs:** position sizing decision and timing
- The RL environment simulates full cost structure including transaction costs, slippage, and market impact

### 7.3 Ensemble Layer

- **Model 1:** LSTM (primary)
- **Model 2:** XGBoost trained on the same feature set
- **Model 3:** Simple momentum model — rule-based baseline and sanity check
- **Ensemble rule:** minimum 2 of 3 models must agree on direction for a signal to proceed
- Disagreement rate between models is logged and monitored as an early indicator of regime change

### 7.4 Market Regime Detection

- Regime classifier runs every 15 minutes based on ADX and short-term volatility
- **Trending regime:** standard signal thresholds, momentum features weighted more heavily
- **Sideways/choppy regime:** confidence threshold raised to 75%, position sizes reduced
- **High volatility regime** (VIX spike or Nifty move exceeding 1.5% intraday): no new positions; existing positions managed to stops
- Regime state is a mandatory input feature for all models

### 7.5 Multiple Timeframe Confirmation

- A 1-minute signal is only acted upon if the 5-minute and 15-minute models agree on direction
- Timeframe disagreement results in a Hold signal regardless of individual confidence

### 7.6 Model Explainability

- SHAP values computed for every live prediction and stored alongside the signal log
- Top 3 contributing features for each signal surfaced in the monitoring dashboard
- Weekly SHAP summary identifies features whose importance has shifted significantly

---

## 8. Backtesting and Validation

> **Anti-Overfitting Mandate:** The permanent validation set (most recent 6 months of historical data) is never used during training or hyperparameter tuning. It is used only for final model evaluation before deployment. Violation of this rule invalidates all performance claims.

### 8.1 Backtesting Engine Requirements

- No look-ahead bias: the model receives only data that would have been available at the exact timestamp of the simulated signal
- Realistic fill simulation: orders fill at the next available price after the signal, not at the signal bar's close
- Transaction costs applied to every simulated trade: brokerage (flat Rs. 20 per order), STT (0.025% sell side), NSE exchange charges, GST on brokerage, stamp duty
- Slippage modelled as a function of the stock's average daily volume and the order size
- Simulation clock mode: the full data pipeline and feature engine support receiving historical data as if it were live

### 8.2 Walk-Forward Validation

- Training: 18 months. Validation: 3 months. Step: 3 months.
- The model must show consistent positive performance across all out-of-sample windows
- A model performing well on one window but poorly on others is considered overfit and is rejected

### 8.3 Benchmark Comparisons

- **Benchmark 1:** Random signal generator with identical position sizing and cost structure
- **Benchmark 2:** Simple EMA crossover strategy (9/21)
- **Benchmark 3:** Nifty 50 index buy-and-hold
- All benchmarks reported on a risk-adjusted basis (Sharpe Ratio) and absolute return

### 8.4 Stress Testing

Monthly, the current production model is run against historical stress periods:
- March 2020: COVID-19 crash
- 2015–2016: Chinese market contagion
- 2022: Global rate hike selloff
- Any recent period where the model experienced its worst recorded drawdown

### 8.5 Historical Replay Tool

- Given any past trading date, the system replays that day through the current production model
- Output: complete record of every signal, every trade, and resulting P&L
- The replay tool uses the same data source hierarchy as live operation: stored database records first, REST API to fill any gaps

---

## 9. Risk Management

> **Hard Limit:** If the risk management layer is unavailable or in an error state, no orders may be submitted. The system must fail closed, not open.

All parameters are configurable but require explicit human confirmation to change, and cannot be changed during market hours.

### 9.1 Position-Level Controls

- Maximum single position size: configurable percentage of total capital (default: 25% single-stock, 20% per stock for multi-stock mode in Phase 1)
- Stop-loss: mandatory for every open position, set at order placement time (default: -1.5%)
- Profit target: set at order placement time (default: +2.0%)
- Maximum holding time: any position open at 15:20 IST is automatically closed
- Bracket orders are the mandatory default order type — stop-loss and target live on the broker's servers
- Stale signal protection: if more than 60 seconds elapse between signal generation and order submission, the order is cancelled

### 9.2 Portfolio-Level Controls

- Maximum simultaneous open positions: configurable (default: 1, maximum 5)
- Maximum sector concentration: no more than 2 open positions in the same NSE sector
- Correlation guard: if two candidate signals have 90-day return correlation above 0.8, only the higher-confidence signal is accepted
- Maximum daily loss limit: if total P&L reaches a configurable negative threshold (default: -3% of capital), all positions close and no new positions open for the remainder of the day
- Maximum daily trade count: configurable upper limit on orders per day

### 9.3 Kill Switch

- Closes all open positions immediately and halts all new signal processing
- Accessible via: internal API endpoint, Telegram bot command (two-step confirmation), and dashboard keyboard shortcut
- After activation, the system enters a locked state — requires explicit human unlock to resume

### 9.4 Pre-Trade Checks

Every signal must pass all checks before an order is submitted. Any failed check cancels the order and logs the reason.

- Market is open and not in pre-open or post-close session
- Stock is not in an active circuit breaker halt
- Current bid-ask spread within acceptable maximum (default: 0.2% of mid-price)
- Signal is not older than 60 seconds
- Daily loss limit has not been reached
- Maximum position count has not been reached
- Sector concentration limit would not be breached
- Order value passes fat-finger sanity check
- No pending corporate action for this stock today
- Not within 30 minutes of a scheduled major market event
- System clock confirmed NTP-synchronised — trading suspended if drift exceeds 2 seconds
- Current data source for this stock is not `REST_POLL` (insufficient granularity for signal generation)

---

## 10. Execution Engine

### 10.1 Internal API

All order activity passes through an internal FastAPI service — the sole interface between the ML layer and the broker API.

| Endpoint | Purpose |
|---|---|
| `POST /signal` | Receives signal from ML layer, runs pre-trade checks, submits or rejects |
| `POST /order` | Direct order submission (manual override only, still passes all risk checks) |
| `GET /positions` | All currently open positions with real-time P&L |
| `GET /pnl` | Daily, weekly, and monthly P&L summary |
| `GET /health` | System health check endpoint, polled by dead man's switch |
| `POST /kill` | Activates the kill switch (requires authentication token) |
| `POST /mode` | Switches between paper and live trading modes (market-hours locked) |
| `GET /data/status` | Current data source mode per stock and last tick timestamps |

### 10.2 Order State Machine

| State | Description |
|---|---|
| `SIGNAL_RECEIVED` | ML layer has emitted a signal |
| `PRE_TRADE_CHECKS` | Risk layer is evaluating the signal |
| `REJECTED` | Signal failed one or more pre-trade checks — terminal |
| `ORDER_SENT` | Order submitted to Breeze API |
| `ORDER_CONFIRMED` | Broker has acknowledged the order |
| `POSITION_OPEN` | Order filled — position is live |
| `EXIT_SIGNAL` | Exit condition met (target, stop-loss, time, or kill switch) |
| `EXIT_SENT` | Exit order submitted to broker |
| `EXIT_CONFIRMED` | Exit order acknowledged by broker |
| `FLAT` | Position closed — terminal, outcome logged |
| `ERROR` | Unexpected state — safe-close procedure initiated |

Execution policy requirements:

- Partial fills are first-class states; risk checks continue until fully filled or cancelled
- Order rejection handling includes bounded retry logic with reason-code aware backoff
- Cancel/replace is allowed only within configured limits to avoid runaway order churn
- Any order not acknowledged within timeout enters `ERROR` and triggers safe-close workflow
- Exchange session boundaries (pre-open, auction, post-close) enforce mode-specific order restrictions

### 10.2.1 Broker and Exchange Constraint Controls

- Broker REST/WebSocket/API rate limits are treated as hard constraints with per-endpoint token buckets
- Max subscription count, max request burst, and session token TTL are externalized in configuration
- If broker returns stale, empty, or malformed payloads above threshold, system enters degraded mode (`monitor-only`) and suppresses new entries
- Symbol-level tradeability gate blocks signals for instruments on surveillance/restriction lists (ASM/GSM/trade-for-trade/corporate suspension)
- Quantity freeze limits and lot-size constraints are validated pre-order to prevent exchange-side rejections

### 10.3 Paper Trading Mode

- Identical to live mode except: `ORDER_SENT` step simulates a fill at current mid-price plus a configurable slippage factor
- Simulated fills marked as such in all logs
- Paper trading P&L tracked separately from live P&L
- Minimum paper trading period before live deployment: 90 consecutive trading days with net positive performance

### 10.4 Staged Capital Deployment

| Stage | Period | Capital |
|---|---|---|
| Stage 1 | Days 1–90 | Paper trading only |
| Stage 2 | Days 91–110 | Live at 25% of intended capital |
| Stage 3 | Days 111–130 | Live at 50% of intended capital |
| Stage 4 | Day 131+ | Full intended capital, subject to risk limits |

Each stage transition requires explicit human sign-off and is logged.

---

## 11. Continuous Retraining Pipeline

### 11.1 Label Generation

- **Correct Up:** signal was Buy, price rose more than threshold within target window
- **Correct Down:** signal was Sell, price fell more than threshold
- **Incorrect:** signal direction was wrong
- **Inconclusive:** price remained within the threshold range
- The `data_source` of ticks used to generate the label is stored — `REST_POLL` labels are flagged as lower reliability

### 11.2 Nightly Retraining Procedure

- Runs at 22:00 IST every trading day
- Model retrained on all accumulated data except the permanent validation set
- New model deployed only if it outperforms the current production model on at least 3 of 5 key metrics: accuracy, Sharpe ratio, maximum drawdown, precision, recall
- If the new model does not pass, the current model is retained and the failure is logged
- Every model version archived with training date, data date range, validation metrics, and git commit hash

### 11.5 Champion-Challenger and Rollback Policy

- Current production model is the Champion; every candidate is a Challenger evaluated in shadow mode first
- Challenger promotion requires passing statistical and risk gates on rolling out-of-sample windows
- Rollout policy: paper shadow -> limited-capital canary -> full deployment
- Rollback SLA: any severe degradation alert triggers automatic rollback to last stable Champion within 5 minutes
- No model deployment, parameter mutation, or feature schema change is permitted during market hours

### 11.3 Model Drift Detection

- Population Stability Index (PSI) calculated daily on prediction confidence distributions — PSI above 0.2 triggers alert
- Rolling accuracy over last 20 trading days compared to validation baseline — drop of more than 5 percentage points triggers alert
- Feature distribution monitoring: mean or variance shift of more than 2 standard deviations triggers alert

### 11.4 Model Retirement Policy

- If rolling 60-day accuracy falls below 50% AND the model has produced a net loss of more than 15% of capital in that period, the model is automatically retired
- On retirement, live trading immediately suspended and system enters paper-only mode
- Retirement requires a full training and walk-forward validation cycle on fresh data before re-activation

---

## 12. Blackout and Calendar Management

### 12.1 Full Trading Blackouts — No New Positions

- Union Budget day
- RBI Monetary Policy Committee announcement days
- General election result days
- NSE-declared market holidays
- Any day where Nifty opens with a gap exceeding 2% in either direction

### 12.2 Partial Restrictions

- F&O expiry day (last Thursday of each month): confidence thresholds raised by 10 percentage points, maximum position size halved
- Day before a scheduled major event: no new positions after 14:00 IST
- First 15 minutes of trading (09:15 to 09:30 IST): observation only, no new positions
- Last 10 minutes of trading (15:20 to 15:30 IST): no new positions, existing positions managed to stops

### 12.4 Regime-Change and Confidence-Collapse Safeguards

- If rolling live precision or win-rate falls below configured floor over N recent trades, new entries are throttled
- If model disagreement rate or uncertainty rises above threshold, system downgrades to rule-based baseline signals
- If collapse conditions persist beyond configured duration, system automatically switches to paper-only mode pending human review

### 12.3 Calendar Data Source

- NSE publishes the holiday calendar annually — loaded into the system at the start of each calendar year
- RBI and government event dates sourced manually and loaded into the event blackout calendar
- The calendar is version-controlled and any modification is logged

---

## 13. Monitoring and Alerting

### 13.1 Real-Time Dashboard

A local web dashboard (Streamlit) provides a single-screen operational view during market hours — readable in under 10 seconds.

- Current open positions: stock, entry price, current price, unrealised P&L, stop-loss, time in position
- Today's closed trades: entry/exit prices, realised P&L, signal confidence at entry
- Daily P&L summary and running daily loss limit status
- Model signal status for each watchlist stock with confidence score and last update time
- System health: WebSocket connection status, current data source mode per stock, API latency, last tick received
- Market context: Nifty 50 current move, VIX, USD/INR, current regime classification
- Kill switch button: prominently visible, requires two-click confirmation

### 13.2 Notification System (Telegram Bot)

- Order placed, position closed, stop-loss hit, daily loss limit reached, kill switch activated
- WebSocket disconnection alert and data source mode switch (WebSocket ↔ REST_POLL)
- System health alert: API error, model prediction failure, clock sync loss
- Dead man's switch alert: main system has not responded in more than 5 minutes
- Model drift alert: PSI threshold exceeded or accuracy degradation detected
- Morning briefing (08:45 IST): watchlist, events, capital status, model health, data source status
- Daily reconciliation alert (after close): broker vs internal positions/fills/charges mismatch summary

### 13.3 Dead Man's Switch

- Lightweight independent monitoring process polls the main system health endpoint every 60 seconds
- If the main system does not respond within 120 seconds during market hours, immediate Telegram alert
- Dead man's switch process sends a daily heartbeat to confirm it is operational

### 13.4 Weekly Performance Report

Generated automatically every Sunday evening:

- Week's P&L broken down by stock, time of day, and regime
- Model accuracy: overall and by confidence tier
- Percentage of time in WebSocket vs REST_POLL mode, and correlation with signal quality
- Feature importance summary: top 5 and bottom 5 contributing features
- Running capital total, cumulative P&L since inception, current drawdown from peak

---

## 14. Operational Resilience

### 14.1 Network Resilience

- Primary: home broadband with static IP (mandatory for Breeze API compliance)
- Automatic failover to mobile hotspot on connection loss — detected within 30 seconds
- On hotspot failover, WebSocket reconnection is attempted; if unsuccessful within 90 seconds, REST polling mode activates

### 14.2 Power Resilience

- UPS providing minimum 20 minutes runtime
- On UPS power-only mode, immediate alert; graceful shutdown if mains not restored within 5 minutes

### 14.3 Graceful Shutdown Procedure

1. Check for open positions
2. If market is open and positions exist: submit exit orders at market price
3. Wait for exit confirmation (maximum 30 seconds)
4. Flush all pending log writes to disk
5. Save complete system state to disk
6. Send shutdown notification via Telegram
7. Terminate all processes cleanly

### 14.4 Startup Recovery

1. Load last saved system state from disk
2. Run startup gap fill procedure (section 4.5.2) before accepting any signals
3. Query Breeze API for current positions and compare to last known state
4. If discrepancy exists, alert the operator and wait for manual confirmation before proceeding
5. Verify NTP clock synchronisation and static IP
6. Only enter active trading mode after all startup checks pass

### 14.5 Disaster Recovery Documentation

A physical printed document stored securely must contain:

- ICICI Direct web login and manual position management URL
- ICICI Direct customer care phone number for emergency order cancellation
- Steps to disable API access from the ICICI Direct portal
- Instructions for closing all positions manually via the broker web interface
- Contact details of a technically-capable trusted person who can execute the above steps

### 14.6 Disaster Recovery Drills and Targets

- Quarterly DR drills are mandatory and evidence must be archived
- Recovery Time Objective (RTO): trading-safe state restoration within 30 minutes
- Recovery Point Objective (RPO): maximum 1 minute market data loss for active watchlist
- Drill report must include timeline, deviations, and corrective actions

### 14.7 Security and Secrets Management

- All broker/API credentials stored in encrypted secrets manager; no plaintext secrets in repo or logs
- API keys, session secrets, and Telegram tokens rotated on a scheduled cadence and on incident
- Least-privilege access enforced for internal services and operator accounts
- Encrypted backups with periodic restore validation are mandatory
- Incident response runbook covers credential compromise, unauthorized trading activity, and data tampering scenarios

---

## 15. Performance Tracking and Attribution

### 15.1 Trade-Level Metrics

- Entry and exit timestamps, prices, quantities, gross P&L, transaction costs (itemised), net P&L
- Signal confidence at entry, model version that generated the signal, market regime at time of signal
- Data source quality flag at time of signal (`WEBSOCKET` vs `REST_POLL`)
- Whether exit was via target, stop-loss, time-based close, or kill switch

### 15.2 Portfolio-Level Metrics

- Daily, weekly, monthly, and inception-to-date net P&L
- Sharpe Ratio (annualised, using current RBI repo rate as risk-free rate)
- Maximum drawdown (percentage and absolute rupee amount) and duration
- Win rate, average win size, average loss size, profit factor
- Turnover (running total for tax computation)
- Return on capital deployed

### 15.3 Performance Attribution

- P&L breakdown by stock, time of day, market regime, and signal confidence tier
- P&L breakdown by data source quality — `WEBSOCKET` vs `REST_POLL` signals compared
- Attribution reports generated weekly and archived

### 15.4 Model Version Tracking

- Every trade log entry records the model version ID that generated the signal
- Performance metrics computed separately for each model version
- Enables precise measurement of whether a model update improved or degraded performance

### 15.5 Daily Broker Reconciliation

- End-of-day reconciliation compares broker contract notes/tradebook with internal execution logs
- Mandatory checks: positions, fills, fees/charges, timestamps, and net P&L
- Any unresolved mismatch blocks next-day live mode until cleared or explicitly waived with audit note

---

## 16. Behavioural and Psychological Safeguards

### 16.1 Manual Intervention Journal

- Every manual override requires the operator to enter a text reason before execution
- Reason, timestamp, market state, and subsequent outcome stored permanently
- Monthly report summarises all interventions and whether they helped or hurt performance

### 16.2 Configuration Lockout

- All model parameters, risk limits, and watchlist configurations locked from 09:00 to 15:45 IST
- Any modification attempt during market hours requires a secondary password and generates an audit log entry

### 16.3 Win Streak Risk Reduction

- After 5 consecutive winning trades, position sizes automatically reduced by 20% for the following 5 trades
- Operator notified when this reduction is applied and when it is lifted

### 16.4 Post-Loss Cooling-Off

- After hitting the daily loss limit, a 24-hour lockout before live trading resumes
- Operator must actively unlock the system — it cannot resume automatically
- The unlock action is logged

### 16.5 Profit Withdrawal Discipline

- Monthly profit withdrawal reminder generated with recommended withdrawal amount
- Prevents accumulated profits from being silently reabsorbed into the risk pool

---

## 17. Tax and Financial Record-Keeping

### 17.1 Tax-Ready Trade Log

- Every trade: date, stock, exchange, buy/sell, quantity, rate, brokerage, STT, exchange charges, GST, stamp duty, net amount
- Daily, monthly, and annual P&L summaries in standard format
- Running turnover figure updated after every trade
- Export in CSV and PDF formats

### 17.2 Tax Audit Threshold Monitoring

- Running turnover tracked continuously
- Alert generated when turnover crosses 80% of the mandatory audit threshold (Rs. 10 crore)

### 17.4 Compliance Operations Cadence

- Monthly CA-ready package generated automatically (tradebook, turnover, charges, P&L, intervention logs)
- Advance tax estimate support report generated quarterly
- Export schema is versioned to maintain continuity across accounting periods

### 17.3 Future: External Capital Accounting

- The data model must accommodate separate capital accounts for multiple investors without a breaking schema change
- This is a future-state requirement — not built in v1.0

---

## 18. Recommended Technology Stack

| Component | Recommended Technology | Notes |
|---|---|---|
| Language | Python 3.11+ | Primary language for all components |
| Broker API — WebSocket | breeze-connect (official SDK) | Static IP mandatory from April 2026 |
| Broker API — REST/CRUD | breeze-connect REST methods | Historical data, gap fill, fallback polling |
| Time-series database | TimescaleDB (PostgreSQL extension) | Parquet for archival storage |
| ML framework | PyTorch with CUDA 12.8 | Required for RTX 5060 Ti (Blackwell) |
| RL framework | stable-baselines3 | Phase 2 only |
| Technical indicators | pandas-ta | Lightweight, Pandas-native |
| Backtesting | vectorbt or backtesting.py | Supports simulation clock mode |
| Internal API | FastAPI | Async, fast, well-documented |
| Model tracking | MLflow (local) | Experiment tracking and model registry |
| Scheduling | APScheduler | Retraining and report generation |
| Monitoring dashboard | Streamlit | Local web dashboard |
| Alerting | python-telegram-bot | Push notifications |
| Data validation | Great Expectations or Pydantic | Tick data quality checks |
| Explainability | SHAP | Feature importance per prediction |
| Version control | Git (local + remote backup) | All code and model configs |

### 18.1 Hardware Requirements

| Component | Specification |
|---|---|
| GPU | RTX 5060 Ti 16GB — sufficient for LSTM Phase 1; Phase 2 RL is tight but workable |
| RAM | Minimum 32GB system RAM recommended |
| Storage | Minimum 1TB NVMe SSD for tick data (3 years, all NSE stocks) |
| Network | Static IP broadband (mandatory) + mobile hotspot (failover) |
| Power | UPS rated for minimum 20 minutes runtime at full load |
| OS | Ubuntu 24.04 LTS or Windows 11 with WSL2 |

---

## 19. Phased Delivery Plan

| Phase | Scope | Gate Criterion |
|---|---|---|
| Phase 1 (Months 1–2) | Data ingestion (WebSocket + REST CRUD), historical bootstrap, gap fill, dedup layer, storage, feature engineering, backtesting engine, LSTM model, paper trading, basic dashboard and alerting | Walk-forward validation shows positive risk-adjusted returns. Paper trading running stably for 30 days. REST fallback tested and verified. |
| Phase 2 (Months 2–3) | Ensemble layer, regime detection, full risk management, pre-trade checks, retraining pipeline, complete audit trail, WebSocket disconnection recovery and REST polling mode | All Must Have features operational. 60 days cumulative paper trading. All data source modes tested under simulated failure conditions. |
| Phase 3 (Months 3–4) | Live trading at staged capital levels, RL agent training (parallel, not yet in production), performance attribution, full tax reporting | Minimum 90 consecutive trading days paper mode with positive Sharpe, max drawdown under 10%, and stable reconciliation. |
| Phase 4 (Ongoing) | RL agent deployed to production ensemble, advanced features, external capital structure if required | RL agent shows improvement over LSTM-only baseline in 30-day live shadow mode before deployment. |

---

## 20. Open Questions and Decisions Required

| Question | Notes |
|---|---|
| What is the initial target watchlist size? | Suggest 10–15 liquid Nifty 100 stocks to keep training fast and interpretable |
| What is the initial confidence threshold? | 65% suggested as default; tune empirically during paper trading |
| What is the REST polling interval during WebSocket failure? | 60 seconds suggested; shorter intervals increase API load |
| What is the maximum acceptable REST_POLL duration before trading halts? | Suggest: if WebSocket unavailable for more than 30 minutes, suspend all new signals |
| What backup/DR infrastructure for the dead man's switch? | Free-tier cloud VM or Raspberry Pi on a separate network connection |
| Will the system trade BSE in addition to NSE? | NSE recommended for v1.0 due to higher equity liquidity |
| What is the formal legal structure for external capital? | Partnership Firm or LLP — requires CA and legal counsel; out of scope for v1.0 |

### 20.1 Decision Ownership and Deadlines

All open questions must include:

- Decision owner
- Decision deadline
- Default action if deadline is missed

This avoids unresolved choices blocking implementation.

---

## 21. Data Contract and Feature Availability Matrix

Every production feature and data field must have a contract entry:

- Source system and endpoint
- Event timestamp vs ingestion timestamp semantics
- Expected latency class (`REAL_TIME`, `DELAYED_INTRADAY`, `EOD_ONLY`)
- Nullability, fallback behavior, and max staleness TTL
- Data quality checks and failure handling policy

Signal generation is blocked if mandatory contract requirements are not met.

---

## 22. Execution Policy and Broker API Constraints

### 22.1 Canonical Signal Behavior in `REST_POLL` Mode

When stock data source is `REST_POLL`:

- No new entry signals are permitted
- Existing positions may be managed (stops/exits) only
- Confidence adjustment logic does not override this suppression rule

### 22.2 Slippage and Cost Realism

Execution simulation and live impact analytics must include:

- Time-of-day spread regimes
- Volatility-adjusted market impact
- Liquidity bucket by stock ADV and order size percentile
- Rejection/partial-fill penalties and retry latency

---

## 23. Model Governance and Deployment Controls

- Champion-challenger registry with explicit promotion criteria
- Shadow-mode evaluation before any production promotion
- Canary capital stage required before full-capital rollout
- Automatic rollback trigger matrix with severity levels
- Model card mandatory for each deployed version (data range, leakage audit result, known failure modes)

---

## 24. SLOs, Reconciliation, and Operational Readiness

### 24.1 Core SLOs

- Market data freshness SLO (e.g., latest tick age <= 5 seconds during normal market conditions)
- Signal decision latency SLO (feature-ready to decision <= configured threshold)
- Order acknowledgment SLO (submission to broker ack <= configured threshold)
- Service availability SLO during market hours

### 24.2 Reconciliation and Go/No-Go Gate

- Daily broker reconciliation must pass before next-day live activation
- Any critical reconciliation breach forces paper-only mode until resolved
- Market-open checklist includes static IP verification, NTP sync, data pipeline health, and broker session validity

---

> **Document Control:** Version 1.2. This product brief is a living document. All changes must be version-controlled, dated, and logged with the reason for change. The version current at the time of each model deployment is the authoritative reference for that deployment's intended behaviour.
