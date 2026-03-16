# OpenFang Documentation

**OpenFang** is an open-source Agent Operating System written in Rust. A single ~32 MB static binary delivering autonomous AI agents that run 24/7 on schedules, respond across 40 messaging channels, orchestrate multi-agent workflows, and enforce 16 independent security layers — with no Python runtime or Node.js required.

**Current stable release: v0.4.4** (2026-03-15)
**GitHub:** https://github.com/RightNow-AI/openfang
**Website:** https://www.openfang.sh
**Stars:** 14,595 | **Forks:** 1,722 | **License:** Apache-2.0 OR MIT

---

## Documentation Index

| Document | Description |
|----------|-------------|
| [Installation](installation.md) | Install binary, build from source, Docker |
| [CLI Reference](cli-reference.md) | All CLI commands and flags |
| [API Reference](api-reference.md) | 140+ REST/WS/SSE endpoints |
| [Configuration](configuration.md) | config.toml reference |
| [Architecture](architecture.md) | 14-crate workspace, subsystems, boot sequence |
| [Agents](agents.md) | Agent lifecycle, templates, modes |
| [Hands](hands.md) | 7 autonomous capability packages |
| [Channels](channels.md) | 40 messaging channel adapters |
| [Skills](skills.md) | 60 bundled skills + authoring guide |
| [Workflows](workflows.md) | Multi-agent pipeline orchestration |
| [LLM Providers](llm-providers.md) | 27 providers, 130+ models |
| [Security](security.md) | 16-layer security architecture |
| [SDK](sdk.md) | JavaScript and Python SDKs |
| [Migration](migration.md) | Migrating from OpenClaw |
| [Contributing](contributing.md) | Development setup and contribution guide |
| [Changelog](changelog.md) | Full release history |

---

## Quick Start

```bash
# Install (Linux/macOS)
curl -sSf https://openfang.sh | sh

# Install (Windows PowerShell)
irm https://openfang.sh/install.ps1 | iex

# Set a free LLM API key (Groq has a free tier)
export GROQ_API_KEY="gsk_..."

# Initialize and start
openfang init
openfang start            # Dashboard at http://127.0.0.1:4200

# Spawn an agent and chat
openfang agent spawn agents/assistant/agent.toml
openfang chat
```

---

## Key Features

| Feature | Detail |
|---------|--------|
| **Language** | Rust 2021, single ~32 MB binary |
| **LLM Providers** | 27 providers, 130+ models |
| **Channel Adapters** | 40 messaging platforms |
| **Built-in Tools** | 53 core tools + MCP + A2A |
| **Bundled Skills** | 60 expert knowledge skills |
| **Agent Templates** | 30 pre-built templates in 4 tiers |
| **Autonomous Hands** | 8 pre-built 24/7 capability packages |
| **Security Layers** | 16 independent defense systems |
| **Cold Start** | <200 ms |
| **Idle Memory** | ~40 MB |

---

## Performance vs Alternatives

| Metric | OpenFang | OpenClaw | CrewAI | AutoGen |
|--------|----------|----------|--------|---------|
| Cold start | 180 ms | 5.98 s | 3.0 s | 4.0 s |
| Idle memory | 40 MB | 394 MB | 200 MB | 250 MB |
| Install size | 32 MB | 500 MB | 100 MB | 200 MB |
| Security systems | 16 | 3 | 1 | Docker |
| Channel adapters | 40 | 13 | 0 | 0 |
| LLM providers | 27 | 6 | varies | varies |

---

## Architecture Overview

OpenFang is a 14-crate Rust workspace:

```
openfang-cli / openfang-desktop (Tauri 2.0)
    ↓
openfang-api          (Axum 0.8, 140+ endpoints)
    ↓
openfang-kernel       (orchestration, workflows, RBAC, scheduling)
    ├── openfang-runtime   (agent loop, LLM drivers, 53 tools, WASM, MCP, A2A)
    ├── openfang-memory    (SQLite, embeddings, sessions)
    ├── openfang-channels  (40 channel adapters)
    ├── openfang-skills    (60 skills, FangHub marketplace)
    ├── openfang-hands     (8 autonomous Hands)
    ├── openfang-extensions (MCP templates, OAuth2, vault)
    └── openfang-wire      (OFP peer-to-peer protocol)
        openfang-types     (shared types, taint tracking, Ed25519)
```

---

## The 8 Autonomous Hands

Hands are pre-built autonomous capability packages that run on schedules without human prompting:

| Hand | What It Does |
|------|-------------|
| **Clip** | YouTube → vertical shorts with AI captions |
| **Lead** | Daily prospect discovery with ICP scoring |
| **Collector** | OSINT monitoring with knowledge graph |
| **Predictor** | Superforecasting with Brier score tracking |
| **Researcher** | Deep research with CRAAP-method validation |
| **Twitter** | X/Twitter account management (7 content formats) |
| **Browser** | Web automation with purchase-approval gates |
| **Trading** | 8-phase market intelligence + Alpaca API execution |

See [Hands documentation](hands.md) for full details.
