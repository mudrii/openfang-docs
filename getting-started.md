# Getting Started

Get OpenFang v0.5.7 running and interact with your first agent in under five minutes.

## 1. Install

### Linux / macOS

```bash
curl -fsSL https://openfang.sh/install | sh
```

### Windows (PowerShell)

```powershell
irm https://openfang.sh/install.ps1 | iex
```

### Build from source

```bash
cargo install --git https://github.com/RightNow-AI/openfang openfang-cli
```

Or clone and build locally:

```bash
git clone https://github.com/RightNow-AI/openfang.git
cd openfang
cargo build --release -p openfang-cli
# Binary is at target/release/openfang
```

Minimum Rust version: 1.75. The release profile uses LTO and `codegen-units = 1` for a ~32 MB binary.

## 2. Initialize

```bash
openfang init
```

This creates the `~/.openfang/` directory with:

- `config.toml` -- main configuration file
- `agents/` -- agent manifest storage
- `skills/` -- user-installed skills
- `hands/` -- hand instance data
- `data/` -- SQLite databases (sessions, memory, usage)
- `integrations.toml` -- MCP integration state

## 3. Configure a model provider

Set an API key for at least one LLM provider. OpenFang auto-detects configured providers at startup.

```bash
# Any one of these is enough to get started:
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export GEMINI_API_KEY="AI..."
export GROQ_API_KEY="gsk_..."
export DEEPSEEK_API_KEY="sk-..."
```

For local models (no API key needed):

```bash
# Ollama
ollama pull llama3.2
```

Then set in `~/.openfang/config.toml`:

```toml
[default_model]
provider = "ollama"
model = "llama3.2"
base_url = "http://localhost:11434"
api_key_env = ""
```

OpenFang supports 27 LLM providers. See [llm-providers.md](llm-providers.md) for the full catalog.

## 4. Verify the environment

```bash
openfang doctor
```

This checks:

- OpenFang home directory exists
- Configuration is valid
- At least one LLM provider is reachable
- Optional dependencies (Docker, Playwright, ffmpeg, etc.)

Also available:

```bash
openfang status    # Daemon status and agent count
openfang health    # System health summary
```

## 5. Start the daemon

```bash
openfang start
```

The daemon boots the kernel, loads agents and skills, connects channel bridges, and starts the HTTP server.

| Surface | Default URL |
|---------|-------------|
| Dashboard | http://127.0.0.1:4200 |
| API | http://127.0.0.1:4200/api/ |

The daemon writes `~/.openfang/daemon.json` so the CLI can discover it. To run in the foreground with logs:

```bash
RUST_LOG=info openfang start
```

## 6. Chat with an agent

### Interactive TUI

```bash
openfang chat
```

This opens the Ratatui-based terminal UI connected to the default agent.

### Target a specific agent

```bash
openfang agent list
openfang agent chat <agent-id>
```

### Via the API

```bash
# List agents
curl -s http://127.0.0.1:4200/api/agents | jq

# Send a message
curl -s -X POST http://127.0.0.1:4200/api/agents/<agent-id>/message \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello, what can you do?"}'
```

### Via the dashboard

Open http://127.0.0.1:4200 in a browser. The dashboard provides a web chat interface, agent management, channel configuration, Hand marketplace, and system monitoring.

## 7. Spawn an agent from a template

```bash
# Interactive template selection
openfang agent new

# Or spawn directly from a manifest file
openfang agent spawn agents/hello-world/agent.toml
```

Agent manifests are TOML files that define the agent's name, model, system prompt, capabilities, and tools. See [agents.md](agents.md) for the full manifest format.

## 8. Activate a Hand

Hands are curated autonomous agent packages. The 9 bundled Hands are: browser, clip, collector, infisical-sync, lead, predictor, researcher, trader, and twitter.

```bash
# List available Hands
openfang hand list

# See details and requirements for a Hand
openfang hand info researcher

# Check if dependencies are met
openfang hand check-deps browser

# Activate a Hand
openfang hand activate researcher
```

Hands can also be activated from the dashboard's Hands tab. See [hands.md](hands.md) for details on each bundled Hand.

## 9. Enable a channel

Connect OpenFang to a messaging platform by configuring a channel adapter in `config.toml`:

```toml
[channels.telegram]
bot_token_env = "TELEGRAM_BOT_TOKEN"
allowed_users = ["123456789"]

[channels.discord]
bot_token_env = "DISCORD_BOT_TOKEN"
```

Or configure channels through the dashboard's Channels tab, which provides guided setup for all 43 supported adapters.

After configuring, restart the daemon:

```bash
openfang stop
openfang start
```

## 10. Install a skill

Skills extend agent capabilities. 61 skills ship bundled; additional skills can be installed from ClawHub or local directories:

```bash
# List bundled skills
openfang skill list

# Install from ClawHub marketplace
openfang skill install <skill-name>

# Install from a local directory
openfang skill install ./my-skill/
```

See [skill-development.md](skill-development.md) for authoring custom skills.

## What next

- [architecture.md](architecture.md) -- understand the 14-crate workspace structure
- [agents.md](agents.md) -- agent manifest format and lifecycle
- [hands.md](hands.md) -- bundled Hand catalog and activation
- [channels.md](channels.md) -- channel adapter setup guides
- [llm-providers.md](llm-providers.md) -- full provider catalog and model routing
- [cli-reference.md](cli-reference.md) -- complete CLI command reference
- [api-reference.md](api-reference.md) -- REST/WebSocket/SSE API surface
- [security.md](security.md) -- capability-based security model
- [configuration.md](configuration.md) -- full `config.toml` reference
