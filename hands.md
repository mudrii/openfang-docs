# Hands

Validated against OpenFang release `v0.5.7`.

Hands are bundled autonomous capability packages managed by the `openfang-hands` crate. They are not the same thing as the `agents/` template catalog.

Release fact:

- `v0.5.7` bundles `9` Hands in `crates/openfang-hands/bundled`

## Released Hand Catalog

| Hand ID | Purpose |
|---------|---------|
| `clip` | Turn source video into short-form clips |
| `lead` | Prospect discovery and enrichment |
| `collector` | Continuous OSINT-style monitoring |
| `predictor` | Forecasting and evidence tracking |
| `researcher` | Deep research and report generation |
| `twitter` | X/Twitter content management |
| `browser` | Browser automation with approval controls |
| `trader` | Market intelligence and trading workflows |
| `infisical-sync` | Secret synchronization with Infisical |

## How Hands differ from agent templates

| Surface | Backing source |
|---------|----------------|
| Agent templates | `agents/*/agent.toml` |
| Hands | `crates/openfang-hands/bundled/*/HAND.toml` |

Agent templates are manifests you spawn directly. Hands are curated packages with their own definitions, settings, requirement checks, lifecycle, and metrics.

## Common Hand Operations

```bash
openfang hand list
openfang hand active
openfang hand info researcher
openfang hand check-deps browser
openfang hand activate researcher
openfang hand pause <instance-id>
openfang hand resume <instance-id>
openfang hand deactivate researcher
```

## Dependency-sensitive Hands

Some Hands have requirement checks before they are fully usable.

Examples from the released source:

- `browser` checks for `python3` and optionally Chromium
- `twitter` requires Twitter/X credentials
- `infisical-sync` requires Infisical connection settings

The release CLI exposes:

- `openfang hand check-deps <id>`
- `openfang hand install-deps <id>`

## Packaging Model

Each released Hand definition includes:

- a `HAND.toml`
- a bundled `SKILL.md`
- settings metadata
- dashboard metrics
- requirement metadata

The bundled registry test in `crates/openfang-hands/src/bundled.rs` asserts `hands.len() == 9`.

## Notes

- Upstream README prose still says `7` Hands in some places; that is stale for `v0.5.7`.
- This docs repo uses the bundled-hand registry, not the README, as the source of truth.
