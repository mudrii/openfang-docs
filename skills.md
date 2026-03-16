# Skills

Skills inject expert knowledge into agents. They are YAML-frontmatter Markdown files (PromptOnly), Python scripts, WASM modules, or Node.js scripts that extend agent capabilities.

---

## Skill Runtimes

| Runtime | Description | Security |
|---------|-------------|----------|
| `promptonly` | Markdown injected into system prompt | None (text only) |
| `python` | Python subprocess, JSON stdin/stdout | Subprocess sandbox |
| `wasm` | Wasmtime sandbox, fuel metering | WASM dual-metered |
| `node` | Node.js subprocess (OpenClaw compat) | Subprocess sandbox |
| `shell` | Shell script (use sparingly) | Subprocess sandbox |
| `builtin` | Compiled Rust, part of binary | Native |

---

## Skill Manifest Format

```toml
# skill.toml
[skill]
name = "my-skill"
version = "0.1.0"
description = "Expert knowledge for my domain"
author = "yourname"
license = "MIT"
tags = ["domain", "expert"]

[runtime]
type = "promptonly"   # or python, wasm, node, shell, builtin
entry = "main.py"     # Only for non-promptonly runtimes

[[tools.provided]]
name = "my_tool"
description = "What this tool does"
input_schema = {
  type = "object",
  properties = {
    query = { type = "string", description = "The query" }
  },
  required = ["query"]
}

[requirements]
tools = ["web_fetch"]                    # Built-in tools this skill uses
capabilities = ["NetConnect(*)"]          # Required capabilities
```

For `promptonly` skills, the SKILL.md file content is injected directly into the agent's system prompt when the skill is active.

---

## 60 Bundled Skills

All bundled skills ship inside the binary (no downloads). They are `promptonly` type — pure expert knowledge injected as system prompt context.

### DevOps & Infrastructure (8)

| Skill | Description |
|-------|-------------|
| `github` | GitHub operations: PRs, issues, Actions, `gh` CLI workflows |
| `docker` | Container building, Compose, multi-stage builds, health checks |
| `kubernetes` | kubectl, Helm, deployments, debugging, RBAC |
| `terraform` | IaC provisioning, modules, state management |
| `ansible` | Configuration management, playbooks, roles |
| `ci-cd` | Pipeline design, GitHub Actions, GitLab CI, Jenkins |
| `nginx` | Reverse proxy, load balancing, SSL, config optimization |
| `sysadmin` | Server administration, troubleshooting, monitoring |

### Cloud Platforms (3)

| Skill | Description |
|-------|-------------|
| `aws` | EC2, S3, Lambda, IAM, CLI, CDK |
| `gcp` | GKE, Cloud Run, BigQuery, IAM, CLI |
| `azure` | VMs, AKS, Functions, ARM templates |

### Languages (8)

| Skill | Description |
|-------|-------------|
| `rust-expert` | Ownership, lifetimes, async, unsafe, performance |
| `python-expert` | Idiomatic Python, typing, async, packaging |
| `typescript-expert` | TypeScript, generics, decorators, tooling |
| `golang-expert` | Go patterns, goroutines, interfaces, testing |
| `react-expert` | React 19, hooks, state management, performance |
| `nextjs-expert` | Next.js App Router, SSR/SSG, deployment |
| `css-expert` | Modern CSS, Tailwind, animations, layouts |
| `shell-scripting` | POSIX sh, bash, zsh, cross-platform scripts |

### Databases (6)

| Skill | Description |
|-------|-------------|
| `postgres-expert` | Query optimization, indexing, tuning, replication |
| `redis-expert` | Data structures, caching patterns, Lua scripting |
| `mongodb` | Schema design, aggregation, indexing |
| `sqlite-expert` | Embedded SQLite, WAL, FTS, virtual tables |
| `elasticsearch` | Mappings, queries, aggregations, relevance tuning |
| `sql-analyst` | SQL query writing, optimization, analytics |

### APIs & Web (5)

| Skill | Description |
|-------|-------------|
| `api-tester` | HTTP testing, REST/GraphQL debugging, Postman patterns |
| `graphql-expert` | Schema design, resolvers, federation, performance |
| `openapi-expert` | OpenAPI 3.x spec writing, validation, code generation |
| `oauth-expert` | OAuth 2.0, OIDC, JWT, token management |
| `web-search` | Information research with source citation |

### AI & ML (4)

| Skill | Description |
|-------|-------------|
| `ml-engineer` | Model training, evaluation, MLOps, serving |
| `llm-finetuning` | LoRA, QLoRA, RLHF, dataset preparation |
| `vector-db` | Embeddings, similarity search, Pinecone, Qdrant, pgvector |
| `prompt-engineer` | Prompt design, chain-of-thought, few-shot, evaluation |

### Security (3)

| Skill | Description |
|-------|-------------|
| `security-audit` | Vulnerability assessment, threat modeling, OWASP |
| `crypto-expert` | Cryptographic primitives, TLS, key management |
| `compliance` | SOC2, GDPR, HIPAA, ISO27001 compliance guidance |

