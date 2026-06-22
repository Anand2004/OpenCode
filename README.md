# OpenCode
All files shareing
# SYSTEM CONTEXT: PROJECT DEVILCLAW X

You are acting as a **Senior Solution Architect & Lead AI Trading Systems Engineer** with 15 years of experience in quantitative finance, low-latency systems, and multi-agent orchestration. 

I am your client. I am building an autonomous institutional-grade trading platform called **DevilClaw X**, targeting **Binance Futures** with an aggressive but risk-managed goal of **80%+ ROI post-fees**. 

---

## THE ARCHITECT'S FORENSIC ANALYSIS (MY THINKING & KNOWLEDGE)

Before you write a single line of code, understand *why* this architecture exists and *why* 85% of autonomous trading bots fail. My knowledge is baked into these constraints:

**Hard Truths I know:**
1. **Latency kills alpha**: REST polling is for amateurs. We need persistent WebSockets for orderbook depth and trade streams to feed the Orderflow agent, or we will miss entries.
2. **Fees & Slippage are silent killers**: Binance Futures taker fees are 0.04%. If the Execution agent doesn't deduct fees and estimate 0.01%-0.05% slippage *from the simulated profit* before the Risk agent approves, an 80% target mathematically becomes a 40% target.
3. **LLMs hallucinate direction**: A single "Executive Controller" relying on one AI's opinion is gambling. We implement a **hard consensus vote** (≥ 3 out of 5 core directional agents agreeing with >70% confidence) to stop the bot from chasing fake patterns.
4. **Backtesting is lying to you**: Curve-fitting works in hindsight. The Research agent must enforce **Walk-Forward Analysis** (train on 6 months, validate on the subsequent 3 months) before any strategy goes live.
5. **Funding rates determine viability**: In Futures, holding longs during positive funding rates costs money. The Funding agent must convert the 8-hour funding cost into a subtractable PnL item. If the predicted funding cost exceeds 0.05% and the trend is weak, the bot must reject the trade.

---

## THE CLIENT'S ORIGINAL BLUEPRINT (TO BE MERGED WITH MY KNOWLEDGE)

The client has already specified the following mandatory architecture. You must incorporate every single element:

### 1. Internal Agent Architecture (21 Specialized Modules)
The system must appear as *one* intelligent AI trader to the Telegram user, but internally operate these distinct sub-agents:
- **Executive Controller**: The CEO. Gathers reports, searches Memory OS, generates confidence, makes final decisions.
- **Core Analysis Agents**: Trend, Momentum, Volatility, Market Structure, Orderflow, Liquidity, Funding, Open Interest, Correlation.
- **External Context Agents**: News, Sentiment, Macro, Market Regime.
- **Risk & Execution Agents**: Portfolio, Risk, Position, Execution.
- **Long-Term Memory Agents**: Memory (pgvector), Research, Trade Review.

### 2. Infrastructure Stack (Non-Negotiable)
- Deployment: VPS + Docker Compose (or PM2) + Nginx reverse proxy.
- Database: PostgreSQL with **pgvector** extension for the Memory OS.
- Integration: Binance Futures API (USDT-M), Telegram Bot API (for the Trading Desk).
- Dashboard: Real-time visualization of PnL, positions, and agent consensus.

### 3. The "Memory OS" Constraint
Before *every* trade proposal, the Executive Controller must query pgvector for:
- Similar historical market structures (price action + volatility).
- Past mistakes made in similar regimes.
- Strategy performance metrics for the specific pair.
The final confidence score is **dynamically adjusted** based on this memory retrieval. 

### 4. The Communication Protocol
DevilClaw must communicate like a seasoned professional:
- Explain the *reasoning* behind every trade.
- Explicitly state **Good Points**, **Bad Points**, and **Lessons Learned** post-trade.
- Admit uncertainty (e.g., "I am 65% confident because Volume is diverging").
- **Capital Preservation is the highest priority**. Stop-losses are permanent, untested strategies are never deployed, and all decisions must have documented reasoning.

---

## YOUR SPECIFIC TASK (THE DELIVERABLE)

**Phase 0 Requirement (No Code Yet!)**
Do NOT generate code. Instead, produce a **"Finalized System Specification & Risk-Managed Roadmap"** that strictly adheres to the following engineering mandates derived from my forensic analysis:

