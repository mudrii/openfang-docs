# Hands

Validated against OpenFang release `v0.5.7`.

Hands are bundled autonomous capability packages managed by the `openfang-hands` crate. They are not the same thing as the `agents/` template catalog. Where agent templates are manifests you spawn and chat with directly, Hands are curated, domain-complete packages that work for you in the background. You check in on them rather than driving them interactively.

Release fact: `v0.5.7` bundles **9** Hands in `crates/openfang-hands/bundled`.

---

## How Hands differ from agent templates

| Surface | Backing source |
|---------|----------------|
| Agent templates | `agents/*/agent.toml` |
| Hands | `crates/openfang-hands/bundled/*/HAND.toml` |

Agent templates are manifests you spawn directly. Hands are curated packages with their own definitions, settings, requirement checks, lifecycle, and dashboard metrics. Each Hand ships with a `HAND.toml` manifest and a `SKILL.md` containing domain expertise that is injected into the agent's context at activation time.

---

## CLI commands

```bash
openfang hand list                        # List all registered hand definitions
openfang hand active                      # List running hand instances
openfang hand info <id>                   # Show details for a hand definition
openfang hand check-deps <id>             # Check requirement satisfaction
openfang hand install-deps <id>           # Install missing dependencies
openfang hand activate <id>               # Activate a hand (single instance)
openfang hand activate <id> --name <name> # Activate a named instance (multi-instance)
openfang hand pause <instance-id>         # Pause a running instance
openfang hand resume <instance-id>        # Resume a paused instance
openfang hand deactivate <instance-id>    # Deactivate and remove an instance
openfang hand status <id>                 # Show readiness and instance status
openfang hand install <path>              # Install a custom hand from a directory
```

---

## Hand lifecycle

Every Hand instance moves through these states:

```
Inactive --> Active --> Paused --> Active
                  \               /
                   --> Error ----/
                   --> Deactivated (removed)
```

- **Inactive** -- the definition is registered but no instance is running.
- **Active** -- an agent has been spawned and is running autonomously.
- **Paused** -- the instance exists but the agent is suspended. Resume to return to Active.
- **Error** -- the instance hit a runtime error. The error message is stored on the instance.
- **Deactivated** -- the instance is removed from the registry entirely.

Lifecycle operations:

| Operation | CLI | API |
|-----------|-----|-----|
| Activate | `openfang hand activate <id>` | `POST /api/hands/<id>/activate` |
| Pause | `openfang hand pause <instance-id>` | `POST /api/hands/instances/<id>/pause` |
| Resume | `openfang hand resume <instance-id>` | `POST /api/hands/instances/<id>/resume` |
| Deactivate | `openfang hand deactivate <instance-id>` | `DELETE /api/hands/instances/<id>` |
| Status | `openfang hand status <id>` | `GET /api/hands/<id>/readiness` |

---

## Multi-instance activation (v0.5.7)

By default, activating a hand that is already active is rejected. The `v0.5.7` release adds support for running multiple instances of the same hand simultaneously, as long as each instance has a distinct `instance_name`.

**CLI:**

```bash
openfang hand activate clip --name clip-youtube
openfang hand activate clip --name clip-podcast
```

**API:**

```bash
curl -X POST http://127.0.0.1:4200/api/hands/clip/activate \
  -H "Content-Type: application/json" \
  -d '{"instance_name": "clip-youtube", "config": {}}'
```

Rules enforced by the registry:

- Each `(hand_id, instance_name)` pair must be unique among active instances.
- Unnamed activations follow the legacy single-instance-per-hand rule -- a second unnamed activation of the same hand is rejected.
- Named and unnamed instances of the same hand can coexist.

---

## Custom hand installation

You can install hands from a local directory containing a `HAND.toml` (and optional `SKILL.md`):

```bash
openfang hand install /path/to/my-hand/
```

The daemon copies the hand directory to `~/.openfang/hands/<hand_id>/` so that custom hands persist across daemon restarts. On startup, the registry calls `load_workspace_hands` to re-register all hands found under `~/.openfang/hands/`.

If the source path is already the persistence directory (e.g. `~/.openfang/hands/foo`), the copy step is skipped.

---

## Bundled Hand catalog

### browser

| Field | Value |
|-------|-------|
| **ID** | `browser` |
| **Name** | Browser Hand |
| **Description** | Autonomous web browser -- navigates sites, fills forms, clicks buttons, and completes multi-step web tasks with user approval for purchases |
| **Category** | Productivity |
| **Icon** | globe |

