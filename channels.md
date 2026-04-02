# Channel Adapters

OpenFang supports 41 messaging channel adapters. All channels share the same unified bridge architecture: exponential backoff, graceful shutdown, secret zeroization, message splitting, and per-channel configuration overrides.

---

## Channel Configuration

Channels are configured in `~/.openfang/config.toml` or via the web dashboard **Channels** tab:

```toml
[channels.telegram]
enabled = true
bot_token = "${TELEGRAM_BOT_TOKEN}"
allowed_users = [123456789]
model_override = "claude-haiku-4-5"
output_format = "telegram_html"     # telegram_html | markdown | plain

[channels.discord]
enabled = true
bot_token = "${DISCORD_BOT_TOKEN}"
channel_ids = ["1234567890"]
ignore_bots = true
send_in_thread = false

[channels.slack]
enabled = true
app_token = "${SLACK_APP_TOKEN}"
bot_token = "${SLACK_BOT_TOKEN}"
```

### Per-Channel Overrides (All Channels)

| Field | Description |
|-------|-------------|
| `model_override` | Use a different model for this channel |
| `system_prompt_override` | Custom persona for this channel |
| `output_format` | `telegram_html`, `slack_mrkdwn`, `markdown`, `plain` |
| `allow_dms` | Allow direct messages |
| `allow_groups` | Allow group/channel messages |
| `rate_limit` | Messages per minute per user |
| `allowed_users` | Whitelist of user IDs |

---

## All 41 Channel Adapters

### Core Messaging (7)

#### Telegram
- **Protocol:** Bot API long-polling (30s timeout)
- **Config:** `bot_token`, `allowed_users`, `api_url` (proxy support), `forum_thread_id`
- **Features:** Images (magic-byte MIME detection), voice (auto-download), typing indicator (refreshes every 4s), forum topics, group chat
- **Setup:** `openfang channel setup telegram`

#### Discord
- **Protocol:** Gateway WebSocket v10 with heartbeat/resume
- **Config:** `bot_token`, `channel_ids`, `ignore_bots`, `send_in_thread`
- **Features:** Multi-channel listening, mention-only or all-messages mode, threading
- **Requires:** Message Content Intent enabled in Discord Developer Portal
- **Setup:** `openfang channel setup discord`

#### Slack
- **Protocol:** Socket Mode (WebSocket) for private workspaces
- **Config:** `app_token` (xapp-), `bot_token` (xoxb-)
- **Features:** Threading (`thread_ts`), `send_in_thread` config
- **Setup:** `openfang channel setup slack`

#### WhatsApp
- **Two modes:**
  1. **Cloud API** — Official Meta/Twilio Cloud API with webhook
  2. **Baileys gateway** — QR-code web gateway (`packages/whatsapp-gateway/`)
- **Config:** `phone_number_id`, `access_token` (Cloud) or `gateway_url` (Baileys)
- **Features:** Images, voice, video, documents, stickers, sender identity in system prompt
- **QR pairing:** `GET /api/channels/whatsapp/qr` + poll `/api/channels/whatsapp/qr/poll`

#### Signal
- **Protocol:** `signal-cli` subprocess JSON-RPC
- **Config:** `phone_number`, `signal_cli_url`
- **Requires:** Running `signal-cli` daemon

#### Matrix
- **Protocol:** Client-Server API `/sync` polling
- **Config:** `homeserver_url`, `user_id`, `access_token`, `room_ids`

#### Email
- **Protocol:** IMAP (inbound polling) + SMTP (outbound replies)
- **Config:** IMAP host/port/credentials/TLS, SMTP host/port/credentials/TLS
- **Features:** Reply threading, HTML and plain text, attachments (planned)
- **Limit:** 20,000 chars per message

---

### Enterprise Platforms (8)

#### Microsoft Teams
- **Protocol:** Bot Framework REST API + webhook
- **Config:** `app_id`, `app_password`, `webhook_url`

#### Mattermost
- **Protocol:** WebSocket v4 + REST API v4
- **Config:** `server_url`, `bot_token`, `team_name`, `channel_name`
- **Limit:** 16,383 chars per message

#### Google Chat
- **Protocol:** Service account JWT + webhook
- **Config:** `service_account_json`, `room_id`
- **Auth:** Google service account with OAuth2 scopes

#### Webex (Cisco)
- **Protocol:** Mercury WebSocket + REST API
- **Config:** `bot_token`, `room_filter`

#### Feishu / Lark
- **Protocol:** Open Platform API
- **Config:** `app_id`, `app_secret`, `region` (`cn` or `intl`)

#### Rocket.Chat
- **Protocol:** REST API + long-polling history
- **Config:** `server_url`, `auth_token`, `user_id`

#### Zulip
- **Protocol:** REST API event queue polling
- **Config:** `server_url`, `bot_email`, `api_key`, `stream`, `topic`
- **Auth:** HTTP Basic
- **Limit:** 10,000 chars per message

#### DingTalk
- **Two modes:**
  1. **HTTP webhook** — Simple outbound + robot API
  2. **Stream mode** — WebSocket (Corp ID + agent secret)
- **Config (webhook):** `access_token`, `webhook_url`, `secret` (HMAC-SHA256)
- **Config (stream):** `corp_id`, `agent_secret`

---

### Social Platforms (8)

#### LINE
- **Protocol:** REST API + webhook
- **Config:** `channel_access_token`, `channel_secret`
- **Auth:** Bearer token

#### Viber
- **Protocol:** REST webhook + API
- **Config:** `account_access_token`, `webhook_url`

