# CLI Reference

Validated against OpenFang release `v0.5.7`.

Derived from the `Commands` enum and subcommand structs in `crates/openfang-cli/src/main.rs`.

## Global Form

```bash
openfang [--config <path>] <command> [subcommand] [options]
```

**Global flag:**

| Flag | Description |
|------|-------------|
| `--config <path>` | Path to config file (overrides default `~/.openfang/config.toml`) |

When invoked with no command in an interactive terminal, OpenFang shows a launcher menu with options: Get Started, Chat, Dashboard, Desktop App, Terminal UI, Show Help, Quit.

When piped (non-interactive stdout), it prints the help text.

---

## Daemon Lifecycle

### `openfang init`

Initialize OpenFang: create `~/.openfang/` directory, default `config.toml`, install bundled agent templates.

| Flag | Description |
|------|-------------|
| `--quick` | Non-interactive mode (no prompts, auto-detect provider, for CI/scripts) |

In interactive mode, runs a multi-step TUI onboarding wizard. Non-interactive terminals automatically fall back to quick mode.

### `openfang start`

Start the OpenFang kernel daemon (API server + kernel).

| Flag | Description |
|------|-------------|
| `--yolo` | Auto-approve all tool calls (no confirmation prompts) |

Writes `daemon.json` to `~/.openfang/` so the CLI can locate the running daemon. Detects and refuses to start if another daemon is already running on the same port.

### `openfang stop`

Stop the running daemon. Sends a shutdown request to the API, waits up to 5 seconds for clean exit, then force-kills via PID if necessary.

### `openfang status`

Show kernel status (daemon or in-process).

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

Displays: status, agent count, default provider/model, API URL, data directory, uptime, and active agents.

### `openfang doctor`

Run diagnostic health checks.

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |
| `--repair` | Attempt to auto-fix issues (create missing dirs/config, fix permissions, remove stale files) |

Checks: OpenFang directory, `.env` file permissions, config.toml syntax, port availability, daemon status, stale daemon.json, database file, disk space, agent manifests, LLM provider API keys (with live validation), channel token formats, config deserialization, skill registry, extension registry, and daemon health (when running).

### `openfang health`

Quick daemon health check.

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

### `openfang dashboard`

Open the web dashboard in the default browser. Auto-starts the daemon if not running.

### `openfang tui`

Launch the interactive terminal dashboard (ratatui-based TUI).

### `openfang logs`

Tail the OpenFang log file (`~/.openfang/tui.log`).

| Flag | Description |
|------|-------------|
| `--lines <N>` | Number of lines to show (default: 50) |
| `-f`, `--follow` | Follow log output in real time |

---

## Chat

### `openfang chat [agent]`

Quick chat with the default agent (or a named agent). Launches an interactive TUI chat session.

| Argument | Description |
|----------|-------------|
| `agent` | Optional agent name or ID to chat with |

### `openfang message <agent> <text>`

Send a one-shot message to an agent and print the response.

| Argument | Description |
|----------|-------------|
| `agent` | Agent name or ID |
| `text` | Message text |

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

### `openfang sessions [agent]`

List conversation sessions.

| Argument | Description |
|----------|-------------|
| `agent` | Optional agent name or ID to filter by |

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

---

## Agent Management

### `openfang agent new [template]`

Spawn a new agent from a template. Shows interactive picker if template name is omitted.

| Argument | Description |
|----------|-------------|
| `template` | Template name (e.g., "coder", "assistant") |

### `openfang agent spawn <manifest>`

Spawn a new agent from a manifest TOML file.

| Argument | Description |
|----------|-------------|
| `manifest` | Path to the agent manifest TOML file |

### `openfang agent list`

List all running agents.

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

### `openfang agent chat <agent_id>`

Interactive chat with a specific agent by UUID.

### `openfang agent kill <agent_id>`

Kill (terminate) an agent.

### `openfang agent set <agent_id> <field> <value>`

Set an agent property. Currently supported fields: `model`.

---

## Workflow Management

### `openfang workflow list`

List all registered workflows.

### `openfang workflow create <file>`

