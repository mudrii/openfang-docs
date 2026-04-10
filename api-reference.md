# API Reference

Validated against OpenFang release `v0.5.7`.

Derived from `crates/openfang-api/src/server.rs` route registrations and `middleware.rs` auth allowlist.

Default local base URL:

```text
http://127.0.0.1:4200
```

---

## Authentication

OpenFang uses Bearer token authentication controlled by the `api_key` field in `config.toml`.

**When `api_key` is set (non-empty, non-whitespace):**

- All mutating requests (POST, PUT, PATCH, DELETE) require the header:
  ```
  Authorization: Bearer <api_key>
  ```
- Read-only GET requests to public endpoints (listed below) are allowed without auth.
- The `/api/shutdown` endpoint is restricted to loopback connections only (127.0.0.1) and bypasses token auth.

**When `api_key` is empty or unset:**

- All endpoints are open without authentication.

**Dashboard session auth:**

- When `[auth] enabled = true` is set in `config.toml`, dashboard login uses Argon2id password hashing.
- Session cookies are accepted as an alternative to Bearer tokens.
- Generate a password hash with: `openfang auth hash-password`

**Public endpoints (no auth required, GET only unless noted):**

| Endpoint | Notes |
|----------|-------|
| `/` | WebChat UI page |
| `/logo.png`, `/favicon.ico` | Static assets |
| `/api/health`, `/api/health/detail` | Any method |
| `/api/status`, `/api/version` | GET |
| `/api/agents` | GET only (listing) |
| `/api/profiles` | GET |
| `/api/config`, `/api/config/schema` | GET |
| `/api/uploads/{file_id}` | GET |
| `/api/models`, `/api/models/aliases` | GET |
| `/api/providers` | GET |
| `/api/budget`, `/api/budget/agents`, `/api/budget/agents/{id}` | GET |
| `/api/network/status` | GET |
| `/api/a2a/agents`, `/a2a/*` | GET |
| `/api/approvals`, `/api/approvals/*` | GET |
| `/api/channels` | GET |
| `/api/hands`, `/api/hands/active`, `/api/hands/*` | GET |
| `/api/skills` | GET |
| `/api/sessions` | GET |
| `/api/integrations`, `/api/integrations/available`, `/api/integrations/health` | GET |
| `/api/workflows` | GET |
| `/api/logs/stream` | SSE stream, any method |
| `/api/cron/*` | GET |
| `/api/providers/github-copilot/oauth/*` | Any method |
| `/api/auth/login`, `/api/auth/logout` | Any method |
| `/api/auth/check` | GET |
| `/.well-known/agent.json` | GET (A2A discovery) |

---

## Health & Status

### `GET /api/health`

Basic health check. Returns 200 when the daemon is running.

- **Auth:** Public
- **Response:** `{"status": "ok", "uptime_secs": 1234}`

### `GET /api/health/detail`

Detailed health check including agent count, database status, and uptime.

- **Auth:** Public
- **Response:** `{"status": "ok", "agent_count": 3, "uptime_secs": 1234, "database": "connected"}`

### `GET /api/status`

Full daemon status including default provider, model, agent list, and data directory.

- **Auth:** Public
- **Response:** `{"status": "running", "agent_count": 2, "default_provider": "groq", "default_model": "llama-3.3-70b-versatile", "uptime_seconds": 600, "data_dir": "...", "agents": [...]}`

### `GET /api/version`

Returns the daemon version.

- **Auth:** Public
- **Response:** `{"version": "0.5.7"}`

### `GET /api/metrics`

Prometheus-format metrics endpoint.

- **Auth:** Required
- **Response:** Prometheus text format

---

## Agents

### `GET /api/agents`

List all running agents.

- **Auth:** Public (GET)
- **Response:** Array of agent objects with `id`, `name`, `state`, `model_provider`, `model_name`

### `POST /api/agents`

Spawn a new agent from a manifest.

- **Auth:** Required
- **Request:** `{"manifest_toml": "<TOML content>"}`
- **Response:** `{"agent_id": "uuid", "name": "assistant", "model_provider": "groq", "model_name": "..."}`

### `GET /api/agents/{id}`

Get details for a specific agent.

- **Auth:** Required
- **Response:** Agent object

### `DELETE /api/agents/{id}`

Kill (terminate) an agent.

- **Auth:** Required
- **Response:** `{"status": "killed"}`

### `PATCH /api/agents/{id}`

Patch agent properties.

