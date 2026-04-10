# WhatsApp Web Gateway (QR Code)

Validated against OpenFang release `v0.5.7`.

Connect your personal WhatsApp account to OpenFang via QR code -- the same way WhatsApp Web works. No Meta Business account required.

The gateway is a standalone Node.js process (`packages/whatsapp-gateway/`) that uses the [Baileys](https://github.com/WhiskeySockets/Baileys) library to maintain a WhatsApp Web session. It bridges messages bidirectionally between WhatsApp and the OpenFang API.

---

## How It Works

```
WhatsApp (phone) <--WA Web Protocol--> Gateway (Baileys) <--HTTP--> OpenFang API
                                           :3009                      :4200
```

1. The gateway connects to WhatsApp's servers using the Baileys library
2. On first run, a QR code is generated for phone pairing
3. Incoming messages are forwarded to `POST /api/agents/{agent}/message` on OpenFang
4. The agent's response is sent back to the WhatsApp conversation via Baileys
5. Outgoing messages from OpenFang are sent via `POST /message/send` on the gateway

---

## Prerequisites

- **Node.js >= 18** (check with `node --version`)
- **OpenFang** running and accessible at `http://127.0.0.1:4200` (or custom URL)
- A WhatsApp account on your phone

---

## Setup

### 1. Install dependencies

```bash
cd packages/whatsapp-gateway
npm install
```

This installs:
- `@whiskeysockets/baileys` (v6) -- WhatsApp Web protocol
- `qrcode` -- QR code generation
- `pino` -- Logging

### 2. Configure `config.toml`

Add the WhatsApp channel to your OpenFang configuration:

```toml
[channels.whatsapp]
gateway_url_env = "WHATSAPP_WEB_GATEWAY_URL"
default_agent = "assistant"
```

The `gateway_url_env` field tells OpenFang which environment variable holds the gateway URL. When this env var is set, the adapter routes outgoing messages through the gateway instead of the Cloud API.

### 3. Set environment variables

Add to your shell profile (`~/.zshrc`) for persistence:

```bash
export WHATSAPP_WEB_GATEWAY_URL="http://127.0.0.1:3009"
```

Then reload:

```bash
source ~/.zshrc
```

Or set inline when starting:

```bash
export WHATSAPP_WEB_GATEWAY_URL="http://127.0.0.1:3009"
```

### 4. Start the gateway

```bash
node packages/whatsapp-gateway/index.js
```

The gateway starts an HTTP server on `127.0.0.1:3009` and logs:

```
[gateway] WhatsApp Web gateway listening on http://127.0.0.1:3009
[gateway] OpenFang URL: http://127.0.0.1:4200
[gateway] Default agent: assistant
```

If credentials from a previous session exist (`auth_store/creds.json`), the gateway auto-connects without requiring a new QR scan.

### 5. Start OpenFang

```bash
openfang start
```

### 6. Scan the QR code

Open the OpenFang dashboard and navigate to **Channels** > **WhatsApp**. A QR code will appear. Scan it with your phone:

> **WhatsApp** > **Settings** > **Linked Devices** > **Link a Device**

Once scanned, the gateway status changes to `connected` and incoming messages are routed to your configured agent.

---

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `WHATSAPP_WEB_GATEWAY_URL` | Gateway URL for OpenFang to connect to | *(empty = gateway disabled)* |
| `WHATSAPP_GATEWAY_PORT` | Port the gateway listens on | `3009` |
| `OPENFANG_URL` | OpenFang API URL the gateway forwards messages to | `http://127.0.0.1:4200` |
| `OPENFANG_DEFAULT_AGENT` | Agent name that handles incoming messages | `assistant` |

---

## Gateway API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/login/start` | Start Baileys connection, returns QR code as `data:image/png;base64,...` |
| `GET` | `/login/status` | Poll connection status (`connected`, `message`, `expired`) |
| `POST` | `/message/send` | Send outgoing message: `{ "to": "+5511999999999", "text": "Hello" }` |
| `GET` | `/health` | Health check (returns `status`, `connected`, `session_id`) |

### POST /login/start

Starts a new Baileys connection. If already connected, returns success immediately.

Response:
```json
{
  "qr_data_url": "data:image/png;base64,...",
  "session_id": "uuid",
  "message": "Scan this QR code with WhatsApp -> Linked Devices",
  "connected": false
}
```

The endpoint waits up to 15 seconds for the QR code to generate before responding.

### GET /login/status

Poll this endpoint to check if the QR code was scanned.

Response:
```json
{
  "connected": true,
  "message": "Connected to WhatsApp",
  "expired": false
}
```

### POST /message/send

Send a message through WhatsApp. The `to` field accepts:
- Phone numbers: `"+5511999999999"` (converted to `5511999999999@s.whatsapp.net`)
- Full JIDs: `"5511999999999@s.whatsapp.net"` (user) or `"120363...@g.us"` (group)

Request:
```json
{
  "to": "+5511999999999",
  "text": "Hello from OpenFang"
}
```

### GET /health

Returns the current gateway health status.

Response:
```json
{
  "status": "ok",
  "connected": true,
  "session_id": "uuid"
}
```

---

## Message Flow

### Incoming (WhatsApp to Agent)

1. User sends a WhatsApp message
2. Baileys receives via `messages.upsert` event
3. Gateway extracts text, sender info, and group context
4. Forwards to OpenFang: `POST {OPENFANG_URL}/api/agents/{DEFAULT_AGENT}/message`
5. Agent processes and returns a response
6. Gateway sends the response back to the same WhatsApp conversation

The gateway handles:
- **DMs:** Reply goes to the sender's JID
- **Groups:** Reply goes to the group JID
- **Media messages:** Images, voice notes, videos, documents, and stickers are described as text placeholders (e.g., `[Image received]`, `[Voice note received]`)
- **Self-messages and status broadcasts:** Automatically filtered out

### Outgoing (Agent to WhatsApp)

When OpenFang needs to send a proactive message, the WhatsApp adapter calls:

```
POST {WHATSAPP_WEB_GATEWAY_URL}/message/send
{ "to": "+5511999999999", "text": "..." }
```

Long messages are automatically split at the 4,096 character limit before being sent.

---

## Session Persistence

The gateway stores WhatsApp session credentials in `packages/whatsapp-gateway/auth_store/`. On restart:

- **If `auth_store/creds.json` exists:** Auto-connects without requiring a new QR scan
- **If logged out from phone:** Auth store is cleared automatically and a new QR code is needed

---

## Reconnection

The gateway handles disconnections with exponential backoff:

- **Recoverable disconnects** (timeout, connection lost, restart required): Automatically reconnects with backoff up to 60 seconds
- **Logged out** (`DisconnectReason.loggedOut`): Clears auth store, stops reconnection. Use `POST /login/start` to begin a new QR flow.

---

## Troubleshooting

### QR code not appearing

- Verify the gateway is running: `curl http://127.0.0.1:3009/health`
- Check that port 3009 is not in use by another process
- Try `POST http://127.0.0.1:3009/login/start` directly to trigger QR generation

### "WhatsApp not connected" when sending

- Check gateway status: `curl http://127.0.0.1:3009/login/status`
- If `connected: false`, the session may have expired. Scan a new QR code.
- Ensure `WHATSAPP_WEB_GATEWAY_URL` is set in the environment where OpenFang runs

### Messages not reaching the agent

- Verify OpenFang is running: `curl http://127.0.0.1:4200/api/health`
- Check that `OPENFANG_URL` points to the correct OpenFang instance
- Verify the agent specified in `OPENFANG_DEFAULT_AGENT` exists: `curl http://127.0.0.1:4200/api/agents`
- Check gateway logs for `Forward/reply failed` errors

### Session keeps disconnecting

- WhatsApp allows a limited number of linked devices. Remove old linked devices from your phone.
- Ensure your phone has a stable internet connection (WhatsApp Web requires the phone to be online)
- Check for `409 Conflict` errors indicating another instance is running

### Port conflicts

Override the gateway port:

```bash
WHATSAPP_GATEWAY_PORT=3010 node packages/whatsapp-gateway/index.js
```

Update the gateway URL accordingly:

```bash
export WHATSAPP_WEB_GATEWAY_URL="http://127.0.0.1:3010"
```

### Clearing session state

To force a fresh QR code pairing:

```bash
rm -rf packages/whatsapp-gateway/auth_store
```

Then restart the gateway and scan a new QR code.

---

## Alternative: WhatsApp Cloud API

For production workloads, use the official [WhatsApp Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api) with a Meta Business account. This provides:

- Verified business profiles
- Higher message throughput
- Rich media support (images, documents, locations) without gateway limitations
- No dependency on a paired phone

Configure Cloud API mode in `config.toml`:

```toml
[channels.whatsapp]
phone_number_id = "123456789012345"
access_token_env = "WHATSAPP_ACCESS_TOKEN"
verify_token_env = "WHATSAPP_VERIFY_TOKEN"
webhook_port = 8443
```

When `WHATSAPP_WEB_GATEWAY_URL` is **not** set, the adapter defaults to Cloud API mode.
