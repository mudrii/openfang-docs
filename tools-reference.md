# Tools Reference

Validated against OpenFang `v0.5.7` source (`crates/openfang-runtime/src/tool_runner.rs`).

OpenFang ships **59 built-in tools** across 17 categories. Every tool is available to agents through the `execute_tool` dispatcher. Access is controlled by capability-based security: an agent can only call tools listed in its `allowed_tools` manifest field. Tools not in the list are rejected with a "Permission denied" error before execution begins.

---

## Filesystem (4 tools)

### `file_read`

Read the contents of a file. Paths are relative to the agent workspace.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | yes | The file path to read |

### `file_write`

Write content to a file. Creates parent directories if needed.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | yes | The file path to write to |
| `content` | string | yes | The content to write |

### `file_list`

List files in a directory. Returns sorted entries with `/` suffix for directories.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | yes | The directory path to list |

### `apply_patch`

Apply a multi-hunk diff patch to add, update, move, or delete files. Requires a workspace root.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `patch` | string | yes | Patch in Begin Patch / End Patch format. Use Add File, Update File, Delete File markers. Hunks use @@ headers with space (context), - (remove), + (add) prefixed lines. |

**Security:** All filesystem tools reject path traversal (`..` components are forbidden). When a `workspace_root` is configured, paths are resolved through the workspace sandbox.

---

## Web (2 tools)

### `web_fetch`

Fetch a URL with SSRF protection. For GET requests, HTML is converted to Markdown. For other methods, returns raw response body.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | yes | The URL to fetch (http/https only) |
| `method` | string | no | HTTP method: `GET`, `POST`, `PUT`, `PATCH`, `DELETE` (default: `GET`) |
| `headers` | object | no | Custom HTTP headers as key-value pairs |
| `body` | string | no | Request body for POST/PUT/PATCH |

**Security:** Taint tracking blocks URLs containing secrets/PII patterns (`api_key=`, `token=`, `secret=`, `password=`, `Authorization:`) to prevent data exfiltration.

### `web_search`

Search the web using multiple providers (Tavily, Brave, Perplexity, DuckDuckGo) with automatic fallback. Returns structured results with titles, URLs, and snippets.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | yes | The search query |
| `max_results` | integer | no | Maximum results to return (default: 5, max: 20) |

---

## Shell (1 tool)

### `shell_exec`

Run a shell command and return its output.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `command` | string | yes | The command to run |
| `timeout_seconds` | integer | no | Timeout in seconds (default: 30, overridden by exec_policy timeout) |

**Security:** Three layers of protection:
1. Shell metacharacter injection is always blocked (backticks, `$(`, `${`, etc.)
2. `exec_policy` enforcement: `Allowlist` mode uses direct argv splitting without a shell interpreter; `Full` mode uses `sh -c`
3. Taint heuristics block suspicious patterns (`curl`, `wget`, `| sh`, `eval`) unless exec_policy is `Full`

Environment variables are isolated via `subprocess_sandbox`. The command runs in the agent's workspace directory.

---

## Agent (4 tools)

### `agent_send`

Send a message to another agent and receive their response. Supports UUID or agent name lookup.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `agent_id` | string | yes | The target agent's UUID or name |
| `message` | string | yes | The message to send |

Inter-agent call depth is capped at 5 to prevent infinite recursion (A->B->C->...). Requires a running kernel.

### `agent_spawn`

Spawn a new agent from a TOML manifest. Returns the new agent's ID and name.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `manifest_toml` | string | yes | Agent manifest in TOML format (must include name, module, [model], and [capabilities]) |

### `agent_list`

List all currently running agents with their IDs, names, states, and models.

No parameters.

### `agent_kill`

Kill (terminate) another agent by its ID.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `agent_id` | string | yes | The agent's UUID to kill |

---

## Memory (2 tools)

### `memory_store`

Store a value in shared memory accessible by all agents. Use for cross-agent coordination and data sharing.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `key` | string | yes | The storage key |
| `value` | string | yes | The value to store (JSON-encode objects/arrays, or pass a plain string) |

### `memory_recall`

Recall a value from shared memory by key.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `key` | string | yes | The storage key to recall |

---

## Tasks (4 tools)

### `task_post`

