# Architecture

OpenFang v0.5.7 is a Cargo workspace with 14 crates (13 under `crates/` plus `xtask`), totaling ~178K lines of code across 253 source files. The workspace compiles to a single ~32 MB statically linked binary with 2,077+ tests and zero clippy warnings.

## Workspace Members

### Core crates

These six crates form the vertical spine from shared types up through the user-facing CLI.

| Crate | Description | Key dependencies |
|-------|-------------|-----------------|
| `openfang-types` | Shared types: agents, tools, events, config, capabilities, taint labels, model catalog entries, manifest signing. Contains no business logic. | serde, chrono, uuid, ed25519-dalek |
| `openfang-memory` | SQLite-backed persistence with three storage backends: structured key-value store, semantic text search (LIKE matching, Qdrant planned), and a knowledge graph for entities/relations. | openfang-types, rusqlite |
| `openfang-runtime` | Agent execution loop, LLM driver abstraction (3 native drivers: Anthropic, Gemini, OpenAI-compatible), 59 built-in tools, WASM sandboxing, MCP client, A2A protocol, browser automation, media understanding, taint tracking, audit helpers. | openfang-types, openfang-memory, openfang-skills, wasmtime, rmcp, reqwest |
| `openfang-kernel` | System assembly: agent registry, capability manager, event bus, scheduler, supervisor, workflow engine, trigger engine, cron scheduler, approval manager, metering engine, config hot-reload. | openfang-types, openfang-memory, openfang-runtime, openfang-skills, openfang-hands, openfang-extensions, openfang-wire, openfang-channels |
| `openfang-api` | HTTP/WebSocket/SSE server (Axum), REST routes, dashboard, rate limiting, session auth (Argon2id), OpenAI-compatible endpoint, channel bridge orchestration. | openfang-kernel, openfang-runtime, openfang-types, axum, tower-http, governor |
| `openfang-cli` | End-user CLI and TUI (Ratatui), daemon lifecycle (`start`/`stop`), `init` wizard, `chat`, `agent` subcommands, shell completions, migration commands. | openfang-kernel, openfang-api, openfang-types, clap, ratatui |

### Integration crates

| Crate | Description | Key dependencies |
|-------|-------------|-----------------|
| `openfang-channels` | 43 channel adapters converting platform messages into unified `ChannelMessage` events. Covers Telegram, Discord, Slack, WhatsApp, Signal, Matrix, Teams, email (SMTP/IMAP), IRC, MQTT, and 30+ more across messaging, social, enterprise, and niche categories. | openfang-types, reqwest, tokio-tungstenite, lettre, imap, rumqttc, prost |
| `openfang-skills` | Skill registry, bundled skill loader (61 skills via `include_str!()`), marketplace client (ClawHub), OpenClaw compatibility layer, skill verification/security scanning, installer. Skill runtimes: Python, WASM, Node, Shell, Builtin, PromptOnly. | openfang-types, walkdir, zip |
| `openfang-hands` | Hand registry and lifecycle for 9 bundled Hands (browser, clip, collector, infisical-sync, lead, predictor, researcher, trader, twitter). A Hand is a curated autonomous agent package that users activate from a marketplace-style UI. | openfang-types, dashmap |
| `openfang-extensions` | Integration system: 25 bundled MCP server templates, AES-256-GCM encrypted credential vault, OAuth2 PKCE flows (Google/GitHub/Microsoft/Slack), health monitor with auto-reconnect. | openfang-types, aes-gcm, argon2, reqwest |
| `openfang-wire` | OpenFang Protocol (OFP) for peer-to-peer agent networking. PeerNode listens for connections, PeerRegistry tracks known peers, WireMessage is the JSON-RPC framed protocol. | openfang-types, hmac, sha2, dashmap |
| `openfang-migrate` | Migration engine for importing agents, memory, sessions, skills, and channel configs from OpenClaw (LangChain and AutoGPT planned). | openfang-types, serde_yaml, json5, walkdir |

### Build crate

