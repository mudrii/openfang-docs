# Contributing

OpenFang is an open-source project. Contributions are welcome via GitHub pull requests.

**Repository:** https://github.com/RightNow-AI/openfang

---

## Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Rust | 1.75+ | `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs \| sh` |
| Python | 3.8+ | Optional (for Python skills) |
| LLM API key | any | Groq free tier works for testing |

---

## Development Setup

```bash
git clone https://github.com/RightNow-AI/openfang
cd openfang

# Set a test API key
export GROQ_API_KEY="gsk_..."

# Development build (faster, not optimized)
cargo build --profile release-fast -p openfang-cli

# Full test suite (1,767+ tests)
cargo test --workspace

# Linting (zero warnings policy — must pass before commit)
cargo clippy --workspace --all-targets -- -D warnings

# Formatting (required before commit)
cargo fmt --all

# Security audit
cargo audit
```

---

## Workspace Structure

```
openfang/
├── crates/
│   ├── openfang-types/       # Shared data structures
│   ├── openfang-memory/      # SQLite memory substrate
│   ├── openfang-runtime/     # Agent loop, tools, LLM drivers
│   ├── openfang-kernel/      # Orchestration, workflows, RBAC
│   ├── openfang-api/         # Axum HTTP server
│   ├── openfang-channels/    # 40 channel adapters
│   ├── openfang-skills/      # 60 skills, FangHub
│   ├── openfang-hands/       # 8 autonomous Hands
│   ├── openfang-wire/        # OFP peer protocol
│   ├── openfang-cli/         # CLI interface
│   ├── openfang-desktop/     # Tauri desktop app
│   ├── openfang-extensions/  # MCP, OAuth2, vault
│   └── openfang-migrate/     # OpenClaw migration
├── agents/                   # 30 agent templates
├── docs/                     # Internal documentation
├── sdk/                      # JS and Python SDKs
├── xtask/                    # Build automation
└── scripts/                  # Install scripts
```

---

## Code Style

- **Doc comments** on all public types and functions
- Use `thiserror` for error types — no manual `Display` implementations
- **No `unwrap()`** in library code — use `?` and proper error propagation
- Every feature needs tests — use `tempfile::TempDir` for filesystem isolation
- `cargo fmt --all` before every commit
- `cargo clippy --workspace --all-targets -- -D warnings` must pass with zero warnings

---

## Mandatory Live Testing

After every feature, run live integration tests (unit tests alone are insufficient):

```bash
# 1. Build
cargo build --release -p openfang-cli

# 2. Start daemon
export GROQ_API_KEY="gsk_..."
./target/release/openfang start &
sleep 2

# 3. Verify health
curl -s http://127.0.0.1:4200/api/health

# 4. Spawn a test agent
AGENT_ID=$(curl -s -X POST http://127.0.0.1:4200/api/agents \
  -H "Content-Type: application/json" \
  -d '{"template": "hello-world", "name": "test"}' | python3 -c "import sys,json; print(json.load(sys.stdin)['id'])")

# 5. Send a real LLM message
curl -s -X POST "http://127.0.0.1:4200/api/agents/$AGENT_ID/message" \
  -H "Content-Type: application/json" \
  -d '{"message": "Say hello in exactly 5 words."}'

# 6. Check budget/usage
curl -s http://127.0.0.1:4200/api/budget

# 7. Kill test agent
curl -s -X DELETE "http://127.0.0.1:4200/api/agents/$AGENT_ID"
```

---

## Adding a New Agent Template

1. Create directory: `agents/my-template/`
2. Create `agents/my-template/agent.toml`:

```toml
[agent]
name = "my-template"
description = "What this agent does"
version = "1.0.0"
tier = 3                  # 1=Frontier, 2=Smart, 3=Balanced, 4=Fast

[model]
provider = "groq"
model = "llama-3.3-70b-versatile"

[persona]
system_prompt = """
You are an expert in [domain]. [Detailed instructions...]
"""

[capabilities]
required = ["ToolInvoke(web_fetch)", "MemoryRead", "MemoryWrite"]
```

