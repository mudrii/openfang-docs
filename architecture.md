# Architecture

Validated against OpenFang release `v0.5.7`.

This page documents the released source layout rather than the marketing copy in the upstream README and website.

## Release Baseline

| Item | Value |
|------|-------|
| Workspace members | 14 |
| Crates under `crates/` | 13 |
| Bundled agent templates | 30 |
| Bundled Hands | 9 |
| Bundled skills | 61 |
| Channel registry entries | 42 |
| Security layers | 16 |

## Workspace Layout

OpenFang is a Cargo workspace with `13` crates under `crates/` plus `xtask`.

### Core crates

| Crate | Responsibility |
|-------|----------------|
| `openfang-types` | Shared types for agents, tools, events, config, taint labels, approvals, and model metadata |
| `openfang-memory` | SQLite-backed persistence, sessions, usage data, and knowledge structures |
| `openfang-runtime` | Agent loop, drivers, tool execution, sandboxing, MCP/A2A glue, web tooling, audit helpers |
| `openfang-kernel` | System assembly, agent registry, scheduler, workflow engine, routing, approvals, orchestration |
| `openfang-api` | HTTP, WebSocket, and SSE server plus dashboard-facing routes |
| `openfang-cli` | End-user CLI, TUI entrypoints, daemon control, setup flows |

### Integration and packaging crates

| Crate | Responsibility |
|-------|----------------|
| `openfang-channels` | Channel adapters and bridge infrastructure |
| `openfang-skills` | Bundled skills, installed skills, verification, marketplace support |
| `openfang-hands` | Bundled Hand definitions and hand registry/lifecycle |
| `openfang-extensions` | Integration templates and extension support |
| `openfang-wire` | OFP peer-to-peer networking |
| `openfang-migrate` | Migration helpers from other agent frameworks |
| `openfang-desktop` | Tauri desktop shell |
| `xtask` | Build and release automation |

## Runtime Topology

```text
CLI / TUI / Desktop
        |
        v
  openfang-api
        |
        v
  openfang-kernel
    |      |      |      |
    |      |      |      +-- openfang-wire
    |      |      +--------- openfang-channels
    |      +---------------- openfang-memory
    +----------------------- openfang-runtime
              |       |       |
              |       |       +-- openfang-skills
              |       +---------- openfang-hands
              +------------------ openfang-types
```

## Boot Sequence

At a high level, the released daemon path does the following:

1. reads `config.toml`
2. initializes data directories and SQLite state
3. builds the model catalog and provider status view
4. loads agents, skills, Hands, and configured integrations
5. starts schedulers, background tasks, and channel bridges
6. serves the API and dashboard

The CLI and desktop app both sit on top of the same core runtime rather than maintaining separate logic paths.

## Agent Execution Model

The released agent loop combines:

- prompt construction
- session loading and repair
- model invocation
- tool-call detection
- capability checks
- tool execution
- usage accounting
- loop-guard protection
- optional compaction when context pressure grows

Important release-era stability and safety subsystems include:

- loop guard
- session repair
- tool-output truncation
- SSRF filtering for web tools
- approval workflows for sensitive actions

## Channels

For `v0.5.7`, the most defensible count is the API's `CHANNEL_REGISTRY`, which contains `42` channel entries.

That count is higher than some upstream prose docs because the release registry includes distinct entries such as:

- `dingtalk`
- `dingtalk_stream`
- `wecom`
- `feishu`

This docs repo treats the API registry as the released source of truth for channel counts.

## Hands

The released `openfang-hands` crate bundles `9` Hands:

- `clip`
- `lead`
- `collector`
- `predictor`
- `researcher`
- `twitter`
- `browser`
- `trader`
- `infisical-sync`

Some upstream prose still says `7`; that is stale for `v0.5.7`.

## Skills

The released skill registry test in `crates/openfang-skills/src/bundled.rs` asserts `61` bundled skills. Upstream comments still mention `60`, so this docs repo uses the tested value.

## Providers and Models

Provider counts are easy to misstate because the runtime catalog mixes brands, protocol variants, coding-specific entries, and CLI-backed integrations.

Safe release facts:

- `ModelCatalog::list_providers()` has `41` entries in `v0.5.7`
- `builtin_models()` contains `197` builtin model entries
- the runtime uses three main driver families: Anthropic, Gemini, and OpenAI-compatible

For user-facing docs, this repo prefers explaining the catalog shape over collapsing it into an ambiguous headline number.

## Security

The release keeps `16` security layers in active documentation and release notes, including:

- capability enforcement
- WASM dual metering
- taint tracking
- Merkle audit trail
- manifest signing
- SSRF filtering
- secret zeroization
- subprocess sandboxing
- loop guard
- session repair
- Argon2id dashboard password hashing

See [security.md](security.md) for the detailed layer-by-layer writeup.