- **Auth:** Required
- **Request:** JSON with fields to update
- **Response:** Updated agent object

### `PUT /api/agents/{id}/update`

Full update of an agent.

- **Auth:** Required
- **Request:** JSON agent definition
- **Response:** Updated agent object

### `PUT /api/agents/{id}/mode`

Set the agent's operating mode.

- **Auth:** Required
- **Request:** `{"mode": "..."}`

### `POST /api/agents/{id}/restart`

Restart an agent. Also available as `POST /api/agents/{id}/start`.

- **Auth:** Required
- **Response:** `{"status": "restarted"}`

### `POST /api/agents/{id}/stop`

Stop an agent without killing it.

- **Auth:** Required
- **Response:** `{"status": "stopped"}`

### `PUT /api/agents/{id}/model`

Set the LLM model for an agent.

- **Auth:** Required
- **Request:** `{"model": "gpt-4o"}`
- **Response:** `{"status": "updated"}`

### `POST /api/agents/{id}/clone`

Clone an existing agent.

- **Auth:** Required
- **Response:** `{"agent_id": "new-uuid", "name": "...-clone"}`

### `PATCH /api/agents/{id}/identity`

Update the agent's identity (name, description, etc.).

- **Auth:** Required
- **Request:** JSON identity fields

### `PATCH /api/agents/{id}/config`

Patch agent-specific configuration.

- **Auth:** Required
- **Request:** JSON config fields

### `GET /api/agents/{id}/deliveries`

Get pending deliveries for an agent.

- **Auth:** Required

### `GET /api/profiles`

List available agent profiles/templates.

- **Auth:** Public

---

## Messaging

### `POST /api/agents/{id}/message`

Send a message to an agent and receive a response.

- **Auth:** Required
- **Request:** `{"message": "Hello, what can you do?"}`
- **Response:** `{"reply": "I can help with...", "agent_id": "..."}`

### `POST /api/agents/{id}/message/stream`

Send a message and receive a streaming SSE response.

- **Auth:** Required
- **Request:** `{"message": "Explain quantum computing"}`
- **Response:** Server-Sent Events stream

### `GET /api/agents/{id}/ws`

WebSocket endpoint for real-time bidirectional chat with an agent.

- **Auth:** Required (token passed as query parameter or header)
- **Protocol:** WebSocket
- **URL:** `ws://127.0.0.1:4200/api/agents/{id}/ws`

---

## Sessions

### `GET /api/agents/{id}/session`

Get the current active session for an agent.

- **Auth:** Required

### `GET /api/agents/{id}/sessions`

List all sessions for an agent.

- **Auth:** Required

### `POST /api/agents/{id}/sessions`

Create a new session for an agent.

- **Auth:** Required

### `POST /api/agents/{id}/sessions/{session_id}/switch`

Switch the agent to a different session.

- **Auth:** Required

### `POST /api/agents/{id}/session/reset`

Reset the current session (clear conversation history).

- **Auth:** Required

### `POST /api/agents/{id}/session/compact`

Compact the session (summarize and reduce token usage).

- **Auth:** Required

### `DELETE /api/agents/{id}/history`

Clear the agent's conversation history.

- **Auth:** Required

### `GET /api/sessions`

List all sessions across all agents.

- **Auth:** Public (GET)

### `DELETE /api/sessions/{id}`

Delete a specific session.

- **Auth:** Required

### `PUT /api/sessions/{id}/label`

Set a label on a session.

- **Auth:** Required
- **Request:** `{"label": "my-session"}`

### `GET /api/agents/{id}/sessions/by-label/{label}`

Find a session by label for a specific agent.

- **Auth:** Required

---

## Tools & Skills

### `GET /api/tools`

List all available tools across the system.

- **Auth:** Required

### `GET /api/agents/{id}/tools`

Get tools available to a specific agent.

- **Auth:** Required

### `PUT /api/agents/{id}/tools`

Set the tool list for an agent.

- **Auth:** Required
- **Request:** JSON array of tool definitions

### `GET /api/agents/{id}/skills`

Get skills assigned to a specific agent.

- **Auth:** Required

### `PUT /api/agents/{id}/skills`

Set skills for an agent.

- **Auth:** Required

### `GET /api/agents/{id}/mcp_servers`

Get MCP servers configured for an agent.

- **Auth:** Required

### `PUT /api/agents/{id}/mcp_servers`

Set MCP servers for an agent.

- **Auth:** Required

### `GET /api/skills`

