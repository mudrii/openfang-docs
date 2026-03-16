# Hands (Autonomous Capability Packages)

Hands are pre-built, opinionated autonomous capability packages that run on schedules 24/7 — no human prompting required. Each Hand includes a multi-phase operational system prompt (500+ words), a domain-specific SKILL.md, guardrails with approval gates for sensitive actions, and full configuration options.

All 8 Hands ship compiled into the binary — no external downloads required.

---

## The 8 Bundled Hands

### Clip — YouTube Content Repurposing

**Purpose:** Converts long-form YouTube videos into vertical short-form clips with AI-generated voice-over.

**Pipeline:**
1. Download source video (`yt-dlp`)
2. Transcribe audio (Whisper)
3. Identify high-value segments (LLM scene scoring)
4. Extract and reframe clips (`FFmpeg`)
5. Generate voice-over script (LLM)
6. Synthesize TTS audio
7. Overlay captions and branding
8. Queue for publishing (approval gate before upload)

**Key tools:** `yt-dlp`, `FFmpeg`, TTS provider, content platform APIs

---

### Lead — Prospect Discovery

**Purpose:** Runs daily to discover, enrich, and score potential leads based on your Ideal Customer Profile (ICP).

**Pipeline:**
1. Load ICP criteria from config
2. Search multiple sources (LinkedIn, web, company directories)
3. Enrich prospect data (company size, tech stack, funding)
4. Score against ICP (LLM-based scoring)
5. Deduplicate against existing leads
6. Export to CSV/JSON or CRM
7. Send summary report via configured channel

**Key tools:** `web_fetch`, `web_search`, file I/O, channel notification

---

### Collector — OSINT Intelligence Monitor

**Purpose:** Continuously monitors web sources for changes, builds a knowledge graph of entities and relationships, and alerts on significant developments.

**Pipeline:**
1. Crawl configured URLs on schedule
2. Detect content changes (diff-based)
3. Extract entities and relationships (LLM)
4. Update knowledge graph
5. Sentiment analysis on changes
6. Alert on high-significance events
7. Generate intelligence digest

**Key tools:** `web_fetch`, `memory_store`, `memory_recall`, entity extraction, channel notification

---

### Predictor — Superforecasting Engine

**Purpose:** Applies superforecasting methodology to track predictions, update probabilities as evidence arrives, and score forecast accuracy using Brier scores.

**Pipeline:**
1. Monitor configured news and data sources
2. Collect multi-signal evidence
3. Generate reasoning chains (LLM chain-of-thought)
4. Update probability estimates with confidence intervals
5. Track prediction resolution
6. Compute Brier scores for accuracy calibration
7. Report forecast updates

**Key tools:** `web_fetch`, `memory_store`, `memory_recall`, structured reasoning

---

### Researcher — Deep Research

**Purpose:** Conducts multi-source, multi-iteration deep research with academic-grade credibility evaluation.

**Methodology:**
- **CRAAP evaluation:** Currency, Relevance, Authority, Accuracy, Purpose
- **Cross-reference validation:** Multiple independent sources required
- **Citation format:** APA bibliography
- **Multi-language support:** Sources in any language, output in specified language
- **Iterative refinement:** Follow-up questions until depth threshold met

**Pipeline:**
1. Parse research question
2. Generate search queries (multiple angles)
3. Fetch and evaluate sources
4. CRAAP credibility scoring per source
5. Synthesize findings with citations
6. Identify gaps → additional research iterations
7. Generate final report with full bibliography

**Key tools:** `web_search`, `web_fetch`, `memory_store`, document generation

---

### Twitter — X/Twitter Account Manager

**Purpose:** Autonomously manages an X/Twitter account with content generation, scheduling, and audience engagement.

**7 Content Formats:**
1. Thread (numbered insights)
2. Hot take (contrarian opinion)
3. Question (community engagement)
4. Behind-the-scenes
5. Tips (numbered list)
6. Story (narrative arc)
7. Data/stat highlight

**Pipeline:**
1. Analyze account performance metrics
2. Select content format based on schedule
3. Generate content (LLM with persona)
4. Draft approval queue (mandatory human review before posting)
5. Post on schedule
6. Monitor replies and mentions
7. Engage with responses (LLM-generated replies)
8. Report weekly performance summary

