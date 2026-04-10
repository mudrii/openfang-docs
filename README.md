# OpenFang Documentation

Documentation for [OpenFang](https://github.com/RightNow-AI/openfang), an open-source Agent Operating System written in Rust.

**Current version: v0.5.7** (released April 8, 2026)

## At a Glance

| Metric | Value |
|--------|-------|
| Workspace crates | 14 |
| Source files | 253 |
| Lines of code | ~178K |
| Built-in tools | 59 |
| Channel adapters | 43 |
| Bundled Hands | 9 |
| Bundled skills | 61 across 14 categories |
| LLM providers | 27, with 3 native drivers |
| Tests | 2,077+ |
| Clippy warnings | 0 |
| Binary size | ~32 MB (single static binary) |
| License | Apache-2.0 OR MIT |

## Getting Started

| Document | Description |
|----------|-------------|
| [getting-started.md](getting-started.md) | Install, initialize, start the daemon, chat with your first agent |
| [installation.md](installation.md) | Platform-specific install paths, source builds, release binaries |

## Core Concepts

| Document | Description |
|----------|-------------|
| [architecture.md](architecture.md) | 14-crate workspace layout, dependency graph, key patterns |
| [agents.md](agents.md) | Agent manifest format, lifecycle, execution model |
| [hands.md](hands.md) | Bundled Hands: curated autonomous capability packages |
| [skills.md](skills.md) | Skill system overview and bundled skill catalog |
| [workflows.md](workflows.md) | Workflow engine: multi-step agent orchestration |
| [channels.md](channels.md) | 43 channel adapters for messaging platforms |
| [llm-providers.md](llm-providers.md) | Provider catalog, driver architecture, model routing |
| [security.md](security.md) | Capability-based security, taint tracking, WASM sandboxing, audit trail |

## Integration

| Document | Description |
|----------|-------------|
| [mcp-a2a.md](mcp-a2a.md) | MCP server connections and A2A protocol support |
| [sdk.md](sdk.md) | JavaScript and Python SDK usage |
| [agent-templates.md](agent-templates.md) | Bundled agent template catalog |
| [skill-development.md](skill-development.md) | Authoring custom skills (TOML + Python/WASM/Node/Shell/PromptOnly) |
| [configuration.md](configuration.md) | `config.toml` reference and hot-reload behavior |
| [desktop.md](desktop.md) | Tauri 2.0 native desktop application |
| [migration.md](migration.md) | Importing agents from OpenClaw and other frameworks |

## Reference

| Document | Description |
|----------|-------------|
| [cli-reference.md](cli-reference.md) | CLI command map derived from `openfang-cli` |
| [api-reference.md](api-reference.md) | REST, WebSocket, and SSE API surface |
| [changelog.md](changelog.md) | Release history |
| [troubleshooting.md](troubleshooting.md) | Common issues and solutions |

## Contributor Docs

| Document | Description |
|----------|-------------|
| [contributing.md](contributing.md) | Contributor workflow and code conventions |
| [production-checklist.md](production-checklist.md) | Release and deployment checklist |

## Audit and Integrity

| Document | Description |
|----------|-------------|
| [review-plan-v0.5.7.md](review-plan-v0.5.7.md) | Review plan used for this documentation pass |
| [documentation_integrity_findings_v0.5.7.md](documentation_integrity_findings_v0.5.7.md) | Fact-check report and drift analysis |
| [coverage-matrix-v0.5.7.md](coverage-matrix-v0.5.7.md) | Source-to-doc coverage map |