List all installed skills.

- **Auth:** Public (GET)

### `POST /api/skills/install`

Install a skill.

- **Auth:** Required
- **Request:** `{"source": "skill-name-or-url"}`

### `POST /api/skills/uninstall`

Uninstall a skill.

- **Auth:** Required
- **Request:** `{"name": "skill-name"}`

### `POST /api/skills/reload`

Hot-reload the skill registry.

- **Auth:** Required

### `POST /api/skills/create`

Create a new skill scaffold.

- **Auth:** Required

### `GET /api/marketplace/search`

Search the FangHub marketplace for skills.

- **Auth:** Required
- **Query params:** `?q=search-term`

---

## Channels

### `GET /api/channels`

List all configured channels and their status.

- **Auth:** Public (GET)

### `POST /api/channels/{name}/configure`

Configure a channel (Telegram, Discord, Slack, etc.).

- **Auth:** Required
- **Request:** Channel-specific configuration JSON

### `DELETE /api/channels/{name}/configure`

Remove a channel's configuration.

- **Auth:** Required

### `POST /api/channels/{name}/test`

Send a test message through a channel.

- **Auth:** Required

### `POST /api/channels/reload`

Reload all channel bridges.

- **Auth:** Required

### `POST /api/channels/whatsapp/qr/start`

Start WhatsApp QR login flow.

- **Auth:** Required

### `GET /api/channels/whatsapp/qr/status`

Poll WhatsApp QR login status.

- **Auth:** Required

---

## Hands

### `GET /api/hands`

List all available hands.

- **Auth:** Public (GET)

### `GET /api/hands/active`

List currently active hand instances.

- **Auth:** Public (GET)

### `GET /api/hands/{hand_id}`

Get detailed info about a specific hand.

- **Auth:** Public (GET)

### `POST /api/hands/install`

Install a hand from HAND.toml content.

- **Auth:** Required
- **Request:** `{"toml_content": "...", "skill_content": "..."}`

### `POST /api/hands/upsert`

Create or update a hand definition.

- **Auth:** Required

### `POST /api/hands/{hand_id}/activate`

Activate a hand (spawns a dedicated agent).

- **Auth:** Required
- **Request:** `{"instance_name": "optional-name"}` (optional)
- **Response:** `{"instance_id": "uuid", "agent_name": "..."}`

### `POST /api/hands/{hand_id}/check-deps`

Check dependency status for a hand.

- **Auth:** Required

### `POST /api/hands/{hand_id}/install-deps`

Install missing dependencies for a hand.

- **Auth:** Required

### `GET /api/hands/{hand_id}/settings`

Get settings for a hand.

- **Auth:** Required

### `PUT /api/hands/{hand_id}/settings`

Update settings for a hand.

- **Auth:** Required

### `POST /api/hands/instances/{id}/pause`

Pause a running hand instance.

- **Auth:** Required

### `POST /api/hands/instances/{id}/resume`

Resume a paused hand instance.

- **Auth:** Required

### `DELETE /api/hands/instances/{id}`

Deactivate (stop) a hand instance.

- **Auth:** Required

### `GET /api/hands/instances/{id}/stats`

Get runtime statistics for a hand instance.

- **Auth:** Required

### `GET /api/hands/instances/{id}/browser`

Get browser state for a hand instance (browser-based hands).

- **Auth:** Required

---

## Workflows

### `GET /api/workflows`

List all registered workflows.

- **Auth:** Public (GET)

### `POST /api/workflows`

Create a new workflow.

- **Auth:** Required
- **Request:** JSON workflow definition with `name`, `steps`, etc.
- **Response:** `{"workflow_id": "uuid"}`

### `GET /api/workflows/{id}`

Get a workflow by ID.

- **Auth:** Required

### `PUT /api/workflows/{id}`

Update a workflow.

- **Auth:** Required
- **Request:** JSON workflow definition

### `DELETE /api/workflows/{id}`

Delete a workflow.

- **Auth:** Required

### `POST /api/workflows/{id}/run`

Execute a workflow.

- **Auth:** Required
- **Request:** `{"input": "text input for the workflow"}`
- **Response:** `{"run_id": "uuid", "output": "..."}`

### `GET /api/workflows/{id}/runs`

List runs for a workflow.

- **Auth:** Required

---

## Triggers

### `GET /api/triggers`

List all triggers. Supports `?agent_id=<uuid>` filter.

- **Auth:** Required

### `POST /api/triggers`