**Tools:** `browser_navigate`, `browser_click`, `browser_type`, `browser_screenshot`, `browser_read_page`, `browser_close`, `web_search`, `web_fetch`, `memory_store`, `memory_recall`, `knowledge_add_entity`, `knowledge_add_relation`, `knowledge_query`, `schedule_create`, `schedule_list`, `schedule_delete`, `file_write`, `file_read`

**Requirements:**

| Key | Label | Type | Required | Install (macOS) |
|-----|-------|------|----------|-----------------|
| `python3` | Python 3 must be installed | binary | yes | `brew install python3` |
| `chromium` | Chromium or Google Chrome | binary | **optional** | `brew install --cask google-chrome` |

Python 3 is required for the Playwright browser automation library. Chromium is optional because Playwright can install its own bundled browser. The requirement checker looks for `python3`, `python`, `chromium`, `chromium-browser`, `google-chrome`, and `google-chrome-stable` on PATH, plus well-known install paths and the Playwright cache.

**Settings:**

| Key | Label | Type | Default |
|-----|-------|------|---------|
| `headless` | Headless Mode | toggle | `true` |
| `approval_mode` | Purchase Approval | toggle | `true` |
| `max_pages_per_task` | Max Pages Per Task | select | `20` (options: 10, 20, 50) |
| `default_wait` | Default Wait After Action | select | `auto` (options: auto, 1s, 3s) |
| `screenshot_on_action` | Screenshot After Actions | toggle | `false` |

**Dashboard metrics:** Pages Visited, Tasks Completed, Screenshots

---

### clip

| Field | Value |
|-------|-------|
| **ID** | `clip` |
| **Name** | Clip Hand |
| **Description** | Turns long-form video into viral short clips with captions and thumbnails |
| **Category** | Content |
| **Icon** | clapper |

**Tools:** `shell_exec`, `file_read`, `file_write`, `file_list`, `web_fetch`, `memory_store`, `memory_recall`

**Requirements:**

| Key | Label | Type | Required | Install (macOS) |
|-----|-------|------|----------|-----------------|
| `ffmpeg` | FFmpeg must be installed | binary | yes | `brew install ffmpeg` |
| `ffprobe` | FFprobe must be installed | binary | yes | `brew install ffmpeg` |
| `yt-dlp` | yt-dlp must be installed | binary | yes | `brew install yt-dlp` |

**Settings:**

| Key | Label | Type | Default |
|-----|-------|------|---------|
| `stt_provider` | Speech-to-Text Provider | select | `auto` (options: auto, whisper_local, groq_whisper, openai_whisper, deepgram) |
| `tts_provider` | Text-to-Speech Provider | select | `none` (options: none, edge_tts, openai_tts, elevenlabs) |
| `elevenlabs_api_key` | ElevenLabs API Key | text | (empty, env_var: `ELEVENLABS_API_KEY`) |
| `publish_target` | Publish Clips To | select | `local_only` (options: local_only, telegram, whatsapp, both) |
| `telegram_bot_token` | Telegram Bot Token | text | (empty) |
| `telegram_chat_id` | Telegram Chat ID | text | (empty) |
| `whatsapp_token` | WhatsApp Access Token | text | (empty) |
| `whatsapp_phone_id` | WhatsApp Phone Number ID | text | (empty) |
| `whatsapp_recipient` | WhatsApp Recipient | text | (empty) |

**Dashboard metrics:** Jobs Completed, Clips Generated, Total Duration, Published to Telegram, Published to WhatsApp

---

### collector

| Field | Value |
|-------|-------|
| **ID** | `collector` |
| **Name** | Collector Hand |
| **Description** | Autonomous intelligence collector -- monitors any target continuously with change detection and knowledge graphs |
| **Category** | Data |
| **Icon** | magnifying glass |

**Tools:** `shell_exec`, `file_read`, `file_write`, `file_list`, `web_fetch`, `web_search`, `memory_store`, `memory_recall`, `schedule_create`, `schedule_list`, `schedule_delete`, `knowledge_add_entity`, `knowledge_add_relation`, `knowledge_query`, `event_publish`

**Requirements:** None

**Settings:**

