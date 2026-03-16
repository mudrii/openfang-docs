# Architecture

OpenFang is a 14-crate Rust workspace (137,000+ LOC, 1,767+ tests, zero Clippy warnings) that compiles to a single ~32 MB static binary.

---

## Crate Dependency Graph

```
User Interfaces
├── openfang-cli        Command-line interface + daemon management
└── openfang-desktop    Tauri 2.0 native desktop app (system tray, auto-update)
        ↓
openfang-api            Axum 0.8 HTTP/WS/SSE server (140+ endpoints)
        ↓
openfang-kernel         Orchestration layer
├── openfang-runtime    Agent execution loop, LLM drivers, 53 built-in tools
│                       WASM sandbox (Wasmtime), MCP client/server, A2A protocol
├── openfang-memory     SQLite KV store, vector embeddings, session management
├── openfang-channels   40 channel adapters (unified bridge + router)
├── openfang-skills     60 bundled skills + FangHub marketplace
├── openfang-hands      8 autonomous capability packages
├── openfang-extensions MCP templates, OAuth2, secret vault
└── openfang-wire       OFP peer-to-peer TCP protocol
        ↓
openfang-types          Shared data structures, taint tracking, Ed25519 signing
openfang-migrate        OpenClaw → OpenFang migration tooling
xtask                   Build automation
```

---

## Daemon Boot Sequence

```
openfang start
  1. Load ~/.openfang/config.toml
  2. Initialize SQLite databases (memory, sessions, audit trail)
  3. Load agent manifests from agents directory
  4. Connect to external MCP servers (stdio/SSE)
  5. Start background tasks:
     - Config hot-reload watcher (30-second poll)
     - Cron scheduler
     - Event bus
     - Channel adapters
     - OFP peer listener (if enabled)
  6. Start Axum HTTP server (SO_REUSEADDR)
  7. Write daemon.json (pid, addr, version) with mode 0600
  8. Channel bridge starts routing
```

---

## Agent Execution Loop

```
Message arrives (API / channel / webhook / schedule)
  ↓
Kernel routes to agent
  ↓
Agent run loop (openfang-runtime):
  1. Build system prompt (persona + skills + context)
  2. Load session from SQLite
  3. Apply tool profile filtering (4-10 relevant tools sent to LLM)
  4. Token budget allocation (system + tools + history + response)
  5. Call LLM driver → stream tokens
  6. Detect tool calls in response (<invoke> / function_call)
  7. Check capability permission for each tool
  8. Execute tool (WASM sandbox or native subprocess)
  9. Validate result (taint check, length cap 15K chars)
  10. Append to session
  11. Check loop guard (SHA256 dedup, max iterations)
  12. If session >70% context capacity → compact (summarize)
  13. Emit SSE / WebSocket events to callers
  14. Record token usage + cost metering
```

---

## Memory Substrate

```
~/.openfang/
├── config.toml          Main configuration
├── daemon.json          Running daemon metadata (mode 0600)
├── agents/              Spawned agent records
│   └── {agent-id}/
│       ├── memory.db    SQLite: KV store + vector embeddings
│       ├── sessions.db  Conversation history
│       └── audit.db     Merkle hash chain audit trail
└── skills/              Installed user skills
```

**SQLite memory tables:**
- `kv_store` — structured key-value memory
- `semantic_memory` — `MemoryFragment` records with vector embeddings
- `entities` — knowledge graph nodes
- `relations` — knowledge graph edges
- `usage_stats` — per-model token tracking
- `cron_jobs` — scheduled automation
- `workflow_runs` — execution history

---

## LLM Abstraction Layer

```
LLM Request
  ↓
ModelRoutingConfig → select provider + model by task complexity
  ↓
Driver dispatch:
  ├── AnthropicDriver    → api.anthropic.com (native streaming)
  ├── GeminiDriver       → generativelanguage.googleapis.com
  └── OpenAICompatDriver → any OpenAI-compatible endpoint
  ↓
Fallback chain (if primary fails):
  └── [[fallback_providers]] in config.toml
  ↓
Token usage recorded → hourly quota enforcement
Cost estimation via per-model pricing table
```

---

## Capability System

Every agent declares required capabilities in its `agent.toml`. The kernel enforces these at execution time.

**23 capability types:**

| Category | Capabilities |
|----------|-------------|
| File | `FileRead(glob)`, `FileWrite(glob)` |
| Network | `NetConnect(host)`, `NetListen(port)` |
| Tools | `ToolInvoke(name)`, `ToolAll` |
| LLM | `LlmQuery`, `LlmMaxTokens(n)` |
| Agent | `AgentSpawn`, `AgentMessage(id)`, `AgentKill` |
| Memory | `MemoryRead`, `MemoryWrite` |
| Shell | `ShellExec`, `EnvRead` |
| Wire | `OfpDiscover`, `OfpConnect`, `OfpAdvertise` |
| Economic | `EconSpend`, `EconEarn`, `EconTransfer` |