Create a workflow from a JSON file.

### `openfang workflow get <workflow_id>`

Get a workflow by ID.

### `openfang workflow update <workflow_id> <file>`

Update a workflow from a JSON file.

### `openfang workflow delete <workflow_id>`

Delete a workflow by ID.

### `openfang workflow run <workflow_id> <input>`

Run a workflow with input text.

---

## Trigger Management

### `openfang trigger list`

List all triggers.

| Flag | Description |
|------|-------------|
| `--agent-id <id>` | Optional agent ID to filter by |

### `openfang trigger create <agent_id> <pattern_json> [options]`

Create a trigger for an agent.

| Argument | Description |
|----------|-------------|
| `agent_id` | Agent UUID that owns the trigger |
| `pattern_json` | Trigger pattern as JSON (e.g., `'{"lifecycle":{}}'`, `'{"agent_spawned":{"name_pattern":"*"}}'`) |

| Flag | Description |
|------|-------------|
| `--prompt <template>` | Prompt template, use `{{event}}` placeholder (default: `"Event: {{event}}"`) |
| `--max-fires <N>` | Maximum fire count (0 = unlimited, default: 0) |

### `openfang trigger delete <trigger_id>`

Delete a trigger by ID.

---

## Skill Management

### `openfang skill install <source>`

Install a skill from FangHub, a local directory, or a git URL. Detects and converts OpenClaw skill format automatically.

### `openfang skill list`

List installed skills.

### `openfang skill remove <name>`

Remove an installed skill.

### `openfang skill search <query>`

Search FangHub for skills.

### `openfang skill create`

Interactive scaffold for a new skill (prompts for name, description, runtime).

---

## Channel Management

### `openfang channel list`

List configured channels and their status (webchat, telegram, discord, slack, whatsapp, signal, matrix, email).

### `openfang channel setup [channel]`

Interactive setup wizard for a channel. Shows channel picker if name is omitted.

| Argument | Description |
|----------|-------------|
| `channel` | Channel name: `telegram`, `discord`, `slack`, `whatsapp`, `email`, `signal`, `matrix` |

### `openfang channel test <channel>`

Test a channel by sending a test message.

### `openfang channel enable <channel>`

Enable a channel.

### `openfang channel disable <channel>`

Disable a channel without removing its configuration.

---

## Hand Management

### `openfang hand list`

List all available hands.

### `openfang hand active`

Show currently active hand instances.

### `openfang hand install <path>`

Install a hand from a local directory containing `HAND.toml`.

### `openfang hand activate <id>`

Activate a hand by ID (spawns a dedicated agent).

| Flag | Description |
|------|-------------|
| `-n`, `--name <name>` | Optional instance name (required to run multiple instances of the same hand) |

### `openfang hand deactivate <id>`

Deactivate an active hand instance.

### `openfang hand info <id>`

Show detailed info about a hand.

### `openfang hand check-deps <id>`

Check dependency status for a hand.

### `openfang hand install-deps <id>`

Install missing dependencies for a hand.

### `openfang hand pause <id>`

Pause a running hand instance (by instance ID from `hand active`).

### `openfang hand resume <id>`

Resume a paused hand instance.

---

## Configuration

### `openfang config show`

Print the current `config.toml` contents.

### `openfang config edit`

Open the config file in your editor (`$EDITOR` or `$VISUAL`, falls back to `vi`/`notepad`).

### `openfang config get <key>`

Get a config value by dotted key path.

```bash
openfang config get default_model.provider
openfang config get api_listen
```

### `openfang config set <key> <value>`

Set a config value. Preserves the type of existing values. Creates a backup at `config.toml.bak`.

**Warning:** Strips TOML comments when writing.

```bash
openfang config set default_model.model gpt-4o
openfang config set api_listen "0.0.0.0:4200"
```

### `openfang config unset <key>`

Remove a config key. Creates a backup at `config.toml.bak`.

**Warning:** Strips TOML comments when writing.

### `openfang config set-key <provider>`

Save an API key for a provider to `~/.openfang/.env` (prompts interactively). Also stores in the encrypted vault if initialized. Runs a live validation test after saving.

