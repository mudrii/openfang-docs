# Documentation Integrity Findings — OpenFang v0.5.7

**Audit date:** 2026-04-02
**Source repo:** `~/src/claw/openfang/` (read-only snapshot)
**Docs repo:** `~/src/claw/openfang-docs/`
**Auditor:** automated fact-check against source code and public web sources

---

## 1. Source Code Verification

### 1.1 Version

| Claim location | Stated version | Actual (source) | Status |
|----------------|---------------|-----------------|--------|
| docs README.md line 5 | v0.4.4 | Cargo.toml: **0.5.5**, changelog.md: **v0.5.7** | DISCREPANCY |
| security.md line 220 (health endpoint example) | v0.4.4 | 0.5.5 / 0.5.7 | DISCREPANCY |
| Source README.md badge | v0.3.30 | Cargo.toml: 0.5.5 | STALE (source repo README) |
| changelog.md line 5 | v0.5.7 | Latest release per changelog | OK (most up-to-date) |

**Note:** The docs README still says v0.4.4. The changelog says v0.5.7. Cargo.toml workspace version is 0.5.5 (the 0.5.6 and 0.5.7 releases may have been tagged without bumping Cargo.toml yet). The source repo's own README badge still shows v0.3.30, which is severely outdated.

### 1.2 Crate Count

| Claim | Docs say | Source (Cargo.toml members) | Status |
|-------|----------|---------------------------|--------|
| Workspace crates | "14-crate workspace" (README, architecture.md) | 13 crates + xtask = 14 workspace members; 13 actual `crates/` directories | ACCEPTABLE |

The docs say "14-crate workspace." Cargo.toml lists 13 crates under `crates/` plus `xtask` (build automation), totaling 14 workspace members. Whether xtask counts as a "crate" is debatable. The `ls crates/` directory shows exactly 13 entries. The claim of "14 crates" is reasonable if xtask is included, but could be more precise.

### 1.3 Agent Templates

| Claim | Docs say | Source (`agents/`) | Status |
|-------|----------|-------------------|--------|
| Agent templates | "30 pre-built templates" (README Key Features table) | **31** directories in `agents/` | DISCREPANCY (minor, undercount) |

### 1.4 Bundled Skills

| Claim | Docs say | Source (`crates/openfang-skills/bundled/`) | Status |
|-------|----------|------------------------------------------|--------|
| Bundled skills | "60 bundled skills" (README, architecture.md, skills.md) | **61** directories | DISCREPANCY (minor, undercount) |

### 1.5 Channel Adapters

| Claim | Docs say | Source (`crates/openfang-channels/src/`) | Status |
|-------|----------|----------------------------------------|--------|
| Channel adapters | "40 messaging channels" | **43** adapter .rs files (excluding lib.rs, types.rs, router.rs, formatter.rs, bridge.rs). Includes dingtalk.rs + dingtalk_stream.rs (likely 1 platform with 2 files). | DISCREPANCY |

Actual adapter files: bluesky, dingtalk (+stream), discord, discourse, email, feishu, flock, gitter, google_chat, gotify, guilded, irc, keybase, line, linkedin, mastodon, matrix, mattermost, messenger, mqtt, mumble, nextcloud, nostr, ntfy, pumble, reddit, revolt, rocketchat, signal, slack, teams, telegram, threema, twist, twitch, viber, webex, webhook, wecom, whatsapp, xmpp, zulip.

If dingtalk + dingtalk_stream count as 1 platform, that is 42 unique adapters. If webhook is excluded as not a "messaging channel," it would be 41. Either way, the docs undercount at 40.

### 1.6 Autonomous Hands

| Claim | Docs say | Source (`crates/openfang-hands/bundled/`) | Status |
|-------|----------|----------------------------------------|--------|
| Hands count | "8 autonomous Hands" (README, architecture.md, hands.md) | **9** directories: browser, clip, collector, infisical-sync, lead, predictor, researcher, trader, twitter | DISCREPANCY |

