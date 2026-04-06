# Review Plan — OpenFang `v0.5.7`

This is the execution plan used for the release-accuracy pass in this docs repo.

## Objective

Review the OpenFang source tree and documentation in detail, validate released behavior only, and publish a corrected docs set in `openfang-docs`.

## Release boundary

Use `v0.5.7` as the documentation baseline.

Reason:

- `origin/main` contains unreleased commits beyond the latest release tag
- the docs repo should describe released behavior, not current branch drift

## Review phases

### Phase 1: sync and baseline

1. fetch `openfang` and `openfang-docs`
2. confirm branch divergence
3. identify the latest released tag
4. treat `v0.5.7` as authoritative for product behavior

### Phase 2: source-of-truth extraction

1. verify workspace structure from `Cargo.toml`
2. verify released template count from `agents/*/agent.toml`
3. verify bundled Hands from `openfang-hands`
4. verify bundled skills from `openfang-skills`
5. verify channel count from `CHANNEL_REGISTRY`
6. verify provider and model catalog counts from `model_catalog.rs`
7. verify CLI surface from `openfang-cli/src/main.rs`
8. verify API families from `openfang-api/src/routes.rs`

### Phase 3: docs drift audit

1. compare existing docs-repo claims against released counts
2. identify broken links, stale versions, and unsupported install claims
3. flag any document that mixes released and unreleased behavior
4. flag any count that depends on ambiguous grouping rules

### Phase 4: public validation

1. check the official GitHub release page
2. check the public website and docs site
3. use those public sources only to confirm external drift, not as the final product truth

### Phase 5: rewrite

Priority order:

1. `README.md`
2. `installation.md`
3. `getting-started.md`
4. `architecture.md`
5. `hands.md`
6. `llm-providers.md`
7. `agents.md`
8. `cli-reference.md`
9. `api-reference.md`
10. integrity report and supporting consistency fixes

### Phase 6: verification

1. run a repo-wide search for stale counts and stale version strings
2. inspect git diff for accidental regressions
3. commit in `openfang-docs`
4. push to GitHub

## Parallel review ownership used for this pass

To speed up the review, work was split across multiple agents:

- architecture, runtime, security, workflows
- CLI, config, install, API, desktop
- channels, Hands, skills, providers, SDKs, templates
- docs-repo structural audit and rewrite priorities

## Documentation rules for future updates

- prefer release tags over `main` for public docs
- prefer source-derived counts over prose counts
- avoid headline numbers when the grouping rule is ambiguous
- treat the public website as a drift signal, not the final authority