| Key | Label | Type | Default |
|-----|-------|------|---------|
| `target_subject` | Target Subject | text | (empty) |
| `collection_depth` | Collection Depth | select | `deep` (options: surface, deep, exhaustive) |
| `update_frequency` | Update Frequency | select | `daily` (options: hourly, every_6h, daily, weekly) |
| `focus_area` | Focus Area | select | `general` (options: market, business, competitor, person, technology, general) |
| `alert_on_changes` | Alert on Changes | toggle | `true` |
| `report_format` | Report Format | select | `markdown` (options: markdown, json, html) |
| `max_sources_per_cycle` | Max Sources Per Cycle | select | `30` (options: 10, 30, 50, 100) |
| `track_sentiment` | Track Sentiment | toggle | `false` |

**Dashboard metrics:** Data Points, Entities Tracked, Reports Generated, Last Update

---

### infisical-sync

| Field | Value |
|-------|-------|
| **ID** | `infisical-sync` |
| **Name** | Infisical Sync Hand |
| **Description** | Autonomous secrets synchronisation between a self-hosted Infisical instance and the agent's local credential vault |
| **Category** | Security |
| **Icon** | lock |

**Tools:** `schedule_create`, `schedule_list`, `schedule_delete`, `memory_store`, `memory_recall`, `knowledge_add_entity`, `knowledge_add_relation`, `knowledge_query`, `event_publish`, `shell_exec`, `file_read`, `file_write`, `vault_set`, `vault_get`, `vault_list`, `vault_delete`

**Requirements:**

| Key | Label | Type | Required |
|-----|-------|------|----------|
| `INFISICAL_URL` | Infisical Instance URL | env_var | yes |
| `INFISICAL_CLIENT_ID` | Machine Identity Client ID | env_var | yes |
| `INFISICAL_CLIENT_SECRET` | Machine Identity Client Secret | env_var | yes |

**Settings:**

| Key | Label | Type | Default |
|-----|-------|------|---------|
| `sync_interval_minutes` | Sync Interval | select | `15` (options: 5, 15, 30, 60) |
| `environment` | Infisical Environment | select | `prod` (options: prod, staging, dev) |
| `push_on_vault_write` | Push on Vault Write | toggle | `true` |
| `delete_orphans` | Delete Orphaned Local Secrets | toggle | `false` |

**Dashboard metrics:** Secrets in Vault, Last Sync, Last Error, Projects Synced, Secrets Pushed, Secrets Pulled

---

### lead

| Field | Value |
|-------|-------|
| **ID** | `lead` |
| **Name** | Lead Hand |
| **Description** | Autonomous lead generation -- discovers, enriches, and delivers qualified leads on a schedule |
| **Category** | Data |
| **Icon** | chart |

**Tools:** `shell_exec`, `file_read`, `file_write`, `file_list`, `web_fetch`, `web_search`, `memory_store`, `memory_recall`, `schedule_create`, `schedule_list`, `schedule_delete`, `knowledge_add_entity`, `knowledge_add_relation`, `knowledge_query`

**Requirements:** None

**Settings:**

| Key | Label | Type | Default |
|-----|-------|------|---------|
| `target_industry` | Target Industry | text | (empty) |
| `target_role` | Target Role | text | (empty) |
| `company_size` | Company Size | select | `any` (options: any, startup, smb, enterprise) |
| `lead_source` | Lead Source | select | `web_search` (options: web_search, linkedin_public, crunchbase, custom) |
| `output_format` | Output Format | select | `csv` (options: csv, json, markdown_table) |
| `leads_per_report` | Leads Per Report | select | `25` (options: 10, 25, 50, 100) |
| `delivery_schedule` | Delivery Schedule | select | `daily_9am` (options: daily_7am, daily_9am, weekdays_8am, weekly_monday) |
| `geo_focus` | Geographic Focus | text | (empty) |
| `enrichment_depth` | Enrichment Depth | select | `standard` (options: basic, standard, deep) |

**Dashboard metrics:** Leads Found, Reports Generated, Last Report, Unique Companies

---

### predictor

| Field | Value |
|-------|-------|
| **ID** | `predictor` |
| **Name** | Predictor Hand |
| **Description** | Autonomous future predictor -- collects signals, builds reasoning chains, makes calibrated predictions, and tracks accuracy |
| **Category** | Data |
| **Icon** | crystal ball |

**Tools:** `shell_exec`, `file_read`, `file_write`, `file_list`, `web_fetch`, `web_search`, `memory_store`, `memory_recall`, `schedule_create`, `schedule_list`, `schedule_delete`, `knowledge_add_entity`, `knowledge_add_relation`, `knowledge_query`