The changelog for v0.5.4 explicitly states: "Infisical Sync Hand -- Secret synchronization with Infisical vault (9th bundled Hand)." The main README and architecture/hands docs were not updated to reflect this addition. Additionally, the README table lists "Trading" but the directory is named `trader`.

### 1.7 LLM Providers

| Claim | Docs say | Source (model_catalog.rs) | Status |
|-------|----------|--------------------------|--------|
| Provider count | "27 providers" (README, llm-providers.md) | **41** ProviderInfo entries in `builtin_providers()` | NEEDS CONTEXT |
| Model count | "130+ models" | Source comment says "130+ builtin models across 28 providers"; 217 ModelCatalogEntry references in file | NEEDS CONTEXT |

The 41 ProviderInfo entries include coding-specific variants (zhipu_coding, zai_coding, kimi_coding, volcengine_coding) and CLI-based providers (claude-code, qwen-code, codex). If coding variants are grouped with their parent and CLI tools are excluded, approximately 28-30 distinct provider "brands" exist. The docs claim of "27 providers" is an undercount. The source code's own docstring says "28 providers."

### 1.8 Built-in Tools

| Claim | Docs say | Source | Status |
|-------|----------|--------|--------|
| Built-in tools | "53 core tools" (README), "53 built-in tools" (architecture.md line 17), "41 built-in tools" (architecture.md line 239) | Not independently verified | INTERNAL INCONSISTENCY |

architecture.md contradicts itself: line 17 says "53 built-in tools" while line 239 says "41 built-in tools -> 4-10 for chat agents." The README says 53. The online sources (openfang.sh, GitHub README) also say 53.

### 1.9 Security Layers

| Claim | Docs say | Source | Status |
|-------|----------|--------|--------|
| Security layers | "16 independent security layers" | 16 layers enumerated in architecture.md and security.md | OK |

Both documents list the same 16 layers consistently.

### 1.10 governor Crate Version

| Claim | security.md says | Source (Cargo.toml) | Status |
|-------|-----------------|---------------------|--------|
| governor version | "governor 0.8" (security.md line 277) | `governor = "0.10"` | DISCREPANCY |

### 1.11 Test Count

| Claim | Docs say | Source README | Status |
|-------|----------|--------------|--------|
| Tests | "1,767+ tests" (architecture.md) | Source README badge says "1,767+ passing" | CONSISTENT (but not independently verified) |

### 1.12 License

| Claim | Docs say | Source / GitHub | Status |
|-------|----------|----------------|--------|
| License | README line 8: "Apache-2.0 OR MIT" | Cargo.toml: `license = "Apache-2.0 OR MIT"` | OK |
| GitHub search result | One web source says "fully open source under the MIT license" | Cargo.toml: dual-licensed | EXTERNAL SOURCE INACCURATE (not our docs) |

---

## 2. Online Verification

### 2.1 GitHub Repository

| Claim | Docs say | Web search (2026-04-02) | Status |
|-------|----------|------------------------|--------|
| Repo URL | github.com/RightNow-AI/openfang | Confirmed exists | OK |
| Stars | 14,595 | **16,113** (web search result) | DISCREPANCY (docs stale) |
| Forks | 1,722 | **2,005** (web search result) | DISCREPANCY (docs stale) |
| Description | "Agent Operating System" | Confirmed | OK |

### 2.2 Official Website

| Claim | Docs say | Web search | Status |
|-------|----------|-----------|--------|
| Website URL | https://www.openfang.sh | Confirmed — title: "OpenFang -- The Agent Operating System" | OK |

### 2.3 crates.io Publication

| Question | Result |
|----------|--------|
| Is OpenFang published on crates.io? | **No.** Not found on crates.io. The 14 crates are part of the internal workspace, installed from GitHub source. |

### 2.4 Community & Media Presence

| Source | Finding |
|--------|---------|
| Product Hunt | Listed: "OpenFang: Open-Source Agent Operating System" |
| Medium | Multiple articles (March 2026) |
| SitePoin | Benchmark article comparing to CrewAI & LangGraph |
| LinkedIn | Feature article exists |
| SourceForge | Mirror exists |
| Mintlify | Documentation site hosted |

