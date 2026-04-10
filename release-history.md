# Release History

Structured version history for OpenFang from initial release through v0.5.7.

All releases are published at [github.com/RightNow-AI/openfang/releases](https://github.com/RightNow-AI/openfang/releases).

---

## Major Milestones

| Version | Date | Headline |
|---------|------|----------|
| v0.1.0 | 2026-02-26 | Initial public release |
| v0.2.0 | 2026-02-28 | Provider fixes, email channel, MCP guidance |
| v0.3.0 | 2026-03-02 | Stability and provider resilience |
| v0.4.0 | 2026-03-12 | Community batch, Docker runtimes, WeCom channel |
| v0.5.0 | 2026-03-20 | Community contributions, RMCP protocol, SearXNG search |

---

## Detailed Release Table

### v0.5.x -- Community Contributions & Hardening

| Version | Date | Key Changes |
|---------|------|-------------|
| [v0.5.7](https://github.com/RightNow-AI/openfang/releases/tag/v0.5.7) | 2026-03-31 | **BREAKING:** Dashboard password hashing switched from SHA256 to Argon2id. Multi-instance hands, `openfang auth hash-password` CLI, 8 community fixes. |
| [v0.5.6](https://github.com/RightNow-AI/openfang/releases/tag/v0.5.6) | 2026-03-30 | SearXNG search provider with pagination/categories, SSRF allowlist tuning, Ollama context fixes, embedding detection. |
| [v0.5.5](https://github.com/RightNow-AI/openfang/releases/tag/v0.5.5) | 2026-03-28 | 5 bug fixes (#771, #811, #752, #772, #661). Silent reinforcement fix, token estimation for ToolUse. |
| [v0.5.4](https://github.com/RightNow-AI/openfang/releases/tag/v0.5.4) | 2026-03-27 | 5 bug fixes + 1 resolved (#875, #872, #867, #824, #833, #766). Dependency bumps (governor, toml). |
| [v0.5.3](https://github.com/RightNow-AI/openfang/releases/tag/v0.5.3) | 2026-03-27 | 7 bug fixes (#825, #828, #856, #770, #774, #851/#808, #785). RMCP protocol support. |
| [v0.5.2](https://github.com/RightNow-AI/openfang/releases/tag/v0.5.2) | 2026-03-26 | 6 bug fixes (#845, #844, #823, #767, #802, #816). Nested XML tool call recovery, agent skills reload, Alpine UI fix. |
| [v0.5.1](https://github.com/RightNow-AI/openfang/releases/tag/v0.5.1) | 2026-03-20 | Post-v0.5.0 hotfixes. |
| [v0.5.0](https://github.com/RightNow-AI/openfang/releases/tag/v0.5.0) | 2026-03-20 | Community contribution milestone. Major community PRs merged. |

### v0.4.x -- Channels & Integration

| Version | Date | Key Changes |
|---------|------|-------------|
| [v0.4.9](https://github.com/RightNow-AI/openfang/releases/tag/v0.4.9) | 2026-03-19 | Image pipeline improvements. |
| [v0.4.8](https://github.com/RightNow-AI/openfang/releases/tag/v0.4.8) | 2026-03-19 | Batch bug fixes across channels and runtime. |
| [v0.4.5](https://github.com/RightNow-AI/openfang/releases/tag/v0.4.5) | 2026-03-18 | Feature batch: WeCom channel adapter, shell skill runtime. |
| [v0.4.1](https://github.com/RightNow-AI/openfang/releases/tag/v0.4.1) | 2026-03-14 | Docker runtime improvements, channel agent re-resolution. |
| [v0.4.0](https://github.com/RightNow-AI/openfang/releases/tag/v0.4.0) | 2026-03-12 | Community batch release. Docker multi-runtime support, WeCom channel, shell skill runtime, channel agent re-resolution. |

### v0.3.x -- Stability & Security Hardening

| Version | Date | Key Changes |
|---------|------|-------------|
| [v0.3.49](https://github.com/RightNow-AI/openfang/releases/tag/v0.3.49) | 2026-03-12 | Final v0.3 release. Trader dashboard, 11+ community issue fixes. |
| [v0.3.45](https://github.com/RightNow-AI/openfang/releases/tag/v0.3.45) | 2026-03-10 | Community fixes (inspired by 11 community PRs). Streaming think fix, cron orphan cleanup. |
| [v0.3.40](https://github.com/RightNow-AI/openfang/releases/tag/v0.3.40) | 2026-03-10 | Claude Code driver fixes, tool schema improvements. |
| [v0.3.30](https://github.com/RightNow-AI/openfang/releases/tag/v0.3.30) | 2026-03-08 | Shell hardening, security hardening, CSP fixes. |
| [v0.3.20](https://github.com/RightNow-AI/openfang/releases/tag/v0.3.20) | 2026-03-05 | Think-tag stripping, driver resilience, model catalog composite, channel resilience. |
| [v0.3.10](https://github.com/RightNow-AI/openfang/releases/tag/v0.3.10) | 2026-03-04 | Community batch fixes, default resilience improvements. |
| [v0.3.0](https://github.com/RightNow-AI/openfang/releases/tag/v0.3.0) | 2026-03-02 | Provider hardening, batch stability fixes. |

### v0.2.x -- Provider Polish

| Version | Date | Key Changes |
|---------|------|-------------|
| [v0.2.9](https://github.com/RightNow-AI/openfang/releases/tag/v0.2.9) | 2026-03-02 | Batch fixes across providers and channels. |
| [v0.2.5](https://github.com/RightNow-AI/openfang/releases/tag/v0.2.5) | 2026-03-01 | Provider fixes, stability improvements. |
| [v0.2.0](https://github.com/RightNow-AI/openfang/releases/tag/v0.2.0) | 2026-02-28 | Provider switching, MCP fixes, email channel, installer fixes. |

### v0.1.x -- Launch Week

| Version | Date | Key Changes |
|---------|------|-------------|
| [v0.1.9](https://github.com/RightNow-AI/openfang/releases/tag/v0.1.9) | 2026-02-28 | Bug fixes. |
| [v0.1.8](https://github.com/RightNow-AI/openfang/releases/tag/v0.1.8) | 2026-02-28 | Real email channel support. |
| [v0.1.6](https://github.com/RightNow-AI/openfang/releases/tag/v0.1.6) | 2026-02-27 | MCP guidance and documentation. |
| [v0.1.4](https://github.com/RightNow-AI/openfang/releases/tag/v0.1.4) | 2026-02-27 | Custom provider URL support. Community fixes. |
| [v0.1.0](https://github.com/RightNow-AI/openfang/releases/tag/v0.1.0) | 2026-02-26 | **Initial public release.** 14-crate Rust workspace with 1,731+ tests. 27 LLM providers, 40 channel adapters, 60 skills, 7 Hands, 41 tools. Desktop app (Tauri 2.0). Full MCP + A2A protocol support. OpenClaw migration engine. JS and Python SDKs. |

---

## Breaking Changes Summary

| Version | Breaking Change |
|---------|-----------------|
| v0.5.7 | Dashboard password hashing switched from SHA256 to Argon2id. Existing `password_hash` values in `config.toml` must be regenerated with `openfang auth hash-password`. |

---

## Platform Statistics by Release

| Version | Tests | Providers | Channels | Skills | Hands | Tools |
|---------|-------|-----------|----------|--------|-------|-------|
| v0.1.0 | 1,731+ | 27 | 40 | 60 | 7 | 41 |
| v0.5.7 | 2,077+ | 27 | 43 | 61 | 9 | 59 |

---

## Release Cadence

OpenFang follows rapid-release during active development. In the first 5 weeks (Feb 26 -- Mar 31, 2026), 87 releases were published -- roughly 2-3 per day during launch week, tapering to weekly patch releases.

- **v0.1.x** (10 releases, Feb 26-28): Launch week rapid iteration
- **v0.2.x** (10 releases, Feb 28 -- Mar 2): Provider stability
- **v0.3.x** (50 releases, Mar 2-12): Hardening, security, community fixes
- **v0.4.x** (10 releases, Mar 12-19): Integration and channels
- **v0.5.x** (8 releases, Mar 20-31): Community contributions, bug fixes
