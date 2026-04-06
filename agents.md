# Agents

Validated against OpenFang release `v0.5.7`.

An OpenFang agent is a persisted manifest plus runtime state: identity, model settings, capabilities, resources, memory, session history, and optional automation behavior.

## Agent states

The released `AgentState` enum includes:

- `created`
- `running`
- `suspended`
- `terminated`
- `crashed`

## Agent modes

The released `AgentMode` enum includes:

| Mode | Effect |
|------|--------|
| `observe` | no tool access |
| `assist` | read-only tool subset |
| `full` | all granted tools remain available |

## Released manifest shape

The `v0.5.7` templates use a flat manifest layout rather than nested `[agent]` or `[persona]` sections.

Example:

```toml
name = "assistant"
version = "0.1.0"
description = "General-purpose assistant"
author = "openfang"
module = "builtin:chat"
tags = ["general", "assistant"]

[model]
provider = "default"
model = "default"
max_tokens = 8192
temperature = 0.5
system_prompt = """You are a helpful AI agent."""

[[fallback_models]]
provider = "default"
model = "gemini-2.0-flash"
api_key_env = "GEMINI_API_KEY"

[resources]
max_llm_tokens_per_hour = 300000
max_concurrent_tools = 10

[capabilities]
tools = ["file_read", "file_write", "web_fetch"]
network = ["*"]
memory_read = ["*"]
memory_write = ["self.*", "shared.*"]

[autonomous]
max_iterations = 100
```

## Released template count

For `v0.5.7`, the released tree contains `30` bundled agent templates under `agents/`.

Spawn paths:

```bash
openfang agent new
openfang agent spawn agents/assistant/agent.toml
openfang agent spawn agents/hello-world/agent.toml
```

## Common agent commands

```bash
openfang agent list
openfang agent chat <agent-id>
openfang agent kill <agent-id>
openfang agent set <agent-id> model <model-id>
openfang sessions
openfang message <agent> "hello"
```

## Related docs

- [agent-templates.md](agent-templates.md) for the released template catalog
- [hands.md](hands.md) for autonomous bundled Hands
- [cli-reference.md](cli-reference.md) for command coverage
- [api-reference.md](api-reference.md) for agent-related endpoints
