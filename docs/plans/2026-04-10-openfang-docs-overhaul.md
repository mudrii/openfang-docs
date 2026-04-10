# OpenFang Documentation Overhaul — v0.5.7

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Produce comprehensive, fact-checked documentation for OpenFang v0.5.7 in the openfang-docs repository, covering all 14 crates, 9 Hands, 61 skills, 40 channels, 27 providers, security systems, API, CLI, desktop app, and migration — verified against actual source code.

**Architecture:** Each doc file is a standalone markdown document targeting a specific topic. All claims (feature counts, tool lists, config options, API endpoints) are verified against the v0.5.7 source at `~/src/claw/openfang`. The docs repo at `~/src/claw/openfang-docs` is a flat structure of .md files with a README.md index.

**Tech Stack:** Markdown, git, GitHub (mudrii/openfang-docs)

---

## Fact-Check Register (Verified Discrepancies)

These are confirmed inaccuracies found in existing docs that MUST be corrected:

| Claim | Old Value | Correct Value (v0.5.7) | Source |
|-------|-----------|------------------------|--------|
| Bundled Hands count | 7 | **9** (added trader, infisical-sync) | `ls crates/openfang-hands/bundled/` |
| Bundled Skills count | 60 | **61** | `find crates/openfang-skills/bundled -name SKILL.md \| wc -l` |
| Test count | 1,767 / 1,744 | **2,077+** | `grep -r '#\[test\]' crates/ \| wc -l` |
| Version badge in README | v0.3.30 | **v0.5.7** | `Cargo.toml workspace.package.version` |
| SECURITY.md supported | 0.3.x only | **0.5.x** | Current release is v0.5.7 |
| CHANGELOG.md | Only v0.1.0 | **Missing 86 releases** | `git tag --list \| wc -l` = 87 tags |
| CONTRIBUTING.md tools | 38 built-in | Needs verification from tool_runner.rs |
| CONTRIBUTING.md hands | 7 | **9** |
| Workspace version | 0.5.7 | Correct | Cargo.toml |

---

## File Structure

### Files to UPDATE (existing):
- `README.md` — Index page, version refs, feature counts
- `getting-started.md` — Installation, init, first agent
- `architecture.md` — 14-crate overview, dependency graph, design patterns
- `configuration.md` — config.toml reference, env vars
- `security.md` — 16 security systems documented from source
- `llm-providers.md` — 27 providers, model catalog, routing
- `channels.md` — 40 adapters, config, per-channel details
- `hands.md` — All 9 hands with HAND.toml specs
- `skills.md` — 61 skills, categories, development guide
- `api-reference.md` — All REST/WS/SSE endpoints
- `cli-reference.md` — All commands and flags
- `desktop.md` — Tauri 2.0 app details
- `mcp-a2a.md` — MCP + A2A protocol support
- `workflows.md` — Workflow engine, triggers, pipelines
- `agents.md` — Agent templates, manifests, lifecycle
- `agent-templates.md` — Pre-built agent catalog
- `sdk.md` — JS/Python SDKs
- `migration.md` — OpenClaw migration
- `troubleshooting.md` — FAQ, common issues
- `contributing.md` — Dev setup, PR process
- `changelog.md` — Full release history
- `production-checklist.md` — Deployment guide
- `skill-development.md` — Creating custom skills

### Files to CREATE (new):
- `tools-reference.md` — Complete built-in tools catalog (currently undocumented)
- `hands-development.md` — Creating custom Hands (HAND.toml spec, lifecycle)
- `whatsapp-gateway.md` — WhatsApp Web QR gateway setup (currently only in README)
- `environment-variables.md` — Complete env var reference
- `release-history.md` — Structured version history with dates and highlights

---

## Task 1: Verify Source-of-Truth Data from Code

**Files:**
- Read: `crates/openfang-runtime/src/tool_runner.rs` (tool definitions)
- Read: `crates/openfang-api/src/server.rs` (route registrations)
- Read: `crates/openfang-channels/src/lib.rs` (channel adapter list)
- Read: `crates/openfang-types/src/` (model catalog, config structs)
- Read: `crates/openfang-cli/src/main.rs` (CLI commands)
- Read: All 9 `HAND.toml` files
- Read: `crates/openfang-kernel/src/kernel.rs` (security systems)

