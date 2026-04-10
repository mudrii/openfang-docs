# Channel Adapters

Validated against OpenFang release `v0.5.7`.

OpenFang ships **43 channel adapter modules** in `crates/openfang-channels/src/` (excluding utility modules `bridge`, `formatter`, `router`, `types`). The API's `CHANNEL_REGISTRY` in `crates/openfang-api/src/routes.rs` tracks all registered adapters.

All channels share the same bridge infrastructure: exponential backoff with capped retry, graceful shutdown via `watch` channels, secret zeroization on drop (`zeroize` crate), automatic message splitting at platform limits, per-channel override support, and lifecycle reaction indicators.

---

## Common Configuration

Every channel is configured under `[channels.<name>]` in `~/.openfang/config.toml` (or via the dashboard **Channels** tab). All channels support a shared set of behavioral overrides in the `[channels.<name>.overrides]` section.

### Per-Channel Overrides

```toml
[channels.telegram.overrides]
model = "claude-haiku-4-5"          # Use a different model for this channel
system_prompt = "You are a concise assistant."  # Custom persona
output_format = "telegram_html"     # telegram_html | slack_mrkdwn | markdown | plain_text
dm_policy = "respond"               # respond | allowed_only | ignore
group_policy = "mention_only"       # all | mention_only | commands_only | ignore
rate_limit_per_user = 10            # Messages per minute per user (0 = unlimited)
threading = true                    # Enable thread replies
usage_footer = "full"               # off | tokens | cost | full
typing_mode = "instant"             # instant | message | thinking | never
lifecycle_reactions = true          # Send emoji reactions (queued/thinking/done/error)
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `model` | string | agent default | Override the LLM model for this channel |
| `system_prompt` | string | agent default | Custom system prompt / persona |
| `output_format` | enum | `markdown` | `telegram_html`, `slack_mrkdwn`, `markdown`, `plain_text` |
| `dm_policy` | enum | `respond` | How to handle direct messages |
| `group_policy` | enum | `mention_only` | How to handle group messages |
| `rate_limit_per_user` | u32 | `0` | Messages per minute per user (0 = unlimited) |
| `threading` | bool | `false` | Enable threaded replies |
| `usage_footer` | enum | `full` | What usage info to show in response footers |
| `typing_mode` | enum | `instant` | When to send typing indicators |
| `lifecycle_reactions` | bool | `true` | Show emoji status reactions on messages |

### DM Policy

| Value | Behavior |
|-------|----------|
| `respond` | Respond to all DMs (default) |
| `allowed_only` | Only respond to DMs from allowed users |
| `ignore` | Ignore all DMs |

### Group Policy

| Value | Behavior |
|-------|----------|
| `all` | Respond to all group messages |
| `mention_only` | Only respond when @mentioned (default) |
| `commands_only` | Only respond to `/slash` commands |
| `ignore` | Ignore all group messages |

### Default Agent Routing

Every channel config has a `default_agent` field that sets which agent handles incoming messages:

```toml
[channels.telegram]
default_agent = "assistant"
```

When unset, the bridge uses the first running agent. Users can switch agents at runtime with the `/agent <name>` command.

### Lifecycle Reactions

When `lifecycle_reactions = true`, the bot sets emoji reactions on the user's message to indicate progress:

| Phase | Emoji | Meaning |
|-------|-------|---------|
| Queued | hourglass | Message waiting for agent |
| Thinking | thinking face | LLM is reasoning |
| Tool Use | gear | Executing a tool |
| Streaming | writing hand | Streaming response tokens |
| Done | check mark | Agent finished |
| Error | cross mark | Agent encountered an error |

Each new reaction replaces the previous one automatically. Supported on Telegram (via `setMessageReaction`). Set `lifecycle_reactions = false` on channels that don't support reactions or where you want clean UX.

---

## Core Messaging (7)

### Telegram

**Protocol:** Bot API long-polling (`getUpdates` with 30s timeout)

**Source:** `telegram.rs` -- no external Telegram crate, raw `reqwest` for full control.

**Constructor fields:** `token`, `allowed_users`, `poll_interval`, `api_url`

**Features:**
- Text, photos (magic-byte MIME detection, picks largest size), documents, voice messages, locations
- File upload via multipart form (for proactive `channel_send` tool)
- Typing indicator (refreshes every 4s via `sendChatAction`)
- Forum topic support (`message_thread_id` for supergroup topics)
- Group @mention detection (in both `entities` and `caption_entities`, case-insensitive)
- Reply-to-message context (quoted text prepended to content)
- Bot command registration via `setMyCommands` on startup
- Edited message handling (`edited_message` updates)
- HTML sanitization (only allows Telegram-safe tags: `b`, `i`, `u`, `s`, `a`, `code`, `pre`, `blockquote`, `tg-spoiler`)
- `sender_chat` fallback for channel-forwarded messages
- 409 Conflict recovery (stale polling sessions on daemon restart)

**Config:**
```toml
[channels.telegram]
bot_token_env = "TELEGRAM_BOT_TOKEN"
allowed_users = [123456789, 987654321]
default_agent = "assistant"
poll_interval_secs = 1
api_url = "https://api.telegram.org"   # override for proxy/mirror
default_chat_id = ""                   # for proactive messaging

