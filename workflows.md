# Workflows

Workflows are multi-agent pipelines that chain agents together in configurable patterns: sequential, parallel fan-out, conditional, and loops.

---

## Workflow Definition

Workflows are defined as JSON (or created via the visual drag-and-drop builder in the web dashboard):

```json
{
  "name": "Research and Write",
  "description": "Deep research followed by article writing",
  "steps": [
    {
      "name": "research",
      "agent": "researcher",
      "mode": "sequential",
      "prompt": "Research the following topic thoroughly: {{input}}",
      "output_var": "research_results",
      "timeout_secs": 300,
      "on_error": "fail"
    },
    {
      "name": "write",
      "agent": "writer",
      "mode": "sequential",
      "prompt": "Write a comprehensive article based on this research:\n\n{{research_results}}",
      "timeout_secs": 120,
      "on_error": "retry",
      "max_retries": 2
    }
  ]
}
```

---

## Step Modes

### `sequential`

Steps execute one after another. Output of step N becomes `{{input}}` for step N+1.

```json
{"mode": "sequential"}
```

### `fan_out`

Multiple consecutive `fan_out` steps execute in parallel, each receiving the same input.

```json
[
  {"name": "analyze-tone",     "agent": "analyst",  "mode": "fan_out"},
  {"name": "check-facts",      "agent": "researcher","mode": "fan_out"},
  {"name": "review-structure", "agent": "editor",   "mode": "fan_out"},
  {"name": "collect-results",  "agent": "writer",   "mode": "collect"}
]
```

### `collect`

Follows a group of `fan_out` steps. Joins all parallel outputs with `\n\n---\n\n` separator.

### `conditional`

Executes only if the previous output contains a specified substring.

```json
{
  "mode": "conditional",
  "condition": "NEEDS_REVIEW",
  "prompt": "This content needs review: {{input}}"
}
```

### `loop`

Repeats until a stop condition matches or max iterations is reached.

```json
{
  "mode": "loop",
  "max_iterations": 5,
  "stop_condition": "COMPLETE",
  "prompt": "Continue improving this: {{input}}"
}
```

---

## Variable Substitution

| Variable | Value |
|----------|-------|
| `{{input}}` | Output from the previous step (or initial workflow input) |
| `{{named_var}}` | Named variable stored with `output_var` in a previous step |
| `{{event}}` | Human-readable description (in trigger-fired workflows) |

---

## Error Handling

| Policy | Behavior |
|--------|----------|
| `fail` | Abort entire workflow (default) |
| `skip` | Continue workflow, pass previous input to next step |
| `retry` | Retry up to `max_retries` times with full timeout per attempt |

---

## Execution Limits

| Limit | Default | Notes |
|-------|---------|-------|
| `timeout_secs` per step | 60 | Range: 1–3600 |
| `max_retries` | 0 | Only with `"on_error": "retry"` |
| `max_iterations` (loop) | 5 | |
| Retained run history | 200 | Oldest completed/failed evicted |

---

## Workflow API

### Create workflow

```bash
curl -X POST http://127.0.0.1:4200/api/workflows \
  -H "Authorization: Bearer <key>" \
  -H "Content-Type: application/json" \
  -d @workflow.json
```

### List workflows

```bash
GET /api/workflows
```

### Run workflow

```bash
curl -X POST http://127.0.0.1:4200/api/workflows/<id>/run \
  -H "Authorization: Bearer <key>" \
  -d '{"input": "Your topic here"}'
```

### Get run history

```bash
GET /api/workflows/<id>/runs
```

### Update workflow

```bash
PATCH /api/workflows/<id>
```

### Delete workflow

```bash
DELETE /api/workflows/<id>
```

---

## Workflow CLI

```bash
openfang workflow list
openfang workflow create workflow.json
openfang workflow run <workflow-id> --input "Research AI safety"
openfang workflow run <workflow-id> --input-file prompt.txt
```

---

## Visual Workflow Builder

The web dashboard **Workflow Builder** tab provides a drag-and-drop SVG canvas:

- 7 node types: Start, Agent, Fan-out, Collect, Conditional, Loop, End
- Connect nodes with directional edges
- Configure each node via sidebar panel
- Export to TOML format
- Real-time workflow graph visualization

---

## Trigger Engine

Workflows (and individual agents) can be activated by events:

### Event Pattern Types

| Pattern | Fires When |
|---------|-----------|
| `All` | Any event |
| `Lifecycle` | Any agent lifecycle event |
| `AgentSpawned` | Specific agent spawned |
| `AgentTerminated` | Specific agent terminated |
| `System` | Any system event |
| `SystemKeyword` | System event containing keyword |
| `MemoryUpdate` | Agent memory is updated |
| `MemoryKeyPattern` | Memory key matches glob pattern |
| `ContentMatch` | Output text matches substring/regex |

### Creating Triggers

```bash
curl -X POST http://127.0.0.1:4200/api/events/triggers \
  -H "Authorization: Bearer <key>" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "<workflow-id>",
    "pattern": {"content_match": {"pattern": "deploy complete"}},
    "message": "Deployment completed. Run post-deploy checks: {{event}}",
    "max_fires": 0
  }'
```

`max_fires: 0` means unlimited. A positive number auto-disables the trigger after N fires.

### CLI

```bash
openfang trigger list
openfang trigger create <agent-id> '{"lifecycle": {}}' \
  --message "Agent lifecycle event: {{event}}"
openfang trigger delete <trigger-id>
```

---

## Example Workflows

### Code Review Pipeline

```json
{
  "name": "Code Review",
  "steps": [
    {
      "name": "review",
      "agent": "code-reviewer",
      "mode": "fan_out",
      "prompt": "Review this code for correctness: {{input}}"
    },
    {
      "name": "security-check",
      "agent": "security-auditor",
      "mode": "fan_out",
      "prompt": "Security audit this code: {{input}}"
    },
    {
      "name": "synthesize",
      "agent": "architect",
      "mode": "collect",
      "prompt": "Synthesize these reviews into actionable feedback:\n\n{{input}}"
    }
  ]
}
```

### Parallel Research + Write

```json
{
  "name": "Multi-angle Research",
  "steps": [
    {
      "name": "technical",
      "agent": "researcher",
      "mode": "fan_out",
      "prompt": "Research technical aspects of: {{input}}"
    },
    {
      "name": "business",
      "agent": "analyst",
      "mode": "fan_out",
      "prompt": "Research business implications of: {{input}}"
    },
    {
      "name": "risks",
      "agent": "security-auditor",
      "mode": "fan_out",
      "prompt": "Research risks and downsides of: {{input}}"
    },
    {
      "name": "write",
      "agent": "writer",
      "mode": "collect",
      "prompt": "Write a balanced analysis using these research angles:\n\n{{input}}"
    }
  ]
}
```

### Iterative Refinement Loop

```json
{
  "name": "Iterative Refinement",
  "steps": [
    {
      "name": "draft",
      "agent": "writer",
      "mode": "sequential",
      "prompt": "Write a first draft about: {{input}}"
    },
    {
      "name": "refine",
      "agent": "editor",
      "mode": "loop",
      "max_iterations": 3,
      "stop_condition": "FINAL",
      "prompt": "Improve this draft (output FINAL when satisfied):\n\n{{input}}"
    }
  ]
}
```
