# Skills

Skills inject expert knowledge into agents. They are Markdown files (PromptOnly), Python scripts, Node.js scripts, WASM modules, or shell scripts that extend agent capabilities. OpenFang v0.5.7 ships with **61 bundled skills** compiled into the binary.

---

## Skill Runtimes

| Runtime | Language | Sandboxed | Description |
|---------|----------|-----------|-------------|
| `promptonly` | Markdown | N/A | Expert knowledge injected into the LLM system prompt. No code execution. |
| `python` | Python 3.8+ | Subprocess (`env_clear()`) | JSON stdin/stdout protocol. Environment isolated to prevent secret leakage. |
| `node` | JavaScript | Subprocess (`env_clear()`) | OpenClaw-compatible. Same isolation as Python. |
| `wasm` | Rust, C, Go | Wasmtime sandbox | Dual metering (fuel + epoch). Best for security-sensitive tools. |
| `shell` | Bash/sh | Subprocess (`env_clear()`) | Shell script execution. Use sparingly. |
| `builtin` | Rust | N/A | Compiled into the binary. For core tools only. |

All subprocess runtimes (`python`, `node`, `shell`) call `env_clear()` before spawning. Only `PATH`, `HOME`, and platform essentials (`SYSTEMROOT`/`TEMP` on Windows, `PYTHONIOENCODING` for Python) are forwarded. This prevents third-party skill code from reading API keys, tokens, or credentials from the host environment.

---

## 61 Bundled Skills

All bundled skills ship inside the binary via `include_str!()` -- no downloads needed. They are `promptonly` type: pure expert knowledge injected as system prompt context when assigned to an agent.

User-installed skills with the same name override bundled ones.

### DevOps & Infrastructure (11)

| Skill | Description |
|-------|-------------|
| `ansible` | Ansible automation expert for playbooks, roles, inventories, and infrastructure management |
| `ci-cd` | CI/CD pipeline expert for GitHub Actions, GitLab CI, Jenkins, and deployment automation |
| `docker` | Docker expert for containers, Compose, Dockerfiles, and debugging |
| `helm` | Helm chart expert for Kubernetes package management, templating, and dependency management |
| `kubernetes` | Kubernetes operations expert for kubectl, pods, deployments, and debugging |
| `linux-networking` | Linux networking expert for iptables, nftables, routing, DNS, and network troubleshooting |
| `nginx` | Nginx configuration expert for reverse proxy, load balancing, TLS, and performance tuning |
| `shell-scripting` | Shell scripting expert for Bash, POSIX compliance, error handling, and automation |
| `sysadmin` | System administration expert for Linux, macOS, Windows, services, and monitoring |
| `terraform` | Terraform IaC expert for providers, modules, state management, and planning |
| `prometheus` | Prometheus monitoring expert for PromQL, alerting rules, Grafana dashboards, and observability |

### Cloud Platforms (3)

| Skill | Description |
|-------|-------------|
| `aws` | AWS cloud services expert for EC2, S3, Lambda, IAM, and AWS CLI |
| `azure` | Microsoft Azure expert for az CLI, AKS, App Service, and cloud infrastructure |
| `gcp` | Google Cloud Platform expert for gcloud CLI, GKE, Cloud Run, and managed services |

### Languages (5)

| Skill | Description |
|-------|-------------|
| `golang-expert` | Go programming expert for goroutines, channels, interfaces, modules, and concurrency patterns |
| `python-expert` | Python expert for stdlib, packaging, type hints, async/await, and performance optimization |
| `rust-expert` | Rust programming expert for ownership, lifetimes, async/await, traits, and unsafe code |
| `typescript-expert` | TypeScript expert for type system, generics, utility types, and strict mode patterns |
| `wasm-expert` | WebAssembly expert for WASI, component model, Rust/C compilation, and browser integration |

### Frontend (3)

| Skill | Description |
|-------|-------------|
| `css-expert` | CSS expert for flexbox, grid, animations, responsive design, and modern layout techniques |
| `nextjs-expert` | Next.js expert for App Router, SSR/SSG, API routes, middleware, and deployment |
| `react-expert` | React expert for hooks, state management, Server Components, and performance optimization |

### Databases (6)