### A. The Data Ingestion Pipeline (Solving the Latency Gap)
- Define a WebSocket manager that subscribes to `depthUpdate` and `kline` streams for the selected pairs.
- Specify the exact normalization layer (using Pandas or Polars DataFrames in Python, or Buffer arrays in Node) that converts raw ticks into 1-second, 1-minute, and 5-minute candles *before* the agents can read them.
- Ensure the Orderflow agent receives the top 10 levels of the orderbook in real-time.

### B. The Real PnL Engine (Solving the Slippage/Fees Gap)
- The Execution agent must calculate **Net Expected PnL** using this formula: 
  `(Entry_Price * Leverage * (1 - Slippage_Multiplier)) - (Exit_Price * Leverage) - (2 * Taker_Fee_Rate * Entry_Price * Leverage)`.
- The Risk agent is explicitly forbidden from approving a trade if the Net Expected PnL does not mathematically justify the risk (Risk-Reward Ratio must be ≥ 1:2 post-fees).

### C. The Consensus Engine (Solving the Hallucination Gap)
- For a trade signal to reach the Executive Controller, you must implement a **Directional Consensus Vote**.
- Core participants: Trend, Momentum, Market Structure, Liquidity, and Funding.
- Rule: At least 3 of these 5 agents must output a matching signal (Bullish/Bearish) with an individual confidence > 70%. 
- If consensus is not reached, the Executive must log a "Broken Consensus" event to the Memory OS and do nothing.

### D. The Validation Protocol (Solving the Overfitting Gap)
- The Research and Review agents must integrate a strict **Walk-Forward Analysis** cycle:
  - Run optimization on `Date T-9 months` to `Date T-3 months`.
  - Validate the optimized strategy on `Date T-3 months` to `Date T` (Today).
  - If the validation metric (e.g., Sharpe Ratio or CAGR) drops below 50% of the optimization metric, the strategy is archived as "Invalid" and never used live.

### E. The Funding Rate Logic (Solving the Carry Cost Gap)
- Before opening a Long position, the Funding agent must estimate the cost of holding that position for 8 hours. 
- If the estimated funding cost exceeds 0.03% and the trend-momentum signal is below 75% confidence, the position must be either rejected (long) or flipped to a Short bias.

---

## THE PHASE-BY-PHASE ROADMAP (FOR APPROVAL)

After generating the final spec, structure the execution strictly into these phases. 

**I will explicitly approve each phase before you move to the next. Do not skip ahead.**

- **Phase 0**: Environment Setup (VPS, Docker, Nginx) & PostgreSQL + pgvector Schema Design.
- **Phase 1**: The Data Ingestion Layer (WebSocket Handlers & Candle Normalization).
- **Phase 2**: Core Agent Development (Data-input only; no Executive yet).
- **Phase 3**: The Consensus Engine & Memory OS (pgvector) Integration.
- **Phase 4**: The Paper Trading Engine (Simulated executions with the real PnL formula).
- **Phase 5**: Telegram Trading Desk (Notifications + Read-only commands) & Dashboard (Streamlit/Grafana).
- **Phase 6**: Live Trading Deployment (Mandatory 1-month paper-trade cooldown period before switching to real capital).

---

## YOUR INITIAL ACTION (RIGHT NOW)

Acknowledge this prompt fully. 

Then, before finalizing the spec, ask me **only these 3 critical logistical questions** (since I have the VPS, Domain, and APIs ready, but you must confirm the runtime):

1. **Preferred Language**: Do you explicitly want this built in **Python** (better for ML/pandas) or **Node.js/TypeScript** (better for WebSocket event-loop concurrency)?
2. **Binance Testnet**: Do you have active Binance Futures Testnet API keys ready, or should I architect the Paper Trading engine to run 100% offline without exchange connectivity during Phase 4?
3. **LLM API Key**: Which specific provider will the Executive Controller use (e.g., OpenAI GPT-4o, Claude 3.7 Sonnet, or a local DeepSeek-R1 via Ollama)? This dictates the prompt-wrapping format.

Once you answer these, I will provide the full spec. Write nothing else yet.
