# LLM Providers

Validated against OpenFang `v0.5.7` source (`crates/openfang-runtime/src/drivers/mod.rs` and `crates/openfang-runtime/src/model_catalog.rs`).

## Catalog Counts

The runtime model catalog contains:

- **41** provider entries in `builtin_providers()`
- **37** entries in `known_providers()` (the driver dispatch list)
- **197** builtin model entries in `builtin_models()`

These counts include brand variants (e.g., `zhipu` and `zhipu_coding`), coding-specific provider variants, and CLI-backed integrations. This page groups providers by driver family and documents each with its env var name, base URL, and API key requirement.

---

## Driver Families

OpenFang uses three native driver implementations plus several special-case drivers:

| Driver | API format | Source file |
|--------|-----------|-------------|
| **AnthropicDriver** | Anthropic Messages API | `drivers/anthropic.rs` |
| **GeminiDriver** | Google Gemini API | `drivers/gemini.rs` |
| **OpenAIDriver** | OpenAI Chat Completions API (and compatibles) | `drivers/openai.rs` |
| **FallbackDriver** | Wraps multiple drivers in a chain | `drivers/fallback.rs` |
| **CopilotDriver** | GitHub Copilot (wraps OpenAI with token exchange) | `drivers/copilot.rs` |
| **ClaudeCodeDriver** | Subprocess-based (Claude Code CLI) | `drivers/claude_code.rs` |
| **QwenCodeDriver** | Subprocess-based (Qwen Code CLI) | `drivers/qwen_code.rs` |
| **VertexAIDriver** | Google Cloud Vertex AI (OAuth) | `drivers/vertex.rs` |

The `OpenAIDriver` handles the majority of providers since most expose an OpenAI-compatible `/v1/chat/completions` endpoint. Azure OpenAI uses a variant of `OpenAIDriver` with `api-key` header authentication.

---

## Provider Reference

### Cloud Providers (API key required)

| Provider ID | Display Name | Env Var | Base URL | Driver |
|-------------|-------------|---------|----------|--------|
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | `https://api.anthropic.com` | Anthropic |
| `openai` | OpenAI | `OPENAI_API_KEY` | `https://api.openai.com/v1` | OpenAI |
| `gemini` / `google` | Google Gemini | `GEMINI_API_KEY` (or `GOOGLE_API_KEY`) | `https://generativelanguage.googleapis.com` | Gemini |
| `deepseek` | DeepSeek | `DEEPSEEK_API_KEY` | `https://api.deepseek.com/v1` | OpenAI |
| `groq` | Groq | `GROQ_API_KEY` | `https://api.groq.com/openai/v1` | OpenAI |
| `openrouter` | OpenRouter | `OPENROUTER_API_KEY` | `https://openrouter.ai/api/v1` | OpenAI |
| `mistral` | Mistral AI | `MISTRAL_API_KEY` | `https://api.mistral.ai/v1` | OpenAI |
| `together` | Together AI | `TOGETHER_API_KEY` | `https://api.together.xyz/v1` | OpenAI |
| `fireworks` | Fireworks AI | `FIREWORKS_API_KEY` | `https://api.fireworks.ai/inference/v1` | OpenAI |
| `perplexity` | Perplexity AI | `PERPLEXITY_API_KEY` | `https://api.perplexity.ai` | OpenAI |
| `cohere` | Cohere | `COHERE_API_KEY` | `https://api.cohere.com/v2` | OpenAI |
| `ai21` | AI21 Labs | `AI21_API_KEY` | `https://api.ai21.com/studio/v1` | OpenAI |
| `cerebras` | Cerebras | `CEREBRAS_API_KEY` | `https://api.cerebras.ai/v1` | OpenAI |
| `sambanova` | SambaNova | `SAMBANOVA_API_KEY` | `https://api.sambanova.ai/v1` | OpenAI |
| `huggingface` | Hugging Face | `HF_API_KEY` | `https://api-inference.huggingface.co/v1` | OpenAI |
| `xai` | xAI (Grok) | `XAI_API_KEY` | `https://api.x.ai/v1` | OpenAI |
| `replicate` | Replicate | `REPLICATE_API_TOKEN` | `https://api.replicate.com/v1` | OpenAI |
| `chutes` | Chutes.ai | `CHUTES_API_KEY` | `https://llm.chutes.ai/v1` | OpenAI |
| `venice` | Venice.ai | `VENICE_API_KEY` | `https://api.venice.ai/api/v1` | OpenAI |
| `nvidia` / `nvidia-nim` | NVIDIA NIM | `NVIDIA_API_KEY` | `https://integrate.api.nvidia.com/v1` | OpenAI |

