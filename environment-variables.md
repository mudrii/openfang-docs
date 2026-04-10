# Environment Variables

Complete reference for all environment variables recognized by OpenFang v0.5.7.

---

## Runtime

| Variable | Default | Description |
|----------|---------|-------------|
| `OPENFANG_HOME` | `~/.openfang` | Override the OpenFang home directory. All configuration, data, skills, and logs are stored here. |
| `OPENFANG_VAULT_KEY` | -- | Base64-encoded AES-256 vault master key for headless/CI environments. When unset, the vault uses OS keyring or prompts interactively. |
| `RUST_LOG` | `openfang=info` | Tracing verbosity filter. Examples: `debug`, `openfang=debug`, `openfang_channels=debug,openfang_runtime=info`. |

---

## LLM Provider API Keys

Set at least one provider key to enable agents. Keys are resolved at runtime via `std::env::var`.

### Tier 1 -- Major Cloud Providers

| Variable | Provider | Required |
|----------|----------|----------|
| `ANTHROPIC_API_KEY` | Anthropic (Claude) | If using Anthropic models |
| `OPENAI_API_KEY` | OpenAI (GPT, o-series) | If using OpenAI models |
| `GEMINI_API_KEY` | Google Gemini | If using Gemini models |
| `GOOGLE_API_KEY` | Google Gemini (alias) | Accepted as fallback when `GEMINI_API_KEY` is unset |
| `DEEPSEEK_API_KEY` | DeepSeek | If using DeepSeek models |
| `GROQ_API_KEY` | Groq | If using Groq models (free tier available) |

### Tier 2 -- Additional Cloud Providers

| Variable | Provider | Required |
|----------|----------|----------|
| `OPENROUTER_API_KEY` | OpenRouter | If using OpenRouter |
| `TOGETHER_API_KEY` | Together AI | If using Together |
| `MISTRAL_API_KEY` | Mistral AI | If using Mistral |
| `FIREWORKS_API_KEY` | Fireworks AI | If using Fireworks |
| `COHERE_API_KEY` | Cohere | If using Cohere |
| `PERPLEXITY_API_KEY` | Perplexity AI | If using Perplexity |
| `XAI_API_KEY` | xAI (Grok) | If using xAI |
| `AI21_API_KEY` | AI21 Labs | If using AI21 |
| `CEREBRAS_API_KEY` | Cerebras | If using Cerebras |
| `SAMBANOVA_API_KEY` | SambaNova | If using SambaNova |
| `HF_API_KEY` | Hugging Face Inference | If using HF Inference API |
| `HUGGINGFACE_API_KEY` | Hugging Face (legacy) | Recognized in subprocess env stripping |
| `REPLICATE_API_TOKEN` | Replicate | If using Replicate |
| `NVIDIA_API_KEY` | NVIDIA NIM | If using NVIDIA NIM |
| `CHUTES_API_KEY` | Chutes.ai | If using Chutes |
| `VENICE_API_KEY` | Venice.ai | If using Venice |

### Tier 3 -- Enterprise / Cloud Platform

| Variable | Provider | Required |
|----------|----------|----------|
| `AZURE_OPENAI_API_KEY` | Azure OpenAI | If using Azure OpenAI |
| `AWS_ACCESS_KEY_ID` | AWS Bedrock | If using Bedrock |
| `GITHUB_TOKEN` | GitHub Copilot | If using GitHub Copilot models |

### Tier 4 -- Chinese Providers

| Variable | Provider | Required |
|----------|----------|----------|
| `DASHSCOPE_API_KEY` | Qwen (Alibaba DashScope) | If using Qwen models |
| `MOONSHOT_API_KEY` | Moonshot (Kimi) | If using Moonshot |
| `KIMI_API_KEY` | Kimi for Code | If using Kimi coding models |
| `MINIMAX_API_KEY` | MiniMax | If using MiniMax |
| `ZHIPU_API_KEY` | Zhipu AI (GLM / CodeGeeX / Z.AI) | If using Zhipu or Z.AI models |
| `QIANFAN_API_KEY` | Baidu Qianfan | If using Qianfan |
| `VOLCENGINE_API_KEY` | Volcano Engine (Doubao) | If using Volcano Engine |

### Local Providers (No Key Required)

| Variable | Provider | Notes |
|----------|----------|-------|
| `OLLAMA_API_KEY` | Ollama | Optional; Ollama does not require a key by default |
| `VLLM_API_KEY` | vLLM | Optional; vLLM does not require a key by default |
| `LMSTUDIO_API_KEY` | LM Studio | Optional; LM Studio does not require a key by default |
| `LEMONADE_API_KEY` | Lemonade | Optional; local provider |

---

## Search & Tool API Keys

| Variable | Provider | Used By |
|----------|----------|---------|
| `BRAVE_API_KEY` | Brave Search | `web_search` tool, search skills |
| `TAVILY_API_KEY` | Tavily | `web_search` tool, research skills |
| `ELEVENLABS_API_KEY` | ElevenLabs | `tts_speak` tool |

---

## Channel Tokens

### Telegram

