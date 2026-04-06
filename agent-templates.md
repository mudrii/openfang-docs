# Agent Templates

OpenFang ships with **30 pre-built agent templates** organized into 4 performance tiers. Each template is a ready-to-spawn `agent.toml` manifest located in the `agents/` directory. Templates cover software engineering, business operations, personal productivity, and everyday tasks.

Counting note:

- the release tree contains `31` directories under `agents/`
- `30` of those are spawnable templates with an `agent.toml`
- `langchain-code-reviewer` is present as an integration/example directory, not a regular bundled spawn template

---

## Quick Start

```bash
# Spawn by template name
openfang spawn orchestrator
openfang spawn coder

# Spawn from a TOML file path
openfang spawn --template agents/writer/agent.toml

# Spawn via the REST API
curl -X POST http://localhost:4200/api/agents \
  -H "Content-Type: application/json" \
  -d '{"template": "coder"}'

# Spawn with model override
curl -X POST http://localhost:4200/api/agents \
  -H "Content-Type: application/json" \
  -d '{"template": "writer", "model": "gemini-2.5-flash"}'
```

---

## Template Tiers

Templates are organized into 4 tiers based on task complexity and the LLM models they use. Higher tiers use more capable (and more expensive) models.

### Tier 1 -- Frontier (DeepSeek)

For tasks requiring the deepest reasoning: multi-agent orchestration, system architecture, and security analysis.

| Template | Provider | Model |
|----------|----------|-------|
| orchestrator | deepseek | deepseek-chat |
| architect | deepseek | deepseek-chat |
| security-auditor | deepseek | deepseek-chat |

All Tier 1 agents fall back to `groq/llama-3.3-70b-versatile` if the DeepSeek API key is unavailable.

### Tier 2 -- Smart (Gemini 2.5 Flash)

For strong analytical and coding tasks: software engineering, data science, research, testing, and legal review.

| Template | Provider | Model |
|----------|----------|-------|
| coder | gemini | gemini-2.5-flash |
| code-reviewer | gemini | gemini-2.5-flash |
| data-scientist | gemini | gemini-2.5-flash |
| debugger | gemini | gemini-2.5-flash |
| researcher | gemini | gemini-2.5-flash |
| analyst | gemini | gemini-2.5-flash |
| test-engineer | gemini | gemini-2.5-flash |
| legal-assistant | gemini | gemini-2.5-flash |

All Tier 2 agents fall back to `groq/llama-3.3-70b-versatile` if the Gemini API key is unavailable.

### Tier 3 -- Balanced (Groq + Gemini Fallback)

For everyday business and productivity tasks: planning, writing, email, customer support, sales, recruiting, and meetings.

| Template | Provider | Model | Fallback |
|----------|----------|-------|----------|
| planner | groq | llama-3.3-70b-versatile | gemini/gemini-2.0-flash |
| writer | groq | llama-3.3-70b-versatile | gemini/gemini-2.0-flash |
| doc-writer | groq | llama-3.3-70b-versatile | gemini/gemini-2.0-flash |
| devops-lead | groq | llama-3.3-70b-versatile | gemini/gemini-2.0-flash |
| assistant | groq | llama-3.3-70b-versatile | gemini/gemini-2.0-flash |
| email-assistant | groq | llama-3.3-70b-versatile | gemini/gemini-2.0-flash |
| social-media | groq | llama-3.3-70b-versatile | gemini/gemini-2.0-flash |
| customer-support | groq | llama-3.3-70b-versatile | gemini/gemini-2.0-flash |
| sales-assistant | groq | llama-3.3-70b-versatile | gemini/gemini-2.0-flash |
| recruiter | groq | llama-3.3-70b-versatile | gemini/gemini-2.0-flash |
| meeting-assistant | groq | llama-3.3-70b-versatile | gemini/gemini-2.0-flash |

### Tier 4 -- Fast (Groq Only)

For lightweight, high-speed tasks: ops monitoring, translation, tutoring, wellness tracking, budgeting, travel, and home automation. No fallback model configured (except `ops` which uses an 8B model for speed).

| Template | Provider | Model |
|----------|----------|-------|
| ops | groq | llama-3.1-8b-instant |
| hello-world | groq | llama-3.3-70b-versatile |
| translator | groq | llama-3.3-70b-versatile |
| tutor | groq | llama-3.3-70b-versatile |
| health-tracker | groq | llama-3.3-70b-versatile |
| personal-finance | groq | llama-3.3-70b-versatile |
| travel-planner | groq | llama-3.3-70b-versatile |
| home-automation | groq | llama-3.3-70b-versatile |