3. Submit a PR with the new template

---

## Adding a Channel Adapter

1. Create `crates/openfang-channels/src/my_channel.rs`
2. Implement the `ChannelAdapter` trait:

```rust
use async_trait::async_trait;
use crate::{ChannelAdapter, ChannelBridgeHandle, ChannelMessage, OutputFormat};

pub struct MyChannelAdapter {
    config: MyChannelConfig,
}

#[async_trait]
impl ChannelAdapter for MyChannelAdapter {
    fn name(&self) -> &str {
        "my_channel"
    }

    async fn start(&self, bridge: Arc<dyn ChannelBridgeHandle>) -> anyhow::Result<()> {
        // Start listening for messages
        // Call bridge.send_message() to forward to kernel
        Ok(())
    }

    async fn stop(&self) -> anyhow::Result<()> {
        Ok(())
    }

    async fn send(&self, message: &ChannelMessage) -> anyhow::Result<()> {
        // Send message to the platform
        Ok(())
    }

    fn supported_formats(&self) -> Vec<OutputFormat> {
        vec![OutputFormat::Markdown, OutputFormat::PlainText]
    }
}
```

3. Register in `crates/openfang-channels/src/lib.rs`
4. Add config struct to `crates/openfang-types/src/config.rs`
5. Wire into channel bridge in `crates/openfang-api/src/channel_bridge.rs`
6. Add CLI setup wizard step

---

## Adding a New Tool

Tools are implemented in `crates/openfang-runtime/src/tool_runner.rs`:

```rust
// Add to the tool dispatch match arm
"my_new_tool" => {
    let input: MyToolInput = serde_json::from_value(call.input.clone())?;
    let result = execute_my_tool(input, &context).await?;
    Ok(ToolResult {
        tool_use_id: call.id.clone(),
        content: serde_json::to_string(&result)?,
        is_error: false,
    })
}
```

Register the tool definition in the tool catalog so it appears in `/api/tools`.

---

## Adding a New Bundled Skill

1. Create directory: `crates/openfang-skills/bundled/my-skill/`
2. Create `SKILL.md`:

```markdown
---
name: my-skill
version: 0.1.0
description: Expert in my domain
author: yourname
license: MIT
tags: [domain, expert]
runtime: promptonly
---

## My Domain Expert

### Key Principles
...

### Common Patterns
...

### Pitfalls to Avoid
...
```

3. The skill is automatically compiled into the binary via `include_str!`
4. No Rust code changes needed

---

## Adding a Web API Handler

```rust
// crates/openfang-api/src/routes.rs

pub async fn get_my_resource(
    State(state): State<AppState>,
    Path(agent_id): Path<String>,
) -> impl IntoResponse {
    let kernel = state.kernel.lock().await;
    // ... business logic
    Json(json!({"success": true, "data": result}))
}

// Register in build_router():
.route("/api/agents/:id/my-resource", get(get_my_resource))
```

---

## Pull Request Process

1. Fork: https://github.com/RightNow-AI/openfang/fork
2. Create branch: `git checkout -b feat/my-feature`
3. Implement feature with tests
4. Run `cargo test --workspace` — all tests must pass
5. Run `cargo clippy --workspace --all-targets -- -D warnings` — zero warnings
6. Run `cargo fmt --all` — code must be formatted
7. Run live integration tests (see above)
8. Commit with conventional commit message: `feat: add my feature`
9. Open PR against `main`

**Commit format:**
```
feat: add trading hand
fix: telegram typing indicator refresh race condition
docs: add workflow examples
chore: bump wasmtime to 42
test: add integration test for session compaction
```

---

## AI-Assisted Contributions

The `evolve_core` and `contribute` skills allow OpenFang to improve itself:

```bash
# Set GITHUB_TOKEN with fork + PR permissions
openfang config set-key GITHUB_TOKEN ghp_...

# Agent will open a draft PR with improvements
# (Requires explicit user confirmation due to needs_confirmation = true)
```
