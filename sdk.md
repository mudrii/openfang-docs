# SDKs

OpenFang provides official SDKs for JavaScript/TypeScript and Python, plus an OpenAI-compatible API that works with any OpenAI SDK.

---

## JavaScript / TypeScript SDK

**Package:** `@openfang/sdk`

```bash
npm install @openfang/sdk
# or
yarn add @openfang/sdk
# or
pnpm add @openfang/sdk
```

### Basic Usage

```javascript
const { OpenFang } = require("@openfang/sdk");

const client = new OpenFang("http://localhost:4200", {
  apiKey: "your-api-key"   // Optional; required if auth is enabled
});

// List agents
const agents = await client.agents.list();

// Spawn an agent
const agent = await client.agents.create({
  template: "assistant",
  name: "my-assistant"
});

// Send a message
const response = await client.agents.message(agent.id, "Hello!");
console.log(response.text);

// Delete an agent
await client.agents.delete(agent.id);
```

### Streaming

```javascript
// Streaming chat with async generator
for await (const event of client.agents.stream(agent.id, "Explain async/await")) {
  if (event.type === "text") {
    process.stdout.write(event.delta);
  } else if (event.type === "token_usage") {
    console.log(`\nTokens: ${event.input} in / ${event.output} out`);
  } else if (event.type === "done") {
    console.log("\n[Complete]");
  }
}
```

### Resource Classes

| Class | Description |
|-------|-------------|
| `client.agents` | Agent CRUD, messaging, sessions |
| `client.sessions` | Session management |
| `client.workflows` | Workflow creation and execution |
| `client.skills` | Skill installation and management |
| `client.channels` | Channel configuration |
| `client.tools` | Tool discovery |
| `client.models` | Model catalog |
| `client.providers` | Provider management |
| `client.memory` | KV and semantic memory |
| `client.triggers` | Event trigger management |
| `client.schedules` | Cron job management |

### TypeScript

Full TypeScript type definitions are included in `index.d.ts`:

```typescript
import { OpenFang, Agent, StreamEvent } from "@openfang/sdk";

const client = new OpenFang("http://localhost:4200");

const agent: Agent = await client.agents.create({
  template: "coder",
  name: "my-coder"
});

for await (const event: StreamEvent of client.agents.stream(agent.id, "Write fizzbuzz in Rust")) {
  if (event.type === "text") {
    process.stdout.write(event.delta);
  }
}
```

### Error Handling

```javascript
const { OpenFang, OpenFangError } = require("@openfang/sdk");

try {
  const response = await client.agents.message("nonexistent-id", "Hello");
} catch (e) {
  if (e instanceof OpenFangError) {
    console.error(`Status: ${e.status}, Error: ${e.message}`);
  }
}
```

---

## Python SDK

**Package:** `openfang`

```bash
pip install openfang
```

### Agent Framework (for building OpenFang-compatible agents)

```python
from openfang_sdk import Agent

agent = Agent()

@agent.on_setup
def setup(context: dict) -> None:
    """Called once when agent starts."""
    print("Agent initialized")

@agent.on_message
def handle_message(message: str, context: dict) -> str:
    """Called for each incoming message. Returns response."""
    return f"You said: {message}"

@agent.on_teardown
def teardown(context: dict) -> None:
    """Called on agent shutdown."""
    print("Agent shutting down")

if __name__ == "__main__":
    agent.run()
```

### REST Client

```python
from openfang_client import OpenFangClient

client = OpenFangClient(
    base_url="http://localhost:4200",
    api_key="your-api-key"  # Optional
)

# List agents
agents = client.agents.list()

# Send message
response = client.agents.message(agents[0]["id"], "Hello from Python!")
print(response["text"])

# Streaming
for event in client.agents.stream(agents[0]["id"], "Write a poem"):
    if event["type"] == "text":
        print(event["delta"], end="", flush=True)
```

### Utilities

```python
from openfang_sdk import read_input, respond, log

def main():
    # Read JSON tool call from stdin (for skill scripts)
    data = read_input()

    # Log to stderr (doesn't pollute stdout)
    log(f"Processing: {data['tool']}")

    # Write JSON result to stdout
    respond({"result": "processed"})

main()
```

### Echo Agent Example

```python
from openfang_sdk import Agent

agent = Agent()

@agent.on_message
def echo(message: str, context: dict) -> str:
    return f"Echo: {message}"

agent.run()
```

---

## OpenAI-Compatible API

Use OpenFang as a drop-in replacement for the OpenAI API:

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:4200/v1",
    api_key="your-api-key"
)

# Non-streaming
response = client.chat.completions.create(
    model="llama-3.3-70b-versatile",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)

# Streaming
stream = client.chat.completions.create(
    model="claude-sonnet-4-6",
    messages=[{"role": "user", "content": "Write a haiku"}],
    stream=True
)
for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

```javascript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "http://localhost:4200/v1",
  apiKey: "your-api-key"
});

const stream = await client.chat.completions.create({
  model: "llama-3.3-70b-versatile",
  messages: [{ role: "user", content: "Hello!" }],
  stream: true
});

for await (const chunk of stream) {
  process.stdout.write(chunk.choices[0]?.delta?.content ?? "");
}
```

### Available endpoints

```
POST /v1/chat/completions    # Chat completion (streaming + non-streaming)
GET  /v1/models              # List available models
```

---

## Direct HTTP API

Use any HTTP client directly:

```bash
# List agents
curl http://127.0.0.1:4200/api/agents \
  -H "Authorization: Bearer <api_key>"

# Send message
curl -X POST http://127.0.0.1:4200/api/agents/<id>/message \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello!"}'

# Stream response
curl -N -X POST http://127.0.0.1:4200/api/agents/<id>/message/stream \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-Type: application/json" \
  -d '{"message": "Write a story", "stream": true}'
```

See [API Reference](api-reference.md) for the complete endpoint list.