| Crate | Description |
|-------|-------------|
| `openfang-desktop` | Tauri 2.0 native desktop shell. Embeds the kernel and API server, adds system tray, global shortcuts, notifications, auto-update. |
| `xtask` | Build and release automation (cross-compilation, packaging, checksums). |

## Dependency Graph

```
openfang-cli -----> openfang-api -----> openfang-kernel
    |                   |                   |   |   |   |
    |                   |                   |   |   |   +-- openfang-wire
    |                   |                   |   |   +------ openfang-channels
    |                   |                   |   +---------- openfang-hands
    |                   |                   +-------------- openfang-extensions
    |                   |                   |
    +-------------------+-------> openfang-runtime ------> openfang-skills
                                      |         |
                                      |         +-------> openfang-memory
                                      |                       |
                                      +-----------------------+---> openfang-types
```

`openfang-desktop` parallels `openfang-cli`, depending on `openfang-kernel` and `openfang-api` directly. `openfang-migrate` depends only on `openfang-types`. `xtask` has no internal dependencies.

## Key Architectural Patterns

### KernelHandle trait

The `KernelHandle` trait (defined in `openfang-runtime/src/kernel_handle.rs`) breaks the circular dependency between runtime and kernel. The runtime needs to call kernel operations (spawn agents, send messages, access shared memory, manage tasks) during tool execution, but the kernel depends on the runtime for agent loops. The solution:

1. `openfang-runtime` defines the `KernelHandle` trait with methods like `spawn_agent()`, `send_to_agent()`, `memory_store()`, `task_post()`, `hand_activate()`, `send_channel_message()`, etc.
2. `openfang-kernel` implements `KernelHandle` on `OpenFangKernel`.
3. The kernel passes itself (as `Arc<dyn KernelHandle>`) into `run_agent_loop()`.

This means the runtime crate never imports the kernel crate, keeping the dependency graph acyclic.

### AppState bridge

`AppState` (in `openfang-api/src/server.rs`) bridges the kernel into the Axum HTTP layer. It wraps:

- `kernel: Arc<OpenFangKernel>` -- the kernel reference shared across all route handlers
- `bridge_manager` -- channel bridge lifecycle (Telegram bots, Discord gateways, etc.)
- `channels_config` -- hot-reloadable channel configuration
- `budget_config` -- runtime-updatable budget limits
- `provider_probe_cache` -- LLM provider health check results
- `clawhub_cache` -- marketplace metadata cache
- `shutdown_notify` -- coordinated graceful shutdown

Route handlers receive `AppState` via Axum's state extraction and call into the kernel.

### Daemon detection

The CLI and daemon communicate through a `daemon.json` file written to `~/.openfang/`:

```json
{
  "pid": 12345,
  "listen_addr": "127.0.0.1:4200",
  "started_at": "2026-04-08T12:00:00Z",
  "version": "0.5.7",
  "platform": "aarch64-apple-darwin"
}
```

CLI commands read this file to discover the running daemon's address, then issue HTTP requests to the API. If no daemon is running, the CLI reports the status and suggests `openfang start`.

### Capability-based security

Every agent declares its capabilities in its manifest. The kernel's `CapabilityManager` enforces these at runtime before any tool execution. Capability types include:

- `FileRead(glob)` / `FileWrite(glob)` -- filesystem access scoped by pattern
- `NetConnect(pattern)` / `NetListen(port)` -- network access
- `ToolInvoke(name)` / `ToolAll` -- tool execution permissions
- `ShellExec(pattern)` -- shell command filtering
- `AgentSpawn` / `AgentMessage(pattern)` / `AgentKill(pattern)` -- inter-agent permissions
- `MemoryRead(scope)` / `MemoryWrite(scope)` -- memory access scoping
- `OfpDiscover` / `OfpConnect(pattern)` -- OFP networking permissions

Child agents inherit capabilities from their parent. The `spawn_agent_checked()` method on `KernelHandle` verifies that every capability in a child manifest is covered by the parent's grants.

## Boot Sequence

