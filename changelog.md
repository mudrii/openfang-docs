# Changelog

All notable changes to OpenFang are documented here, following [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and [Semantic Versioning](https://semver.org/).

**Current stable release: v0.4.4** (2026-03-15)

---

## [0.4.4] — 2026-03-15

Automated release build.

---

## [0.4.3] — 2026-03-14

### Bug Fixes
- External links now open in new tabs in the web dashboard
- Homebrew install detection improved
- WhatsApp media handling: images, voice, video, documents, and stickers
- WhatsApp sender identity (phone + name) flows to system prompt

### Enhancements
- Added `free`, `openrouter/free`, `free-reasoning` model aliases for OpenRouter free models
- Config overwrite warning toast notification
- OpenSSL statically compiled (no runtime `libssl` dependency)
- Minimax switched to global endpoint for better reliability

---

## [0.4.2] — 2026-03-14

### Bug Fixes
- LoopGuard poll detection now purely keyword-based (less false positives)
- Custom model provider now shows `provider:model` format
- Telegram typing indicator refreshes every 4 seconds
- "No agent selected" error on API-created agents
- Tool call denial now includes actionable guidance
- WhatsApp sender metadata added to MessageRequest

### Enhancements
- **NVIDIA NIM provider** — 5 models (Llama 3.1 405B, Llama 3.3 70B, Mistral Large, and more)
- **YOLO mode** — `openfang start --yolo` or `auto_approve = true` in config: auto-approve all tool calls (development only)
- Skill output cards default to expanded state

---

## [0.4.1] — 2026-03-14

### Bug Fixes
- Memory recall loop fixed
- Raw LLM errors sanitized before sending to channel users
- `HAND.toml` accepts both flat and `[hand]` table formats
- Pre-emptive quota-aware compaction
- `log_level` in config.toml now takes effect
- Max iterations error includes config guidance
- `config.toml` backed up to `.bak` before any auto-rewrite

### Enhancements
- Spawn wizard reads default provider/model from `/api/status`

---

## [0.4.0] — 2026-03-12

Major release.

---

## [0.3.49] — 2026-03-12

Release build.

---

## [0.3.48] — 2026-03-12

### Bug Fixes
- Chromium detection for Browser hand
- Gemini 2.5 Flash empty responses (filtering out `thought:true` parts)
- Twitter hand text area not editable
- EOF parse error now marked as retryable
- 4 new tool-call recovery patterns
- WhatsApp auto-reconnect on disconnect

### Enhancements
- **Slack threading** — `thread_ts` support and `send_in_thread()` method
- **`OPENFANG_API_KEY` env var fallback** for configuration
- **Qwen Code CLI provider** — 3 models (`qwen-coder-plus`, `qwen-coder`, `qwen-long`)

*Stats: 2,103 tests*

---

## [0.3.47] — 2026-03-11

### Bug Fixes (11 community issues)
- Agent tool filtering — only declared tools sent to LLM (saves significant tokens)
- Streaming text leak fixed
- Telegram image MIME magic-byte detection
- Custom provider name preservation on reload
- Gemini 3.x `thoughtSignature` handling
- Telegram group `sender_chat` fallback
- Trigger reassignment after restart
- Hand re-activation tool profile
- Claude Code `skip_permissions` config field
- `channel_send` `thread_id` and `file_path` parameters
- Hand readiness API enriched

*Stats: 2,074 tests*

---

## [0.3.46] — 2026-03-11

### Bug Fixes (community PRs)
- IME composition guard for CJK input
- Default `User-Agent` header (fixes 403s from some sites)
- Token quota defaulted to unlimited (0) by default
- Safe string slicing + desktop shutdown race condition fix
- Embedding driver respects `provider_urls` config
- Wizard multi-line TOML escaping

### Enhancements
- **Z.AI Coding models:** `glm-5-coding`, `glm-4.7-coding`, `glm-4.7`
- **Kimi for Code** — `kimi-k2.5-0711`
- **XML-attribute tool call recovery** — Pattern 9 for malformed XML tool calls

*Stats: 187 models, 40+ providers, 2,040+ tests*

---

## [0.3.45] — 2026-03-10

### Added
- **Trading Hand (8th bundled Hand)** — Autonomous 8-phase trading pipeline:
  - State recovery, portfolio setup, market intelligence, multi-factor analysis
  - Adversarial bull/bear debate (two separate LLM sessions)
  - Risk management gate (mandatory approval before execution)
  - Alpaca API execution (market/limit orders, stop-loss, take-profit)
  - Trade analytics and journal
  - 12 configuration settings, 10 dashboard metrics
  - 470-line system prompt, 937-line SKILL.md with RSI/MACD/Bollinger/22 candlestick patterns, Kelly criterion

### Bug Fixes
- Streaming `<think>` tag leak fixed with `StreamingThinkFilter`
- Cron jobs orphaned after agent deletion fixed

*Stats: 2,031 tests, 8 bundled Hands, 184 models*

---

## [0.3.44] — 2026-03-10

### Bug Fixes
- Gemini `thought_signature` on function calls (new `provider_metadata` field)
- Auth error over-matching refined with API key redaction
- ClawHub rate limit backoff (5 attempts, respects `Retry-After` header)
- Telegram forum `message_thread_id` support

### Enhancements
- MiniMax M2.5-highspeed + abab7-chat models
- **Chutes.ai provider** — DeepSeek-V3, DeepSeek-R1, Llama-4, Qwen3-235B

*Stats: 1,994 tests*

---

## [0.3.43] — 2026-03-10

### Bug Fixes
- `web_fetch` gzip/brotli decompression (Yahoo Finance and others now work)
- UTF-8 CJK boundary panic fixed (9 affected sites)
- Gemini tool schema normalizer extended (strips `const`, `format`, flattens `oneOf`/type arrays)
- Local LLM empty response recovery (captures `reasoning_content`, extracts `<think>` tags)
- Voice messages concurrent dispatch (Semaphore(32))
- Custom model API keys persist on config reload

### Configuration
- `[budget] default_max_llm_tokens_per_hour` now configurable (0 = unlimited)

---

## [0.3.42] — 2026-03-10

### Bug Fixes
- Gemini tool schema fix (strips `$defs`, `$ref`, `additionalProperties`, inlines `$ref`)
- Temperature omitted for reasoning models (`gpt-5-mini`, `o1`, `o3`, `o4-mini`)
- Tool `input_schema` string/null handling

### Enhancements
- 6 OpenRouter free model variants added

---

## [0.3.40] — 2026-03-09

### Bug Fixes
- OpenAI tool schema fix (non-object `input_schema` normalized)
- Claude Code CLI env fix (`env_remove()` instead of `env_clear()`, removed `--dangerously-skip-permissions`)

---

## [0.3.38] — 2026-03-09

### Enhancements
- **Claude Code provider** — "Detect Claude Code" button in spawn wizard
- Credentials detection for both `~/.claude/.credentials.json` and `~/.claude/credentials.json`
- Cross-platform environment variable handling

*Stats: 1,949 tests*

---

## [0.3.37] — 2026-03-09

### Bug Fixes (Community Batch 3)
- Tool name alias normalization (fs-write → write_file, etc.)
- Provider keys reload without daemon restart
- Moonshot/kimi model IDs corrected
- Telegram lifecycle emoji reactions
- Telegram proxy `api_url` config option
- Discord `ignore_bots` configuration

### Enhancements
- Linux init crash: 7-browser fallback chain
- Text tool call fallback parser (25 new test cases)

*Stats: 1,948 tests*

---

## [0.3.36] — 2026-03-09

### Bug Fixes (Community Batch 2)
- Empty LLM response after ~4 conversation rounds (message pair re-validation)
- MCP tools bypass `ToolInvoke` filter (intentional — MCP tools always allowed)
- Telegram photos downloaded and base64-encoded for vision models
- Workflow visual builder SVG double-click fix
- Python 3 detection fix
- Hand state persisted in `hand_state.json`
- CLI auth on all commands

*Stats: 1,921 tests*

---

## [0.3.35] — 2026-03-09

### Bug Fixes (13 fixes)
- SSE `stream_options` for accurate token counts
- UTF-8 safe string operations
- `chrono-tz` timezone support in cron schedules
- TOML multiline strings in wizard
- Dashboard 401 interceptor
- Custom provider auto-detect `{PROVIDER}_API_KEY` pattern
- Cron stale agent fix (jobs survive agent restarts)
- Provider probing (concurrent, 60s cache)
- Model switch sync
- Embedding URL auto-append `/v1`
- ZHIPU non-null content fix
- Fish shell PATH detection
- 10 real OpenRouter models (replaced fake ones)

*Stats: 1,915 tests*

---

## [0.3.34] — 2026-03-09

### Bug Fixes
- Streaming empty text and 0 token counts fixed
- `stream_options: {include_usage: true}` for Groq/OpenAI/OpenRouter/DeepSeek
- Auto-fallback for providers that don't support `stream_options`

---

## [0.3.33] — 2026-03-08

### Bug Fixes
- Bluesky 400 error (invalid `seenAt` datetime format)
- OpenClaw migration nested map conversion
- Local LLM providers with empty `api_key_env`
- Gemini API key auto-switch on quota exhaustion
- Localhost dashboard accessible without API key

---

## [0.3.32] — 2026-03-08

### Bug Fixes
- CSP header blocking Alpine.js v3 on dashboard (regression from v0.3.30)

---

## [0.3.31] — 2026-03-08

### Critical Fixes
- `api_key` config migration (moved from root level)
- Dashboard unlock UX improvements
- MCP stdio JSON-RPC fix
- `doctor` health CLI fix
- `init` wizard non-TTY fallback
- Tool error guidance injection
- Budget defaults sentinel fix
- ClawHub retry backoff

### Enhancements
- Template spawn via API
- `/api/config/schema` made public (no auth required)

*Stats: 1,885 tests*

---

## [0.3.30] — 2026-03-08

### Security Hardening Release

- **Shell execution rewrite:** `shell_exec` now uses direct `execve()` — no shell interpreter, eliminating shell injection class entirely
- **Authentication hardening:** All mutating endpoints (POST/PUT/DELETE) now require auth; GET-only endpoints remain public
- **WebSocket constant-time comparison** for token validation
- **OFP per-message HMAC** — wire protocol messages individually authenticated
- **Audit trail persistence** — Merkle hash chain stored in SQLite
- Gemini API key fix
- CJK IME fix
- Discord `ignore_bots` configuration
- Browser Hand now requires Chromium (auto-detected)

*Stats: 1,886 tests*

---

## [0.1.0] — 2026-02-24

### Initial Release

#### Core Platform
- 15-crate Rust workspace (later refined to 14)
- Agent lifecycle: spawn, list, kill, clone, mode switching (Full/Assist/Observe)
- SQLite memory substrate: structured KV, semantic recall, vector embeddings
- 41 built-in tools (filesystem, web, shell, browser, scheduling, inter-agent, TTS, media, image analysis)
- WASM sandbox with dual metering (fuel + epoch + watchdog)
- Workflow engine: sequential, fan-out, collect, conditional, loop
- Visual workflow builder: SVG canvas, 7 node types, TOML export
- Trigger system: 9 event pattern types, fire limits
- Event bus with pub/sub and correlation IDs
- 7 bundled Hands: Clip, Lead, Collector, Predictor, Researcher, Twitter, Browser

#### LLM
- 3 native drivers: Anthropic, Google Gemini, OpenAI-compatible
- 27 providers, 130+ models
- Model catalog with aliases, tier classification
- Intelligent model routing
- Fallback driver for automatic failover
- Cost estimation and metering
- Streaming (SSE) across all drivers

#### Token Management
- Token-aware session compaction (70% threshold)
- Emergency trimming at 70%/90% with summary injection
- Tool profile filtering (41 tools → 4-10 for chat, saves 15-20K tokens)
- `MAX_TOOL_RESULT_CHARS` = 15K

#### Security
- 16-layer defense-in-depth architecture
- Capability-based access control
- WASM dual metering
- Merkle audit trail
- Information flow taint tracking
- Ed25519 signed manifests
- SSRF protection
- GCRA rate limiter
- Secret zeroization
- Loop guard
- Session repair
- Health endpoint redaction

#### Channels
- 40 channel adapters
- Unified bridge with agent routing, RBAC, message splitting

#### API
- 100+ REST/WS/SSE endpoints (Axum 0.8)
- OpenAI-compatible `/v1/chat/completions`
- Google A2A protocol
- Prometheus metrics
- Multi-session management

#### Desktop
- Tauri 2.0 native app (system tray, auto-update)

#### Tests
- 1,731+ tests at initial release

---

*Earlier pre-release versions (v0.3.24–v0.3.29) are rapid iteration builds from 2026-03-05 to 2026-03-08 without public release notes.*