Post a task to the shared task queue for another agent to pick up.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | yes | Short task title |
| `description` | string | yes | Detailed task description |
| `assigned_to` | string | no | Agent name or ID to assign the task to |

### `task_claim`

Claim the next available task from the task queue assigned to you or unassigned.

No parameters.

### `task_complete`

Mark a previously claimed task as completed with a result.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `task_id` | string | yes | The task ID to complete |
| `result` | string | yes | The result or outcome of the task |

### `task_list`

List tasks in the shared queue, optionally filtered by status.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | no | Filter by status: `pending`, `in_progress`, `completed` |

---

## Events (1 tool)

### `event_publish`

Publish a custom event that can trigger proactive agents. Use to broadcast signals to the agent fleet.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `event_type` | string | yes | Type identifier for the event (e.g., `code_review_requested`) |
| `payload` | object | no | JSON payload data for the event |

---

## Scheduling (3 tools)

### `schedule_create`

Schedule a recurring task using natural language or cron syntax.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `description` | string | yes | What this schedule does (e.g., "Check for new emails") |
| `schedule` | string | yes | Natural language or cron expression (e.g., `every 5 minutes`, `daily at 9am`, `weekdays at 6pm`, `0 */5 * * *`) |
| `agent` | string | no | Agent name or ID to run this task (defaults to self) |

Supported natural language patterns: `every N minutes`, `every N hours`, `every day/week`, `daily at Xam/pm`, `weekdays at Xam/pm`, `weekends at Xam/pm`, `hourly`, `daily`, `weekly`, `monthly`. Cron expressions with 5 space-separated fields pass through directly.

### `schedule_list`

List all scheduled tasks with their IDs, descriptions, schedules, and next run times.

No parameters.

### `schedule_delete`

Remove a scheduled task by its ID.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | yes | The schedule ID to remove |

---

## Cron Scheduling (3 tools)

These are kernel-level cron jobs with richer scheduling semantics than `schedule_*` tools.

### `cron_create`

Create a scheduled/cron job. Supports one-shot (`at`), recurring (`every N seconds`), and cron expressions. Max 50 jobs per agent.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | yes | Job name (max 128 chars, alphanumeric + spaces/hyphens/underscores) |
| `schedule` | object | yes | Schedule object: `{"kind":"at","at":"2025-01-01T00:00:00Z"}` or `{"kind":"every","every_secs":300}` or `{"kind":"cron","expr":"0 */6 * * *"}` |
| `action` | object | yes | Action object: `{"kind":"system_event","text":"..."}` or `{"kind":"agent_turn","message":"...","timeout_secs":300}` |
| `delivery` | object | no | Delivery target: `{"kind":"none"}` or `{"kind":"channel","channel":"telegram"}` or `{"kind":"last_channel"}` |
| `one_shot` | boolean | no | If true, auto-delete after running (default: false) |

### `cron_list`

List all scheduled/cron jobs for the current agent.

No parameters.

### `cron_cancel`

Cancel a scheduled/cron job by its ID.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `job_id` | string | yes | The UUID of the cron job to cancel |

---

## Knowledge Graph (3 tools)

### `knowledge_add_entity`

Add an entity to the knowledge graph.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | yes | Display name of the entity |
| `entity_type` | string | yes | Type: `person`, `organization`, `project`, `concept`, `event`, `location`, `document`, `tool`, or a custom type |
| `properties` | object | no | Arbitrary key-value properties |

### `knowledge_add_relation`

Add a relation between two entities in the knowledge graph.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `source` | string | yes | Source entity ID or name |
| `relation` | string | yes | Relation type: `works_at`, `knows_about`, `related_to`, `depends_on`, `owned_by`, `created_by`, `located_in`, `part_of`, `uses`, `produces`, or a custom type |
| `target` | string | yes | Target entity ID or name |
| `confidence` | number | no | Confidence score 0.0-1.0 (default: 1.0) |
| `properties` | object | no | Arbitrary key-value properties |

### `knowledge_query`

Query the knowledge graph. Returns matching entity-relation-entity triples.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `source` | string | no | Filter by source entity name or ID |
| `relation` | string | no | Filter by relation type |
| `target` | string | no | Filter by target entity name or ID |
| `max_depth` | integer | no | Maximum traversal depth (default: 1) |

---

## Browser (10 tools)