1. Load `~/.openfang/config.toml` (or create defaults via `openfang init`)
2. Initialize data directories and SQLite state (`MemorySubstrate`)
3. Build the model catalog (27 providers, detect API key availability)
4. Create the kernel: agent registry, capability manager, event bus, scheduler, supervisor, workflow engine, trigger engine, cron scheduler, approval manager, metering engine, audit log, WASM sandbox, browser manager, media engine, TTS engine, hand registry, extension registry, credential vault
5. Load agents from `~/.openfang/agents/` and bundled templates
6. Load skills: 61 bundled + user-installed from `~/.openfang/skills/`
7. Connect MCP servers (from config + installed extensions)
8. Start channel bridges (Telegram, Discord, Slack, etc.)
9. Start OFP peer node (if configured)
10. Launch background agents (autonomous agents, scheduled tasks)
11. Bind the Axum HTTP server on `127.0.0.1:4200` (default)
12. Write `daemon.json` for CLI discovery
13. Serve API, dashboard, and WebSocket connections

## Agent Execution Model

The agent loop (`openfang-runtime/src/agent_loop.rs`) runs each conversation turn:

1. **Prompt construction** -- system prompt + skill context + session history + tool definitions
2. **Session loading and repair** -- recovers from truncated or corrupted session state
3. **Model invocation** -- sends to the configured LLM via the appropriate driver
4. **Tool-call detection** -- parses tool_use blocks from the response
5. **Capability check** -- verifies the agent has permission for each requested tool
6. **Approval gate** -- checks if the tool requires human approval (configurable per-tool policy)
7. **Taint tracking** -- validates inputs against taint sinks (e.g., shell metacharacter injection)
8. **Tool execution** -- runs the tool and captures output (with output truncation for large results)
9. **Usage accounting** -- records token usage and cost via the metering engine
10. **Loop guard** -- prevents infinite loops (configurable iteration limit)
11. **Context compaction** -- compresses session history when context pressure grows

The loop repeats steps 3-11 until the model produces a final response (no more tool calls) or hits a guard limit.

## LLM Driver Architecture

Three native driver implementations handle protocol differences:

| Driver | Protocol | Providers |
|--------|----------|-----------|
| `AnthropicDriver` | Anthropic Messages API | Anthropic, Kimi Coding |
| `GeminiDriver` | Google Generative Language API | Gemini |
| `OpenAIDriver` | OpenAI Chat Completions API | OpenAI, Groq, DeepSeek, Together, Mistral, Fireworks, Ollama, vLLM, LM Studio, Perplexity, Cohere, AI21, Cerebras, SambaNova, HuggingFace, xAI, Replicate, OpenRouter, Chutes, Venice, NVIDIA NIM, MiniMax, Zhipu, Qwen, Qianfan, Volcengine, Azure OpenAI, and any custom OpenAI-compatible endpoint |

Additionally, `ClaudeCodeDriver`, `QwenCodeDriver`, `CopilotDriver`, and `VertexAIDriver` handle subprocess-based or OAuth-authenticated providers.

## Security Layers

Key security subsystems in v0.5.7:

- **Capability enforcement** -- immutable per-agent permissions checked before every tool call
- **WASM dual metering** -- fuel-based execution limits for untrusted plugin code (Wasmtime)
- **Taint tracking** -- labels data with provenance (`ExternalNetwork`, etc.) and blocks dangerous sinks
- **Merkle audit trail** -- hash-chained log of all agent actions for tamper detection
- **Manifest signing** -- Ed25519 signatures on agent and skill manifests
- **SSRF filtering** -- blocks private/internal IPs in web_fetch and web_search tools
- **Secret zeroization** -- API keys and credentials are zeroized from memory after use
- **Subprocess sandboxing** -- shell commands run with metacharacter injection detection
- **Docker sandboxing** -- optional container-isolated execution via `docker_exec`
- **Loop guard** -- configurable iteration cap prevents runaway agent loops
- **Session repair** -- automatic recovery from corrupted conversation state
- **Argon2id password hashing** -- dashboard authentication
- **Rate limiting** -- per-IP and per-agent request throttling (Governor)
- **Approval workflows** -- configurable human-in-the-loop gates for sensitive tools
- **Capability inheritance** -- child agents cannot exceed parent's permissions
- **Health endpoint redaction** -- sensitive config values omitted from status responses