```bash
openfang config set-key groq
openfang config set-key anthropic
```

### `openfang config delete-key <provider>`

Remove an API key from `~/.openfang/.env` and the vault.

### `openfang config test-key <provider>`

Test provider connectivity with the stored API key (live HTTP request to the provider's API).

---

## Models & Providers

### `openfang models list`

List available models from the catalog.

| Flag | Description |
|------|-------------|
| `--provider <name>` | Filter by provider name |
| `--json` | Output as JSON for scripting |

### `openfang models aliases`

Show model aliases (shorthand names that resolve to full model IDs).

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

### `openfang models providers`

List known LLM providers and their auth status.

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

### `openfang models set [model]`

Set the default model for the daemon. Shows interactive picker if model is omitted.

---

## Scheduled Jobs (Cron)

### `openfang cron list`

List scheduled jobs.

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

### `openfang cron create <agent> <spec> <prompt>`

Create a new scheduled job.

| Argument | Description |
|----------|-------------|
| `agent` | Agent name or ID to run |
| `spec` | Cron expression (e.g., `"0 */6 * * *"`) |
| `prompt` | Prompt to send when the job fires |

| Flag | Description |
|------|-------------|
| `--name <name>` | Optional job name (auto-generated if omitted) |

```bash
openfang cron create researcher "0 */6 * * *" "check for major changes"
```

### `openfang cron delete <id>`

Delete a scheduled job.

### `openfang cron enable <id>`

Enable a disabled job.

### `openfang cron disable <id>`

Disable a job without deleting it.

---

## Approvals

### `openfang approvals list`

List pending execution approvals.

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

### `openfang approvals approve <id>`

Approve a pending request.

### `openfang approvals reject <id>`

Reject a pending request.

---

## Security & Audit

### `openfang security status`

Show security status summary (audit trail, taint tracking, WASM sandbox, wire protocol, API key handling, manifest signing).

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

### `openfang security audit`

Show recent audit trail entries.

| Flag | Description |
|------|-------------|
| `--limit <N>` | Maximum entries to show (default: 20) |
| `--json` | Output as JSON for scripting |

### `openfang security verify`

Verify audit trail integrity (Merkle hash chain).

---

## Auth

### `openfang auth hash-password`

Generate an Argon2id password hash for dashboard authentication. Prompts for password with confirmation. Outputs the hash and the config.toml snippet to add.

---

## Memory (Agent KV Store)

### `openfang memory list <agent>`

List KV pairs for an agent.

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

### `openfang memory get <agent> <key>`

Get a specific KV value.

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

### `openfang memory set <agent> <key> <value>`

Set a KV value.

### `openfang memory delete <agent> <key>`

Delete a KV pair.

---

## Device Pairing

### `openfang devices list`

List paired devices.

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

### `openfang devices pair`

Start a new device pairing flow (generates QR code).

### `openfang devices remove <id>`

Remove a paired device.

### `openfang qr`

Alias for `openfang devices pair`.

---

## Webhooks

### `openfang webhooks list`

List configured webhooks.

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

### `openfang webhooks create <agent> <url>`

Create a new webhook trigger for an agent.

### `openfang webhooks delete <id>`

Delete a webhook.

### `openfang webhooks test <id>`

Send a test payload to a webhook.

---

## Integrations

### `openfang add <name>`

Install an integration (one-click MCP server setup). Resolves credentials interactively.

| Argument | Description |
|----------|-------------|
| `name` | Integration name (e.g., "github", "slack", "notion") |

| Flag | Description |
|------|-------------|
| `--key <token>` | API key or token to store in the vault |

```bash
openfang add github
openfang add github --key ghp_xxxx
```

### `openfang remove <name>`

Remove an installed integration.

### `openfang integrations [query]`

List or search integrations. Groups by category with status badges (Ready, Setup, Available, Error, Disabled).

---

## Credential Vault

### `openfang vault init`

Initialize the encrypted credential vault (`~/.openfang/vault.enc`).

### `openfang vault set <key>`

Store a credential in the vault (prompts for value interactively).

### `openfang vault list`

List all keys in the vault (values are hidden).

### `openfang vault remove <key>`

Remove a credential from the vault.

---

## Gateway (Daemon Control)

Alternative daemon control surface, equivalent to `start`/`stop`/`status`.

### `openfang gateway start`

Same as `openfang start`.

### `openfang gateway stop`

Same as `openfang stop`.

### `openfang gateway status`

Same as `openfang status`.

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

---

## System

### `openfang system info`

Show detailed system info (version, daemon status, agents, provider, model, uptime).

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

### `openfang system version`

Show version information.

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for scripting |

---

## Migration

### `openfang migrate`

Migrate from another agent framework to OpenFang.

| Flag | Description |
|------|-------------|
| `--from <framework>` | Source framework: `openclaw`, `langchain`, `autogpt` |
| `--source-dir <path>` | Path to source workspace (auto-detected if omitted) |
| `--dry-run` | Show what would be imported without making changes |

```bash
openfang migrate --from openclaw
openfang migrate --from langchain --source-dir ~/my-langchain-project --dry-run
```

---

## Scaffolding

### `openfang new <kind>`

Scaffold a new skill or integration template in the current directory.

| Argument | Description |
|----------|-------------|
| `kind` | What to scaffold: `skill` or `integration` |

---

## Shell Completions

### `openfang completion <shell>`

Generate shell completion scripts. Output to stdout.

| Argument | Description |
|----------|-------------|
| `shell` | Shell to generate for: `bash`, `zsh`, `fish`, `elvish`, `powershell` |

```bash
openfang completion zsh > ~/.zfunc/_openfang
openfang completion bash > /etc/bash_completion.d/openfang
openfang completion fish > ~/.config/fish/completions/openfang.fish
```

---

## MCP Server Mode

### `openfang mcp`

Start an MCP (Model Context Protocol) server over stdio. This exposes OpenFang's capabilities to MCP-compatible clients (e.g., Claude Code, Cursor).

---

## Setup Aliases

These commands are aliases for `openfang init`:

| Command | Description |
|---------|-------------|
| `openfang onboard` | Interactive onboarding wizard (same as `init`) |
| `openfang onboard --quick` | Quick non-interactive mode |
| `openfang setup` | Same as `init` |
| `openfang setup --quick` | Quick mode |
| `openfang configure` | Interactive setup (same as `init` without `--quick`) |

---

## Reset & Uninstall

### `openfang reset`

Delete all data in `~/.openfang/` (config, database, agents, credentials).

| Flag | Description |
|------|-------------|
| `--confirm` | Skip the confirmation prompt |

### `openfang uninstall`

Completely remove OpenFang from your system: stop daemon, remove data directory, remove binary, clean auto-start entries, clean PATH from shell configs.

| Flag | Description |
|------|-------------|
| `--confirm` (alias `--yes`) | Skip the confirmation prompt |
| `--keep-config` | Keep config files (config.toml, .env, secrets.env) |

---

## Daemon Detection

The CLI automatically detects a running daemon by reading `~/.openfang/daemon.json`. Commands that need a daemon (workflows, triggers, cron, approvals, etc.) will:

1. Check for a running daemon via the health endpoint.
2. If found, send requests to the daemon over HTTP.
3. If not found, print an error with instructions to start the daemon.

Some commands (agent list, status, chat) can also operate in **in-process mode** when no daemon is running, booting a temporary kernel for the duration of the command.

The CLI reads `api_key` from `config.toml` (or `OPENFANG_API_KEY` env var) and automatically includes `Authorization: Bearer <key>` headers when communicating with the daemon.

## Environment Variables

| Variable | Description |
|----------|-------------|
| `OPENFANG_HOME` | Override the OpenFang home directory (default: `~/.openfang`) |
| `OPENFANG_API_KEY` | API key for daemon authentication (fallback if not in config.toml) |
| `EDITOR` / `VISUAL` | Editor for `config edit` |

The CLI loads `~/.openfang/.env` into the process environment at startup (system env vars take priority).