| Skill | Description |
|-------|-------------|
| `elasticsearch` | Elasticsearch expert for queries, mappings, aggregations, index management, and cluster operations |
| `mongodb` | MongoDB operations expert for queries, aggregation pipelines, indexes, and schema design |
| `postgres-expert` | PostgreSQL expert for query optimization, indexing, extensions, and database administration |
| `redis-expert` | Redis expert for data structures, caching patterns, Lua scripting, and cluster operations |
| `sql-analyst` | SQL query expert for optimization, schema design, and data analysis |
| `sqlite-expert` | SQLite expert for WAL mode, query optimization, embedded patterns, and advanced features |

### APIs & Web (6)

| Skill | Description |
|-------|-------------|
| `api-tester` | API testing expert for curl, REST, GraphQL, authentication, and debugging |
| `graphql-expert` | GraphQL expert for schema design, resolvers, subscriptions, and performance optimization |
| `oauth-expert` | OAuth 2.0 and OpenID Connect expert for authorization flows, PKCE, and token management |
| `openapi-expert` | OpenAPI/Swagger expert for API specification design, validation, and code generation |
| `searxng` | Privacy-respecting metasearch specialist using SearXNG instances |
| `web-search` | Web search and research specialist for finding and synthesizing information |

### AI & ML (4)

| Skill | Description |
|-------|-------------|
| `llm-finetuning` | LLM fine-tuning expert for LoRA, QLoRA, dataset preparation, and training optimization |
| `ml-engineer` | Machine learning engineer expert for PyTorch, scikit-learn, model evaluation, and MLOps |
| `prompt-engineer` | Prompt engineering expert for chain-of-thought, few-shot learning, evaluation, and LLM optimization |
| `vector-db` | Vector database expert for embeddings, similarity search, RAG patterns, and indexing strategies |

### Security (3)

| Skill | Description |
|-------|-------------|
| `compliance` | Compliance expert for SOC 2, GDPR, HIPAA, PCI-DSS, and security frameworks |
| `crypto-expert` | Cryptography expert for TLS, symmetric/asymmetric encryption, hashing, and key management |
| `security-audit` | Security audit expert for OWASP Top 10, CVE analysis, code review, and penetration testing methodology |

### Developer Tools (7)

| Skill | Description |
|-------|-------------|
| `code-reviewer` | Code review specialist focused on patterns, bugs, security, and performance |
| `git-expert` | Git operations expert for branching, rebasing, conflicts, and workflows |
| `github` | GitHub operations expert for PRs, issues, code review, Actions, and gh CLI |
| `jira` | Jira project management expert for issues, sprints, workflows, and reporting |
| `linear-tools` | Linear project management expert for issues, cycles, projects, and workflow automation |
| `regex-expert` | Regular expression expert for crafting, debugging, and explaining patterns |
| `sentry` | Sentry error tracking and debugging specialist |

### Productivity & Writing (6)

| Skill | Description |
|-------|-------------|
| `email-writer` | Professional email writing expert for tone, structure, clarity, and business communication |
| `presentation` | Presentation expert for slide structure, storytelling, visual design, and audience engagement |
| `project-manager` | Project management expert for Agile, estimation, risk management, and stakeholder communication |
| `technical-writer` | Technical writing expert for API docs, READMEs, ADRs, and developer documentation |
| `writing-coach` | Writing improvement specialist for grammar, style, clarity, and structure |
| `pdf-reader` | PDF content extraction and analysis specialist |

### Data (2)

| Skill | Description |
|-------|-------------|
| `data-analyst` | Data analysis expert for statistics, visualization, pandas, and exploration |
| `data-pipeline` | Data pipeline expert for ETL, Apache Spark, Airflow, dbt, and data quality |

### Collaboration (4)

| Skill | Description |
|-------|-------------|
| `confluence` | Confluence wiki expert for page structure, spaces, macros, and content organization |
| `figma-expert` | Figma design expert for components, auto-layout, design systems, and developer handoff |
| `notion` | Notion workspace management and content creation specialist |
| `slack-tools` | Slack workspace management and automation specialist |

### Career (1)

| Skill | Description |
|-------|-------------|
| `interview-prep` | Technical interview preparation expert for algorithms, system design, and behavioral questions |