---

## Template Catalog

### orchestrator

**Tier 1** | `deepseek/deepseek-chat` | Fallback: `groq/llama-3.3-70b-versatile`

Meta-agent that decomposes complex tasks, delegates to specialist agents, and synthesizes results. Analyzes user requests, breaks them into subtasks, uses `agent_list` to discover available specialists, delegates work via `agent_send`, spawns new agents when needed, and synthesizes all responses into a coherent final answer.

- **Temperature**: 0.3 | **Max tokens**: 8192 | **Token quota**: 500,000/hr
- **Schedule**: Continuous check every 120 seconds
- **Tools**: `agent_send`, `agent_spawn`, `agent_list`, `agent_kill`, `memory_store`, `memory_recall`, `file_read`, `file_write`

### architect

**Tier 1** | `deepseek/deepseek-chat` | Fallback: `groq/llama-3.3-70b-versatile`

System architect. Designs software architectures, evaluates trade-offs, creates technical specifications. Follows principles of separation of concerns, performance-aware design, and simplicity over cleverness.

- **Temperature**: 0.3 | **Max tokens**: 8192 | **Token quota**: 200,000/hr
- **Tools**: `file_read`, `file_list`, `memory_store`, `memory_recall`, `agent_send`

### security-auditor

**Tier 1** | `deepseek/deepseek-chat` | Fallback: `groq/llama-3.3-70b-versatile`

Security specialist. Reviews code for vulnerabilities, checks configurations, performs threat modeling. Focuses on OWASP Top 10, input validation, auth flaws, cryptographic misuse, injection attacks, and secrets management. Reports with severity levels: CRITICAL/HIGH/MEDIUM/LOW/INFO.

- **Temperature**: 0.2 | **Max tokens**: 4096 | **Token quota**: 150,000/hr
- **Schedule**: Proactive on `event:agent_spawned`, `event:agent_terminated`
- **Tools**: `file_read`, `file_list`, `shell_exec`, `memory_store`, `memory_recall`

### coder

**Tier 2** | `gemini/gemini-2.5-flash` | Fallback: `groq/llama-3.3-70b-versatile`

Expert software engineer. Reads, writes, and analyzes code. Writes clean, production-quality code with step-by-step reasoning. Always writes tests for produced code.

- **Temperature**: 0.3 | **Max tokens**: 8192 | **Token quota**: 200,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `shell_exec`
- **Shell**: `cargo *`, `rustc *`, `git *`, `npm *`, `python *`

### code-reviewer

**Tier 2** | `gemini/gemini-2.5-flash` | Fallback: `groq/llama-3.3-70b-versatile`

Senior code reviewer. Reviews by priority: correctness, security, performance, maintainability, style. Groups feedback by file with severity tags: `[MUST FIX]`, `[SHOULD FIX]`, `[NIT]`, `[PRAISE]`.

- **Temperature**: 0.3 | **Max tokens**: 4096 | **Token quota**: 150,000/hr
- **Tools**: `file_read`, `file_list`, `shell_exec`, `memory_store`, `memory_recall`

### data-scientist

**Tier 2** | `gemini/gemini-2.5-flash` | Fallback: `groq/llama-3.3-70b-versatile`

Data scientist. Analyzes datasets, builds models, creates visualizations, performs statistical analysis. Covers descriptive stats, hypothesis testing, correlation/regression, time series, clustering, and A/B test design.

- **Temperature**: 0.3 | **Max tokens**: 4096 | **Token quota**: 150,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `shell_exec`, `memory_store`, `memory_recall`

### debugger

**Tier 2** | `gemini/gemini-2.5-flash` | Fallback: `groq/llama-3.3-70b-versatile`

Expert debugger. Follows: reproduce, isolate, identify root cause, fix (minimal correct fix), verify (regression tests). Presents findings as Bug Report, Root Cause, Fix, Prevention.

- **Temperature**: 0.2 | **Max tokens**: 4096 | **Token quota**: 150,000/hr
- **Tools**: `file_read`, `file_list`, `shell_exec`, `memory_store`, `memory_recall`

### researcher