**Requirements:** None

**Settings:**

| Key | Label | Type | Default |
|-----|-------|------|---------|
| `prediction_domain` | Prediction Domain | select | `tech` (options: tech, finance, geopolitics, climate, general) |
| `time_horizon` | Time Horizon | select | `3_months` (options: 1_week, 1_month, 3_months, 1_year) |
| `data_sources` | Data Sources | select | `all` (options: news, social, financial, academic, all) |
| `report_frequency` | Report Frequency | select | `weekly` (options: daily, weekly, biweekly, monthly) |
| `predictions_per_report` | Predictions Per Report | select | `5` (options: 3, 5, 10, 20) |
| `track_accuracy` | Track Accuracy | toggle | `true` |
| `confidence_threshold` | Confidence Threshold | select | `medium` (options: low, medium, high) |
| `contrarian_mode` | Contrarian Mode | toggle | `false` |

**Dashboard metrics:** Predictions Made, Accuracy, Reports Generated, Active Predictions

---

### researcher

| Field | Value |
|-------|-------|
| **ID** | `researcher` |
| **Name** | Researcher Hand |
| **Description** | Autonomous deep researcher -- exhaustive investigation, cross-referencing, fact-checking, and structured reports |
| **Category** | Productivity |
| **Icon** | test tube |

**Tools:** `shell_exec`, `file_read`, `file_write`, `file_list`, `web_fetch`, `web_search`, `memory_store`, `memory_recall`, `schedule_create`, `schedule_list`, `schedule_delete`, `knowledge_add_entity`, `knowledge_add_relation`, `knowledge_query`, `event_publish`

**Requirements:** None

**Settings:**

| Key | Label | Type | Default |
|-----|-------|------|---------|
| `research_depth` | Research Depth | select | `thorough` (options: quick, thorough, exhaustive) |
| `output_style` | Output Style | select | `detailed` (options: brief, detailed, academic, executive) |
| `source_verification` | Source Verification | toggle | `true` |
| `max_sources` | Max Sources | select | `30` (options: 10, 30, 50, unlimited) |
| `auto_follow_up` | Auto Follow-Up | toggle | `true` |
| `save_research_log` | Save Research Log | toggle | `false` |
| `citation_style` | Citation Style | select | `inline_url` (options: inline_url, footnotes, academic_apa, numbered) |
| `language` | Language | select | `english` (options: english, spanish, french, german, chinese, japanese, auto) |

**Agent note:** The researcher hand sets `heartbeat_interval_secs = 120` (overriding the kernel default of 30s) to avoid false-positive recovery triggers during long LLM calls.

**Dashboard metrics:** Queries Solved, Sources Cited, Reports Generated, Active Investigations

---

### trader

| Field | Value |
|-------|-------|
| **ID** | `trader` |
| **Name** | Trading Hand |
| **Description** | Autonomous market intelligence and trading engine -- multi-signal analysis, adversarial bull/bear reasoning, calibrated confidence scoring, strict risk management, and portfolio-level analytics |
| **Category** | Data |
| **Icon** | chart with upward trend |

**Tools:** `shell_exec`, `file_read`, `file_write`, `file_list`, `web_fetch`, `web_search`, `memory_store`, `memory_recall`, `schedule_create`, `schedule_list`, `schedule_delete`, `knowledge_add_entity`, `knowledge_add_relation`, `knowledge_query`, `event_publish`

**Requirements:** None (Alpaca API keys are optional, needed only for live trading mode)

**Settings:**

| Key | Label | Type | Default |
|-----|-------|------|---------|
| `trading_mode` | Trading Mode | select | `paper` (options: analysis, paper, live) |
| `market_focus` | Market Focus | select | `us_stocks` (options: us_stocks, crypto, multi_asset) |
| `strategy_style` | Strategy Style | select | `swing` (options: scalping, day, swing, position) |
| `risk_per_trade` | Risk Per Trade | select | `2` (options: 1%, 2%, 3%, 5%) |
| `max_daily_loss` | Max Daily Loss | select | `5` (options: 2%, 5%, 10%) |
| `analysis_depth` | Analysis Depth | select | `standard` (options: quick, standard, deep) |
| `scan_schedule` | Scan Schedule | select | `4h` (options: 15m, 1h, 4h, daily) |
| `watchlist` | Watchlist | text | `SPY,QQQ,AAPL,MSFT,NVDA,BTC,ETH` |
| `initial_capital` | Initial Capital | text | `10000` |
| `alpaca_api_key` | Alpaca API Key | text | (empty, env_var: `ALPACA_API_KEY`) |
| `alpaca_secret_key` | Alpaca Secret Key | text | (empty, env_var: `ALPACA_SECRET_KEY`) |
| `approval_mode` | Approval Mode | toggle | `true` |