### Developer Tools (5)

| Skill | Description |
|-------|-------------|
| `git-expert` | Advanced Git: rebasing, bisect, reflog, hooks, worktrees |
| `code-reviewer` | Code review across languages, best practices |
| `jira` | Issue tracking, sprints, JQL queries, project management |
| `linear-tools` | Linear project management, workflows |
| `sentry` | Error tracking, performance monitoring, alerts |

### Productivity & Writing (6)

| Skill | Description |
|-------|-------------|
| `technical-writer` | Technical documentation, READMEs, API docs |
| `writing-coach` | Writing improvement, style, clarity |
| `email-writer` | Professional email drafting, tone matching |
| `presentation` | Slide decks, storytelling, executive summaries |
| `project-manager` | Project planning, risk management, stakeholder comms |
| `pdf-reader` | PDF extraction, summarization, OCR |

### Collaboration (4)

| Skill | Description |
|-------|-------------|
| `slack-tools` | Slack API, workflow builder, message formatting |
| `notion` | Notion database API, blocks, workflows |
| `confluence` | Confluence pages, spaces, macros |
| `figma-expert` | Figma components, design tokens, developer handoff |

### Data (3)

| Skill | Description |
|-------|-------------|
| `data-analyst` | SQL, pandas, data visualization, statistics |
| `data-pipeline` | ETL/ELT, orchestration, dbt, streaming |
| `prometheus` | Metrics, PromQL, alerting, Grafana dashboards |

### Career & Learning (2)

| Skill | Description |
|-------|-------------|
| `interview-prep` | Technical and behavioral interview preparation |
| `wasm-expert` | WebAssembly, WASI, Wasmtime, component model |

---

## Installing Skills

```bash
# From FangHub marketplace
openfang skill install web-summarizer
openfang skill install kubernetes-debugger

# From Git URL
openfang skill install https://github.com/user/my-skill

# From local directory
openfang skill install ./path/to/my-skill
```

---

## Creating a PromptOnly Skill

The simplest skill type — pure expert knowledge in Markdown:

```
my-skill/
├── skill.toml
└── SKILL.md
```

`skill.toml`:
```toml
[skill]
name = "my-domain-expert"
version = "0.1.0"
description = "Expert in my domain"
author = "yourname"
license = "MIT"
tags = ["domain", "expert"]

[runtime]
type = "promptonly"
```

`SKILL.md`:
```markdown
---
skill: my-domain-expert
version: 0.1.0
---

## My Domain Expert

### Key Principles
- Always validate inputs before processing
- Use established patterns when available
- Document edge cases explicitly

### Common Patterns
...

### Pitfalls to Avoid
...
```

---

## Creating a Python Skill

```
my-tool/
├── skill.toml
├── SKILL.md
└── main.py
```

`skill.toml`:
```toml
[skill]
name = "my-tool"
version = "0.1.0"
description = "Does something useful"
author = "yourname"

[runtime]
type = "python"
entry = "main.py"

[[tools.provided]]
name = "my_action"
description = "Performs the action"
input_schema = {
  type = "object",
  properties = {
    input = { type = "string" }
  },
  required = ["input"]
}
```

`main.py`:
```python
import sys
import json

def main():
    # Read tool call from stdin
    data = json.load(sys.stdin)
    tool_name = data["tool"]
    params = data["params"]

    if tool_name == "my_action":
        result = do_something(params["input"])
        print(json.dumps({"result": result}))
    else:
        print(json.dumps({"error": f"Unknown tool: {tool_name}"}))

def do_something(input_str):
    return f"Processed: {input_str}"

if __name__ == "__main__":
    main()
```

---

## FangHub Marketplace

FangHub is the community marketplace for OpenFang skills.

```bash
# Search
openfang skill search "kubernetes"
openfang skill search --category devops

# Browse by category (18 categories)
# Coding, DevOps, AI, Data, Productivity, Security, Cloud, Database,
# APIs, Frontend, DevTools, Writing, Collaboration, Career, ML,
# Infrastructure, Networking, Testing

# Install
openfang skill install <slug>

# View details
openfang skill info <slug>
```

Web dashboard: **Skills** tab → **ClawHub Marketplace** section.

---

## Assigning Skills to Agents

Skills can be assigned globally or per-agent:

```toml
# In agent.toml
[skills]
enabled = ["github", "docker", "rust-expert"]
```

Via API:
```bash
# Get agent's active skills
GET /api/agents/<id>/skills

# Update agent skills
PUT /api/agents/<id>/skills
{ "skills": ["github", "kubernetes", "terraform"] }
```

---

## Security Scanning

All skills are scanned for prompt injection patterns before installation:
- Override instructions (e.g., "ignore previous instructions")
- Data exfiltration attempts
- Capability escalation patterns

The scanner runs automatically on `openfang skill install`.