[channels.telegram.overrides]
output_format = "telegram_html"
lifecycle_reactions = true
group_policy = "mention_only"
dm_policy = "respond"
```

**Setup:** `openfang channel setup telegram`

**Limit:** 4,096 chars per message (auto-split).

---

### Discord

**Protocol:** Gateway WebSocket v10 with heartbeat/resume + REST API v10 for sending

**Source:** `discord.rs` -- `tokio-tungstenite` + `reqwest`, no external Discord crate.

**Constructor fields:** `token`, `allowed_guilds`, `allowed_users`, `ignore_bots`, `intents`

**Features:**
- Full Gateway lifecycle: HELLO, IDENTIFY, RESUME, HEARTBEAT, RECONNECT, INVALID_SESSION
- Heartbeat ACK monitoring (forces reconnect on zombie connections)
- Session resume with stored `session_id` and `resume_gateway_url`
- @mention detection (both `mentions` array and `<@bot_id>` / `<@!bot_id>` content patterns)
- Group vs DM detection via `guild_id` presence
- Guild and user allowlists
- Configurable `ignore_bots` (default: true, set false for bot-to-bot multi-agent setups)
- Edited message handling (`MESSAGE_UPDATE`)
- Typing indicator via `POST /channels/{id}/typing`

**Config:**
```toml
[channels.discord]
bot_token_env = "DISCORD_BOT_TOKEN"
allowed_guilds = ["1234567890"]
allowed_users = []
default_agent = "assistant"
intents = 37376                  # GUILD_MESSAGES | DIRECT_MESSAGES | MESSAGE_CONTENT
ignore_bots = true
default_channel_id = ""          # for proactive messaging

[channels.discord.overrides]
output_format = "markdown"
group_policy = "mention_only"
```

**Requires:** Message Content Intent enabled in the Discord Developer Portal.

**Setup:** `openfang channel setup discord`

**Limit:** 2,000 chars per message (auto-split).

---

### Slack

**Protocol:** Socket Mode WebSocket (app-level token) for receiving, Web API (bot token) for sending

**Source:** `slack.rs` -- `tokio-tungstenite` + `reqwest`, no external Slack crate.

**Constructor fields:** `app_token`, `bot_token`, `allowed_channels`, `auto_thread_reply`, `thread_ttl_hours`, `unfurl_links`

**Features:**
- Socket Mode connection with envelope acknowledgment
- Thread tracking: when @mentioned in a thread, auto-replies to follow-up messages in that thread
- Configurable thread TTL (default: 24h) with periodic cleanup (every 5 min)
- @mention detection (both `app_mention` events and `<@bot_id>` in text)
- `message_changed` subtype handling (edited messages)
- Channel allowlist filtering
- Configurable link unfurling (`unfurl_links` / `unfurl_media`)
- Thread replies via `thread_ts`

**Config:**
```toml
[channels.slack]
app_token_env = "SLACK_APP_TOKEN"       # xapp-... (Socket Mode token)
bot_token_env = "SLACK_BOT_TOKEN"       # xoxb-... (Bot User OAuth Token)
allowed_channels = ["C0123456789"]
default_agent = "assistant"
auto_thread_reply = true                # reply to follow-ups in tracked threads
thread_ttl_hours = 24                   # how long to track threads
unfurl_links = true                     # expand link previews

