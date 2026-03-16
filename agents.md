# Agents

An agent is an autonomous AI entity with its own identity, persona, memory, capabilities, and skill set. Agents run continuously, respond to messages across multiple channels, execute tools, and can spawn other agents.

---

## Agent States

```
Spawned → Running → Suspended ⇄ Resumed → Terminated
                              ↘ Crashed
```

| State | Description |
|-------|-------------|
| `Spawned` | Agent record created, not yet initialized |
| `Running` | Active, processing messages and executing tools |
| `Suspended` | Paused (can be resumed) |
| `Terminated` | Cleanly shut down |
| `Crashed` | Unexpected failure |

---

## Agent Modes

| Mode | Description |
|------|-------------|
| `Full` | Complete autonomy — executes all permitted tool calls |
| `Assist` | Suggests actions but waits for confirmation |
| `Observe` | Read-only — no tool calls, only analysis |

---

## Agent Manifest (agent.toml)

Every agent is defined by a TOML manifest:

```toml
[agent]
name = "my-assistant"
description = "General-purpose helpful assistant"
version = "1.0.0"

[model]
provider = "groq"
model = "llama-3.3-70b-versatile"
temperature = 0.7
max_tokens = 4096

[persona]
system_prompt = """
You are a helpful assistant. Be concise and accurate.
Always use available tools when they would help you give a better answer.
"""

[capabilities]
required = [
  "ToolInvoke(web_fetch)",
  "ToolInvoke(file_read)",
  "ToolInvoke(shell_exec)",
  "MemoryRead",
  "MemoryWrite",
  "NetConnect(*)",
]

[tools]
# Optional: restrict to specific tools (omit = use default profile)
allowed = ["web_fetch", "file_read", "file_write", "memory_store", "memory_recall"]

[skills]
# Skills to inject into agent's system prompt
enabled = ["github", "docker", "git-expert"]

[automation]
require_approval = ["shell_exec"]    # Tools requiring human confirmation
auto_approve_autonomous = false      # In autonomous mode, skip approval

[quota]
max_tokens_per_hour = 0             # 0 = unlimited
```

---

## 30 Pre-built Agent Templates

Templates live in the `agents/` directory. Spawn any with:
```bash
openfang agent spawn agents/<name>/agent.toml
```

### Tier 1 — Frontier (Claude Opus / DeepSeek)

Best for complex reasoning, architecture, and high-stakes tasks.

| Template | Description |
|----------|-------------|
| `orchestrator` | Multi-agent task coordination and delegation |
| `architect` | System design, technical architecture, ADRs |
| `security-auditor` | Vulnerability assessment, threat modeling, compliance |

### Tier 2 — Smart (Gemini Pro / Claude Sonnet)

Balanced intelligence for technical work.

| Template | Description |
|----------|-------------|
| `coder` | Software development across languages |
| `code-reviewer` | Code review, refactoring, best practices |
| `data-scientist` | Data analysis, ML modeling, statistics |
| `researcher` | Deep research with source validation |
| `analyst` | Business analysis, reporting, insights |
| `test-engineer` | Test strategy, TDD, coverage analysis |
| `legal-assistant` | Contract review, compliance, legal research |
| `devops-lead` | Infrastructure, CI/CD, deployment |
| `debugger` | Root cause analysis, bug investigation |

### Tier 3 — Balanced (Gemini Flash / Groq Llama)

Cost-efficient for general work and communication.

| Template | Description |
|----------|-------------|
| `assistant` | General-purpose helpful assistant |
| `planner` | Project planning, task breakdown, scheduling |
| `writer` | Long-form writing, content creation |
| `doc-writer` | Technical documentation, READMEs, API docs |
| `email-assistant` | Email drafting, tone adjustment, responses |
| `customer-support` | Support ticket handling, FAQ responses |
| `sales-assistant` | Lead qualification, outreach, proposals |
| `recruiter` | Job descriptions, candidate screening |
| `meeting-assistant` | Meeting notes, action items, summaries |
| `social-media` | Social content creation and scheduling |
| `personal-finance` | Budgeting, expense tracking, financial advice |

### Tier 4 — Fast (Groq / Local)

Ultra-low latency for real-time and high-volume tasks.

| Template | Description |
|----------|-------------|
| `ops` | Operations monitoring, incident response |
| `hello-world` | Minimal example agent for testing |
| `translator` | Multi-language translation |
| `tutor` | Educational explanations and Socratic teaching |
| `health-tracker` | Fitness, nutrition, wellness tracking |
| `travel-planner` | Itinerary planning, bookings, travel advice |
| `home-automation` | Smart home, IoT, automation scripts |

---

## Spawning Agents

### Via CLI

```bash
openfang agent spawn agents/coder/agent.toml
openfang agent spawn agents/researcher/agent.toml --name research-bot
```

### Via API

```bash
curl -X POST http://127.0.0.1:4200/api/agents \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "template": "coder",
    "name": "my-coder",
    "model": "claude-sonnet-4-6"
  }'
```

### Via Web Dashboard

Open **Overview** → **Spawn Wizard** (multi-step: identity → soul → confirm).

The wizard includes emoji picker, archetype selector, and personality presets:
- Professional, Friendly, Technical, Creative, Concise, Mentor

---

## Multi-Session Support

Each agent supports multiple concurrent sessions (e.g., one per Telegram user, one for the web dashboard):

```bash
# List sessions
curl http://127.0.0.1:4200/api/agents/<id>/sessions \
  -H "Authorization: Bearer <key>"

# Create a named session
curl -X POST http://127.0.0.1:4200/api/agents/<id>/sessions \
  -d '{"label": "project-alpha"}'

# Switch active session
curl -X POST http://127.0.0.1:4200/api/agents/<id>/sessions/<session_id>/switch
```

---

## Model Routing

Agents can automatically select the best model based on task complexity:

```toml
[model_routing]
enabled = true
simple_threshold = 0.3      # Route to fast model
complex_threshold = 0.8     # Route to frontier model
simple_model = "llama-3.1-8b-instant"
balanced_model = "llama-3.3-70b-versatile"
complex_model = "claude-opus-4-6"
```

---

## Approval Workflows

For dangerous operations, agents can request human approval:

```toml
[automation]
require_approval = ["shell_exec", "file_write"]
approval_timeout_secs = 60     # 10-300 seconds
```

**Risk levels:**
- 🟢 `Low` — Read-only operations
- 🟡 `Medium` — Reversible writes
- 🔴 `High` — Potentially destructive
- 🆘 `Critical` — Irreversible actions

Approvals are visible in the web dashboard **Approvals** tab and via API:

```bash
# List pending approvals
GET /api/approvals

# Approve
POST /api/approvals/<id>/approve

# Reject
POST /api/approvals/<id>/reject
```

---

## Cloning Agents

```bash
openfang agent clone <agent-id> --name my-clone
```

Clones copy the agent manifest, persona, and skill configuration. Memory is not copied (fresh start).

---

## Agent-to-Agent Communication

Agents can send messages to each other:

```bash
# Via the agent_message tool
<invoke name="agent_message">{"target": "researcher", "message": "Research quantum computing advances"}</invoke>

# Via API
POST /api/comms/send
{
  "from": "orchestrator-id",
  "to": "researcher-id",
  "message": "Research quantum computing advances"
}
```

For agents on different machines, use the OFP wire protocol — see [Architecture](architecture.md#openfang-wire-protocol-ofp).
