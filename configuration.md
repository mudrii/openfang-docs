# Configuration

OpenFang is configured via `~/.openfang/config.toml`. All fields support `#[serde(default)]` for forward compatibility -- unknown fields are silently ignored.

Config hot-reloads automatically (mode configurable via `[reload]`). Force a reload: `POST /api/config/reload`.

---

## Minimal Configuration

```toml
# ~/.openfang/config.toml

[default_model]
provider = "groq"
model = "llama-3.3-70b-versatile"
api_key_env = "GROQ_API_KEY"
```

---

## Full Configuration Reference

### Top-Level Fields

```toml
api_key = ""                                   # Bearer token for API auth (empty = loopback bypass)
api_listen = "127.0.0.1:4200"                  # HTTP API bind address
log_level = "info"                             # trace | debug | info | warn | error
usage_footer = "full"                          # off | tokens | cost | full
mode = "default"                               # stable | default | dev
language = "en"                                # Locale for CLI and messages
network_enabled = false                        # Enable OFP peer protocol
max_cron_jobs = 500                            # Max cron jobs across all agents
```

### `[default_model]`

Primary LLM provider and model.

```toml
[default_model]
provider = "anthropic"                         # Provider ID (anthropic, openai, groq, gemini, ollama, etc.)
model = "claude-sonnet-4-20250514"             # Model identifier
api_key_env = "ANTHROPIC_API_KEY"              # Env var holding API key
# base_url = "https://api.anthropic.com"       # Optional: override API endpoint
# system_prompt = "You are a helpful assistant."
# temperature = 0.7                            # 0.0-2.0 (null = provider default)
# max_tokens = 4096                            # Max output tokens
```

### `[memory]`

Memory substrate configuration.

```toml
[memory]
decay_rate = 0.05                              # Memory confidence decay rate per day
# sqlite_path = "~/.openfang/data/openfang.db" # Custom DB path
# embedding_enabled = true                     # Enable vector embeddings
# embedding_provider = "openai"                # Embedding provider
# embedding_model = "text-embedding-3-small"
# context_window_chars = 100000                # Characters before compaction
```

### `[network]`

OFP (OpenFang Wire Protocol) peer network settings.

```toml
[network]
listen_addr = "127.0.0.1:4200"                # OFP listen address
# shared_secret = ""                           # Required for P2P authentication (HMAC-SHA256)
```

### `[compaction]`

LLM-based session context compaction.

```toml
[compaction]
threshold = 80                                 # Compact when messages exceed this count
keep_recent = 20                               # Keep this many recent messages after compaction
max_summary_tokens = 1024                      # Max tokens for LLM summary
```

### `[web]`

Web tools configuration (search + fetch).

```toml
[web]
search_provider = "auto"                       # auto | brave | tavily | perplexity | searxng | duck_duck_go
cache_ttl_minutes = 15                         # Cache TTL (0 = disabled)
```

#### `[web.brave]`

```toml
[web.brave]
api_key_env = "BRAVE_API_KEY"
max_results = 5
# country = "US"
# search_lang = "en"
# freshness = "pw"                             # pd = past day, pw = past week
```

#### `[web.tavily]`

```toml
[web.tavily]
api_key_env = "TAVILY_API_KEY"
search_depth = "basic"                         # basic | advanced
max_results = 5
include_answer = true
```

#### `[web.perplexity]`

```toml
[web.perplexity]
api_key_env = "PERPLEXITY_API_KEY"
model = "sonar"
```

#### `[web.searxng]`

SearXNG self-hosted search aggregator. No API key required.

```toml
[web.searxng]
url = "https://searxng.example.com"            # SearXNG instance URL
```

*Added in v0.5.4.*

#### `[web.fetch]`

Web fetch configuration with SSRF protection.

```toml
[web.fetch]
max_chars = 50000                              # Max content chars returned
max_response_bytes = 10485760                  # Max response body (10 MB)
timeout_secs = 30                              # HTTP request timeout
readability = true                             # Enable HTML-to-Markdown extraction
ssrf_allowed_hosts = []                        # SSRF allowlist (see Security docs)
```

`ssrf_allowed_hosts` accepts exact hostnames (`"n8n.local"`), wildcard domains (`"*.olares.com"`), or CIDR ranges (`"10.0.0.0/8"`). Allowlisted hosts bypass the private-IP check but never bypass cloud metadata endpoint blocking (169.254.169.254, etc.).

### `[auth]`

Dashboard login authentication. Disabled by default.