All browser tools require Chrome/Chromium to be installed. Each agent gets an isolated browser session. URL taint checking applies to `browser_navigate` (same rules as `web_fetch`).

### `browser_navigate`

Navigate a browser to a URL. Returns the page title and readable content as markdown.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | yes | The URL to navigate to (http/https only) |

### `browser_click`

Click an element on the current browser page by CSS selector or visible text.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `selector` | string | yes | CSS selector (e.g., `#submit-btn`, `.add-to-cart`) or visible text to click |

### `browser_type`

Type text into an input field on the current browser page.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `selector` | string | yes | CSS selector for the input field (e.g., `input[name="email"]`, `#search-box`) |
| `text` | string | yes | The text to type into the field |

### `browser_screenshot`

Take a screenshot of the current browser page. Returns a base64-encoded PNG image.

No parameters.

### `browser_read_page`

Read the current browser page content as structured markdown.

No parameters.

### `browser_close`

Close the browser session. The browser also auto-closes when the agent loop ends.

No parameters.

### `browser_scroll`

Scroll the browser page.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `direction` | string | no | Scroll direction: `up`, `down`, `left`, `right` (default: `down`) |
| `amount` | integer | no | Pixels to scroll (default: 600) |

### `browser_wait`

Wait for a CSS selector to appear on the page. Useful for dynamic content.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `selector` | string | yes | CSS selector to wait for |
| `timeout_ms` | integer | no | Max wait time in milliseconds (default: 5000, max: 30000) |

### `browser_run_js`

Run JavaScript on the current browser page and return the result.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `expression` | string | yes | JavaScript expression to run in the page context |

### `browser_back`

Go back to the previous page in browser history.

No parameters.

---

## Media (6 tools)

### `image_analyze`

Analyze an image file. Returns format, dimensions, file size, and a base64 preview. Optionally accepts a prompt for vision-model analysis.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | yes | Path to the image file |
| `prompt` | string | no | Optional prompt for vision analysis (e.g., "Describe what you see") |

Detected formats: PNG, JPEG, GIF, WebP, BMP, ICO. Dimensions are extracted for PNG, JPEG, GIF, and BMP.

### `media_describe`

Describe an image using a vision-capable LLM. Auto-selects the best available provider (Anthropic, OpenAI, or Gemini).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | yes | Path to the image file (relative to workspace) |
| `prompt` | string | no | Optional prompt to guide the description |

Supported image formats: PNG, JPEG, GIF, WebP, BMP, SVG.

### `media_transcribe`

Transcribe audio to text using speech-to-text. Auto-selects the best available provider (Groq Whisper or OpenAI Whisper).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | yes | Path to the audio file (relative to workspace) |
| `language` | string | no | Optional ISO-639-1 language code (e.g., `en`, `es`, `ja`) |

Supported audio formats: MP3, WAV, OGG, FLAC, M4A, WebM.

### `image_generate`

Generate images from a text prompt. Requires `OPENAI_API_KEY`. Generated images are saved to the workspace `output/` directory.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `prompt` | string | yes | Text description of the image to generate (max 4000 chars) |
| `model` | string | no | Model: `dall-e-3` (default), `dall-e-2`, or `gpt-image-1` |
| `size` | string | no | Image size: `1024x1024` (default), `1024x1792`, `1792x1024`, `256x256`, `512x512` |
| `quality` | string | no | Quality: `hd` (default for dall-e-3) or `standard` |
| `count` | integer | no | Number of images (1-4, default: 1). DALL-E 3 only supports 1. |

### `text_to_speech`

Convert text to speech audio. Auto-selects OpenAI or ElevenLabs. Saves audio to workspace `output/` directory.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `text` | string | yes | The text to convert to speech (max 4096 chars) |
| `voice` | string | no | Voice name: `alloy`, `echo`, `fable`, `onyx`, `nova`, `shimmer` (default: `alloy`) |
| `format` | string | no | Output format: `mp3`, `opus`, `aac`, `flac` (default: `mp3`) |

### `speech_to_text`

Transcribe audio to text. Auto-selects Groq Whisper or OpenAI Whisper.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | yes | Path to the audio file (relative to workspace) |
| `language` | string | no | Optional ISO-639-1 language code |

Supported formats: MP3, WAV, OGG, FLAC, M4A, WebM.

---

## Location (1 tool)