[channels.slack.overrides]
output_format = "slack_mrkdwn"
group_policy = "mention_only"
```

**Setup:** `openfang channel setup slack`

**Limit:** 3,000 chars per message (auto-split).

---

### WhatsApp

**Two modes:**
1. **Cloud API** -- Official Meta WhatsApp Business Cloud API with webhook
2. **Web/QR gateway** -- Baileys-based gateway for personal accounts (`packages/whatsapp-gateway/`)

Mode is selected automatically: if `WHATSAPP_WEB_GATEWAY_URL` is set, the adapter uses Web mode. Otherwise it falls back to Cloud API.

**Source:** `whatsapp.rs`

**Constructor fields:** `phone_number_id`, `access_token`, `verify_token`, `webhook_port`, `allowed_users` + optional `gateway_url`

**Features (Cloud API):** Text, images, documents, locations via Graph API v21.0
**Features (Web mode):** Text messages routed through gateway; media types fall back to text descriptions

**Config:**
```toml
[channels.whatsapp]
phone_number_id = "123456789012345"
access_token_env = "WHATSAPP_ACCESS_TOKEN"
verify_token_env = "WHATSAPP_VERIFY_TOKEN"
webhook_port = 8443
gateway_url_env = "WHATSAPP_WEB_GATEWAY_URL"   # set to enable Web/QR mode
allowed_users = ["+1234567890"]
default_agent = "assistant"

[channels.whatsapp.overrides]
dm_policy = "respond"
```

**Limit:** 4,096 chars per message (auto-split).

See [WhatsApp Web Gateway Setup](whatsapp-gateway.md) for the QR code pairing guide.

---

### Signal

**Protocol:** `signal-cli` JSON-RPC via HTTP

**Source:** `signal.rs`

**Config:**
```toml
[channels.signal]
api_url = "http://localhost:8080"
phone_number = "+1234567890"
allowed_users = []
default_agent = "assistant"

[channels.signal.overrides]
dm_policy = "respond"
```

**Requires:** Running `signal-cli` daemon (REST mode).

---

### Matrix

**Protocol:** Client-Server API `/sync` long-polling (30s timeout)

**Source:** `matrix.rs` -- raw `reqwest` against the Matrix CS API.

**Constructor fields:** `homeserver_url`, `user_id`, `access_token`, `allowed_rooms`, `auto_accept_invites`

**Features:**
- `/sync` polling with resume token (skips old messages on first connect via initial sync with `limit:0`)
- Auto-accept room invites (configurable)
- DM vs group detection (checks joined member count: 2 members = DM)
- @mention detection in message text
- Typing indicator via `PUT /rooms/{id}/typing/{user_id}`
- Event dedup (tracks last 500 event IDs)
- `/whoami` validation on startup (prevents self-reply loops from server delegation mismatches)

**Config:**
```toml
[channels.matrix]
homeserver_url = "https://matrix.org"
user_id = "@openfang:matrix.org"
access_token_env = "MATRIX_ACCESS_TOKEN"
allowed_rooms = ["!room1:matrix.org"]   # empty = all joined rooms
default_agent = "assistant"
auto_accept_invites = false

[channels.matrix.overrides]
output_format = "markdown"
group_policy = "mention_only"
```

---

### Email (IMAP/SMTP)

**Protocol:** IMAP polling (inbound) + SMTP (outbound replies)

**Source:** `email.rs` -- uses `imap` crate (blocking, via `spawn_blocking`), `lettre` for SMTP, `mailparse` for parsing.

**Constructor fields:** `imap_host`, `imap_port`, `smtp_host`, `smtp_port`, `username`, `password`, `poll_interval_secs`, `folders`, `allowed_senders`

**Features:**
- IMAP TLS + SASL PLAIN fallback (for servers like Lark that reject LOGIN)
- Multi-folder monitoring (default: INBOX)
- Reply threading via `In-Reply-To` header and subject continuity (`Re: ...`)
- Agent routing from subject brackets: `[coder] Fix the bug` routes to agent `coder`
- Sender allowlist filtering
- Marks fetched messages as `\Seen`
- Batched fetch (up to 50 emails per poll)
- Multipart email parsing (extracts `text/plain` body)

**Config:**
```toml
[channels.email]
imap_host = "imap.gmail.com"
imap_port = 993
smtp_host = "smtp.gmail.com"
smtp_port = 587                          # 587 for STARTTLS, 465 for implicit TLS
username = "bot@example.com"
password_env = "EMAIL_PASSWORD"
poll_interval_secs = 30
folders = ["INBOX"]
allowed_senders = ["boss@company.com"]   # empty = all senders
default_agent = "assistant"