```toml
[auth]
enabled = true
username = "admin"
password_hash = "$argon2id$v=19$m=19456,t=2,p=1$..."  # openfang auth hash-password
session_ttl_hours = 168                                # 7 days
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enabled` | bool | `false` | Enable username/password auth for the dashboard |
| `username` | string | `"admin"` | Admin username |
| `password_hash` | string | `""` | Argon2id hash in PHC format. Generate with `openfang auth hash-password`. |
| `session_ttl_hours` | u64 | `168` | Session token lifetime in hours |

**Breaking change in v0.5.7:** Password hashes must be Argon2id. Older SHA256 hex hashes are no longer accepted.

### `[approval]`

Execution approval policy. Controls which tools require human confirmation before running.

```toml
[approval]
# Configure via agent.toml or the approval API
```

See `crates/openfang-types/src/approval.rs` for the `ApprovalPolicy` struct.

### `[budget]`

Global spending budget limits.

```toml
[budget]
max_hourly_usd = 0.0                          # 0.0 = unlimited
max_daily_usd = 0.0                            # 0.0 = unlimited
max_monthly_usd = 0.0                          # 0.0 = unlimited
alert_threshold = 0.8                          # Trigger warnings at 80% of any limit
default_max_llm_tokens_per_hour = 0            # 0 = unlimited (rolling 1-hour window)
```

### `[exec_policy]`

Shell/exec security policy for the `shell_exec` and `process_start` tools.

```toml
[exec_policy]
mode = "allowlist"                             # deny | allowlist | full
timeout_secs = 30                              # Max execution time
max_output_bytes = 102400                      # Max stdout/stderr (100 KB)
no_output_timeout_secs = 30                    # Kill process after N seconds of no output
allowed_commands = ["cargo", "git", "npm"]     # Additional allowed commands
# safe_bins includes by default: sleep, true, false, cat, sort, uniq, cut, tr, head, tail, wc, date, echo, printf, basename, dirname, pwd, env
```

| Mode | Behavior |
|------|----------|
| `deny` | All shell execution blocked |
| `allowlist` (default) | Only commands in `safe_bins` or `allowed_commands` allowed. Direct argv execution (no shell interpreter). |
| `full` | All commands allowed via `sh -c`. **Unsafe for production.** |

**YOLO mode:** Set `exec_policy.mode = "full"` or pass `--yolo` to the CLI to auto-approve all tool calls and enable unrestricted shell access. This is equivalent to `auto_approve = true` combined with full exec mode.

### `[[fallback_providers]]`

Automatic failover chain. Tried in order when the primary provider fails.

```toml
[[fallback_providers]]
provider = "anthropic"
model = "claude-sonnet-4-6"

[[fallback_providers]]
provider = "openai"
model = "gpt-4o"

[[fallback_providers]]
provider = "groq"
model = "llama-3.3-70b-versatile"
```

Each entry supports: `provider`, `model`, `api_key_env`, `base_url`.

### `[model_routing]`

Automatically select model tier based on task complexity.

```toml
[model_routing]
enabled = false
simple_threshold = 0.3
complex_threshold = 0.8
simple_provider = "groq"
simple_model = "llama-3.1-8b-instant"
balanced_provider = "groq"
balanced_model = "llama-3.3-70b-versatile"
complex_provider = "anthropic"
complex_model = "claude-opus-4-6"
```

### Channel Configuration

Each channel adapter has its own config section under `[channels.*]`.

#### `[channels.telegram]`

```toml
[channels.telegram]
enabled = true
bot_token_env = "TELEGRAM_BOT_TOKEN"           # Env var holding bot token
allowed_users = []                             # Telegram user IDs (empty = allow all)
# model_override = "claude-haiku-4-5"          # Per-channel model
# output_format = "telegram_html"              # telegram_html | markdown | plain_text
# api_url = "https://api.telegram.org"         # Optional: proxy URL
# default_chat_id = ""                         # Default recipient for channel_send
```

#### `[channels.discord]`

```toml
[channels.discord]
enabled = true
bot_token_env = "DISCORD_BOT_TOKEN"
guild_ids = []                                 # Guild snowflakes (empty = all guilds)
# ignore_bots = true
# send_in_thread = false
```

#### `[channels.slack]`

```toml
[channels.slack]
enabled = true
bot_token_env = "SLACK_BOT_TOKEN"              # xoxb-...
app_token_env = "SLACK_APP_TOKEN"              # xapp-... (Socket Mode)
```

#### `[channels.email]`

```toml
[channels.email]
enabled = true
imap_host = "imap.gmail.com"
imap_port = 993
imap_username = "bot@example.com"
imap_password = "${EMAIL_PASSWORD}"
imap_tls = true
smtp_host = "smtp.gmail.com"
smtp_port = 587
smtp_username = "bot@example.com"
smtp_password = "${EMAIL_PASSWORD}"
smtp_starttls = true
```

