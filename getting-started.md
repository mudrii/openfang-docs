# Getting Started

Validated against OpenFang release `v0.5.7`.

This guide stays close to the released source tree and avoids unreleased `main`-branch behavior.

## 1. Install OpenFang

Pick one installer path:

```bash
# Linux / macOS
curl -fsSL https://openfang.sh/install | sh
```

```powershell
# Windows
irm https://openfang.sh/install.ps1 | iex
```

See [installation.md](installation.md) for release-accurate details and source-build options.

## 2. Initialize the local workspace

```bash
openfang init
```

This creates the OpenFang home directory and starter configuration under `~/.openfang/`.

## 3. Configure at least one model provider

The released runtime supports many provider entries, but the quickest path is still to set one working API key or use a local backend.

Examples:

```bash
export GROQ_API_KEY="gsk_..."
```

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

```bash
export OPENAI_API_KEY="sk-..."
```

If you want to use a local model:

```toml
[default_model]
provider = "ollama"
model = "llama3.2"
base_url = "http://localhost:11434"
api_key_env = ""
```

## 4. Verify the environment

```bash
openfang doctor
```

The release CLI also supports:

```bash
openfang status
openfang health
```

## 5. Start the daemon

```bash
openfang start
```

Default local endpoints:

| Surface | Default URL |
|---------|-------------|
| Dashboard | `http://127.0.0.1:4200` |
| API base | `http://127.0.0.1:4200` |

## 6. Spawn a released template

The `v0.5.7` release tree includes `30` bundled agent templates under `agents/`.

Spawn one directly from a manifest:

```bash
openfang agent spawn agents/hello-world/agent.toml
```

Or use the interactive template flow:

```bash
openfang agent new
```

## 7. Chat with an agent

```bash
openfang chat
```

Or target a specific agent:

```bash
openfang agent list
openfang agent chat <agent-id>
```

## 8. Explore Hands

The release also bundles `9` Hands through `openfang-hands`.

```bash
openfang hand list
openfang hand info researcher
openfang hand check-deps browser
openfang hand activate researcher
```

## 9. Explore the release surface

Good next steps:

- [agents.md](agents.md) for manifest shape and lifecycle
- [hands.md](hands.md) for bundled Hands
- [architecture.md](architecture.md) for subsystem layout
- [llm-providers.md](llm-providers.md) for provider and model-catalog guidance
- [cli-reference.md](cli-reference.md) for command coverage
- [api-reference.md](api-reference.md) for the released API families