Create a new trigger.

- **Auth:** Required
- **Request:** `{"agent_id": "uuid", "pattern": {...}, "prompt_template": "Event: {{event}}", "max_fires": 0}`

### `PUT /api/triggers/{id}`

Update a trigger.

- **Auth:** Required

### `DELETE /api/triggers/{id}`

Delete a trigger.

- **Auth:** Required

---

## Schedules (Cron)

### `GET /api/schedules`

List all schedules.

- **Auth:** Required

### `POST /api/schedules`

Create a schedule.

- **Auth:** Required

### `PUT /api/schedules/{id}`

Update a schedule.

- **Auth:** Required

### `DELETE /api/schedules/{id}`

Delete a schedule.

- **Auth:** Required

### `POST /api/schedules/{id}/run`

Manually run a schedule.

- **Auth:** Required

### `GET /api/cron/jobs`

List all cron jobs.

- **Auth:** Public (GET)

### `POST /api/cron/jobs`

Create a cron job.

- **Auth:** Required
- **Request:** `{"agent_id": "name-or-uuid", "name": "job-name", "schedule": {"kind": "cron", "expr": "0 */6 * * *"}, "action": {"kind": "agent_turn", "message": "prompt text"}}`

### `DELETE /api/cron/jobs/{id}`

Delete a cron job.

- **Auth:** Required

### `PUT /api/cron/jobs/{id}/enable`

Enable or disable a cron job.

- **Auth:** Required

### `GET /api/cron/jobs/{id}/status`

Get status of a cron job.

- **Auth:** Public (GET)

### `POST /api/cron/jobs/{id}/run`

Manually trigger a cron job.

- **Auth:** Required

---

## Budget

### `GET /api/budget`

Get global budget status (spend, limits).

- **Auth:** Public (GET)

### `PUT /api/budget`

Update global budget settings.

- **Auth:** Required

### `GET /api/budget/agents`

Get per-agent cost ranking.

- **Auth:** Public (GET)

### `GET /api/budget/agents/{id}`

Get budget status for a specific agent.

- **Auth:** Public (GET)

### `PUT /api/budget/agents/{id}`

Set budget limits for a specific agent.

- **Auth:** Required

---

## Usage

### `GET /api/usage`

Get usage statistics.

- **Auth:** Required

### `GET /api/usage/summary`

Get aggregated usage summary.

- **Auth:** Required

### `GET /api/usage/by-model`

Get usage broken down by model.

- **Auth:** Required

### `GET /api/usage/daily`

Get daily usage statistics.

- **Auth:** Required

---

## Config

### `GET /api/config`

Get the current daemon configuration (sensitive fields redacted).

- **Auth:** Public (GET)

### `GET /api/config/schema`

Get the configuration schema.

- **Auth:** Public (GET)

### `POST /api/config/set`

Set a configuration value by key path.

- **Auth:** Required
- **Request:** `{"key": "default_model.model", "value": "gpt-4o"}`

### `POST /api/config/reload`

Reload configuration from disk.

- **Auth:** Required

---

## Auth (Dashboard)

### `POST /api/auth/login`

Log in to the dashboard.

- **Auth:** Public
- **Request:** `{"password": "..."}`
- **Response:** Sets session cookie

### `POST /api/auth/logout`

Log out of the dashboard.

- **Auth:** Public

### `GET /api/auth/check`

Check if the current session is authenticated.

- **Auth:** Public
- **Response:** `{"authenticated": true}` or `{"authenticated": false}`

---

## Models & Providers

### `GET /api/models`

List all available models in the catalog.

- **Auth:** Public (GET)
- **Query params:** `?provider=groq` (optional filter)

### `GET /api/models/aliases`

List model aliases (shorthand names).

- **Auth:** Public (GET)

### `GET /api/models/{id}`

Get a specific model by ID. Uses wildcard path matching.

- **Auth:** Required

### `POST /api/models/custom`

Add a custom model to the catalog.

- **Auth:** Required

### `DELETE /api/models/custom/{id}`

Remove a custom model from the catalog.

- **Auth:** Required

### `GET /api/providers`

List known LLM providers and their auth status.

- **Auth:** Public (GET)

### `POST /api/providers/{name}/key`

Save an API key for a provider.

- **Auth:** Required

### `DELETE /api/providers/{name}/key`

Delete the API key for a provider.

- **Auth:** Required

### `POST /api/providers/{name}/test`

Test connectivity with a provider's API key.