[channels.email.overrides]
output_format = "plain_text"
```

**Limit:** 20,000 chars per message.

---

## Enterprise Platforms (8)

### Microsoft Teams

- **Protocol:** Bot Framework REST API + webhook
- **Config:** `app_id`, `app_password`, `webhook_url`

### Mattermost

- **Protocol:** WebSocket v4 + REST API v4
- **Config:** `server_url`, `bot_token`, `team_name`, `channel_name`
- **Limit:** 16,383 chars per message

### Google Chat

- **Protocol:** Service account JWT + webhook
- **Config:** `service_account_json`, `room_id`
- **Auth:** Google service account with OAuth2 scopes

### Webex (Cisco)

- **Protocol:** Mercury WebSocket + REST API
- **Config:** `bot_token`, `room_filter`

### Feishu / Lark

- **Protocol:** Open Platform API
- **Config:** `app_id`, `app_secret`, `region` (`cn` or `intl`)

### Rocket.Chat

- **Protocol:** REST API + long-polling history
- **Config:** `server_url`, `auth_token`, `user_id`

### Zulip

- **Protocol:** REST API event queue polling
- **Config:** `server_url`, `bot_email`, `api_key`, `stream`, `topic`
- **Auth:** HTTP Basic
- **Limit:** 10,000 chars per message

### DingTalk

- **Two modes:**
  1. **HTTP webhook** -- Simple outbound + robot API
  2. **Stream mode** -- WebSocket (Corp ID + agent secret)
- **Config (webhook):** `access_token`, `webhook_url`, `secret` (HMAC-SHA256)
- **Config (stream):** `corp_id`, `agent_secret`

---

## Social Platforms (8)

### LINE

- **Protocol:** REST API + webhook
- **Config:** `channel_access_token`, `channel_secret`
- **Auth:** Bearer token

### Viber

- **Protocol:** REST webhook + API
- **Config:** `account_access_token`, `webhook_url`

### Facebook Messenger

- **Protocol:** Graph API v18 + webhook
- **Config:** `page_access_token`, `webhook_url`

### Mastodon

- **Protocol:** REST API + SSE stream
- **Config:** `instance_url`, `access_token`

### Bluesky

- **Protocol:** AT Protocol XRPC
- **Config:** `identifier` (handle), `app_password`
- **Auth:** App password (not account password)

### Reddit

- **Protocol:** OAuth2 script app (password grant)
- **Config:** `client_id`, `client_secret`, `username`, `password`, `subreddit`

### LinkedIn

- **Protocol:** REST API (OAuth2)
- **Config:** `access_token`, `organization_id`
- **Auth:** OAuth2

### Twitch

- **Protocol:** IRC over TLS
- **Config:** `oauth_token`, `channel_names`

---

## Community Platforms (6)

### IRC

- **Protocol:** Raw TCP (plaintext)
- **Config:** `host`, `port`, `nick`, `channels`, `password`
- **Limit:** 510 chars per message

### Guilded

- **Protocol:** WebSocket (Bonfire) + REST
- **Config:** `bot_token`, `channel_id`
- **Auth:** Bearer token

### Revolt

- **Protocol:** REST API + WebSocket (Bonfire)
- **Config:** `bot_token`, `server_id`, `channel_ids`

### Keybase

- **Protocol:** HTTP JSON-RPC (local Keybase daemon)
- **Config:** `username`, `paper_key`, `team`, `channel`

### Discourse

- **Protocol:** REST API polling
- **Config:** `base_url`, `api_key`, `api_username`, `category_id`

### Gitter

- **Protocol:** Streaming API (newline-delimited JSON)
- **Config:** `bearer_token`, `room_id`
- **Auth:** Bearer token

---

## Privacy-First (3)

### Nostr

- **Protocol:** WebSocket relay (NIP-01/04)
- **Config:** `relay_urls`, `private_key`

### Threema (Gateway)

- **Protocol:** HTTP webhook + REST API
- **Config:** `gateway_api_secret`, `webhook_url`
- **Auth:** HMAC-SHA256

### Mumble

- **Protocol:** TCP protobuf framing
- **Config:** `server`, `port`, `username`
- **Note:** VoIP -- text chat only (audio not implemented)

---

## Workplace Tools (4)

### Pumble

- **Protocol:** Webhook + REST API
- **Config:** `bot_token`, `webhook_url`

### Flock

- **Protocol:** Webhook + REST API
- **Config:** `bot_token`, `webhook_url`

### Twist

- **Protocol:** REST API v3 polling
- **Config:** `oauth_token`, `workspace_id`, `thread_ids`
- **Auth:** OAuth2 Bearer

### WeCom (WeChat Work)

- **Protocol:** Work API + webhook
- **Config:** `corp_id`, `agent_id`, `agent_secret`, `webhook_url`
- **Auth:** HMAC-SHA256

---

## IoT, Notifications & Integration (5)

### MQTT

- **Protocol:** MQTT 3.1.1/5.0 pub/sub
- **Config:** `broker_url`, `subscribe_topic`, `publish_topic`, `username_env`, `password_env`, `use_tls`, `qos`
- **Features:** Plain text messages, JSON payloads (`{"text": "message"}`), command messages (`/` prefix), configurable QoS levels
- **Use case:** IoT device integration, home automation, industrial messaging
- **Limit:** 4,096 chars per message
- **Setup:** `openfang channel setup mqtt`

*Added in v0.5.4.*

### Nextcloud Talk

- **Protocol:** OCS REST API v2 polling
- **Config:** `server_url`, `token`, `room_id`

### ntfy.sh

- **Protocol:** SSE (subscribe) + HTTP POST (publish)
- **Config:** `server_url`, `topic`, `auth_token`

### Gotify

- **Protocol:** WebSocket + REST
- **Config:** `server_url`, `app_token`, `client_token`

### Generic Webhook

- **Protocol:** HTTP POST with HMAC-SHA256 verification
- **Config:** `webhook_secret`, `callback_url`
- **Use case:** Any external system that can send HTTP webhooks

---

## Not Yet Implemented

### XMPP

- **Status:** Stub (awaiting `tokio-xmpp` maturity)
- **Config:** `jid`, `password`

---

## Message Limits by Platform

| Platform | Max chars |
|----------|-----------|
| IRC | 510 |
| Telegram | 4,096 (auto-split) |
| Discord | 2,000 |
| Slack | 3,000 |
| Mattermost | 16,383 |
| Zulip | 10,000 |
| Email | 20,000 |
| WhatsApp | 4,096 |
| WeCom | 2,048 |
| MQTT | 4,096 |
| REST APIs | 4,000--32,000 |

OpenFang automatically splits messages that exceed platform limits using `split_message()`.

---

## Building Custom Adapters

Implement the `ChannelAdapter` trait:

```rust
#[async_trait]
pub trait ChannelAdapter: Send + Sync {
    fn name(&self) -> &str;
    fn channel_type(&self) -> ChannelType;
    async fn start(&self) -> Result<Pin<Box<dyn Stream<Item = ChannelMessage> + Send>>, Box<dyn std::error::Error>>;
    async fn send(&self, user: &ChannelUser, content: ChannelContent) -> Result<(), Box<dyn std::error::Error>>;
    async fn stop(&self) -> Result<(), Box<dyn std::error::Error>>;

    // Optional overrides:
    async fn send_typing(&self, _user: &ChannelUser) -> Result<(), Box<dyn std::error::Error>> { Ok(()) }
    async fn send_in_thread(&self, user: &ChannelUser, content: ChannelContent, thread_id: &str) -> Result<(), Box<dyn std::error::Error>> { self.send(user, content).await }
    async fn send_reaction(&self, _user: &ChannelUser, _message_id: &str, _reaction: &LifecycleReaction) -> Result<(), Box<dyn std::error::Error>> { Ok(()) }
}
```

See [Contributing](contributing.md) for the full guide.