### `location_get`

Get approximate geographic location based on IP address. Returns city, country, coordinates, and timezone.

No parameters. Uses ip-api.com (no API key required).

---

## Channels (1 tool)

### `channel_send`

Send a message or media to a user on a configured channel (email, Telegram, Slack, Discord, etc.).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `channel` | string | yes | Channel adapter name (e.g., `email`, `telegram`, `slack`, `discord`) |
| `recipient` | string | yes | Platform-specific recipient identifier (email address, user ID, etc.). Falls back to `default_chat_id` from channel config if empty. |
| `subject` | string | no | Subject line (used for email; ignored for other channels) |
| `message` | string | no | The message body (required for text messages, optional caption for media) |
| `image_url` | string | no | URL of an image to send (Telegram, Discord, Slack) |
| `file_url` | string | no | URL of a file to send as attachment |
| `file_path` | string | no | Local file path to send as attachment (reads from disk) |
| `filename` | string | no | Filename for file attachments (defaults to basename of file_path) |
| `thread_id` | string | no | Thread/topic ID to reply in (e.g., Telegram message_thread_id, Slack thread_ts) |

For email: validates email format and prepends `Subject:` header. Supports sending local files with automatic MIME type detection for 20+ formats.

---

## Hands (4 tools)

Hands are curated autonomous capability packages that spawn specialized agents.

### `hand_list`

List available Hands and their activation status.

No parameters.

### `hand_activate`

Activate a Hand -- spawns a specialized autonomous agent with curated tools and skills.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `hand_id` | string | yes | The ID of the hand to activate (e.g., `researcher`, `clip`, `browser`) |
| `config` | object | no | Optional configuration overrides for the hand's settings |

### `hand_status`

Check the status and metrics of an active Hand.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `hand_id` | string | yes | The ID of the hand to check status for |

### `hand_deactivate`

Deactivate a running Hand and stop its agent.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `instance_id` | string | yes | The UUID of the hand instance to deactivate |

---

## A2A (2 tools)

Cross-instance agent communication using the A2A protocol.

### `a2a_discover`

Discover an external A2A agent by fetching its agent card from a URL. Returns the agent's name, description, skills, and supported protocols.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | yes | Base URL of the remote OpenFang/A2A-compatible agent (e.g., `https://agent.example.com`) |

SSRF protection blocks private/metadata IP addresses.

### `a2a_send`

Send a task/message to an external A2A agent and get the response.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `message` | string | yes | The task/message to send to the remote agent |
| `agent_url` | string | no | Direct URL of the remote agent's A2A endpoint |
| `agent_name` | string | no | Name of a previously discovered A2A agent (looked up from kernel) |
| `session_id` | string | no | Optional session ID for multi-turn conversations |

Either `agent_url` or `agent_name` must be provided.

---

## Process (5 tools)

Persistent process management for long-running processes (REPLs, servers, watchers). Max 5 processes per agent.

### `process_start`

Start a long-running process. Returns a `process_id` for subsequent operations.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `command` | string | yes | The binary to run (e.g., `python`, `node`, `npm`) |
| `args` | array of strings | no | Command-line arguments (e.g., `["-i"]` for interactive Python) |

**Security:** Subject to the same exec_policy enforcement as `shell_exec`. Shell metacharacters are rejected in both the command and arguments. In Allowlist mode, the command must be in `exec_policy.allowed_commands`.

### `process_poll`

Read accumulated stdout/stderr from a running process. Non-blocking: returns whatever output has buffered since the last poll.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `process_id` | string | yes | The process ID returned by process_start |

### `process_write`

Write data to a running process's stdin. A newline is appended automatically if not present.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `process_id` | string | yes | The process ID returned by process_start |
| `data` | string | yes | The data to write to stdin |

### `process_kill`

Terminate a running process and clean up its resources.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `process_id` | string | yes | The process ID returned by process_start |

### `process_list`

List all running processes for the current agent, including their IDs, commands, uptime, and alive status.

No parameters.

---

## Docker (1 tool)

### `docker_exec`

Run a command inside a Docker container sandbox. Provides OS-level isolation with resource limits, network isolation, and capability dropping. Requires Docker to be installed and `docker.enabled = true` in config.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `command` | string | yes | The command to run inside the container |