- [ ] **Step 1:** Extract complete tool list from `tool_runner.rs` — count all `builtin_tool_definitions()` entries
- [ ] **Step 2:** Extract all API routes from `server.rs` — count endpoints by method
- [ ] **Step 3:** Extract channel adapter list from channels crate — verify count of 40
- [ ] **Step 4:** Extract model catalog from types crate — verify 27 providers, count models
- [ ] **Step 5:** Extract CLI subcommands from `main.rs`
- [ ] **Step 6:** Read all 9 HAND.toml files — document name, category, tools, settings
- [ ] **Step 7:** Extract security system implementations from kernel and runtime
- [ ] **Step 8:** Compile master fact sheet with verified counts

---

## Task 2: Update README.md (Index Page)

**Files:**
- Modify: `~/src/claw/openfang-docs/README.md`

- [ ] **Step 1:** Update all version references to v0.5.7
- [ ] **Step 2:** Update feature counts (9 hands, 61 skills, 2077+ tests, correct tool count)
- [ ] **Step 3:** Update table of contents to include new doc files
- [ ] **Step 4:** Add links to new docs (tools-reference, hands-development, etc.)
- [ ] **Step 5:** Verify all internal doc links work

---

## Task 3: Rewrite architecture.md

**Files:**
- Modify: `~/src/claw/openfang-docs/architecture.md`
- Read: All 14 `crates/*/Cargo.toml` for dependency graph
- Read: All `crates/*/src/lib.rs` for public API surface

- [ ] **Step 1:** Document each of the 14 crates with verified info from Cargo.toml and source
- [ ] **Step 2:** Map inter-crate dependency graph (which crate depends on which)
- [ ] **Step 3:** Document key architectural patterns (KernelHandle trait, AppState bridge, daemon detection)
- [ ] **Step 4:** Document LOC per crate and test counts
- [ ] **Step 5:** Include workspace Cargo.toml dependency versions

---

## Task 4: Rewrite hands.md (All 9 Hands)

**Files:**
- Modify: `~/src/claw/openfang-docs/hands.md`
- Read: All 9 `crates/openfang-hands/bundled/*/HAND.toml`
- Read: All 9 `crates/openfang-hands/bundled/*/SKILL.md`
- Read: `crates/openfang-hands/src/lib.rs` and `registry.rs`

- [ ] **Step 1:** Document all 9 hands: clip, lead, collector, predictor, researcher, twitter, browser, trader, infisical-sync
- [ ] **Step 2:** For each hand: name, category, description, required tools, settings, metrics
- [ ] **Step 3:** Document multi-instance activation (v0.5.7 feature)
- [ ] **Step 4:** Document hand lifecycle: activate, pause, resume, deactivate, status
- [ ] **Step 5:** Document custom hand installation and persistence across restarts

---

## Task 5: Create hands-development.md

**Files:**
- Create: `~/src/claw/openfang-docs/hands-development.md`
- Read: `crates/openfang-hands/src/lib.rs` (HandManifest struct)
- Read: Example HAND.toml files for format reference

- [ ] **Step 1:** Document HAND.toml manifest format with all fields
- [ ] **Step 2:** Document SKILL.md format for hand-specific expertise
- [ ] **Step 3:** Document hand lifecycle management API
- [ ] **Step 4:** Document publishing to FangHub
- [ ] **Step 5:** Include complete example of a custom hand

---

## Task 6: Rewrite channels.md (40 Adapters)

**Files:**
- Modify: `~/src/claw/openfang-docs/channels.md`
- Read: `crates/openfang-channels/src/lib.rs`
- Read: `crates/openfang-channels/src/*.rs` (each adapter)

- [ ] **Step 1:** List all 40 channel adapters with verified names from source
- [ ] **Step 2:** Document config format for each major adapter (Telegram, Discord, Slack, WhatsApp, Matrix, Email)
- [ ] **Step 3:** Document common features: per-channel model overrides, DM/group policies, rate limiting
- [ ] **Step 4:** Document channel bridge architecture
- [ ] **Step 5:** Include Discord gateway heartbeat fix (v0.5.7)

---

## Task 7: Create whatsapp-gateway.md

**Files:**
- Create: `~/src/claw/openfang-docs/whatsapp-gateway.md`
- Read: README.md WhatsApp section for reference

- [ ] **Step 1:** Document WhatsApp Web QR gateway setup (Node.js gateway)
- [ ] **Step 2:** Document environment variables (WHATSAPP_WEB_GATEWAY_URL, etc.)
- [ ] **Step 3:** Document gateway API endpoints
- [ ] **Step 4:** Document WhatsApp Cloud API alternative
- [ ] **Step 5:** Include troubleshooting section