### 2.5 Claimed Performance Comparison

The docs include a comparison table (README lines 76-85) comparing OpenFang to OpenClaw, CrewAI, and AutoGen. External benchmark articles (SitePoint) confirm the general claims about cold start and memory usage, though exact numbers may vary by test environment. The comparison table values are plausible but should be treated as approximate.

---

## 3. Discrepancy Summary

### Critical

| # | Location | Issue | Recommended Fix |
|---|----------|-------|----------------|
| 1 | README.md line 5 | Version says v0.4.4; should be v0.5.7 | Update to v0.5.7 |
| 2 | security.md line 220 | Health endpoint example shows v0.4.4 | Update to v0.5.7 |

### Moderate

| # | Location | Issue | Recommended Fix |
|---|----------|-------|----------------|
| 3 | README.md line 69 | "8 pre-built 24/7 capability packages" | Update to 9 (infisical-sync added in v0.5.4) |
| 4 | README.md lines 110-124 | "The 8 Autonomous Hands" table missing Infisical Sync | Add row for Infisical Sync hand |
| 5 | architecture.md line 22 | "8 autonomous capability packages" | Update to 9 |
| 6 | hands.md line 6 | "All 8 Hands ship compiled" | Update to 9 |
| 7 | README.md line 26, llm-providers.md line 3 | "27 providers" | Update to 28+ (source docstring says 28) |
| 8 | architecture.md lines 17 vs 239 | Internal inconsistency: "53 tools" vs "41 tools" | Resolve to single correct count |
| 9 | README.md line 8 | Stars 14,595 / Forks 1,722 | Update to 16,113 / 2,005 (or note these are approximate) |
| 10 | security.md line 277 | "governor 0.8" | Update to "governor 0.10" |

### Minor

| # | Location | Issue | Recommended Fix |
|---|----------|-------|----------------|
| 11 | README.md line 68 | "30 pre-built templates" | Update to 31 |
| 12 | README.md line 24, architecture.md line 22, skills.md | "60 bundled skills" | Update to 61 |
| 13 | README.md line 23, channels.md line 1, architecture.md | "40 channel adapters" | Update to 43 (or 42 counting dingtalk as one) |

---

## 4. Coverage Assessment

### What is well-documented

- Security architecture (16 layers) -- thoroughly described with code examples
- Agent lifecycle and manifest format
- Boot sequence and daemon architecture
- Memory substrate and SQLite schema
- Capability system (23 types, 9 categories)
- WASM sandbox dual metering
- OFP wire protocol
- Channel configuration with per-channel overrides
- Skill manifest format and runtimes

### What needs updating

- All numeric claims (versions, counts) are stale relative to v0.5.4-v0.5.7 changes
- The Infisical Sync hand is completely missing from hands.md, README, and architecture docs
- Provider count lags behind source (27 claimed vs 28+ actual brands)
- Stars/forks are snapshot values that will always drift; consider removing exact numbers or marking as approximate
- The built-in tool count inconsistency (53 vs 41) in architecture.md needs resolution
- governor crate version is wrong in security.md

### What is absent

- No documentation for MQTT channel adapter (added v0.5.4)
- No documentation for Infisical Sync hand (added v0.5.4)
- No mention of Vertex AI driver (added v0.5.4)
- No mention of HTTP memory backend (added v0.5.4)
- SearXNG search provider (added v0.5.7) not in provider docs
- Argon2id password hashing change (v0.5.7, breaking) not reflected in security.md or configuration.md

---

## 5. Methodology

1. **Source verification:** Read Cargo.toml, directory listings, and model_catalog.rs from the source repo
2. **Documentation review:** Read all 17 docs files in the docs repo
3. **Cross-reference:** Compared every numeric claim in docs against source code
4. **Online verification:** Web searches for GitHub repo, website, crates.io presence, and community coverage
5. **Internal consistency:** Checked docs against each other for contradictions
