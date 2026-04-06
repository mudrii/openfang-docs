# CLI Reference

Validated against OpenFang release `v0.5.7`.

This page is derived from the command enums in `crates/openfang-cli/src/main.rs`, not from older prose docs.

## Global form

```bash
openfang <command> [subcommand] [options]
```

Global flag:

```text
--config <path>
```

## Core commands

| Command | Purpose |
|---------|---------|
| `init` | initialize local OpenFang state |
| `start` | start the daemon |
| `stop` | stop the daemon |
| `status` | daemon status |
| `doctor` | diagnostics |
| `dashboard` | open the dashboard |
| `chat` | quick chat |
| `tui` | terminal UI |
| `health` | quick health check |
| `logs` | tail logs |
| `sessions` | list sessions |
| `message` | send a one-shot message |

## Structured command groups

| Group | Released subcommands |
|-------|----------------------|
| `agent` | `new`, `spawn`, `list`, `chat`, `kill`, `set` |
| `workflow` | `list`, `create`, `get`, `update`, `delete`, `run` |
| `trigger` | `list`, `create`, `delete` |
| `skill` | `install`, `list`, `remove`, `search`, `create` |
| `channel` | `list`, `setup`, `test`, `enable`, `disable` |
| `hand` | `list`, `active`, `install`, `activate`, `deactivate`, `info`, `check-deps`, `install-deps`, `pause`, `resume` |
| `config` | `show`, `edit`, `get`, `set`, `unset`, `set-key`, `delete-key`, `test-key` |
| `models` | `list`, `aliases`, `providers`, `set` |
| `gateway` | `start`, `stop`, `status` |
| `approvals` | `list`, `approve`, `reject` |
| `cron` | `list`, `create`, `delete`, `enable`, `disable` |
| `auth` | `hash-password` |
| `security` | `status`, `audit`, `verify` |
| `memory` | `list`, `get`, `set`, `delete` |
| `devices` | `list`, `pair`, `remove` |
| `webhooks` | `list`, `create`, `delete`, `test` |
| `system` | `info`, `version` |
| `vault` | `init`, `set`, `list`, `remove` |

## Additional top-level commands

The release CLI also includes:

- `migrate`
- `completion`
- `mcp`
- `add`
- `remove`
- `integrations`
- `new`
- `qr`
- `onboard`
- `setup`
- `configure`
- `reset`
- `uninstall`

## High-signal examples

### Initialize and start

```bash
openfang init
openfang doctor
openfang start
```

### Manage agents

```bash
openfang agent new
openfang agent spawn agents/assistant/agent.toml
openfang agent list
openfang agent chat <agent-id>
```

### Manage Hands

```bash
openfang hand list
openfang hand info researcher
openfang hand check-deps browser
openfang hand activate researcher
```

### Configure provider keys

```bash
openfang config set-key groq
openfang config test-key groq
openfang models providers
```

### Generate a dashboard password hash

```bash
openfang auth hash-password
```

### Work with schedules

```bash
openfang cron list
openfang cron create researcher "0 */6 * * *" "check for major changes"
```

## Notes

- Earlier docs in this repo described commands such as `agent update`; the released CLI exposes `agent set` instead.
- The command surface is broad; this page is intended as a release-accurate command map, not a replacement for `--help`.
