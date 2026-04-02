# LLM Providers

OpenFang supports 28 LLM providers with 130+ models out of the box. Three native drivers (Anthropic, Gemini, OpenAI-compatible) route to all providers. Intelligent model routing, fallback chains, and cost metering are built in.

---

## Quick Setup

```bash
# Set API key via CLI
openfang config set-key groq gsk_...
openfang config set-key anthropic sk-ant-...
openfang config set-key openai sk-...

# Or via environment variables
export GROQ_API_KEY="gsk_..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

---

## Built-in Providers

### Anthropic (Native Driver)

**Vault key:** `ANTHROPIC_API_KEY`

| Model ID | Name | Tier | Context |
|----------|------|------|---------|
| `claude-opus-4-20250514` | Claude Opus 4 | Frontier | 200K |
| `claude-sonnet-4-20250514` | Claude Sonnet 4 | Smart | 200K |
| `claude-haiku-4-5-20251001` | Claude Haiku 4.5 | Fast | 200K |
| `claude-opus-4-6` | Claude Opus 4.6 | Frontier | 200K |
| `claude-sonnet-4-6` | Claude Sonnet 4.6 | Smart | 200K |
| `claude-haiku-4-5` | Claude Haiku 4.5 | Fast | 200K |

**Aliases:** `claude`, `claude-opus`, `claude-sonnet`, `claude-haiku`

### OpenAI (OpenAI-Compatible Driver)

**Vault key:** `OPENAI_API_KEY`

| Model ID | Name | Tier |
|----------|------|------|
| `gpt-4.1` | GPT-4.1 | Frontier |
| `gpt-4o` | GPT-4o | Smart |
| `gpt-4o-mini` | GPT-4o mini | Balanced |
| `o3` | o3 | Frontier |
| `o4-mini` | o4-mini | Smart |
| `o3-mini` | o3-mini | Balanced |

**Aliases:** `gpt4`, `gpt-4o`, `gpt-mini`

### Google Gemini (Gemini Driver)

**Vault key:** `GEMINI_API_KEY`

| Model ID | Name | Tier |
|----------|------|------|
| `gemini-2.5-pro` | Gemini 2.5 Pro | Frontier |
| `gemini-2.5-flash` | Gemini 2.5 Flash | Smart |
| `gemini-2.0-pro` | Gemini 2.0 Pro | Smart |
| `gemini-2.0-flash` | Gemini 2.0 Flash | Balanced |

**Free tier available.** Note: Gemini 2.5+ requires `thoughtSignature` handling (built into the driver).

### Groq (OpenAI-Compatible)

**Vault key:** `GROQ_API_KEY` | **Free tier available**

| Model ID | Name | Tier |
|----------|------|------|
| `llama-3.3-70b-versatile` | Llama 3.3 70B | Balanced |
| `llama-3.1-8b-instant` | Llama 3.1 8B | Fast |
| `mixtral-8x7b-32768` | Mixtral 8x7B | Balanced |
| `gemma2-9b-it` | Gemma 2 9B | Fast |

**Alias:** `groq`

### DeepSeek

**Vault key:** `DEEPSEEK_API_KEY`

| Model ID | Name | Tier |
|----------|------|------|
| `deepseek-chat` | DeepSeek V3 | Smart |
| `deepseek-reasoner` | DeepSeek R1 | Frontier |

### xAI (Grok)

**Vault key:** `XAI_API_KEY`

| Model ID | Name | Tier |
|----------|------|------|
| `grok-3` | Grok 3 | Frontier |
| `grok-3-mini` | Grok 3 Mini | Smart |
| `grok-2` | Grok 2 | Balanced |

### Mistral

**Vault key:** `MISTRAL_API_KEY`

| Model ID | Name | Tier |
|----------|------|------|
| `mistral-large-latest` | Mistral Large | Smart |
| `mistral-medium-latest` | Mistral Medium | Balanced |
| `mistral-small-latest` | Mistral Small | Fast |
| `codestral-latest` | Codestral | Smart |

### Cohere

**Vault key:** `COHERE_API_KEY`

| Model ID | Name |
|----------|------|
| `command-r-plus` | Command R+ |
| `command-r` | Command R |

### Perplexity

**Vault key:** `PERPLEXITY_API_KEY`

| Model ID | Name |
|----------|------|
| `llama-3.1-sonar-huge-128k-online` | Sonar Huge (online) |
| `llama-3.1-sonar-large-128k-online` | Sonar Large (online) |
| `llama-3.1-sonar-small-128k-online` | Sonar Small (online) |

### Together AI

**Vault key:** `TOGETHER_API_KEY`

Access to 50+ open-source models including Llama, Mistral, Qwen, Falcon, and more.

### Fireworks AI

**Vault key:** `FIREWORKS_API_KEY`

Fast inference for open-source models.

### AI21

**Vault key:** `AI21_API_KEY`

Jamba models (hybrid SSM-Transformer).

### Cerebras

**Vault key:** `CEREBRAS_API_KEY`

Ultra-fast inference on Wafer Scale Engine hardware.

### SambaNova

**Vault key:** `SAMBANOVA_API_KEY`

High-throughput LLM inference.

### Hugging Face

**Vault key:** `HUGGINGFACE_API_KEY`

Access to thousands of open-source models via the Inference API.

### Replicate

**Vault key:** `REPLICATE_API_KEY`

Run any open-source model on serverless GPU infrastructure.

### OpenRouter

**Vault key:** `OPENROUTER_API_KEY`

Multi-provider aggregator. Access 100+ models from a single endpoint with automatic routing.

**Free model aliases:** `openrouter/free`, `free`, `free-reasoning`

```toml
[providers.openrouter]
api_key_env = "OPENROUTER_API_KEY"
base_url = "https://openrouter.ai/api/v1"
```

### NVIDIA NIM

**Vault key:** `NVIDIA_API_KEY`

Access to NVIDIA-hosted Llama, Mistral, and other models.

Models: `meta/llama-3.1-405b-instruct`, `nvidia/llama-3.1-nemotron-70b-instruct`, `mistralai/mistral-large`, `meta/llama-3.3-70b-instruct`, `nvidia/llama-3.3-nemotron-super-49b-v1`

### Qwen Code CLI (via Alibaba)

**Vault key:** `QWEN_API_KEY`

Models: `qwen-coder-plus`, `qwen-coder`, `qwen-long`

### Z.AI (ZHIPU)

**Vault key:** `ZHIPU_API_KEY`

| Model ID | Name |
|----------|------|
| `glm-5-coding` | GLM-5 Coding |
| `glm-4.7-coding` | GLM-4.7 Coding |
| `glm-4.7` | GLM-4.7 |

### Kimi (Moonshot)

**Vault key:** `MOONSHOT_API_KEY`

| Model ID | Name |
|----------|------|
| `kimi-k2.5-0711` | Kimi for Code |
| `moonshot-v1-8k` | Moonshot 8K |
| `moonshot-v1-32k` | Moonshot 32K |

### MiniMax

**Vault key:** `MINIMAX_API_KEY`

Models: `abab7-chat`, `abab6.5s-chat`, `MiniMax-M2.5-highspeed`

### Chutes.ai

**Vault key:** `CHUTES_API_KEY`

Community-run inference: DeepSeek-V3, DeepSeek-R1, Llama-4, Qwen3-235B

### Local Providers (No API Key)

| Provider | Env Var | Default URL |
|----------|---------|-------------|
| Ollama | `OLLAMA_BASE_URL` | `http://localhost:11434` |
| vLLM | `VLLM_BASE_URL` | `http://localhost:8000` |
| LM Studio | `LMSTUDIO_BASE_URL` | `http://localhost:1234` |