The trader hand includes a circuit breaker system: daily loss exceeding `max_daily_loss` halts trading for 24 hours, 3 consecutive losses trigger a cooldown, and drawdown over 25% from peak closes all positions and switches to analysis-only mode.

**Dashboard metrics:** Portfolio Value, Total P&L, Win Rate, Sharpe Ratio, Max Drawdown, Trades Executed, Active Positions, Signals Analyzed, Accuracy, Last Scan

---

### twitter

| Field | Value |
|-------|-------|
| **ID** | `twitter` |
| **Name** | Twitter Hand |
| **Description** | Autonomous Twitter/X manager -- content creation, scheduled posting, engagement, and performance tracking |
| **Category** | Communication |
| **Icon** | X |

**Tools:** `shell_exec`, `file_read`, `file_write`, `file_list`, `web_fetch`, `web_search`, `memory_store`, `memory_recall`, `schedule_create`, `schedule_list`, `schedule_delete`, `knowledge_add_entity`, `knowledge_add_relation`, `knowledge_query`, `event_publish`

**Requirements:**

| Key | Label | Type | Required |
|-----|-------|------|----------|
| `TWITTER_BEARER_TOKEN` | Twitter API Bearer Token | api_key | yes |

Setup: Go to developer.twitter.com, create a Project and App, generate a Bearer Token, and set it as an environment variable.

**Settings:**

| Key | Label | Type | Default |
|-----|-------|------|---------|
| `twitter_bearer_token` | Twitter Bearer Token | text | (empty) |
| `twitter_style` | Content Style | select | `professional` (options: professional, casual, witty, educational, provocative, inspirational) |
| `post_frequency` | Post Frequency | select | `3_daily` (options: 1_daily, 3_daily, 5_daily, hourly) |
| `auto_reply` | Auto Reply | toggle | `false` |
| `auto_like` | Auto Like | toggle | `false` |
| `content_topics` | Content Topics | text | (empty) |
| `brand_voice` | Brand Voice | text | (empty) |
| `thread_mode` | Thread Mode | toggle | `true` |
| `content_queue_size` | Content Queue Size | select | `10` (options: 5, 10, 20, 50) |
| `engagement_hours` | Engagement Hours | select | `business_hours` (options: business_hours, waking_hours, all_day) |
| `approval_mode` | Approval Mode | toggle | `true` |

When `approval_mode` is enabled (the default), tweets are written to a queue file for review rather than posted directly.

**Dashboard metrics:** Tweets Posted, Replies Sent, Queue Size, Engagement Rate

---

## Readiness model

The registry computes a `HandReadiness` status for each hand definition by combining requirement checks with runtime instance state:

| Field | Meaning |
|-------|---------|
| `requirements_met` | All non-optional requirements are satisfied |
| `active` | At least one instance is in Active status |
| `degraded` | Active but one or more requirements (including optional) are unmet |

Example: the browser hand has python3 (required) and chromium (optional). If python3 is present but chromium is not, `requirements_met` is true but `degraded` is true when the hand is active.

---

## Packaging model

Each bundled Hand ships with:

- `HAND.toml` -- the manifest defining id, name, description, category, tools, requirements, settings, agent config, and dashboard metrics
- `SKILL.md` -- domain expertise document injected into the spawned agent's context at activation time
- Settings metadata with per-option availability checks (env var presence, binary existence)
- Dashboard metrics schema mapping memory keys to display labels and formats
- Requirement metadata with platform-specific install instructions

The bundled registry test in `crates/openfang-hands/src/bundled.rs` asserts `hands.len() == 9`.

---

## Notes

- Upstream README prose still says `7` Hands in some places; that is stale for `v0.5.7`.
- This docs repo uses the bundled-hand registry, not the README, as the source of truth.
- State persistence for active hands is written to disk so hands survive daemon restarts.
- The `HandCategory` enum supports: Content, Security, Productivity, Development, Communication, Data, Finance, and Other.
