# API Reference

The OpenFang daemon exposes a REST/WS/SSE API at `http://127.0.0.1:4200` by default.

## Authentication

**Loopback bypass:** Requests from `127.0.0.1` or `::1` bypass authentication by default.

**Bearer token (required when `require_auth = true`):**
```
Authorization: Bearer <api_key>
```

Set the API key in `config.toml`:
```toml
[network]
api_key = "your-secret-key"
```

---

## Health & Status

```
GET  /api/health              # Public: {"status": "ok"}
GET  /api/health              # Authenticated: full diagnostics
GET  /api/status              # System status, version, uptime
GET  /api/metrics             # Prometheus text-format metrics
```

---

## Agents

```
GET    /api/agents                    # List all agents
POST   /api/agents                    # Spawn a new agent
GET    /api/agents/{id}               # Get agent details
DELETE /api/agents/{id}               # Kill and remove agent
PATCH  /api/agents/{id}               # Update agent config
POST   /api/agents/{id}/restart       # Restart agent
POST   /api/agents/{id}/start         # Resume suspended agent
POST   /api/agents/{id}/stop          # Suspend agent
POST   /api/agents/{id}/clone         # Clone agent
```

### Spawn Agent

```bash
curl -X POST http://127.0.0.1:4200/api/agents \
  -H "Content-Type: application/json" \
  -d '{
    "name": "my-assistant",
    "template": "assistant",
    "model": "claude-sonnet-4-6",
    "system_prompt": "You are a helpful assistant."
  }'
```

---

## Chat & Messages

```
POST /api/agents/{id}/message         # Send message (returns full response)
POST /api/agents/{id}/message/stream  # Send message (SSE streaming)
WS   /api/agents/{id}/ws              # WebSocket real-time chat
```

### Send Message

```bash
curl -X POST http://127.0.0.1:4200/api/agents/{id}/message \
  -H "Content-Type: application/json" \
  -d '{"message": "What is the capital of France?"}'
```

### Streaming (SSE)

```bash
curl -N http://127.0.0.1:4200/api/agents/{id}/message/stream \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"message": "Explain quantum computing", "stream": true}'
```

SSE event types: `text`, `tool_call`, `tool_result`, `done`, `error`, `token_usage`

---

## Sessions

```
GET    /api/agents/{id}/sessions            # List sessions
POST   /api/agents/{id}/sessions            # Create named session
POST   /api/agents/{id}/sessions/{sid}/switch  # Switch active session
POST   /api/agents/{id}/sessions/{sid}/reset   # Clear session history
POST   /api/agents/{id}/sessions/{sid}/compact # Force compaction
GET    /api/agents/{id}/sessions/{sid}/history # Get message history
DELETE /api/agents/{id}/sessions/{sid}         # Delete session
```

---

## Tools

```
GET /api/tools                        # List all 53 built-in tools
GET /api/agents/{id}/tools            # Get agent's tool configuration
PUT /api/agents/{id}/tools            # Update agent's allowed/blocked tools
```

---

## Skills

```
GET    /api/skills                    # List all installed + bundled skills
POST   /api/skills/install            # Install skill (slug, URL, or path)
DELETE /api/skills/{name}             # Remove skill
GET    /api/skills/{name}/readme      # Get skill documentation
GET    /api/skills/{name}/code        # Get skill source code
GET    /api/agents/{id}/skills        # Get agent's active skills
PUT    /api/agents/{id}/skills        # Update agent's skill set
```

```
GET  /api/skills/marketplace/search?q=kubernetes     # Search FangHub
GET  /api/skills/marketplace/browse?category=devops  # Browse by category
```

---

## Memory

```
GET    /api/agents/{id}/memory/kv              # List KV memory keys
GET    /api/agents/{id}/memory/kv/{key}        # Get value
PUT    /api/agents/{id}/memory/kv/{key}        # Set value
DELETE /api/agents/{id}/memory/kv/{key}        # Delete key
POST   /api/agents/{id}/memory/recall          # Semantic recall (vector search)
POST   /api/agents/{id}/memory/store           # Store semantic memory
GET    /api/agents/{id}/memory/entities        # List knowledge graph entities
GET    /api/agents/{id}/memory/relations       # List knowledge graph relations
```

```bash
# Semantic recall
curl -X POST http://127.0.0.1:4200/api/agents/{id}/memory/recall \
  -d '{"query": "Python async patterns", "limit": 5}'
```

---

## Schedules (Cron)

```
GET    /api/cron/jobs             # List all scheduled jobs
POST   /api/cron/jobs             # Create a cron job
GET    /api/cron/jobs/{id}        # Get job details
PUT    /api/cron/jobs/{id}        # Update job
DELETE /api/cron/jobs/{id}        # Delete job
POST   /api/cron/jobs/{id}/run    # Run job immediately
GET    /api/cron/jobs/{id}/history # Get execution history
```

```bash
# Create a daily summary job
curl -X POST http://127.0.0.1:4200/api/cron/jobs \
  -d '{
    "agent_id": "<agent-id>",
    "name": "daily-briefing",
    "schedule": "0 9 * * *",
    "message": "Summarize today'\''s news and my tasks"
  }'
```

