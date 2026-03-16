# Configuration

OpenFang is configured via `~/.openfang/config.toml`. All fields support `#[serde(default)]` for forward compatibility — unknown fields are silently ignored.

Config hot-reloads every 30 seconds without daemon restart. Force a reload: `POST /api/config/reload`.

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

### `[default_model]`

```toml
[default_model]
provider = "groq"                           # Provider ID
model = "llama-3.3-70b-versatile"           # Model ID
api_key_env = "GROQ_API_KEY"               # Env var holding API key
system_prompt = "You are a helpful assistant."
temperature = 0.7                           # 0.0–2.0 (null = provider default)
max_tokens = 4096                           # Max output tokens
```

### `[memory]`

```toml
[memory]
decay_rate = 0.1                           # Memory importance decay per day
context_window_chars = 100000              # Characters before compaction
embedding_enabled = true                   # Enable vector embeddings
embedding_provider = "openai"              # Embedding provider
embedding_model = "text-embedding-3-small"
```

### `[network]`

```toml
[network]
listen_addr = "127.0.0.1:4200"            # API server bind address
public_url = ""                            # Public URL (for webhooks)
enable_ofp = false                         # Enable OFP peer protocol
api_key = ""                               # Bearer token (empty = loopback bypass only)
```

### `[web]`

```toml
[web]
cors_origins = ["http://localhost:3000"]   # Allowed CORS origins
enable_swagger = false                     # Enable Swagger UI at /swagger
```

### `[quota]`

```toml
[quota]
default_max_llm_tokens_per_hour = 0       # 0 = unlimited (rolling 1-hour window)
```

### `[log]`

```toml
[log]
level = "info"                             # error | warn | info | debug | trace
```

### `[model_routing]`

Automatically select model tier based on task complexity:

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

### `[[fallback_providers]]`

Automatic failover chain:

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

### Channel Configuration

Each channel adapter has its own config section:

#### Telegram

```toml
[channels.telegram]
enabled = true
bot_token = "${TELEGRAM_BOT_TOKEN}"        # From @BotFather
allowed_users = [123456789]                # Telegram user IDs (empty = all)
model_override = "claude-haiku-4-5"        # Optional: per-channel model
output_format = "telegram_html"            # telegram_html | markdown | plain
api_url = "https://api.telegram.org"       # Optional: proxy URL
```

#### Discord

```toml
[channels.discord]
enabled = true
bot_token = "${DISCORD_BOT_TOKEN}"
channel_ids = ["1234567890123456789"]      # Channel IDs to listen to
ignore_bots = true                         # Ignore messages from other bots
send_in_thread = false                     # Reply in threads
model_override = ""                        # Optional
```

#### Slack

```toml
[channels.slack]
enabled = true
app_token = "${SLACK_APP_TOKEN}"           # xapp-... (Socket Mode)
bot_token = "${SLACK_BOT_TOKEN}"           # xoxb-...
```

#### Email

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

#### WhatsApp (Cloud API)

```toml
[channels.whatsapp]
enabled = true
mode = "cloud"                             # cloud | baileys
phone_number_id = "123456789"
access_token = "${WHATSAPP_TOKEN}"
```

#### WhatsApp (Baileys Gateway)

```toml
[channels.whatsapp]
enabled = true
mode = "baileys"
gateway_url = "http://localhost:3000"      # Local Baileys gateway
```

#### Matrix

```toml
[channels.matrix]
enabled = true
homeserver_url = "https://matrix.example.org"
user_id = "@bot:example.org"
access_token = "${MATRIX_TOKEN}"
room_ids = ["!roomid:example.org"]
```

#### Mattermost

```toml
[channels.mattermost]
enabled = true
server_url = "https://mattermost.example.com"
bot_token = "${MATTERMOST_TOKEN}"
team_name = "my-team"
channel_name = "town-square"
```

### MCP Server Configuration

Connect to external MCP servers:

```toml
# Stdio transport (subprocess)
[[mcp_servers]]
name = "filesystem"
transport = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]

# SSE transport (HTTP)
[[mcp_servers]]
name = "github"
transport = "sse"
url = "https://mcp.github.com/sse"
headers = { Authorization = "Bearer ${GITHUB_TOKEN}" }
```

### A2A Protocol

```toml
[a2a]
enabled = false
public_url = "https://my-agent.example.com"
```

### Multi-User RBAC

```toml
[[users]]
id = "admin"
api_key = "${ADMIN_API_KEY}"
role = "admin"                             # admin | operator | viewer

[[users]]
id = "readonly"
api_key = "${READONLY_API_KEY}"
role = "viewer"
allowed_agents = ["public-assistant"]     # Restrict to specific agents
```

---

## Environment Variables

All config values can reference environment variables with `${VAR_NAME}` syntax:

```toml
[default_model]
api_key_env = "GROQ_API_KEY"   # Reads from GROQ_API_KEY env var

[channels.telegram]
bot_token = "${TELEGRAM_BOT_TOKEN}"   # Inline env var substitution
```

---

## openfang.toml.example

The repository includes a fully commented example configuration at `openfang.toml.example` showing all available options with their defaults and explanations.

---

## Configuration via CLI

```bash
# View current config
openfang config show

# Get a value
openfang config get default_model.provider

# Set a value
openfang config set default_model.provider anthropic
openfang config set default_model.model claude-opus-4-6

# Set API key (stored securely)
openfang config set-key anthropic sk-ant-...
openfang config set-key groq gsk_...

# Test a key
openfang config test-key anthropic

# Open in editor
openfang config edit

# Reload without restart
openfang config reload
```

---

## Configuration via API

```bash
# Get config
GET /api/config

# Get schema (public endpoint)
GET /api/config/schema

# Update config (triggers 30s reload cycle)
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
