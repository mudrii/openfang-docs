# MCP & A2A Integration

OpenFang implements both the **Model Context Protocol (MCP)** and **Agent-to-Agent (A2A)** protocol, enabling deep interoperability with external tools, IDEs, and other agent frameworks.

---

## Part 1: MCP (Model Context Protocol)

### Overview

MCP is a JSON-RPC 2.0 based protocol that standardizes how LLM applications discover and invoke tools. OpenFang supports MCP in both directions:

- **As a client**: OpenFang connects to external MCP servers (GitHub, filesystem, databases, Puppeteer, etc.) and makes their tools available to all agents.
- **As a server**: OpenFang exposes its own agents as MCP tools, so IDEs like Cursor, VS Code, and Claude Desktop can call OpenFang agents directly.

OpenFang implements MCP protocol version `2024-11-05`.

---

### MCP Client -- Connecting to External Servers

MCP servers are configured in `config.toml` using the `[[mcp_servers]]` array:

```toml
[[mcp_servers]]
name = "github"
timeout_secs = 30
env = ["GITHUB_PERSONAL_ACCESS_TOKEN"]

[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | string | required | Display name, used in tool namespacing |
| `transport` | object | required | How to connect (stdio or SSE) |
| `timeout_secs` | u64 | `30` | JSON-RPC request timeout |
| `env` | array | `[]` | Env vars to pass through to the subprocess |

#### Transport Types

**Stdio** -- Spawns a subprocess and communicates via stdin/stdout with newline-delimited JSON-RPC:

```toml
[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
```

**SSE** -- Connects to a remote HTTP endpoint and sends JSON-RPC via POST:

```toml
[mcp_servers.transport]
type = "sse"
url = "https://mcp.example.com/api"
```

#### Tool Namespacing

All tools discovered from MCP servers are namespaced using the pattern `mcp_{server}_{tool}` to prevent collisions with built-in tools. Names are normalized to lowercase with hyphens replaced by underscores.

Examples:
- Server `github`, tool `create_issue` becomes `mcp_github_create_issue`
- Server `my-server`, tool `do_thing` becomes `mcp_my_server_do_thing`

#### Auto-Connection on Kernel Boot

When the kernel starts, it checks `config.mcp_servers`. If any are configured, it spawns a background task that:

1. Iterates each configured MCP server
2. Spawns the subprocess (stdio) or creates an HTTP client (SSE)
3. Sends the `initialize` handshake with client info
4. Calls `tools/list` to discover all available tools
5. Namespaces each tool with `mcp_{server}_{tool}`
6. Caches discovered tool definitions and stores live connections

MCP tools are merged into the agent's available tool set alongside built-in tools (59) and skill tools.

---

### MCP Server -- Exposing OpenFang via MCP

OpenFang can act as an MCP server, exposing its agents as callable tools to external MCP clients.

Each OpenFang agent becomes an MCP tool named `openfang_agent_{name}` (with hyphens replaced by underscores). The tool accepts a single `message` string parameter and returns the agent's response.

#### CLI: `openfang mcp`

The primary way to run the MCP server is the `openfang mcp` command, which starts a stdio-based MCP server:

```bash
openfang mcp
```

This command:
1. Checks if an OpenFang daemon is running
2. If found, proxies all tool calls to the daemon via its HTTP API
3. If no daemon is running, boots an in-process kernel as a fallback
4. Reads Content-Length framed JSON-RPC messages from stdin
5. Writes Content-Length framed JSON-RPC responses to stdout

#### HTTP MCP Endpoint

OpenFang also exposes an MCP endpoint over HTTP at `POST /mcp`. Unlike the stdio server (which only exposes agents), the HTTP endpoint exposes the full tool set (built-in + skills + MCP tools) and executes tools via the kernel's `execute_tool()` pipeline.

#### Supported JSON-RPC Methods

| Method | Description |
|--------|-------------|
| `initialize` | Handshake; returns server capabilities and info |
| `notifications/initialized` | Client confirmation; no response |
| `tools/list` | Returns all available tools with names, descriptions, and input schemas |
| `tools/call` | Executes a tool and returns the result |

#### Connecting from Cursor / VS Code

Add to your MCP configuration file (e.g., `.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "openfang": {
      "command": "openfang",
      "args": ["mcp"]
    }
  }
}
```

#### Connecting from Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "openfang": {
      "command": "openfang",
      "args": ["mcp"],
      "env": {}
    }
  }
}
```

After configuration, all OpenFang agents appear as tools in the IDE.

---

### MCP Configuration Examples

#### GitHub

```toml
[[mcp_servers]]
name = "github"
timeout_secs = 30
env = ["GITHUB_PERSONAL_ACCESS_TOKEN"]
[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
```

#### Filesystem

```toml
[[mcp_servers]]
name = "filesystem"
timeout_secs = 10
[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"]
```

#### PostgreSQL

```toml
[[mcp_servers]]
name = "postgres"
timeout_secs = 30
env = ["DATABASE_URL"]
[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-postgres"]
```

#### Puppeteer (Browser Automation)

```toml
[[mcp_servers]]
name = "puppeteer"
timeout_secs = 60
[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-puppeteer"]
```

#### Remote SSE Server

```toml
[[mcp_servers]]
name = "remote-tools"
timeout_secs = 30
[mcp_servers.transport]
type = "sse"
url = "https://tools.example.com/mcp"
```

---

### MCP API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/mcp/servers` | List configured and connected MCP servers with their tools |
| `POST` | `/mcp` | Handle MCP JSON-RPC requests over HTTP (full tool execution) |

