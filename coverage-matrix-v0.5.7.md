# Coverage Matrix — OpenFang `v0.5.7`

This file maps the main released source areas to the documents in this repo that cover them.

## Coverage map

| Source area | Primary evidence | Docs produced |
|-------------|------------------|---------------|
| Release metadata | GitHub release `v0.5.7`, local tag, latest release page | `README.md`, `changelog.md`, `documentation_integrity_findings_v0.5.7.md` |
| Workspace layout | `Cargo.toml`, `crates/` tree | `architecture.md`, `contributing.md` |
| Agent templates | `agents/*/agent.toml` | `agents.md`, `agent-templates.md`, `getting-started.md` |
| Hands | `crates/openfang-hands/src/bundled.rs` | `hands.md`, `architecture.md` |
| Skills | `crates/openfang-skills/src/bundled.rs`, skill docs | `skill-development.md`, `skills.md`, `architecture.md` |
| CLI surface | `crates/openfang-cli/src/main.rs` | `cli-reference.md`, `getting-started.md`, `installation.md` |
| API surface | `crates/openfang-api/src/routes.rs`, `server.rs`, `openai_compat.rs`, `ws.rs` | `api-reference.md`, `configuration.md`, `channels.md` |
| Configuration model | `crates/openfang-types/src/config.rs`, `openfang.toml.example`, CLI-generated starter config | `configuration.md`, `troubleshooting.md` |
| Security model | release security docs, `SECURITY.md`, auth changes in `v0.5.7` | `security.md`, `troubleshooting.md`, `changelog.md` |
| Channels | `crates/openfang-api/src/routes.rs`, `crates/openfang-channels/src`, channel config structs | `channels.md`, `configuration.md` |
| Providers and models | `crates/openfang-runtime/src/model_catalog.rs` | `llm-providers.md`, `architecture.md` |
| SDKs | `sdk/javascript`, `sdk/python` | `sdk.md` |
| MCP and A2A | released docs and API route surfaces | `mcp-a2a.md`, `api-reference.md` |
| Workflows | released docs and kernel/API surfaces | `workflows.md`, `api-reference.md` |

## Intentionally conservative areas

These areas were kept cautious because the release tree contains internal drift or ambiguous counting rules:

- headline counts for providers and models
- headline counts for channels beyond API-registry vs implementation-level distinctions
- default port claims when the type default and CLI-generated config differ
- deep route-by-route endpoint statistics

## Exclusions

Not every source file was transcribed line-by-line into this docs repo. The intent was release-accurate public documentation, not a mirror of every upstream Markdown file or source comment.

Explicit exclusions for this pass:

- `docs/launch-roadmap.md`: roadmap/planning, not released product behavior
- `docs/VERTEX_AI_LOCAL_TESTING.md`: specialized local test guidance, not core end-user documentation
- `docs/benchmarks/*.svg`: reference assets rather than standalone narrative docs
- internal source comments that conflict with code-backed tests, such as the stale `60`-skill comment in `openfang-skills`