The container is created per invocation and destroyed after completion. Timeout is configured via `docker.timeout_secs` in config.

---

## System (1 tool)

### `system_time`

Get the current date, time, and timezone. Returns ISO 8601 timestamp, Unix epoch seconds, and timezone info.

No parameters. Returns: `utc`, `local`, `unix_epoch`, `timezone`, `utc_offset`, `date`, `time`, `day_of_week`.

---

## UI (1 tool)

### `canvas_present`

Present an interactive HTML canvas to the user. The HTML is sanitized and saved to the workspace. The dashboard renders it in a panel.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `html` | string | yes | The HTML content to present. Must not contain script tags, event handlers, or javascript: URLs. |
| `title` | string | no | Optional title for the canvas panel |

**Security:** HTML is sanitized before rendering:
- Rejects script, iframe, object, embed, applet tags
- Strips all on* event handler attributes
- Blocks javascript:, vbscript:, data:text/html URL schemes
- Enforces configurable size limit (default 512KB, set via `canvas.max_bytes` in config)

---

## Tool Profiles and Capability Gates

### Capability-based security

Each agent declares its allowed tools in the manifest `[capabilities]` section. The runtime enforces this at the `execute_tool` dispatch layer: if `allowed_tools` is set on the agent, only tools in that list may run. Any attempt to call an unlisted tool returns a "Permission denied" error.

```toml
[capabilities]
tools = ["file_read", "file_write", "file_list", "shell_exec", "web_search"]
```

### Approval gate

Tools can be gated behind human approval. When `requires_approval` returns true for a tool, the runtime calls `request_approval` on the kernel handle before running it. The user sees a summary and can approve or deny. Shell tools (`shell_exec` and `process_start`) bypass the approval gate when `exec_policy.mode = "full"` or when `allowed_commands = ["*"]`.

### Exec policy

The `exec_policy` config controls shell and process operations:

| Mode | Behavior |
|------|----------|
| `deny` | All shell/process operations blocked |
| `allowlist` (default) | Only commands in `allowed_commands` may run; uses direct argv splitting (no shell interpreter) |
| `full` | Unrestricted shell access via `sh -c`; user explicitly opted in |

### Taint tracking

The runtime implements taint tracking for two sinks:

- **shell_exec sink** -- blocks shell metacharacter injection and heuristic patterns for injected external data (curl piping, base64 decoding, eval)
- **net_fetch sink** -- blocks URLs containing secrets/PII in query parameters to prevent data exfiltration

Taint checks apply to `shell_exec`, `web_fetch`, and `browser_navigate`.

---

## OpenClaw Tool Name Mappings

OpenFang normalizes tool names from OpenClaw and LLM-hallucinated aliases through `normalize_tool_name()`. The following mappings are applied transparently before dispatch:

| Alias | OpenFang Canonical |
|-------|-------------------|
| `Read`, `read`, `read_file`, `fs-read`, `fsRead`, `readFile` | `file_read` |
| `Write`, `write`, `write_file`, `Edit`, `edit`, `fs-write`, `fsWrite`, `writeFile` | `file_write` |
| `Glob`, `glob`, `list_files`, `Grep`, `grep`, `fs-list`, `fsList`, `listFiles`, `list_dir`, `ls` | `file_list` |
| `Bash`, `bash`, `exec`, `execute_command`, `fs-exec`, `run`, `run_command`, `runCommand`, `execute`, `shell` | `shell_exec` |
| `WebSearch` | `web_search` |
| `WebFetch`, `fetch_url` | `web_fetch` |
| `memory_search` | `memory_recall` |
| `memory_save` | `memory_store` |
| `sessions_send`, `agent_message` | `agent_send` |
| `sessions_list`, `agents_list`, `agent_list` | `agent_list` |
| `sessions_spawn` | `agent_send` |

Capability enforcement operates on the normalized name: if an agent has `file_write` in its allowed tools and the LLM calls `fs-write`, the call normalizes to `file_write` and passes the capability check.

---

## Tool Extension Points

### MCP tools

Tools with the `mcp_{server}_{tool}` naming pattern are dispatched to connected MCP (Model Context Protocol) servers. These are registered at runtime through the MCP connection configuration.

### Skill tools

If a tool name is not a built-in and no MCP match is found, the skill registry is checked. Skills can provide custom tools through their manifests.