#### `[channels.whatsapp]`

```toml
# Cloud API mode
[channels.whatsapp]
enabled = true
mode = "cloud"                                 # cloud | baileys
phone_number_id = "123456789"
access_token = "${WHATSAPP_TOKEN}"

# Baileys gateway mode
[channels.whatsapp]
enabled = true
mode = "baileys"
gateway_url = "http://localhost:3000"
```

#### `[channels.matrix]`

```toml
[channels.matrix]
enabled = true
homeserver_url = "https://matrix.example.org"
user_id = "@bot:example.org"
access_token = "${MATRIX_TOKEN}"
room_ids = ["!roomid:example.org"]
```

#### `[channels.mattermost]`

```toml
[channels.mattermost]
enabled = true
server_url = "https://mattermost.example.com"
bot_token = "${MATTERMOST_TOKEN}"
team_name = "my-team"
channel_name = "town-square"
```

#### `[channels.mqtt]`

```toml
[channels.mqtt]
enabled = true
broker_url = "tcp://broker.hivemq.com:1883"
subscribe_topic = "openfang/inbox"
publish_topic = "openfang/outbox"
username_env = "MQTT_USERNAME"
password_env = "MQTT_PASSWORD"
use_tls = false
qos = 1                                       # 0 | 1 | 2
```

*Added in v0.5.4.*

#### Channel Overrides

All channels support these per-channel behavior overrides:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `model` | string | agent default | Model override for this channel |
| `system_prompt` | string | none | System prompt override |
| `dm_policy` | enum | `respond` | respond, allowed_only, ignore |
| `group_policy` | enum | `mention_only` | all, mention_only, commands_only, ignore |
| `rate_limit_per_user` | u32 | 0 | Messages per minute per user (0 = unlimited) |
| `threading` | bool | false | Enable thread replies |
| `output_format` | enum | `markdown` | markdown, telegram_html, slack_mrkdwn, plain_text |
| `usage_footer` | enum | global | off, tokens, cost, full |
| `typing_mode` | enum | `instant` | instant, message, thinking, never |
| `lifecycle_reactions` | bool | true | Send emoji reactions on message lifecycle |

### `[[mcp_servers]]`

Connect to external MCP (Model Context Protocol) servers.

```toml
# Stdio transport (subprocess)
[[mcp_servers]]
name = "filesystem"
timeout_secs = 30
env = ["GITHUB_PERSONAL_ACCESS_TOKEN"]
[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]

# SSE transport (HTTP)
[[mcp_servers]]
name = "github"
[mcp_servers.transport]
type = "sse"
url = "https://mcp.github.com/sse"
headers = ["Authorization: Bearer ${GITHUB_TOKEN}"]

# Streamable HTTP transport (MCP 2025-03-26+)
[[mcp_servers]]
name = "remote"
[mcp_servers.transport]
type = "http"
url = "https://mcp.example.com/api"
```

### `[a2a]`

Agent-to-Agent protocol for cross-instance communication.

```toml
[a2a]
enabled = false
listen_path = "/a2a"
# public_url = "https://my-agent.example.com"
```

### `[[users]]` (Multi-User RBAC)

```toml
[[users]]
name = "admin"
role = "owner"                                 # owner | admin | user | viewer
channel_bindings = { telegram = "123456789" }
# api_key_hash = "..."
```

### `[[bindings]]` (Agent Routing)

Route specific channel/account/peer patterns to agents.

```toml
[[bindings]]
agent = "support-bot"
[bindings.match_rule]
channel = "telegram"
peer_id = "123456789"

[[bindings]]
agent = "dev-assistant"
[bindings.match_rule]
channel = "discord"
guild_id = "987654321"
```

### `[broadcast]`

Send same message to multiple agents.

```toml
[broadcast]
strategy = "parallel"                          # parallel | sequential

[broadcast.routes]
"123456789" = ["agent-a", "agent-b"]
```

### `[browser]`

Browser automation (CDP-based) configuration.

```toml
[browser]
enabled = true
headless = true
viewport_width = 1280
viewport_height = 720
timeout_secs = 30
idle_timeout_secs = 300
max_sessions = 5
# chromium_path = "/usr/bin/chromium"           # Auto-detected if None
```

### `[docker]`

Docker container sandbox for OS-level agent isolation.

```toml
[docker]
enabled = false
mode = "off"                                   # off | non_main | all
image = "python:3.12-slim"
network = "none"                               # none | bridge | custom
memory_limit = "512m"
cpu_limit = 1.0
timeout_secs = 60
read_only_root = true
pids_limit = 100
scope = "session"                              # session | agent | shared
```