Local providers auto-detect models via `/v1/models` endpoint.

---

## Model Tiers

| Tier | Use Case | Examples |
|------|----------|---------|
| `Frontier` | Complex reasoning, architecture, critical decisions | Claude Opus, GPT-4.1, o3 |
| `Smart` | Technical work, code generation, analysis | Claude Sonnet, Gemini Pro, DeepSeek V3 |
| `Balanced` | General tasks, cost-efficient | Llama 3.3 70B, Gemini Flash, Mistral Large |
| `Fast` | Real-time, high-volume, simple tasks | Llama 3.1 8B, Claude Haiku |
| `Local` | Private data, offline, no API cost | Ollama, vLLM, LM Studio |
| `Custom` | User-defined providers and models | — |

---

## Intelligent Model Routing

Automatically select the best model based on task complexity:

```toml
[model_routing]
enabled = true
simple_threshold = 0.3
complex_threshold = 0.8
simple_provider = "groq"
simple_model = "llama-3.1-8b-instant"
balanced_provider = "groq"
balanced_model = "llama-3.3-70b-versatile"
complex_provider = "anthropic"
complex_model = "claude-opus-4-6"
```

---

## Fallback Chain

Automatically failover when a provider is unavailable:

```toml
[[fallback_providers]]
provider = "anthropic"
model = "claude-sonnet-4-6"

[[fallback_providers]]
provider = "openai"
model = "gpt-4o"

[[fallback_providers]]
provider = "groq"
model = "llama-3.3-70b-versatile"
```

---

## Adding Custom Providers

Any OpenAI-compatible endpoint can be added:

```toml
[[custom_providers]]
id = "my-provider"
name = "My LLM"
base_url = "https://api.myllm.com/v1"
api_key_env = "MY_LLM_API_KEY"
models = [
  { id = "my-model-1", name = "My Model 1", tier = "balanced" }
]
```

Via CLI:
```bash
openfang config set-key my-provider my-api-key-here
```

Via API:
```bash
POST /api/models/custom
{
  "id": "my-model",
  "name": "My Model",
  "provider": "my-provider",
  "context_window": 128000,
  "supports_tools": true,
  "supports_streaming": true
}
```

---

## Cost Tracking

OpenFang tracks token usage and estimated costs per model:

```bash
# Usage summary
GET /api/usage/summary

# Per-model breakdown
GET /api/usage/by-model

# Per-agent ranking
GET /api/usage

# Daily cost timeline
GET /api/usage/daily
```

Web dashboard **Usage** tab shows: cost projection, provider breakdown (pie chart), daily timeline, per-model and per-agent rankings.

---

## Token Quota

Per-agent hourly token limits:

```toml
[quota]
default_max_llm_tokens_per_hour = 0    # 0 = unlimited
```

The quota uses a rolling 1-hour window. When exceeded, requests return a `QuotaExceeded` error with reset time.

---

## OpenAI-Compatible Drop-in API

OpenFang exposes an OpenAI-compatible API for use with any OpenAI SDK:

```bash
# Chat completion
curl http://127.0.0.1:4200/v1/chat/completions \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama-3.3-70b-versatile",
    "messages": [{"role": "user", "content": "Hello!"}],
    "stream": true
  }'

# List models
curl http://127.0.0.1:4200/v1/models \
  -H "Authorization: Bearer <api_key>"
```

Use OpenFang as a backend for any tool that accepts an OpenAI API URL.