---

## Part 2: A2A (Agent-to-Agent Protocol)

### Overview

The Agent-to-Agent (A2A) protocol, originally specified by Google, enables cross-framework agent interoperability. It allows agents built with different frameworks to discover each other's capabilities and exchange tasks.

OpenFang implements A2A in both directions:

- **As a server**: Publishes Agent Cards describing each agent's capabilities, accepts task submissions, and tracks task lifecycle.
- **As a client**: Discovers external A2A agents at boot time, sends tasks to them, and polls for results.

---

### Agent Card

An Agent Card is a JSON document that describes an agent's identity, capabilities, and supported interaction modes. It is served at the well-known path `/.well-known/agent.json` per the A2A specification.

```json
{
  "name": "code-reviewer",
  "description": "Reviews code for bugs, security issues, and style",
  "url": "http://127.0.0.1:50051/a2a",
  "version": "0.1.0",
  "capabilities": {
    "streaming": true,
    "pushNotifications": false,
    "stateTransitionHistory": true
  },
  "skills": [
    {
      "id": "file_read",
      "name": "file read",
      "description": "Can use the file_read tool",
      "tags": ["tool"],
      "examples": []
    }
  ],
  "defaultInputModes": ["text"],
  "defaultOutputModes": ["text"]
}
```

Agent Cards are built from OpenFang agent manifests. Each tool in the agent's capability list becomes an A2A skill descriptor.

---

### A2A Server

OpenFang serves A2A requests through the REST API:

1. **Agent Card publication** at `/.well-known/agent.json`
2. **Agent listing** at `/a2a/agents`
3. **Task submission and tracking** via the in-memory `A2aTaskStore` (bounded to 1000 tasks with FIFO eviction)

#### Task Submission Flow

When `POST /a2a/tasks/send` is called:

1. Extract the message text from the A2A request format
2. Find the target agent
3. Create a task with status `Working` and insert into the task store
4. Send the message to the agent via `kernel.send_message()`
5. On success: complete the task with the agent's response
6. On failure: fail the task with the error message
7. Return the final task state

---

### A2A Client

The A2A client discovers and interacts with external A2A agents:

- `discover(url)` -- fetches `{url}/.well-known/agent.json` and parses the Agent Card
- `send_task(url, message, session_id)` -- sends a JSON-RPC task submission
- `get_task(url, task_id)` -- polls for task status

#### Auto-Discovery at Boot

When A2A is enabled with external agents configured, the kernel spawns a background task that discovers each configured external agent by fetching their Agent Cards. Failed discoveries are logged as warnings but do not prevent boot.

---

### Task Lifecycle

| Status | Description |
|--------|-------------|
| `Submitted` | Task received but not yet started |
| `Working` | Task is being actively processed |
| `InputRequired` | Agent needs more information from the caller |
| `Completed` | Task finished successfully |
| `Cancelled` | Task was cancelled by the caller |
| `Failed` | Task encountered an error |

Tasks can include typed content parts (text, file with base64 data, structured data) and produce artifacts alongside messages.

---

### A2A API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `GET` | `/.well-known/agent.json` | Public | Agent Card for the primary agent |
| `GET` | `/a2a/agents` | Public | List all agent cards |
| `POST` | `/a2a/tasks/send` | Public | Submit a task to an agent |
| `GET` | `/a2a/tasks/{id}` | Public | Get task status and messages |
| `POST` | `/a2a/tasks/{id}/cancel` | Public | Cancel a running task |

---

### A2A Configuration

A2A is configured in `config.toml` under the `[a2a]` section:

```toml
[a2a]
enabled = true
listen_path = "/a2a"

[[a2a.external_agents]]
name = "research-agent"
url = "https://research.example.com"

[[a2a.external_agents]]
name = "data-analyst"
url = "https://data.example.com"
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enabled` | bool | `false` | Whether A2A endpoints are active |
| `listen_path` | string | `"/a2a"` | Base path for A2A endpoints |
| `external_agents` | array | `[]` | External agents to discover at boot |

If `a2a` is not present in config, all A2A features are disabled.

---

## Security

### MCP Security

- **Subprocess sandboxing**: Stdio MCP servers run with `env_clear()` -- the subprocess environment is completely cleared. Only explicitly whitelisted env vars plus `PATH` are passed through.
- **Path traversal prevention**: The command path for stdio MCP servers rejects `..` sequences.
- **SSRF protection**: SSE transport URLs are checked against known metadata endpoints (169.254.169.254, metadata.google).
- **Request timeout**: All MCP requests have a configurable timeout (default 30 seconds).
- **Message size limit**: The stdio MCP server enforces a 10 MB maximum message size.

### A2A Security

- **Rate limiting**: A2A endpoints go through the same GCRA rate limiter as all other API endpoints.
- **API authentication**: When `api_key` is set in config, all API endpoints (including A2A) require a `Authorization: Bearer <key>` header. Exception: `/.well-known/agent.json` is typically public.
- **Task store bounds**: The store is bounded (default 1000 tasks) with FIFO eviction to prevent memory exhaustion.
- **External agent discovery**: The client uses a 30-second timeout and sends a `User-Agent: OpenFang/0.1 A2A` header.

### Shared

Both MCP and A2A tool execution flows through the same security pipeline as all other tool calls:
- Capability-based access control
- Tool result truncation (15K character default, 50K hard cap)
- Universal 60-second tool execution timeout
- Loop guard detection
- Taint tracking on data flowing between tools