### `[tts]`

Text-to-speech configuration.

```toml
[tts]
enabled = false
# provider = "openai"                          # openai | elevenlabs
max_text_length = 4096
timeout_secs = 30

[tts.openai]
voice = "alloy"                                # alloy | echo | fable | onyx | nova | shimmer
model = "tts-1"                                # tts-1 | tts-1-hd
format = "mp3"                                 # mp3 | opus | aac | flac
speed = 1.0
```

### `[canvas]`

Canvas (Agent-to-UI) tool for presenting interactive HTML.

```toml
[canvas]
enabled = false
max_html_bytes = 524288                        # 512 KB
```

### `[reload]`

Config hot-reload settings.

```toml
[reload]
mode = "hybrid"                                # off | restart | hot | hybrid
debounce_ms = 500                              # Debounce window for file changes
```

| Mode | Behavior |
|------|----------|
| `off` | No automatic reloading |
| `restart` | Full restart on config change |
| `hot` | Hot-reload safe sections only (channels, skills, heartbeat) |
| `hybrid` (default) | Hot-reload where possible, flag restart-required otherwise |

### `[heartbeat]`

Heartbeat monitor settings for detecting unresponsive agents.

```toml
[heartbeat]
default_timeout_secs = 180                     # Seconds before marking as unresponsive
```

### `[auto_reply]`

Auto-reply background engine.

```toml
[auto_reply]
enabled = false
max_concurrent = 3
timeout_secs = 120
suppress_patterns = ["/stop", "/pause"]
```

### `[extensions]`

MCP integration lifecycle management.

```toml
[extensions]
auto_reconnect = true
reconnect_max_attempts = 10
reconnect_max_backoff_secs = 300
health_check_interval_secs = 60
```

### `[thinking]`

Extended thinking for supported models.

```toml
[thinking]
budget_tokens = 10000
stream_thinking = false
```

### Provider URL and Key Overrides

```toml
[provider_urls]
ollama = "http://192.168.1.100:11434/v1"

[provider_api_keys]
nvidia = "NVIDIA_API_KEY"
```

---

## Environment Variables

All config values can reference environment variables with `${VAR_NAME}` syntax:

```toml
[default_model]
api_key_env = "GROQ_API_KEY"                   # Reads from GROQ_API_KEY env var

[channels.telegram]
bot_token_env = "TELEGRAM_BOT_TOKEN"           # Env var name reference
access_token = "${WHATSAPP_TOKEN}"             # Inline env var substitution
```

---

## Agent Configuration (`agent.toml`)

Each agent is defined by a TOML manifest. Agents can be created via the API, CLI, or placed in `~/.openfang/agents/`.

```toml
name = "researcher"
description = "Web research assistant"
module = "llm"

[model]
provider = "anthropic"
model = "claude-sonnet-4-6"

[capabilities]
required = [
  "ToolInvoke(web_search)",
  "ToolInvoke(web_fetch)",
  "MemoryRead",
  "MemoryWrite",
]

[exec_policy]
mode = "allowlist"
allowed_commands = ["git", "cargo"]

# Tags for discovery via agent_find
tags = ["research", "web"]
```

Agent manifests support all `[exec_policy]` fields (mode, allowed_commands, safe_bins, timeout_secs) which override the global config for that specific agent.

---

## YOLO Mode

YOLO mode disables approval gates and enables unrestricted shell access:

**Via config:**
```toml
[exec_policy]
mode = "full"

[approval]
auto_approve = true
```

**Via CLI flag:**
```bash
openfang start --yolo
```

This combines `exec_policy.mode = "full"` with `auto_approve = true`, bypassing the approval gate for all tools including `shell_exec` and `process_start`. Not recommended for production.

---

## Config Include Files

Split large configs into multiple files:

```toml
include = ["channels.toml", "agents.toml"]
```

Paths are relative to the root config file. Absolute paths and `..` components are rejected for security. Included files are deep-merged before the root config.

---

## Configuration via CLI

```bash
openfang config show                           # View current config
openfang config get default_model.provider     # Get a value
openfang config set default_model.provider anthropic  # Set a value
openfang config set-key anthropic sk-ant-...   # Set API key (stored in vault)
openfang config test-key anthropic             # Test a key
openfang config edit                           # Open in $EDITOR
openfang config reload                         # Force immediate reload
```

---

## Configuration via API

```bash
# Get config
GET /api/config

# Get schema (public endpoint)
GET /api/config/schema

# Update config
POST /api/config
{
  "default_model": {
    "provider": "anthropic",
    "model": "claude-opus-4-6"
  }
}

# Force immediate reload
POST /api/config/reload
```