- **Auth:** Required

### `PUT /api/providers/{name}/url`

Set a custom base URL for a provider.

- **Auth:** Required

### `POST /api/providers/github-copilot/oauth/start`

Start GitHub Copilot OAuth flow.

- **Auth:** Public

### `GET /api/providers/github-copilot/oauth/poll/{poll_id}`

Poll for GitHub Copilot OAuth completion.

- **Auth:** Public

---

## OpenAI-Compatible API

### `POST /v1/chat/completions`

OpenAI-compatible chat completions endpoint. Supports streaming via SSE when `"stream": true`.

- **Auth:** Required
- **Request:** Standard OpenAI chat completions format
  ```json
  {
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "Hello"}],
    "stream": false
  }
  ```
- **Response:** OpenAI-compatible response format

### `GET /v1/models`

List models in OpenAI-compatible format.

- **Auth:** Required

---

## A2A (Agent-to-Agent Protocol)

### Inbound (this instance as A2A server)

### `GET /.well-known/agent.json`

A2A agent card discovery endpoint.

- **Auth:** Public

### `GET /a2a/agents`

List agents available via A2A protocol.

- **Auth:** Public (GET)

### `POST /a2a/tasks/send`

Send a task to a local agent via A2A.

- **Auth:** Required

### `GET /a2a/tasks/{id}`

Get the status of an A2A task.

- **Auth:** Public (GET)

### `POST /a2a/tasks/{id}/cancel`

Cancel a running A2A task.

- **Auth:** Required

### Outbound (managing external A2A agents)

### `GET /api/a2a/agents`

List discovered external A2A agents.

- **Auth:** Public (GET)

### `POST /api/a2a/discover`

Discover an external A2A agent at a given URL.

- **Auth:** Required
- **Request:** `{"url": "https://remote-agent/.well-known/agent.json"}`

### `POST /api/a2a/send`

Send a task to an external A2A agent.

- **Auth:** Required

### `GET /api/a2a/tasks/{id}/status`

Check the status of an outbound A2A task.

- **Auth:** Required

---

## MCP (Model Context Protocol)

### `POST /mcp`

MCP protocol over HTTP. Exposes this OpenFang instance as an MCP server.

- **Auth:** Required
- **Request:** MCP JSON-RPC messages

### `GET /api/mcp/servers`

List configured MCP servers and their connection status.

- **Auth:** Required

---

## Memory (Agent KV Store)

### `GET /api/memory/agents/{id}/kv`

List all key-value pairs for an agent.

- **Auth:** Required

### `GET /api/memory/agents/{id}/kv/{key}`

Get a specific value by key.

- **Auth:** Required

### `PUT /api/memory/agents/{id}/kv/{key}`

Set a key-value pair.

- **Auth:** Required
- **Request:** `{"value": "..."}`

### `DELETE /api/memory/agents/{id}/kv/{key}`

Delete a key-value pair.

- **Auth:** Required

---

## Agent Files & Uploads

### `GET /api/agents/{id}/files`

List files associated with an agent.

- **Auth:** Required

### `GET /api/agents/{id}/files/{filename}`

Get a specific agent file.

- **Auth:** Required

### `PUT /api/agents/{id}/files/{filename}`

Create or update an agent file.

- **Auth:** Required

### `POST /api/agents/{id}/upload`

Upload a file to an agent.

- **Auth:** Required
- **Request:** Multipart form data

### `GET /api/uploads/{file_id}`

Serve an uploaded file.

- **Auth:** Public (GET)

---

## Templates

### `GET /api/templates`

List available agent templates.

- **Auth:** Required

### `GET /api/templates/{name}`

Get a specific template by name.

- **Auth:** Required

---

## Approvals

### `GET /api/approvals`

List pending execution approvals.

- **Auth:** Public (GET)

### `POST /api/approvals`

Create an approval request.

- **Auth:** Required

### `POST /api/approvals/{id}/approve`

Approve a pending request.

- **Auth:** Required

### `POST /api/approvals/{id}/reject`

Reject a pending request.

- **Auth:** Required

---

## Security & Audit

### `GET /api/security`

Security status dashboard.

- **Auth:** Required

### `GET /api/audit/recent`

Get recent audit trail entries. Supports `?limit=20`.

- **Auth:** Required

### `GET /api/audit/verify`

Verify audit trail integrity (Merkle hash chain).

- **Auth:** Required
- **Response:** `{"valid": true}` or `{"valid": false, "error": "..."}`