---

## Installing Skills

### From ClawHub Marketplace

```bash
openfang skill install web-summarizer
openfang skill install kubernetes-debugger
```

### From a Git Repository

```bash
openfang skill install https://github.com/user/my-skill
```

### From a Local Directory

```bash
openfang skill install ./path/to/my-skill
openfang skill install /absolute/path/to/my-skill
```

### Listing Installed Skills

```bash
openfang skill list
```

### Removing Skills

```bash
openfang skill remove web-summarizer
```

---

## Uninstalling Skills

```bash
openfang skill remove <name>
```

This removes the skill directory from `~/.openfang/skills/` and unregisters it from the skill registry. Bundled skills cannot be removed (they are compiled into the binary), but they can be overridden by installing a user skill with the same name.

---

## FangHub and ClawHub Marketplaces

OpenFang supports two skill marketplaces:

### ClawHub (clawhub.ai)

The primary community marketplace with 3,000+ skills in both SKILL.md and Node.js formats.

```bash
# Search
openfang skill search "kubernetes"

# Browse by sort order
openfang skill search --sort trending
openfang skill search --sort downloads
openfang skill search --sort stars

# Install
openfang skill install <slug>

# View details
openfang skill info <slug>
```

The ClawHub client handles:
- Automatic retry on rate limits (429) and server errors (5xx) with exponential backoff (up to 5 attempts)
- Respect for `Retry-After` headers
- SHA256 integrity verification of downloaded content
- Automatic format detection (SKILL.md vs package.json)
- Prompt injection scanning before installation
- Security scan of skill manifests

### FangHub (GitHub-based)

A GitHub-organization-based registry for skills distributed as GitHub releases.

```bash
openfang skill search "web scraping"
openfang skill install <skill-name>
```

### Web Dashboard

The dashboard **Skills** tab provides a visual interface to the ClawHub marketplace. Browse trending skills, view details, and install with one click.

---

## Assigning Skills to Agents

Skills can be assigned per-agent in the agent manifest:

```toml
# In agent.toml
[skills]
enabled = ["github", "docker", "rust-expert"]
```

Via the REST API:

```bash
# Get agent's active skills
GET /api/agents/<id>/skills

# Update agent skills
PUT /api/agents/<id>/skills
{ "skills": ["github", "kubernetes", "terraform"] }
```

The kernel loads skill tools and prompt context at agent spawn time, merging them with the agent's base capabilities.

---

## Workspace Skills

Skills can be scoped to a workspace by placing them in a `.openfang/skills/` directory inside your project. Workspace skills override global and bundled skills with the same name.

```
my-project/
  .openfang/
    skills/
      my-custom-skill/
        skill.toml
        SKILL.md
```

Workspace skills go through the same security scanning as global skills. Skills blocked for critical prompt injection patterns are counted and reported.

---

## Hot-Reload

OpenFang detects changes to agent skill assignments and MCP server configurations on config reload. When you modify `skills` or `mcp_servers` in an agent's configuration, the changes take effect on the next config reload without restarting the daemon.

---

## Security Scanning

All skills pass through a multi-layer security pipeline before activation:

**Manifest scan** -- checks for dangerous capabilities:
- Shell execution requests (`ShellExec`, `shell_exec`)
- Unrestricted network access (`NetConnect(*)`)
- Filesystem write tools (`file_write`, `file_delete`)
- Node.js runtime (broad filesystem/network access warning)
- Unusually high tool count (>10 tools)

**Prompt injection scan** -- detects malicious patterns in SKILL.md content:
- Override instructions ("ignore previous instructions", "forget your instructions", etc.)
- Data exfiltration attempts ("send to https://", "base64 encode and send", etc.)
- Shell command references (`rm -rf`, `chmod`, `sudo`)
- Excessive content length (>50KB)

Skills with critical-severity prompt injection patterns are **blocked** and will not load. This defense was implemented after 341 malicious skills were discovered on ClawHub in February 2026.

The scanner runs automatically on:
- `openfang skill install` (ClawHub and local installs)
- Skill registry load at startup (both bundled and user-installed skills)
- Workspace skill loading
- OpenClaw skill migration
