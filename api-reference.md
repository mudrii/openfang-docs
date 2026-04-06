# API Reference

Validated against OpenFang release `v0.5.7`.

This page is a release-accurate API surface map derived from `crates/openfang-api/src/routes.rs`, `openai_compat.rs`, `ws.rs`, and the middleware path allowlist.

Default local base URL:

```text
http://127.0.0.1:4200
```

## Authentication

OpenFang exposes both general API auth and dashboard auth concepts.

High-signal release facts:

- the config model contains a top-level `api_key`
- the dashboard/auth surface includes `POST /api/auth/login`, `POST /api/auth/logout`, and `GET /api/auth/check`
- the API middleware whitelists a small set of read-only routes

For deployment-specific auth behavior, validate your actual `config.toml` and middleware behavior together.

## Major endpoint families

| Family | Example routes |
|--------|----------------|
| Agents | `/api/agents`, `/api/agents/{id}`, `/api/agents/{id}/stop`, `/api/agents/{id}/clone` |
| Sessions | `/api/sessions`, `/api/agents/{id}/sessions`, `/api/agents/{id}/session/compact` |
| Workflows | `/api/workflows`, `/api/workflows/{id}`, `/api/workflows/{id}/runs` |
| Triggers | `/api/triggers`, `/api/triggers/{id}` |
| Memory | `/api/memory/agents/{id}/kv`, `/api/memory/agents/{id}/kv/{key}` |
| Channels | `/api/channels`, `/api/channels/{name}/configure`, `/api/channels/{name}/test` |
| Hands | `/api/hands`, `/api/hands/active`, `/api/hands/{hand_id}/activate` |
| Skills and marketplace | `/api/skills`, `/api/skills/install`, `/api/marketplace/search`, `/api/clawhub/*` |
| Models and providers | `/api/models`, `/api/models/aliases`, `/api/providers`, `/api/providers/{name}/key` |
| MCP and integrations | `/api/mcp/servers`, `/api/integrations`, `/api/integrations/available` |
| Audit and security | `/api/audit/recent`, `/api/audit/verify`, `/api/security` |
| Usage and budget | `/api/usage`, `/api/usage/summary`, `/api/budget`, `/api/budget/agents/{id}` |
| Scheduling | `/api/schedules`, `/api/cron/jobs`, `/api/cron/jobs/{id}/run` |
| Files and uploads | `/api/agents/{id}/files`, `/api/agents/{id}/upload`, `/api/uploads/{file_id}` |
| Approvals | `/api/approvals`, `/api/approvals/{id}/approve`, `/api/approvals/{id}/reject` |
| Pairing and devices | `/api/pairing/request`, `/api/pairing/devices`, `/api/pairing/notify` |
| Comms and topology | `/api/comms/topology`, `/api/comms/events`, `/api/comms/send` |
| Health and status | `/api/health`, `/api/health/detail`, `/api/status`, `/api/version`, `/api/metrics` |

## Streaming and realtime surfaces

The release includes:

| Surface | Route |
|---------|-------|
| Agent chat WebSocket | `/api/agents/{id}/ws` |
| Audit log SSE | `/api/logs/stream` |
| Inter-agent comms SSE | `/api/comms/events/stream` |
| OpenAI-compatible SSE | `/v1/chat/completions` with streaming |

## OpenAI-compatible API

Release routes:

```text
POST /v1/chat/completions
GET  /v1/models
```

These are implemented in `openai_compat.rs`.

## Model and provider surfaces

Useful catalog routes:

```text
GET    /api/models
GET    /api/models/aliases
GET    /api/models/{id}
GET    /api/providers
POST   /api/providers/{name}/key
DELETE /api/providers/{name}/key
POST   /api/providers/{name}/test
PUT    /api/providers/{name}/url
```

## Hands and skills

Representative release routes:

```text
GET  /api/hands
GET  /api/hands/active
POST /api/hands/{hand_id}/activate
POST /api/hands/{hand_id}/check-deps
POST /api/hands/{hand_id}/install-deps

GET  /api/skills
POST /api/skills/install
POST /api/skills/uninstall
POST /api/skills/reload
GET  /api/marketplace/search
GET  /api/clawhub/search
```

## System surfaces

Representative release routes:

```text
GET  /api/health
GET  /api/health/detail
GET  /api/status
GET  /api/version
GET  /api/metrics
GET  /api/config
GET  /api/config/schema
POST /api/config/reload
POST /api/config/set
```

## Notes

- Earlier docs in this repo contained several route names that do not match `v0.5.7`; those were removed.
- The upstream source docs remain useful for deeper examples, but this file prefers a source-derived route map over a potentially stale hand-written narrative spec.