### Enterprise / Cloud Platform Providers

| Provider ID | Display Name | Env Var | Base URL | Driver | Notes |
|-------------|-------------|---------|----------|--------|-------|
| `azure` / `azure-openai` | Azure OpenAI | `AZURE_OPENAI_API_KEY` | User-supplied (per-resource URL required) | OpenAI (Azure variant) | Requires `base_url` set to `https://{resource}.openai.azure.com/openai/deployments` |
| `vertex-ai` / `vertex` / `google-vertex` | Google Vertex AI | `GOOGLE_APPLICATION_CREDENTIALS` | `https://us-central1-aiplatform.googleapis.com` | Vertex | Uses OAuth service account; set `GOOGLE_CLOUD_PROJECT` or read from service account JSON |
| `bedrock` | AWS Bedrock | `AWS_ACCESS_KEY_ID` | `https://bedrock-runtime.us-east-1.amazonaws.com` | OpenAI | |
| `github-copilot` / `copilot` | GitHub Copilot | `GITHUB_TOKEN` | `https://api.githubcopilot.com` | Copilot | Exchanges GitHub PAT for a Copilot API token automatically; caches and refreshes on expiry |
| `codex` / `openai-codex` | OpenAI Codex | `OPENAI_API_KEY` | `https://api.openai.com/v1` | OpenAI | Also reads credentials from Codex CLI `~/.codex/auth.json` |

### Chinese Providers

| Provider ID | Display Name | Env Var | Base URL | Driver |
|-------------|-------------|---------|----------|--------|
| `qwen` / `dashscope` / `model_studio` | Qwen (Alibaba) | `DASHSCOPE_API_KEY` | `https://dashscope.aliyuncs.com/compatible-mode/v1` | OpenAI |
| `minimax` | MiniMax | `MINIMAX_API_KEY` | `https://api.minimax.io/v1` | OpenAI |
| `zhipu` / `glm` | Zhipu AI (GLM) | `ZHIPU_API_KEY` | `https://open.bigmodel.cn/api/paas/v4` | OpenAI |
| `zhipu_coding` / `codegeex` | Zhipu Coding (CodeGeeX) | `ZHIPU_API_KEY` | `https://open.bigmodel.cn/api/coding/paas/v4` | OpenAI |
| `zai` / `z.ai` | Z.AI | `ZHIPU_API_KEY` | `https://api.z.ai/api/paas/v4` | OpenAI |
| `zai_coding` | Z.AI Coding | `ZHIPU_API_KEY` | `https://api.z.ai/api/coding/paas/v4` | OpenAI |
| `moonshot` / `kimi` / `kimi2` | Moonshot (Kimi) | `MOONSHOT_API_KEY` | `https://api.moonshot.ai/v1` | OpenAI |
| `kimi_coding` | Kimi for Code | `KIMI_API_KEY` | `https://api.kimi.com/coding` | Anthropic |
| `qianfan` / `baidu` | Baidu Qianfan | `QIANFAN_API_KEY` | `https://qianfan.baidubce.com/v2` | OpenAI |
| `volcengine` / `doubao` | Volcano Engine (Doubao) | `VOLCENGINE_API_KEY` | `https://ark.cn-beijing.volces.com/api/v3` | OpenAI |
| `volcengine_coding` | Volcano Engine Coding Plan | `VOLCENGINE_API_KEY` | `https://ark.cn-beijing.volces.com/api/coding/v3` | OpenAI |

### CLI-backed Providers (no API key required)

| Provider ID | Display Name | Driver | Notes |
|-------------|-------------|--------|-------|
| `claude-code` | Claude Code | ClaudeCode | Subprocess-based; uses the `claude` CLI. No API key needed (CLI handles auth). |
| `qwen-code` | Qwen Code | QwenCode | Subprocess-based; uses the `qwen` CLI. Free via Qwen OAuth. |

### Local Providers (no API key required)

