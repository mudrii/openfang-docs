# Documentation Integrity Findings — OpenFang `v0.5.7`

Audit date: `2026-04-06`

## Scope

This audit was performed against:

- released source tag `v0.5.7`
- the local docs repo at `~/src/claw/openfang-docs`
- official GitHub release metadata
- the public website and docs only as drift signals

The goal was not to preserve existing wording. The goal was to publish release-accurate documentation for `v0.5.7`.

## Source of truth used in this pass

Primary sources:

- `https://github.com/RightNow-AI/openfang/releases/tag/v0.5.7`
- `v0.5.7` source tree in `~/src/claw/openfang`
- `Cargo.toml`
- `crates/openfang-hands/src/bundled.rs`
- `crates/openfang-skills/src/bundled.rs`
- `crates/openfang-api/src/routes.rs`
- `crates/openfang-runtime/src/model_catalog.rs`
- `crates/openfang-cli/src/main.rs`

Secondary drift-check sources:

- `https://www.openfang.sh`
- `https://www.openfang.sh/docs/getting-started`

## Verified release facts

| Area | Released value | Evidence |
|------|----------------|----------|
| Latest released tag | `v0.5.7` | GitHub release page |
| Release date | `2026-03-31` | GitHub release page |
| Workspace members | `14` | `Cargo.toml` members |
| Crates under `crates/` | `13` | release tree directory count |
| Agent templates | `30` | `agents/*/agent.toml` in `v0.5.7` |
| Bundled Hands | `9` | `openfang-hands` bundled hand definitions |
| Bundled skills | `61` | `openfang-skills` bundled-skills test |
| Channel registry entries | `42` | `CHANNEL_REGISTRY` in API routes |
| Provider entries | `41` | `ModelCatalog::list_providers()` test |
| Builtin model entries | `197` | `builtin_models()` entry count |
| Security layers | `16` | release notes and source security docs |

## Key findings from the previous docs state

### 1. Version drift was not isolated from `main`

The source repo `main` branch is ahead of the latest release. Some older docs mixed released claims with current-branch claims. This docs pass documents `v0.5.7` only.

### 2. Upstream prose docs and website counts conflict with the release tree

Examples:

- upstream README still says `7` Hands in places even though `v0.5.7` bundles `9`
- upstream channel prose says `40` while the API registry exposes `42`
- upstream comments still say `60` bundled skills while the release test asserts `61`

### 3. Provider counts were especially unstable

Older docs used `27` or `28` provider headlines. The released runtime catalog contains `41` provider entries and `197` builtin model entries. Because those include variants and CLI-backed entries, simplified provider counts were removed unless the grouping rule is explained.

### 4. Installation docs contained stale or unsupported claims

Removed or corrected in this pass:

- stale installer example using `v0.4.4`
- unverified Homebrew workflow
- implied crates.io install path
- stale daemon metadata examples

### 5. CLI and API docs contained command and route drift

The docs repo previously described commands and routes that did not line up cleanly with the released source. The rewritten reference pages now derive from `openfang-cli` and `openfang-api`.

## Public web drift observed

The public web presence is useful for detecting documentation drift, but it was not reliable enough to serve as the final authority:

- the homepage and docs site expose inconsistent version signals
- prose counts on the website do not consistently match `v0.5.7`
- release tag metadata on GitHub is materially safer for release documentation

## Files rewritten or corrected in this pass

- `README.md`
- `installation.md`
- `getting-started.md`
- `architecture.md`
- `hands.md`
- `llm-providers.md`
- `agents.md`
- `cli-reference.md`
- `api-reference.md`

Consistency fixes were also applied in smaller supporting files where stale counts or release references remained.

## Methodology

1. fetch and sync both local repos
2. anchor all release claims to `v0.5.7`
3. compare docs-repo claims against source-derived counts
4. use parallel codebase reviews to inspect architecture, CLI/API, and integrations
5. use official GitHub release metadata and the public website only for external validation
6. rewrite high-drift docs first, then run a repo-wide consistency sweep

## Remaining caution areas

- The upstream project still contains stale prose inside the source repo itself.
- Exact tool-count language remains easy to drift and should only be changed with code-backed verification.
- Public website copy should not be assumed to match the latest released tag without checking the release tree.