Glob matching: `*`, `prefix*`, `*suffix`, `prefix*suffix`.
Child agents **cannot** inherit more capabilities than their parent (escalation prevention).

---

## WASM Sandbox

Skills and untrusted tool code run inside Wasmtime with dual metering:

1. **Fuel metering** — instruction-level budget (configurable per-tool)
2. **Epoch interruption** — wall-clock timeout enforced by a watchdog thread

The sandbox provides:
- Workspace-only filesystem access (WASI preopened dirs)
- No environment variable inheritance
- Configurable memory limits
- No arbitrary system call access

---

## Security Architecture (16 Layers)

| Layer | Mechanism |
|-------|-----------|
| 1 | Capability-based access control |
| 2 | WASM dual metering (fuel + epoch + watchdog) |
| 3 | Merkle hash chain audit trail (SHA256, SQLite) |
| 4 | Information flow taint tracking |
| 5 | Ed25519 signed agent manifests |
| 6 | SSRF protection (private IP block, cloud metadata block) |
| 7 | Secret zeroization (`Zeroizing<String>` on all API keys) |
| 8 | OFP mutual auth (HMAC-SHA256 nonces, replay prevention) |
| 9 | Security headers (CSP, X-Frame-Options, HSTS) |
| 10 | GCRA rate limiter (cost-aware token bucket per IP) |
| 11 | Path traversal prevention (canonical paths, symlink checks) |
| 12 | Subprocess sandbox (`env_clear()` + selective passthrough) |
| 13 | Prompt injection scanner (patterns in skill SKILL.md files) |
| 14 | Loop guard (SHA256 tool-call dedup, circuit breaker) |
| 15 | Session repair (7-phase message history validation) |
| 16 | Health endpoint redaction (public = minimal, auth = full) |

---

## OpenFang Wire Protocol (OFP)

Agents on different machines communicate via OFP — a peer-to-peer TCP protocol:

- **Framing:** 4-byte big-endian length prefix + JSON payload
- **Protocol version:** 1
- **Authentication:** HMAC-SHA256 handshake with nonce replay prevention (5-min window)
- **Message types:** Handshake, Discover, AgentMessage, Ping, and corresponding responses + Notifications (AgentSpawned, AgentTerminated, ShuttingDown)
- **Peer registry:** Thread-safe, concurrent peer tracking with fuzzy agent search

---

## Token Management

```
Context window budget allocation:
├── System prompt        (persona + skills)
├── Tool definitions     (filtered to 4-10 relevant tools, saves 15-20K tokens)
├── Conversation history (compacted when needed)
└── Response budget

Compaction triggers:
├── 70% context capacity → token-aware session compaction
└── 90% capacity (emergency) → aggressive trimming + summary injection

Token quota: configurable per-agent hourly limit (rolling 1-hour window)
Default: unlimited (0)
```

---

## Event System

The event bus handles publish-subscribe communication:

**EventTarget:** `Agent`, `Broadcast`, `Pattern`, `System`

**EventPayload types:** `Message`, `ToolResult`, `MemoryUpdate`, `Lifecycle`, `Network`, `System`, `Custom`

**LifecycleEvents:** `Spawned`, `Started`, `Suspended`, `Resumed`, `Terminated`, `Crashed`

**SystemEvents:** `KernelStarted`, `KernelStopping`, `QuotaWarning`, `HealthCheck`, `QuotaEnforced`, `ModelRouted`, `UserAction`, `HealthCheckFailed`

Events support TTL (time-to-live) and correlation IDs for tracking request chains.

---

## Context Window Budget

```
Chars/4 heuristic for token estimation
MAX_TOOL_RESULT_CHARS = 15,000   (reduced from 50K to prevent bloat)
Tool profile filtering: 41 built-in tools → 4-10 for chat agents
Emergency in-loop trimming at 70%/90% thresholds with summary injection
```

---

## Binary Characteristics

| Property | Value |
|----------|-------|
| Language | Rust 2021 Edition |
| Min Rust version | 1.75 |
| Async runtime | Tokio (full features) |
| HTTP framework | Axum 0.8 |
| Database | rusqlite 0.31 (bundled SQLite) |
| WASM runtime | Wasmtime 41 |
| Binary size | ~32 MB (LTO, stripped, opt-level=3) |
| Cold start | <200 ms |
| Idle memory | ~40 MB |
| Test count | 1,767+ |
