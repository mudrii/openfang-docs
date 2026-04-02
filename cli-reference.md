# CLI Reference

All commands: `openfang <command> [subcommand] [options]`

Default daemon address: `http://127.0.0.1:4200`

---

## Global Flags

| Flag | Description |
|------|-------------|
| `--config <path>` | Custom config file path |
| `--help`, `-h` | Show help |
| `--version`, `-V` | Show version |

---

## Daemon Management

### `openfang init`

Initialize OpenFang for the first time. Creates `~/.openfang/config.toml` and the directory structure.

```bash
openfang init          # Interactive wizard
openfang init --quick  # Non-interactive with defaults
```

### `openfang start`

Start the daemon (background HTTP server).

```bash
openfang start                     # Start daemon
openfang start --yolo              # YOLO mode: auto-approve all tool calls
openfang start --config /path/to/config.toml
```

The daemon serves the web dashboard at `http://127.0.0.1:4200`.

### `openfang status`

Show daemon status including PID, uptime, agent count.

```bash
openfang status
```

### `openfang doctor`

Diagnose system dependencies and configuration issues.

```bash
openfang doctor
```

Checks: Rust toolchain, LLM connectivity, channel tokens, disk space, port availability.

### `openfang dashboard`

Open the web dashboard in the default browser.

```bash
openfang dashboard
```

---

## Agent Commands

### `openfang agent new`

Create a new agent interactively.

```bash
openfang agent new
openfang agent new --name researcher --template researcher
```

### `openfang agent spawn`

Spawn an agent from a TOML manifest file.

```bash
openfang agent spawn agents/assistant/agent.toml
openfang agent spawn agents/coder/agent.toml --name my-coder
```

### `openfang agent list`

List all running agents.

```bash
openfang agent list
openfang agent list --format json
```

### `openfang agent chat`

Start an interactive chat session with an agent.

```bash
openfang agent chat                   # Chat with default agent
openfang agent chat my-agent          # Chat with named agent
openfang chat                         # Shorthand
```

### `openfang agent kill`

Stop and remove an agent.

```bash
openfang agent kill <agent-id>
openfang agent kill --name my-agent
```

### `openfang agent clone`

Clone an existing agent (copies config and persona).

```bash
openfang agent clone <agent-id> --name new-agent
```

### `openfang agent update`

Update an agent's configuration.

```bash
openfang agent update <agent-id> --model gemini-2.5-pro
openfang agent update <agent-id> --system-prompt "You are a Python expert."
```

---

## Auth Commands

### `openfang auth hash-password`

Generate an Argon2id password hash for dashboard authentication. Prompts for a password interactively and outputs a PHC-format string to paste into `config.toml`.

```bash
openfang auth hash-password
```

The output is an Argon2id hash like `$argon2id$v=19$m=19456,t=2,p=1$...` for use in the `[auth]` config section.

*Added in v0.5.0.*

---

## Workflow Commands

### `openfang workflow list`

List all workflows.

```bash
openfang workflow list
```

### `openfang workflow create`

Create a workflow from a JSON definition file.

```bash
openfang workflow create workflow.json
```

### `openfang workflow run`

Execute a workflow.

```bash
openfang workflow run <workflow-id> --input "Research topic: AI safety"
openfang workflow run <workflow-id> --input-file prompt.txt
```

---

## Trigger Commands

### `openfang trigger list`

List all event triggers.

```bash
openfang trigger list
```

### `openfang trigger create`

Create an event trigger.

```bash
openfang trigger create <agent-id> '{"lifecycle": {}}'
openfang trigger create <agent-id> '{"content_match": {"pattern": "ERROR"}}'
```

### `openfang trigger delete`

Delete a trigger.

```bash
openfang trigger delete <trigger-id>
```

---

## Skill Commands

### `openfang skill list`

List installed and bundled skills.

```bash
openfang skill list
openfang skill list --installed-only
openfang skill list --bundled-only
```

### `openfang skill install`

Install a skill from FangHub or a URL.

```bash
openfang skill install web-summarizer
openfang skill install https://github.com/user/my-skill
openfang skill install ./path/to/local-skill
```

### `openfang skill remove`

Remove an installed skill.

```bash
openfang skill remove <skill-name>
```

### `openfang skill search`

Search the FangHub marketplace.

```bash
openfang skill search kubernetes
openfang skill search "data analysis"
```

### `openfang skill create`

Scaffold a new skill.

```bash
openfang skill create my-skill --runtime python
openfang skill create my-tool --runtime wasm
```

---

## Channel Commands

### `openfang channel list`

List all configured channels and their status.

```bash
openfang channel list
```

### `openfang channel setup`

Interactive setup wizard for a channel.

```bash
openfang channel setup telegram
openfang channel setup discord
openfang channel setup slack
```

### `openfang channel test`

Test channel credentials.

```bash
openfang channel test telegram
openfang channel test email
```

### `openfang channel enable` / `disable`

Enable or disable a channel.

```bash
openfang channel enable telegram
openfang channel disable discord
```

---

## Configuration Commands

### `openfang config show`

Display the current configuration.

```bash
openfang config show
openfang config show --format json
```

### `openfang config edit`

Open config in `$EDITOR`.

```bash
openfang config edit
```

### `openfang config get`

Get a specific config value.

```bash
openfang config get default_model.provider
openfang config get network.listen_addr
```

### `openfang config set`

Set a config value.

```bash
openfang config set default_model.provider groq
openfang config set default_model.model llama-3.3-70b-versatile
```

### `openfang config set-key`

Set a provider API key.

```bash
openfang config set-key groq gsk_...
openfang config set-key anthropic sk-ant-...
openfang config set-key openai sk-...
```

### `openfang config delete-key`

Remove a provider API key.

```bash
openfang config delete-key groq
```

### `openfang config test-key`

Test a provider API key.

```bash
openfang config test-key groq
openfang config test-key anthropic
```

---

## Migration

### `openfang migrate`

Migrate from OpenClaw to OpenFang.

```bash
openfang migrate --from openclaw
openfang migrate --from openclaw --source-dir /path/to/.openclaw
openfang migrate --from openclaw --dry-run     # Preview without applying
```

See [Migration guide](migration.md) for details.

---

## MCP Server

### `openfang mcp`

Start OpenFang as an MCP server over stdio (for IDE integrations).

```bash
openfang mcp
```

This exposes all agents as MCP tools to IDEs like Cursor, VS Code, and Claude Desktop.

---

## Shell Completions

```bash
openfang completion bash   >> ~/.bashrc
openfang completion zsh    >> ~/.zshrc
openfang completion fish   > ~/.config/fish/completions/openfang.fish
openfang completion powershell >> $PROFILE
```

---

## Common Workflows

### Spawn and chat with an agent in 2 steps

```bash
openfang start
openfang agent spawn agents/assistant/agent.toml && openfang chat
```

### Set up Telegram in one command

```bash
openfang channel setup telegram
```

### Run a multi-agent research workflow

```bash
openfang workflow create research-pipeline.json
openfang workflow run <workflow-id> --input "Analyze recent advances in quantum computing"
```

### Install and use a skill

```bash
openfang skill install web-summarizer
openfang chat           # Now the agent has web summarization capability
```