**Tier 2** | `gemini/gemini-2.5-flash` | Fallback: `groq/llama-3.3-70b-versatile`

Research agent. Fetches web content and synthesizes information into clear, structured reports. Always cites sources and separates facts from analysis.

- **Temperature**: 0.5 | **Max tokens**: 4096 | **Token quota**: 150,000/hr
- **Tools**: `web_fetch`, `file_read`, `file_write`, `file_list`

### analyst

**Tier 2** | `gemini/gemini-2.5-flash` | Fallback: `groq/llama-3.3-70b-versatile`

Data analyst. Processes data, generates insights, creates reports. Shows methodology and uses numbers and evidence to support conclusions.

- **Temperature**: 0.4 | **Max tokens**: 4096 | **Token quota**: 150,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `shell_exec`

### test-engineer

**Tier 2** | `gemini/gemini-2.5-flash` | Fallback: `groq/llama-3.3-70b-versatile`

Quality assurance engineer. Designs unit, integration, property-based, edge case, and regression tests. Follows Arrange-Act-Assert with descriptive names (`test_X_when_Y_should_Z`).

- **Temperature**: 0.3 | **Max tokens**: 4096 | **Token quota**: 150,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `shell_exec`, `memory_store`, `memory_recall`

### legal-assistant

**Tier 2** | `gemini/gemini-2.5-flash` | Fallback: `groq/llama-3.3-70b-versatile`

Legal assistant for contract review, legal research, compliance checking, and document drafting. Checks against GDPR, SOC 2, HIPAA, PCI DSS, CCPA/CPRA. Always includes a disclaimer.

- **Temperature**: 0.2 | **Max tokens**: 8192 | **Token quota**: 200,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`, `web_fetch`

### planner

**Tier 3** | `groq/llama-3.3-70b-versatile` | Fallback: `gemini/gemini-2.0-flash`

Project planner. Creates project plans, breaks down epics, estimates effort, identifies risks. Estimates ranges (best/likely/worst) with 20-30% buffer.

- **Temperature**: 0.3 | **Max tokens**: 8192 | **Token quota**: 200,000/hr
- **Tools**: `file_read`, `file_list`, `memory_store`, `memory_recall`, `agent_send`

### writer

**Tier 3** | `groq/llama-3.3-70b-versatile` | Fallback: `gemini/gemini-2.0-flash`

Content writer. Creates documentation, articles, and technical writing. Concise, active voice, structured with headers and bullet points.

- **Temperature**: 0.7 | **Max tokens**: 4096 | **Token quota**: 100,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`

### doc-writer

**Tier 3** | `groq/llama-3.3-70b-versatile` | Fallback: `gemini/gemini-2.0-flash`

Technical writer. Creates READMEs, API docs, architecture docs, tutorials, reference docs, and ADRs. Starts with WHY, then WHAT, then HOW.

- **Temperature**: 0.4 | **Max tokens**: 8192 | **Token quota**: 200,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`

### devops-lead

**Tier 3** | `groq/llama-3.3-70b-versatile` | Fallback: `gemini/gemini-2.0-flash`

DevOps lead. CI/CD, container orchestration, IaC (Terraform, Pulumi), monitoring (Prometheus, Grafana, OpenTelemetry), incident response, security hardening.

- **Temperature**: 0.2 | **Max tokens**: 4096 | **Token quota**: 150,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `shell_exec`, `memory_store`, `memory_recall`, `agent_send`

### assistant

**Tier 3** | `groq/llama-3.3-70b-versatile` | Fallback: `gemini/gemini-2.0-flash`

General-purpose assistant. The default OpenFang agent for everyday tasks, questions, and conversations. Handles most tasks directly and delegates to specialists when they would do better.

- **Temperature**: 0.5 | **Max tokens**: 8192 | **Token quota**: 300,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`, `web_fetch`, `shell_exec`, `agent_send`, `agent_list`

### email-assistant

**Tier 3** | `groq/llama-3.3-70b-versatile` | Fallback: `gemini/gemini-2.0-flash`

Email triage, drafting, scheduling, and inbox management. Triages by urgency, category, and required action. Generates reusable templates for recurring patterns.

