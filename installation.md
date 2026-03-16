# Installation

## Requirements

| Requirement | Notes |
|-------------|-------|
| OS | Linux (x86_64/aarch64), macOS (x86_64/Apple Silicon), Windows 10+ |
| LLM API key | Groq has a free tier; Ollama works locally with no key |
| Rust 1.75+ | Only needed for building from source |

---

## Prebuilt Binary (Recommended)

### Linux / macOS

```bash
curl -sSf https://openfang.sh | sh
```

The install script:
1. Auto-detects OS and architecture
2. Fetches the latest release from GitHub
3. Verifies SHA256 checksum
4. On macOS: performs ad-hoc codesigning (Gatekeeper workaround)
5. Detects shell (zsh/bash/fish) and adds install dir to PATH

### Windows (PowerShell)

```powershell
irm https://openfang.sh/install.ps1 | iex
```

### Custom version

```bash
OPENFANG_VERSION=v0.4.4 curl -sSf https://openfang.sh | sh
```

---

## Package Managers

### Homebrew (macOS/Linux)

```bash
brew install openfang
```

### Cargo (from source via crates.io)

```bash
cargo install --git https://github.com/RightNow-AI/openfang openfang-cli
```

---

## Docker

```bash
# Pull latest
docker pull ghcr.io/rightnow-ai/openfang:latest

# Run with docker-compose (includes persistent volume)
docker-compose up -d
```

Example `docker-compose.yml`:
```yaml
services:
  openfang:
    image: ghcr.io/rightnow-ai/openfang:latest
    ports:
      - "4200:4200"
    volumes:
      - openfang-data:/home/openfang/.openfang
    environment:
      - GROQ_API_KEY=${GROQ_API_KEY}
      - OPENFANG_LISTEN=0.0.0.0:4200

volumes:
  openfang-data:
```

---

## Desktop App

Download the platform-specific installer from the [releases page](https://github.com/RightNow-AI/openfang/releases):

| Platform | File |
|----------|------|
| Windows | `.msi` installer |
| macOS | `.dmg` disk image |
| Linux | `.AppImage` or `.deb` package |

The desktop app is built with Tauri 2.0. It embeds the full daemon and provides a system tray icon, OS notifications, and auto-update (Ed25519 signed).

---

## Build from Source

```bash
git clone https://github.com/RightNow-AI/openfang
cd openfang

# Install Rust 1.75+ via rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Development build (faster)
cargo build --profile release-fast -p openfang-cli

# Production build (optimized, LTO)
cargo build --release -p openfang-cli

# Run all tests (1,767+)
cargo test --workspace

# Linting (must pass — zero warnings policy)
cargo clippy --workspace --all-targets -- -D warnings
```

The release binary uses: LTO, codegen-units=1, opt-level=3, stripped symbols → ~32 MB static binary.

---

## systemd Service (Linux)

A service file is included in the repository:

```bash
sudo cp deploy/openfang.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable openfang
sudo systemctl start openfang
```

---

## First Run

```bash
# Set at least one LLM API key
export GROQ_API_KEY="gsk_..."    # Free tier available
# or
export ANTHROPIC_API_KEY="sk-ant-..."
# or
export OLLAMA_BASE_URL="http://localhost:11434"  # No key needed

# Initialize (creates ~/.openfang/config.toml)
openfang init

# YOLO mode: auto-approve all tool calls (development only)
openfang start --yolo

# Normal mode
openfang start                   # Dashboard at http://127.0.0.1:4200
```

---

## Health Check

```bash
openfang doctor            # Diagnose system dependencies
openfang status            # Show daemon status
curl http://127.0.0.1:4200/api/health
```

---

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `GROQ_API_KEY` | Groq (free tier) |
| `ANTHROPIC_API_KEY` | Anthropic Claude |
| `OPENAI_API_KEY` | OpenAI |
| `GEMINI_API_KEY` | Google Gemini |
| `DEEPSEEK_API_KEY` | DeepSeek |
| `OLLAMA_BASE_URL` | Ollama local LLM |
| `TELEGRAM_BOT_TOKEN` | Telegram channel |
| `DISCORD_BOT_TOKEN` | Discord channel |
| `SLACK_BOT_TOKEN` | Slack channel |
| `OPENFANG_LISTEN` | Override listen address (default: `127.0.0.1:4200`) |
| `OPENFANG_API_KEY` | Override API key |
| `OPENFANG_HOME` | Override config directory (default: `~/.openfang`) |
| `RUST_LOG` | Log level (`info`, `debug`, `trace`) |

See `.env.example` in the repository for the full list.

---

## Daemon Info

When running, the daemon writes metadata to `~/.openfang/daemon.json` (mode 0600):

```json
{
  "pid": 12345,
  "listen_addr": "127.0.0.1:4200",
  "started_at": "2026-03-15T10:00:00Z",
  "version": "0.4.4",
  "platform": "linux"
}
```

Only one daemon instance can run at a time — a lock file prevents duplicates.