| Variable | Description |
|----------|-------------|
| `TELEGRAM_BOT_TOKEN` | Bot token from @BotFather (format: `123456:ABC-DEF...`) |

### Discord

| Variable | Description |
|----------|-------------|
| `DISCORD_BOT_TOKEN` | Bot token from Discord Developer Portal (59+ characters) |

### Slack

| Variable | Description |
|----------|-------------|
| `SLACK_APP_TOKEN` | App-level token for Socket Mode (format: `xapp-1-...`) |
| `SLACK_BOT_TOKEN` | Bot user OAuth token (format: `xoxb-...`) |

### WhatsApp (Cloud API)

| Variable | Description |
|----------|-------------|
| `WHATSAPP_ACCESS_TOKEN` | Meta Cloud API access token |
| `WHATSAPP_VERIFY_TOKEN` | Webhook verification token |

### WhatsApp (Web Gateway)

| Variable | Description |
|----------|-------------|
| `WHATSAPP_WEB_GATEWAY_URL` | URL of the WhatsApp Web gateway process. Auto-set when the gateway starts with the daemon. Set manually for external gateways. |
| `WHATSAPP_GATEWAY_PORT` | Port for the WhatsApp Web gateway Node.js process (default: `3009`). Passed to the gateway subprocess. |
| `OPENFANG_URL` | Callback URL for the WhatsApp gateway to reach the OpenFang API (e.g. `http://127.0.0.1:4200`). Passed to the gateway subprocess. |
| `OPENFANG_DEFAULT_AGENT` | Default agent name for WhatsApp messages. Passed to the gateway subprocess. |

---

## Subprocess Sandbox

When OpenFang spawns child processes (shell tool, skill scripts), the environment is sanitized. Only these variables are inherited:

**Safe variables (always inherited):**
`PATH`, `HOME`, `TMPDIR`, `TMP`, `TEMP`, `LANG`, `LC_ALL`, `TERM`

**Safe variables (Windows only):**
`USERPROFILE`, `SYSTEMROOT`, `APPDATA`, `LOCALAPPDATA`, `COMSPEC`, `WINDIR`, `PATHEXT`

**Stripped variables:** All `*_API_KEY`, `*_TOKEN`, `*_SECRET`, `*_PASSWORD` variables are removed from child processes to prevent secret leakage. The agent's own provider key is injected only when explicitly allowed.

---

## Driver Subprocess Isolation

When OpenFang delegates to CLI-based LLM drivers (Claude Code, Qwen Code), it strips the following sensitive variables from the subprocess environment:

`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, `GOOGLE_API_KEY`, `GROQ_API_KEY`, `DEEPSEEK_API_KEY`, `MISTRAL_API_KEY`, `TOGETHER_API_KEY`, `FIREWORKS_API_KEY`, `OPENROUTER_API_KEY`, `PERPLEXITY_API_KEY`, `COHERE_API_KEY`, `AI21_API_KEY`, `CEREBRAS_API_KEY`, `SAMBANOVA_API_KEY`, `HUGGINGFACE_API_KEY`, `XAI_API_KEY`, `REPLICATE_API_TOKEN`, `BRAVE_API_KEY`, `TAVILY_API_KEY`, `ELEVENLABS_API_KEY`

Additionally, any variable ending with `_SECRET`, `_TOKEN`, or `_PASSWORD` is removed.

---

## Migration

| Variable | Description |
|----------|-------------|
| `OPENCLAW_STATE_DIR` | Override the OpenClaw state directory for `openfang migrate --from openclaw`. Defaults to `~/.openclaw`. |

---

## Desktop App

| Variable | Default | Description |
|----------|---------|-------------|
| `RUST_LOG` | `openfang=info,tauri=info` | Controls tracing verbosity in the Tauri desktop app. |

All other OpenFang environment variables apply as normal since the desktop app boots the same kernel.

---

## Python Runtime

When OpenFang runs Python skill scripts, these additional variables are forwarded if set:

`PYTHONPATH`, `VIRTUAL_ENV`, `CONDA_DEFAULT_ENV`

---

## Setting Environment Variables

### Shell (current session)

```bash
export GROQ_API_KEY="gsk_..."
export TELEGRAM_BOT_TOKEN="123456:ABC-DEF..."
```

### Persistent (secrets.env)

OpenFang loads `~/.openfang/secrets.env` automatically on startup. Add variables there:

```
GROQ_API_KEY=gsk_...
TELEGRAM_BOT_TOKEN=123456:ABC-DEF...
```

The CLI `channel setup` and `config set-key` commands write to this file automatically.

### Persistent (shell profile)

Add `export` lines to `~/.bashrc`, `~/.zshrc`, or equivalent.

### Docker

```bash
docker run -e GROQ_API_KEY=gsk_... -e TELEGRAM_BOT_TOKEN=... ghcr.io/RightNow-AI/openfang
```

### Vault (encrypted storage)

```bash
openfang vault store GROQ_API_KEY gsk_...
openfang vault store TELEGRAM_BOT_TOKEN 123456:ABC-DEF...
```

The vault uses AES-256-GCM encryption with Argon2id key derivation, stored in `~/.openfang/vault.enc`.