---

## Task 8: Rewrite security.md (16 Systems)

**Files:**
- Modify: `~/src/claw/openfang-docs/security.md`
- Read: `crates/openfang-runtime/src/` (WASM sandbox, subprocess sandbox, tool runner)
- Read: `crates/openfang-kernel/src/kernel.rs` (audit trail, loop guard, session repair)
- Read: `crates/openfang-types/src/` (taint tracking, manifest signing)
- Read: `crates/openfang-wire/src/` (OFP mutual auth)
- Read: `crates/openfang-api/src/` (security headers, rate limiter, auth)

- [ ] **Step 1:** Document each of the 16 security systems with code references
- [ ] **Step 2:** Document the v0.5.7 security fix: rm allowlist bypass (#919)
- [ ] **Step 3:** Document SSRF allowlist configuration (v0.5.6 feature)
- [ ] **Step 4:** Document security reporting process (update from 0.3.x to 0.5.x)
- [ ] **Step 5:** Include security dependency audit table

---

## Task 9: Rewrite configuration.md

**Files:**
- Modify: `~/src/claw/openfang-docs/configuration.md`
- Read: `openfang.toml.example`
- Read: `crates/openfang-types/src/` (KernelConfig struct and all config structs)

- [ ] **Step 1:** Document complete config.toml reference with all sections
- [ ] **Step 2:** Document all environment variables
- [ ] **Step 3:** Document config hot-reload behavior
- [ ] **Step 4:** Document per-agent config (agent.toml format)
- [ ] **Step 5:** Document YOLO mode, approval settings, exec_policy

---

## Task 10: Create tools-reference.md

**Files:**
- Create: `~/src/claw/openfang-docs/tools-reference.md`
- Read: `crates/openfang-runtime/src/tool_runner.rs`

- [ ] **Step 1:** List every built-in tool with name, description, input schema
- [ ] **Step 2:** Group tools by category (filesystem, web, shell, agent, memory, scheduling, media)
- [ ] **Step 3:** Document tool profiles and capability gates
- [ ] **Step 4:** Document MCP tool integration
- [ ] **Step 5:** Include tool name compatibility mappings (OpenClaw → OpenFang)

---

## Task 11: Rewrite llm-providers.md

**Files:**
- Modify: `~/src/claw/openfang-docs/llm-providers.md`
- Read: `crates/openfang-runtime/src/drivers/` (all driver files)
- Read: `crates/openfang-types/src/` (model catalog)

- [ ] **Step 1:** Document all 27 providers with API key env vars
- [ ] **Step 2:** Document 3 native drivers (Anthropic, Gemini, OpenAI-compatible)
- [ ] **Step 3:** Document model catalog with tier classification
- [ ] **Step 4:** Document intelligent routing, fallback chains, cost tracking
- [ ] **Step 5:** Document per-agent model override and provider URL overrides

---

## Task 12: Rewrite api-reference.md

**Files:**
- Modify: `~/src/claw/openfang-docs/api-reference.md`
- Read: `crates/openfang-api/src/server.rs` (all route registrations)
- Read: `crates/openfang-api/src/routes.rs` or handler files

- [ ] **Step 1:** List all API endpoints by category with method, path, description
- [ ] **Step 2:** Document OpenAI-compatible endpoints (/v1/chat/completions, /v1/models)
- [ ] **Step 3:** Document WebSocket streaming protocol
- [ ] **Step 4:** Document A2A protocol endpoints
- [ ] **Step 5:** Document authentication (Bearer token, loopback bypass)

---

## Task 13: Rewrite cli-reference.md

**Files:**
- Modify: `~/src/claw/openfang-docs/cli-reference.md`
- Read: `crates/openfang-cli/src/main.rs`

- [ ] **Step 1:** Document all CLI subcommands with flags and options
- [ ] **Step 2:** Document daemon management (start, stop, status, doctor)
- [ ] **Step 3:** Document MCP server mode
- [ ] **Step 4:** Document shell completion generation
- [ ] **Step 5:** Document TUI dashboard screens

---

## Task 14: Rewrite skills.md and skill-development.md

**Files:**
- Modify: `~/src/claw/openfang-docs/skills.md`
- Modify: `~/src/claw/openfang-docs/skill-development.md`
- Read: `crates/openfang-skills/src/` (skill system)
- Read: `crates/openfang-skills/bundled/` (all 61 skill manifests)

- [ ] **Step 1:** List all 61 bundled skills organized by category
- [ ] **Step 2:** Document 4 skill runtimes (Python, Node.js, WASM, PromptOnly)
- [ ] **Step 3:** Document SKILL.md format and manifest structure
- [ ] **Step 4:** Document FangHub/ClawHub marketplace
- [ ] **Step 5:** Document skill installation, hot-reload, SHA256 verification

---

## Task 15: Rewrite changelog.md (Full Release History)

**Files:**
- Modify: `~/src/claw/openfang-docs/changelog.md`
- Source: All GitHub release notes (v0.1.0 through v0.5.7)

- [ ] **Step 1:** Document v0.5.x releases (v0.5.0–v0.5.7) with full details
- [ ] **Step 2:** Document v0.4.x releases (v0.4.0–v0.4.9)
- [ ] **Step 3:** Document v0.3.x notable releases (v0.3.0, v0.3.30 security hardening, v0.3.40+)
- [ ] **Step 4:** Document v0.2.x releases (v0.2.0–v0.2.9)
- [ ] **Step 5:** Include v0.1.0 initial release

---

## Task 16: Create environment-variables.md

**Files:**
- Create: `~/src/claw/openfang-docs/environment-variables.md`
- Read: Source code for all env var references

- [ ] **Step 1:** List all provider API key env vars (ANTHROPIC_API_KEY, etc.)
- [ ] **Step 2:** List all channel token env vars
- [ ] **Step 3:** List runtime config env vars (RUST_LOG, etc.)
- [ ] **Step 4:** List WhatsApp gateway env vars
- [ ] **Step 5:** Group by category with descriptions and defaults

---

## Task 17: Create release-history.md

**Files:**
- Create: `~/src/claw/openfang-docs/release-history.md`

- [ ] **Step 1:** Create structured version table (version, date, headline, highlights)
- [ ] **Step 2:** Document major milestones (v0.1.0, v0.2.0, v0.3.30, v0.4.0, v0.5.0, v0.5.7)
- [ ] **Step 3:** Include GitHub release links
- [ ] **Step 4:** Note breaking changes per version

---

## Task 18: Update remaining docs

**Files:**
- Modify: `getting-started.md`, `desktop.md`, `mcp-a2a.md`, `workflows.md`, `agents.md`, `agent-templates.md`, `sdk.md`, `migration.md`, `troubleshooting.md`, `contributing.md`, `production-checklist.md`

- [ ] **Step 1:** Update version references in all files to v0.5.7
- [ ] **Step 2:** Update feature counts in all files
- [ ] **Step 3:** Verify all code examples work with v0.5.7
- [ ] **Step 4:** Add v0.5.7 features (multi-instance hands, SSRF allowlist, etc.)
- [ ] **Step 5:** Remove outdated/incorrect information

---

## Task 19: Cross-Reference Validation

- [ ] **Step 1:** Verify all internal links between docs resolve
- [ ] **Step 2:** Verify all feature counts are consistent across docs
- [ ] **Step 3:** Verify all version references are v0.5.7
- [ ] **Step 4:** Verify config examples match actual config struct fields
- [ ] **Step 5:** Run final diff review of all changes

---

## Task 20: Git Operations

- [ ] **Step 1:** Initialize clean state in openfang-docs repo
- [ ] **Step 2:** Stage all changes
- [ ] **Step 3:** Create descriptive commit
- [ ] **Step 4:** Push to GitHub (mudrii/openfang-docs)

---

## Parallel Execution Strategy

Tasks can be parallelized in these groups:

**Group A (source extraction — must run first):** Task 1
**Group B (independent doc rewrites — run in parallel after Group A):**
- Agent 1: Tasks 2, 3 (README, architecture)
- Agent 2: Tasks 4, 5 (hands, hands-development)
- Agent 3: Tasks 6, 7 (channels, whatsapp-gateway)
- Agent 4: Tasks 8, 9 (security, configuration)
- Agent 5: Tasks 10, 11 (tools-reference, llm-providers)
- Agent 6: Tasks 12, 13 (api-reference, cli-reference)
- Agent 7: Tasks 14, 15 (skills, changelog)
- Agent 8: Tasks 16, 17 (env-vars, release-history)

**Group C (after Group B):** Tasks 18, 19 (cross-updates, validation)
**Group D (final):** Task 20 (git operations)
