# LLM Providers

Validated against OpenFang release `v0.5.7`.

Provider counts are one of the easiest areas to misstate in OpenFang. The release runtime catalog includes:

- `41` provider entries in `ModelCatalog::list_providers()`
- `197` builtin model entries in `builtin_models()`

Those counts include brand variants, coding-specific provider variants, and CLI-backed integrations. For that reason, this page avoids a simplified "X providers" marketing number unless the grouping rule is explicit.

## Driver Families

The released runtime centers on three main driver families:

| Driver family | Used for |
|---------------|----------|
| Anthropic | Anthropic-native and Anthropic-compatible flows |
| Gemini | Gemini-native flows |
| OpenAI-compatible | OpenAI and most HTTP-compatible providers |

The runtime also contains special handling around Azure OpenAI, Vertex AI, GitHub Copilot, Codex, Claude Code, and Qwen Code.

## Practical setup guidance

If you only want a working install, configure one provider first:

```bash
export GROQ_API_KEY="gsk_..."
```

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

```bash
export OPENAI_API_KEY="sk-..."
```

```bash
export GEMINI_API_KEY="AIza..."
```

For a local backend:

```toml
[default_model]
provider = "ollama"
model = "llama3.2"
base_url = "http://localhost:11434"
api_key_env = ""
```

## Release-safe provider view

Major provider families represented in the `v0.5.7` runtime catalog include:

- Anthropic
- OpenAI
- Gemini
- DeepSeek
- Groq
- OpenRouter
- Mistral
- Together
- Fireworks
- Ollama
- vLLM
- LM Studio
- Perplexity
- Cohere
- AI21
- Cerebras
- SambaNova
- Hugging Face
- xAI
- Replicate
- GitHub Copilot
- NVIDIA
- Qwen
- MiniMax
- Zhipu
- Z.ai
- Moonshot
- Qianfan
- Volcengine
- Bedrock
- Azure
- Codex
- Claude Code
- Qwen Code

Some of these also appear as variant entries, which is why a raw catalog count is higher than a grouped brand count.

## Why earlier docs drifted

Different upstream sources used different rules:

- older prose docs counted only a smaller public subset
- runtime tests count every provider entry
- some counts exclude coding-specific variants
- some counts exclude CLI-backed providers

This docs repo therefore uses:

- raw catalog counts when talking about the source tree
- grouped names when talking about user-facing setup choices

## Model discovery

The release CLI exposes model and provider inspection commands:

```bash
openfang models list
openfang models aliases
openfang models providers
```

The API also exposes catalog surfaces:

```text
GET /api/models
GET /api/models/aliases
GET /api/models/{id}
GET /api/providers
```

## Configuration pattern

Typical model configuration:

```toml
[default_model]
provider = "groq"
model = "llama-3.3-70b-versatile"
api_key_env = "GROQ_API_KEY"
```

Fallback configuration:

```toml
[[fallback_providers]]
provider = "gemini"
model = "gemini-2.0-flash"
api_key_env = "GEMINI_API_KEY"
```

## Notes

- Earlier docs in this repo used `27` or `28` as a provider headline. Those numbers were removed because they did not match the released runtime catalog.
- SDK package version numbers should not be assumed to match the application release version.
