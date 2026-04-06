# OpenFang Documentation

This repository tracks release-validated documentation for OpenFang.

Validated baseline:
- Release tag: `v0.5.7`
- Release date: `2026-03-31`
- Source repo: `https://github.com/RightNow-AI/openfang`

This pass intentionally documents released behavior only. The source repo's `main` branch is ahead of `v0.5.7`, but unreleased changes are excluded here.

## Verified Release Snapshot

These values were checked against the `v0.5.7` source tree and release metadata:

| Item | Released value | Verification basis |
|------|----------------|--------------------|
| Workspace members | 14 | `Cargo.toml` workspace members |
| Crates under `crates/` | 13 | `crates/` directory in release tree |
| Agent templates | 30 | `agents/*/agent.toml` in `v0.5.7` |
| Bundled Hands | 9 | `crates/openfang-hands/bundled/*/HAND.toml` |
| Bundled skills | 61 | `crates/openfang-skills/src/bundled.rs` test |
| Channel registry entries | 42 | `CHANNEL_REGISTRY` in `crates/openfang-api/src/routes.rs` |
| Provider entries | 41 | `ModelCatalog::list_providers()` test |
| Builtin model entries | 197 | `builtin_models()` entries in `model_catalog.rs` |
| Security layers | 16 | release notes and source security docs |

Provider and model counts deserve care: the runtime catalog includes brand variants, coding-specific entries, and CLI-backed providers. This docs repo uses the source-derived catalog counts above when precision matters, and avoids marketing-style grouped counts unless the grouping rule is stated.

## Documentation Index

| Document | Purpose |
|----------|---------|
| [review-plan-v0.5.7.md](review-plan-v0.5.7.md) | Review and rewrite plan used for this release-doc pass |
| [documentation_integrity_findings_v0.5.7.md](documentation_integrity_findings_v0.5.7.md) | Fact-check report, drift analysis, and source methodology |
| [coverage-matrix-v0.5.7.md](coverage-matrix-v0.5.7.md) | Source-doc coverage map showing what was captured and what was intentionally excluded |
| [installation.md](installation.md) | Release-accurate install paths and first-run notes |
| [getting-started.md](getting-started.md) | Minimal path from install to a running daemon |
| [architecture.md](architecture.md) | Source-derived subsystem overview for `v0.5.7` |
| [agents.md](agents.md) | Agent manifest shape, lifecycle, and runtime concepts |
| [hands.md](hands.md) | Bundled Hand catalog and lifecycle |
| [channels.md](channels.md) | Channel adapter overview and sourcing notes |
| [llm-providers.md](llm-providers.md) | Provider/driver/model catalog guidance with release-safe terminology |
| [sdk.md](sdk.md) | JavaScript and Python SDK notes |
| [cli-reference.md](cli-reference.md) | CLI command map derived from `openfang-cli` |
| [api-reference.md](api-reference.md) | API surface map derived from `openfang-api` |
| [configuration.md](configuration.md) | `config.toml` reference |
| [security.md](security.md) | Security architecture and hardening systems |
| [workflows.md](workflows.md) | Workflow engine concepts |
| [mcp-a2a.md](mcp-a2a.md) | MCP and A2A integration notes |
| [agent-templates.md](agent-templates.md) | Bundled template catalog |
| [skill-development.md](skill-development.md) | Skill authoring guide |
| [production-checklist.md](production-checklist.md) | Maintainer-facing release and deployment checklist |
| [contributing.md](contributing.md) | Contributor workflow |
| [changelog.md](changelog.md) | Curated release history |

## Scope Notes

- The GitHub release page for `v0.5.7` is the authoritative source for what was released.
- The public website and upstream prose docs contain conflicting version and count claims; they were used only as drift signals, not as the final source of truth.
- Where a count is unstable or ambiguous, this docs repo prefers a source-backed explanation over a simplified headline number.