| Provider ID | Display Name | Env Var | Base URL | Driver |
|-------------|-------------|---------|----------|--------|
| `ollama` | Ollama | `OLLAMA_API_KEY` (optional) | `http://localhost:11434/v1` | OpenAI |
| `vllm` | vLLM | `VLLM_API_KEY` (optional) | `http://localhost:8000/v1` | OpenAI |
| `lmstudio` | LM Studio | `LMSTUDIO_API_KEY` (optional) | `http://localhost:1234/v1` | OpenAI |
| `lemonade` | Lemonade | `LEMONADE_API_KEY` (optional) | `http://localhost:8888/api/v1` | OpenAI |

Local providers auto-discover available models at startup. Models discovered this way are added to the catalog with `Local` tier and zero cost.

### Custom Providers

Any provider not in the above lists can be used by setting a `base_url`. The runtime treats it as OpenAI-compatible and looks for `{PROVIDER_UPPER}_API_KEY` as the env var convention:

```toml
[default_model]
provider = "my-custom-llm"
model = "my-model"
base_url = "http://localhost:9999/v1"
```

---

## Quick Start

Configure one provider to get a working install:

```bash
# Any one of these is sufficient:
export GROQ_API_KEY="gsk_..."
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."
export GEMINI_API_KEY="AIza..."
```

For a local backend:

```toml
[default_model]
provider = "ollama"
model = "llama3.2"
```

---

## Auto-detection

When no `[default_model]` is configured, `detect_available_provider()` scans environment variables in this priority order:

1. `OPENAI_API_KEY` -> openai / gpt-4o
2. `ANTHROPIC_API_KEY` -> anthropic / claude-sonnet-4-20250514
3. `GEMINI_API_KEY` (or `GOOGLE_API_KEY`) -> gemini / gemini-2.5-flash
4. `GROQ_API_KEY` -> groq / llama-3.3-70b-versatile
5. `DEEPSEEK_API_KEY` -> deepseek / deepseek-chat
6. `OPENROUTER_API_KEY` -> openrouter / gemini-2.5-flash
7. `MISTRAL_API_KEY` -> mistral / mistral-large-latest
8. `TOGETHER_API_KEY` -> together / Llama-3-70b-chat-hf
9. `FIREWORKS_API_KEY` -> fireworks / llama-v3p1-70b-instruct
10. `XAI_API_KEY` -> xai / grok-2
11. `PERPLEXITY_API_KEY` -> perplexity / sonar-large-128k-online
12. `COHERE_API_KEY` -> cohere / command-r-plus

The first provider with a non-empty API key is selected.

---

## Fallback Chains

Configure fallback providers in `config.toml` under `[[fallback_providers]]`. When the primary provider fails (rate limit, overload, network error, or any API error), the `FallbackDriver` tries each fallback in order. Only returns an error when all drivers in the chain are exhausted.

```toml
[default_model]
provider = "anthropic"
model = "claude-sonnet-4-20250514"
api_key_env = "ANTHROPIC_API_KEY"

[[fallback_providers]]
provider = "groq"
model = "llama-3.3-70b-versatile"
api_key_env = "GROQ_API_KEY"

[[fallback_providers]]
provider = "ollama"
model = "llama3.2:latest"
```

Each `[[fallback_providers]]` entry supports:

| Field | Type | Description |
|-------|------|-------------|
| `provider` | string | Provider name (e.g., `ollama`, `groq`) |
| `model` | string | Model to use from this provider |
| `api_key_env` | string | Environment variable for API key (empty for local providers) |
| `base_url` | string (optional) | Base URL override (uses catalog default if omitted) |

The FallbackDriver wraps both `complete` and `stream` paths. On each failure, it substitutes the model name for the fallback entry and retries with the next driver. Rate-limited (`429`), overloaded (`529`), and network errors all trigger fallback progression.

---

## Model Tiers

Every model in the catalog is classified into a tier:

| Tier | Description | Examples |
|------|-------------|---------|
| `frontier` | Most capable, cutting-edge | Claude Opus 4.6, GPT-5.2, Gemini 3.1 Pro, Grok 4 |
| `smart` | Cost-effective and capable | Claude Sonnet 4.6, GPT-4o, Gemini 2.5 Flash, Mistral Large |
| `balanced` | Speed/cost balance | GPT-4.1 Mini, Llama 3.3 70B, Mixtral 8x7B |
| `fast` | Cheapest, fastest | Claude Haiku 4.5, GPT-4o Mini, Gemini 2.0 Flash Lite |
| `local` | Local models (zero cost) | Ollama/vLLM/LM Studio discovered models |
| `custom` | User-defined models added at runtime | Custom models from `custom_models.json` |