- **Temperature**: 0.4 | **Max tokens**: 8192 | **Token quota**: 150,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`, `web_fetch`

### social-media

**Tier 3** | `groq/llama-3.3-70b-versatile` | Fallback: `gemini/gemini-2.0-flash`

Social media content creation and strategy. Crafts platform-optimized content for Twitter/X, LinkedIn, Instagram, Facebook, TikTok, Reddit, Mastodon, Bluesky, and Threads.

- **Temperature**: 0.7 | **Max tokens**: 4096 | **Token quota**: 120,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`, `web_fetch`

### customer-support

**Tier 3** | `groq/llama-3.3-70b-versatile` | Fallback: `gemini/gemini-2.0-flash`

Customer support agent. Triages tickets by category, severity, product area, and customer tier. Writes empathetic, solution-oriented responses.

- **Temperature**: 0.3 | **Max tokens**: 4096 | **Token quota**: 200,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`, `web_fetch`

### sales-assistant

**Tier 3** | `groq/llama-3.3-70b-versatile` | Fallback: `gemini/gemini-2.0-flash`

Sales assistant. Drafts personalized outreach using the AIDA framework. Analyzes pipelines, prepares pre-call briefs, builds competitive battle cards.

- **Temperature**: 0.5 | **Max tokens**: 4096 | **Token quota**: 150,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`, `web_fetch`

### recruiter

**Tier 3** | `groq/llama-3.3-70b-versatile` | Fallback: `gemini/gemini-2.0-flash`

Recruiting agent. Resume screening, candidate outreach, job description writing, structured interview guides with STAR-format behavioral questions.

- **Temperature**: 0.4 | **Max tokens**: 4096 | **Token quota**: 150,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`, `web_fetch`

### meeting-assistant

**Tier 3** | `groq/llama-3.3-70b-versatile` | Fallback: `gemini/gemini-2.0-flash`

Meeting notes, action items, agenda preparation, and follow-up tracking. Transforms transcripts into structured minutes with executive summaries.

- **Temperature**: 0.3 | **Max tokens**: 8192 | **Token quota**: 150,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`

### ops

**Tier 4** | `groq/llama-3.1-8b-instant` | No fallback

DevOps agent. Monitors systems, runs diagnostics, manages deployments. Uses the smallest model in the fleet (8B) for maximum speed. Prefers read-only operations unless explicitly asked.

- **Temperature**: 0.2 | **Max tokens**: 2048 | **Token quota**: 50,000/hr
- **Schedule**: Periodic every 5 minutes
- **Tools**: `shell_exec`, `file_read`, `file_list`
- **Shell**: `docker *`, `git *`, `cargo *`, `systemctl *`, `ps *`, `df *`, `free *`

### hello-world

**Tier 4** | `groq/llama-3.3-70b-versatile` | No fallback

Minimal starter agent with basic read-only capabilities. No system prompt, no shell access. Useful as a starting point for custom agents or for testing.

- **Token quota**: 100,000/hr
- **Tools**: `file_read`, `file_list`, `web_fetch`

### translator

**Tier 4** | `groq/llama-3.3-70b-versatile` | No fallback

Multi-language translation. Handles contextual and cultural adaptation, document format preservation, software localization (JSON, YAML, PO/POT, XLIFF), and glossary management.

- **Temperature**: 0.3 | **Max tokens**: 8192 | **Token quota**: 200,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`, `web_fetch`

### tutor

**Tier 4** | `groq/llama-3.3-70b-versatile` | No fallback

Teaching and explanation agent. Uses the Feynman Technique and Socratic questioning. Walks through problems step-by-step. Creates structured learning plans with spaced repetition.

- **Temperature**: 0.5 | **Max tokens**: 8192 | **Token quota**: 200,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`, `shell_exec`, `web_fetch`

### health-tracker

**Tier 4** | `groq/llama-3.3-70b-versatile` | No fallback

Wellness tracking. Tracks weight, blood pressure, heart rate, sleep, water intake, steps, mood, and custom metrics. Manages medication schedules. Always disclaims it is not a medical professional.

- **Temperature**: 0.3 | **Max tokens**: 4096 | **Token quota**: 100,000/hr
- **Schedule**: Periodic every 1 hour
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`

### personal-finance

**Tier 4** | `groq/llama-3.3-70b-versatile` | No fallback

Personal finance. Budget tracking (50/30/20, zero-based, envelope method), expense categorization, savings goals, debt analysis (avalanche vs. snowball). Always disclaims output is not financial advice.

- **Temperature**: 0.2 | **Max tokens**: 8192 | **Token quota**: 150,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`, `shell_exec`