#### Facebook Messenger
- **Protocol:** Graph API v18 + webhook
- **Config:** `page_access_token`, `webhook_url`

#### Mastodon
- **Protocol:** REST API + SSE stream
- **Config:** `instance_url`, `access_token`

#### Bluesky
- **Protocol:** AT Protocol XRPC
- **Config:** `identifier` (handle), `app_password`
- **Auth:** App password (not account password)

#### Reddit
- **Protocol:** OAuth2 script app (password grant)
- **Config:** `client_id`, `client_secret`, `username`, `password`, `subreddit`

#### LinkedIn
- **Protocol:** REST API (OAuth2)
- **Config:** `access_token`, `organization_id`
- **Auth:** OAuth2

#### Twitch
- **Protocol:** IRC protocol over TLS
- **Config:** `oauth_token`, `channel_names`

---

### Community Platforms (6)

#### IRC
- **Protocol:** Raw TCP (plaintext)
- **Config:** `host`, `port`, `nick`, `channels`, `password`
- **Limit:** 510 chars per message

#### Guilded
- **Protocol:** WebSocket (Bonfire) + REST
- **Config:** `bot_token`, `channel_id`
- **Auth:** Bearer token

#### Revolt
- **Protocol:** REST API + WebSocket (Bonfire)
- **Config:** `bot_token`, `server_id`, `channel_ids`

#### Keybase
- **Protocol:** HTTP JSON-RPC (local Keybase daemon)
- **Config:** `username`, `paper_key`, `team`, `channel`

#### Discourse
- **Protocol:** REST API polling
- **Config:** `base_url`, `api_key`, `api_username`, `category_id`

#### Gitter
- **Protocol:** Streaming API (newline-delimited JSON)
- **Config:** `bearer_token`, `room_id`
- **Auth:** Bearer token

---

### Self-Hosted (1)

#### Nextcloud Talk
- **Protocol:** OCS REST API v2 polling
- **Config:** `server_url`, `token`, `room_id`

---

### Privacy-First (3)

#### Nostr
- **Protocol:** WebSocket relay (NIP-01/04)
- **Config:** `relay_urls`, `private_key`

#### Threema (Gateway)
- **Protocol:** HTTP webhook + REST API
- **Config:** `gateway_api_secret`, `webhook_url`
- **Auth:** HMAC-SHA256

#### Mumble
- **Protocol:** TCP protobuf framing
- **Config:** `server`, `port`, `username`
- **Note:** VoIP — text chat only (audio not implemented)

---

### Workplace Tools (4)

#### Pumble
- **Protocol:** Webhook + REST API
- **Config:** `bot_token`, `webhook_url`

#### Flock
- **Protocol:** Webhook + REST API
- **Config:** `bot_token`, `webhook_url`

#### Twist
- **Protocol:** REST API v3 polling
- **Config:** `oauth_token`, `workspace_id`, `thread_ids`
- **Auth:** OAuth2 Bearer

#### WeCom (WeChat Work)
- **Protocol:** Work API + webhook
- **Config:** `corp_id`, `agent_id`, `agent_secret`, `webhook_url`
- **Auth:** HMAC-SHA256

---

### Notification Services (2)

#### ntfy.sh
- **Protocol:** SSE (subscribe) + HTTP POST (publish)
- **Config:** `server_url`, `topic`, `auth_token`

#### Gotify
- **Protocol:** WebSocket + REST
- **Config:** `server_url`, `app_token`, `client_token`

---

### Integration (1)

#### Generic Webhook
- **Protocol:** HTTP POST with HMAC-SHA256 verification
- **Config:** `webhook_secret`, `callback_url`
- **Use case:** Any external system that can send HTTP webhooks

---

### IoT & Messaging (1)

#### MQTT
- **Protocol:** MQTT 3.1.1/5.0 pub/sub
- **Config:** `broker_url`, `subscribe_topic`, `publish_topic`, `username_env`, `password_env`, `use_tls`, `qos`
- **Features:** Plain text messages, JSON payloads (`{"text": "message"}`), command messages (`/` prefix), configurable QoS levels
- **Use case:** IoT device integration, home automation, industrial messaging, any MQTT broker
- **Limit:** 4,096 chars per message
- **Setup:** `openfang channel setup mqtt`

*Added in v0.5.4.*

---

### Not Yet Implemented

#### XMPP
- **Status:** Stub (awaiting `tokio-xmpp` maturity)
- **Config:** `jid`, `password`

---

## Message Limits by Platform

| Platform | Max chars |
|----------|-----------|
| IRC | 510 |
| Telegram | ~4,096 (auto-split) |
| Discord | 2,000 |
| Slack | 3,000 |
| Mattermost | 16,383 |
| Zulip | 10,000 |
| Email | 20,000 |
| WhatsApp | 4,096 |
| WeCom | 2,048 |
| MQTT | 4,096 |
| REST APIs | 4,000–32,000 |

OpenFang automatically splits messages that exceed platform limits.

---

## Building Custom Adapters

Implement the `ChannelAdapter` trait:

```rust
#[async_trait]
pub trait ChannelAdapter: Send + Sync {
    fn name(&self) -> &str;
    async fn start(&self, bridge: Arc<dyn ChannelBridgeHandle>) -> Result<()>;
    async fn stop(&self) -> Result<()>;
    async fn send(&self, message: &ChannelMessage) -> Result<()>;
    fn supported_formats(&self) -> Vec<OutputFormat>;
}
```

See [Contributing](contributing.md) for the full guide.