**Schedule formats:**
- `"0 9 * * *"` — 5-field cron (daily at 09:00 UTC)
- `{"every": 3600}` — Every N seconds (60–86400)
- `{"at": "2026-12-31T23:59:00Z"}` — Specific UTC time

---

## Workflows

```
GET    /api/workflows             # List workflows
POST   /api/workflows             # Create workflow
GET    /api/workflows/{id}        # Get workflow
PATCH  /api/workflows/{id}        # Update workflow
DELETE /api/workflows/{id}        # Delete workflow
POST   /api/workflows/{id}/run    # Execute workflow
GET    /api/workflows/{id}/runs   # List run history
GET    /api/workflows/{id}/runs/{run_id}  # Get specific run
```

---

## Triggers

```
GET    /api/events/triggers          # List event triggers
POST   /api/events/triggers          # Create trigger
PUT    /api/events/triggers/{id}     # Update trigger
DELETE /api/events/triggers/{id}     # Delete trigger
```

---

## Channels

```
GET  /api/channels                   # List all channels + config schemas
POST /api/channels/{name}/config     # Save channel configuration
POST /api/channels/{name}/test       # Test channel credentials
POST /api/channels/{name}/enable     # Enable channel
POST /api/channels/{name}/disable    # Disable channel
POST /api/channels/{name}/reload     # Reload channel (hot-reload config)
GET  /api/channels/{name}/schema     # Get channel config JSON schema

# WhatsApp QR pairing
GET  /api/channels/whatsapp/qr       # Get QR code
GET  /api/channels/whatsapp/qr/poll  # Poll QR code status
```

---

## Budget & Usage

```
GET /api/budget                      # Current budget status
PUT /api/budget                      # Update budget limits
GET /api/usage                       # Per-agent usage ranking
GET /api/usage/summary               # Overall token and cost summary
GET /api/usage/by-model              # Per-model breakdown
GET /api/usage/daily                 # Daily cost timeline + projection
```

---

## Providers & Models

```
GET  /api/providers                  # List all configured providers
POST /api/providers/{name}/key       # Set API key for provider
POST /api/providers/{name}/test      # Test provider connectivity
GET  /api/models                     # List all available models
POST /api/models/custom              # Add custom model
DELETE /api/models/custom/{id}       # Remove custom model
```

---

## Configuration

```
GET  /api/config                     # Get current config
GET  /api/config/schema              # Get config JSON schema (public)
POST /api/config                     # Update config (triggers hot-reload)
POST /api/config/reload              # Force config reload
```

---

## Approvals

```
GET  /api/approvals                  # List pending approvals
POST /api/approvals                  # Create approval request
POST /api/approvals/{id}/approve     # Approve action
POST /api/approvals/{id}/reject      # Reject action
```

---

## Network (OFP)

```
GET  /api/peers                      # List known peers
GET  /api/network/status             # Network connectivity status
```

---

## Agent-to-Agent (A2A)

```
GET  /api/a2a/agent-card             # Agent discovery card
POST /api/a2a/tasks/send             # Send task to agent
GET  /api/a2a/tasks/{id}             # Get task status
POST /api/a2a/tasks/{id}/cancel      # Cancel task
```

---

## Communications

```
GET  /api/comms/topology             # Agent relationship graph
GET  /api/comms/events               # Recent inter-agent events
POST /api/comms/send                 # Send agent-to-agent message
POST /api/comms/task                 # Post task to another agent
```

---

## Security & Audit

```
GET /api/audit/verify                # Verify Merkle audit chain integrity
GET /api/security/info               # List security features (public)
```

---

## MCP

```
POST /api/mcp                        # MCP JSON-RPC endpoint
GET  /api/mcp/servers                # List connected MCP servers
```

---

## Webhooks (External Event Injection)

```
POST /hooks/wake                     # Inject a system event
POST /hooks/agent                    # Send a message to an agent
```

```bash
# Wake a specific agent with a message
curl -X POST http://127.0.0.1:4200/hooks/agent \
  -H "Content-Type: application/json" \
  -d '{
    "message": "New order received: #12345",
    "agent": "order-processor",
    "deliver": "now"
  }'

# Fire a system event
curl -X POST http://127.0.0.1:4200/hooks/wake \
  -d '{
    "text": "deployment-complete",
    "mode": "now"
  }'
```

**Webhook payload validation:** `text` max 4096 chars, `message` max 16384 chars. Control characters rejected (except newline).

---

## Auth

```
POST /api/auth/login                 # Login and get session token
POST /api/auth/logout                # Invalidate session
GET  /api/auth/check                 # Check current auth status
```

---

## OpenAI-Compatible

```
POST /v1/chat/completions            # OpenAI-compatible chat completion
GET  /v1/models                      # OpenAI-compatible model list
```

---

## Web Dashboard

```
GET  /                               # Web dashboard (Alpine.js SPA)
GET  /manifest.json                  # PWA manifest
GET  /sw.js                          # Service worker
```

---

## Response Formats

### Success

```json
{"success": true, "data": {...}}
```

### Error

```json
{
  "error": "AgentNotFound",
  "message": "Agent 'my-agent' does not exist",
  "code": 404
}
```

### Streaming SSE

```
data: {"type": "text", "delta": "Hello "}
data: {"type": "text", "delta": "world!"}
data: {"type": "token_usage", "input": 12, "output": 4}
data: {"type": "done"}
```