---

## Integrations

### `GET /api/integrations`

List installed integrations.

- **Auth:** Public (GET)

### `GET /api/integrations/available`

List all available integration templates.

- **Auth:** Public (GET)

### `POST /api/integrations/add`

Install an integration.

- **Auth:** Required
- **Request:** `{"name": "github"}`

### `DELETE /api/integrations/{id}`

Remove an integration.

- **Auth:** Required

### `POST /api/integrations/{id}/reconnect`

Reconnect a failed integration.

- **Auth:** Required

### `GET /api/integrations/health`

Health status of all installed integrations.

- **Auth:** Public (GET)

### `POST /api/integrations/reload`

Hot-reload the integration registry.

- **Auth:** Required

---

## Peer Network

### `GET /api/peers`

List connected OFP peers.

- **Auth:** Required

### `GET /api/network/status`

Network connectivity status.

- **Auth:** Public (GET)

---

## Agent Communications (Comms)

### `GET /api/comms/topology`

Get the inter-agent communication topology.

- **Auth:** Required

### `GET /api/comms/events`

Get recent inter-agent communication events.

- **Auth:** Required

### `GET /api/comms/events/stream`

SSE stream of inter-agent communication events.

- **Auth:** Required

### `POST /api/comms/send`

Send a message between agents.

- **Auth:** Required

### `POST /api/comms/task`

Send a task between agents.

- **Auth:** Required

---

## Bindings

### `GET /api/bindings`

List agent bindings (channel-to-agent mappings).

- **Auth:** Required

### `POST /api/bindings`

Add an agent binding.

- **Auth:** Required

### `DELETE /api/bindings/{index}`

Remove a binding by index.

- **Auth:** Required

---

## Migration

### `GET /api/migrate/detect`

Detect available migration sources.

- **Auth:** Required

### `POST /api/migrate/scan`

Scan a source framework for migratable data.

- **Auth:** Required

### `POST /api/migrate`

Run a migration.

- **Auth:** Required

---

## Device Pairing

### `POST /api/pairing/request`

Start a device pairing flow.

- **Auth:** Required
- **Response:** `{"qr_data": "...", "pairing_code": "...", "expires_at": "..."}`

### `POST /api/pairing/complete`

Complete a pairing flow.

- **Auth:** Required

### `GET /api/pairing/devices`

List paired devices.

- **Auth:** Required

### `DELETE /api/pairing/devices/{id}`

Remove a paired device.

- **Auth:** Required

### `POST /api/pairing/notify`

Send a push notification to paired devices.

- **Auth:** Required

---

## ClawHub (Ecosystem)

### `GET /api/clawhub/search`

Search the ClawHub skill registry.

- **Auth:** Required
- **Query params:** `?q=search-term`

### `GET /api/clawhub/browse`

Browse ClawHub categories.

- **Auth:** Required

### `GET /api/clawhub/skill/{slug}`

Get skill details from ClawHub.

- **Auth:** Required

### `GET /api/clawhub/skill/{slug}/code`

Get skill source code from ClawHub.

- **Auth:** Required

### `POST /api/clawhub/install`

Install a skill from ClawHub.

- **Auth:** Required

---

## Webhooks

### `POST /hooks/wake`

External webhook endpoint for wake events.

- **Auth:** Required

### `POST /hooks/agent`

External webhook endpoint for agent events.

- **Auth:** Required

---

## Live Streaming (SSE)

### `GET /api/logs/stream`

Tail daemon logs via Server-Sent Events.

- **Auth:** Public

### `GET /api/comms/events/stream`

Inter-agent communication events via SSE.

- **Auth:** Required

### `POST /api/agents/{id}/message/stream`

Streaming chat response via SSE.

- **Auth:** Required

---

## System

### `POST /api/shutdown`

Gracefully shut down the daemon. Restricted to loopback connections only.

- **Auth:** Loopback only (no token required from 127.0.0.1)

### `GET /api/commands`

List available chat slash commands.

- **Auth:** Required

---

## Rate Limiting

All endpoints are subject to GCRA-based rate limiting. The rate limiter is applied as middleware before route handlers.

## CORS

CORS is configured based on the auth setting:

- **When `api_key` is empty:** CORS restricted to localhost origins and common dev ports (3000, 8080).
- **When `api_key` is set:** CORS restricted to localhost:4200, localhost:8080, and the configured listen address.

Both modes allow any HTTP method and any headers.