**Key tools:** Twitter/X API, `web_fetch`, analytics, channel notification

**Guardrails:** All posts go through approval queue before publishing.

---

### Browser — Web Automation Agent

**Purpose:** Automates web browsing tasks using Playwright: form filling, data extraction, session persistence, and purchase workflows.

**Capabilities:**
- Page navigation and waiting for load states
- Form detection and filling
- Button clicking and interaction
- Screenshot capture and analysis
- Session/cookie persistence across runs
- JavaScript execution in page context
- Data extraction and structured output

**Guardrails:**
- **Purchase approval gate:** Any action involving payment or subscription requires explicit user confirmation
- Browser requires Chromium installation (auto-detected via `doctor`)

**Key tools:** Playwright bridge, `web_fetch`, file I/O, approval workflow

---

### Trading — Market Intelligence + Execution

**Purpose:** 8-phase autonomous trading pipeline combining market intelligence, adversarial analysis, risk management, and Alpaca API execution.

*Added in v0.3.45.*

**8-Phase Pipeline:**
1. **State recovery** — Load portfolio state and open positions
2. **Portfolio setup** — Verify account, buying power, position limits
3. **Market intelligence** — Multi-source data collection (price, volume, news, sentiment)
4. **Multi-factor analysis** — RSI, MACD, Bollinger Bands, VWAP, ATR; 22 candlestick patterns
5. **Adversarial debate** — Bull case vs Bear case (separate LLM sessions)
6. **Risk management gate** — Kelly criterion sizing, max drawdown check (mandatory approval gate)
7. **Alpaca API execution** — Market/limit orders, stop-loss, take-profit
8. **Analytics** — Trade journal, P&L tracking, strategy refinement

**Configuration (12 settings):**
- `max_position_size_pct` — Max % of portfolio per position
- `max_daily_loss_pct` — Daily loss circuit breaker
- `min_confidence_threshold` — Minimum signal confidence to trade
- `symbols` — Watchlist
- `schedule` — Trading schedule (cron)
- `risk_reward_min` — Minimum risk/reward ratio
- `paper_trading` — Use Alpaca paper trading mode
- And 5 more...

**Dashboard metrics (10):** P&L, win rate, Sharpe ratio, max drawdown, trade count, avg holding time, best trade, worst trade, current exposure, buying power remaining.

**Guardrails:** Risk management gate requires explicit approval before any real-money trades.

**Requires:** `ALPACA_API_KEY`, `ALPACA_SECRET_KEY`, market data API key.

---

## Activating Hands

Hands are shipped as agent templates in the `agents/` directory:

```bash
# Spawn a Hand as an agent
openfang agent spawn agents/clip/agent.toml --name my-clip-hand
openfang agent spawn agents/trading/agent.toml --name my-trader

# Configure the Hand
openfang config edit    # Set Hand-specific config in config.toml
```

### Schedule-Based Activation

Hands typically run on cron schedules:

```bash
# Create a schedule for the Researcher hand
POST /api/cron/jobs
{
  "agent_id": "<researcher-hand-id>",
  "name": "daily-research",
  "schedule": "0 9 * * *",
  "message": "Research today's top AI news and summarize findings"
}
```

### Event-Based Activation

```bash
# Activate Collector on system startup
POST /api/events/triggers
{
  "agent_id": "<collector-hand-id>",
  "pattern": {"lifecycle": {"event": "KernelStarted"}},
  "message": "Start monitoring configured sources"
}
```

---

## Hand State Persistence

Hand state is persisted in `hand_state.json` in the agent's directory. This ensures continuity across restarts:

- **Clip:** Queue of processed/pending clips
- **Lead:** Known leads database, dedup index
- **Collector:** Source change hashes, entity graph
- **Predictor:** Active forecasts, resolution tracker
- **Researcher:** Research history, citation database
- **Twitter:** Content calendar, performance metrics
- **Browser:** Session cookies, task history
- **Trading:** Portfolio state, open positions, trade journal

---

## Creating Custom Hands

A Hand is an agent with:
1. A detailed `agent.toml` with operational system prompt
2. Domain-specific SKILL.md file
3. Schedule or trigger configuration
4. Approval gates for sensitive actions

See [Contributing](contributing.md) for the full guide on creating new Hands.