### travel-planner

**Tier 4** | `groq/llama-3.3-70b-versatile` | No fallback

Trip planning. Day-by-day itineraries, destination guides, travel budgets at multiple price tiers, transportation logistics, and packing lists.

- **Temperature**: 0.5 | **Max tokens**: 8192 | **Token quota**: 150,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`, `web_fetch`

### home-automation

**Tier 4** | `groq/llama-3.3-70b-versatile` | No fallback

Smart home control. Manages IoT devices, designs automation workflows, configures scenes, monitors energy consumption. Understands Matter/Thread protocol.

- **Temperature**: 0.2 | **Max tokens**: 4096 | **Token quota**: 100,000/hr
- **Tools**: `file_read`, `file_write`, `file_list`, `memory_store`, `memory_recall`, `shell_exec`, `web_fetch`

---

## Custom Templates

Create your own `agent.toml` following this format:

```toml
name = "my-agent"
version = "0.1.0"
description = "What this agent does in one sentence."
author = "your-name"
module = "builtin:chat"
tags = ["tag1", "tag2"]

[model]
provider = "gemini"
model = "gemini-2.5-flash"
api_key_env = "GEMINI_API_KEY"
max_tokens = 4096
temperature = 0.3
system_prompt = """Your agent's personality and instructions."""

[[fallback_models]]
provider = "groq"
model = "llama-3.3-70b-versatile"
api_key_env = "GROQ_API_KEY"

[resources]
max_llm_tokens_per_hour = 150000
max_concurrent_tools = 5

[capabilities]
tools = ["file_read", "file_write", "file_list", "shell_exec",
         "memory_store", "memory_recall", "web_fetch"]
network = ["*"]
memory_read = ["*"]
memory_write = ["self.*"]
shell = ["python *", "cargo *"]
```

### Available Tools

| Tool | Description |
|------|-------------|
| `file_read` | Read file contents |
| `file_write` | Write/create files |
| `file_list` | List directory contents |
| `shell_exec` | Execute shell commands (restricted by `shell` whitelist) |
| `memory_store` | Persist key-value data to memory |
| `memory_recall` | Retrieve data from memory |
| `web_fetch` | Fetch content from URLs (SSRF-protected) |
| `agent_send` | Send a message to another agent |
| `agent_list` | List all running agents |
| `agent_spawn` | Spawn a new agent |
| `agent_kill` | Terminate a running agent |

### Tips

1. **Start minimal.** Grant only the tools and capabilities the agent actually needs.
2. **Write a clear system prompt.** Be specific about role, methodology, output format, and limitations.
3. **Set appropriate temperature.** 0.2 for precise tasks, 0.5 for balanced, 0.7+ for creative.
4. **Use shell whitelists.** Never grant `shell = ["*"]`. Whitelist specific patterns.
5. **Set token budgets.** Start with 100,000/hr and adjust.
6. **Add fallback models.** Use `[[fallback_models]]` for reliability.

---

## Spawning

### CLI

```bash
openfang spawn coder
openfang spawn coder --name "backend-coder"
openfang spawn --template agents/custom/my-agent.toml
openfang agents
openfang message <agent-id> "Write a function to parse TOML files"
openfang kill <agent-id>
```

### REST API

```bash
POST /api/agents          {"template": "coder"}
POST /api/agents/{id}/message  {"content": "Implement the auth module"}
WS   /api/agents/{id}/ws
GET  /api/agents
DELETE /api/agents/{id}
```

### OpenAI-Compatible API

```bash
POST /v1/chat/completions
{
  "model": "openfang:coder",
  "messages": [{"role": "user", "content": "Write a Rust HTTP server"}],
  "stream": true
}
```

---

## Environment Variables

| Variable | Provider | Used By |
|----------|----------|---------|
| `DEEPSEEK_API_KEY` | DeepSeek | Tier 1 (orchestrator, architect, security-auditor) |
| `GEMINI_API_KEY` | Google Gemini | Tier 2 primary, Tier 3 fallback |
| `GROQ_API_KEY` | Groq | Tier 3 primary, Tier 1/2 fallback, Tier 4 |

At minimum, set `GROQ_API_KEY` to enable all Tier 3 and Tier 4 agents. Add `GEMINI_API_KEY` for Tier 2. Add `DEEPSEEK_API_KEY` for Tier 1.