---

## Cost Tracking

Each model entry in the catalog includes per-million-token pricing:

- `input_cost_per_m` -- cost per million input tokens (USD)
- `output_cost_per_m` -- cost per million output tokens (USD)

The runtime uses these values for budget tracking. Query pricing via the API:

```
GET /api/models/{id}       # includes input_cost_per_m, output_cost_per_m
GET /api/budget             # global cost tracking
GET /api/budget/agents      # per-agent cost ranking
GET /api/budget/agents/{id} # single agent budget detail
```

Local and CLI-backed models have zero cost. Free-tier models (e.g., OpenRouter `:free` suffixed models, GLM-4 Flash, ERNIE Speed 128K) also have zero cost.

---

## Model Discovery

### CLI

```bash
openfang models list          # all models in catalog
openfang models aliases       # all alias mappings
openfang models providers     # all providers with auth status
```

### API

```
GET /api/models              # full catalog
GET /api/models/aliases      # alias -> canonical ID map
GET /api/models/{id}         # single model detail
GET /api/providers           # all providers with auth status and model counts
```

### Dynamic discovery

Local providers (Ollama, vLLM, LM Studio) support dynamic model discovery at startup. Discovered models are merged into the catalog with `Local` tier and zero cost. The provider's `model_count` is updated accordingly.

### Custom models

Add custom models via `custom_models.json` or the API. Custom models use `Custom` tier and are prioritized over builtins with the same lowercased name (so a user's vLLM model `Qwen3-30B-A3B` is preferred over the builtin `qwen3-30b-a3b`).

---

## Provider URL Overrides

Override the default base URL for any provider in `config.toml`:

```toml
[provider_urls]
ollama = "http://192.168.1.100:11434/v1"
vllm = "http://gpu-server:8000/v1"
minimax = "https://api.minimaxi.com/v1"   # China mainland endpoint
```

Unknown provider names are auto-registered as custom OpenAI-compatible entries when a URL override is set.

---

## Model Aliases

Common aliases resolve to canonical model IDs:

| Alias | Resolves to |
|-------|------------|
| `sonnet` | `claude-sonnet-4-6` |
| `opus` | `claude-opus-4-6` |
| `haiku` | `claude-haiku-4-5-20251001` |
| `gpt4`, `gpt4o` | `gpt-4o` |
| `gpt5` | `gpt-5.2` |
| `flash` | `gemini-2.5-flash` |
| `deepseek` | `deepseek-chat` |
| `llama` | `llama-3.3-70b-versatile` |
| `grok` | `grok-4-0709` |
| `mistral` | `mistral-large-latest` |
| `codestral` | `codestral-latest` |
| `sonar` | `sonar-pro` |
| `jamba` | `jamba-1.5-large` |
| `command` | `command-a` |
| `copilot` | `copilot/gpt-4o` |
| `codex` | `codex/gpt-5.4` |
| `claude-code` | `claude-code/sonnet` |
| `qwen-code` | `qwen-code/qwen3-coder` |
| `kimi` | `kimi-k2` |
| `glm` | `glm-5-20250605` |
| `minimax` | `MiniMax-M2.7` |
| `free` | `openrouter/meta-llama/llama-3.1-8b-instruct:free` |
| `free-reasoning` | `openrouter/deepseek/deepseek-r1:free` |

Full alias list available via `openfang models aliases` or `GET /api/models/aliases`.

---

## Configuration Reference

### Default model

```toml
[default_model]
provider = "groq"
model = "llama-3.3-70b-versatile"
api_key_env = "GROQ_API_KEY"
# base_url = "..."  # optional override
```

### Fallback providers

```toml
[[fallback_providers]]
provider = "gemini"
model = "gemini-2.0-flash"
api_key_env = "GEMINI_API_KEY"
# base_url = "..."  # optional override

[[fallback_providers]]
provider = "ollama"
model = "llama3.2:latest"
```

### Provider URL overrides

```toml
[provider_urls]
ollama = "http://remote-server:11434/v1"
```
