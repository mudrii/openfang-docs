# Hands Development

Validated against OpenFang release `v0.5.7`.

This guide covers creating custom Hands for OpenFang. A Hand is a pre-built, domain-complete agent configuration that users activate from the marketplace. Unlike regular agents (you chat with them), Hands work for you (you check in on them).

---

## Directory structure

A Hand is a directory containing at minimum a `HAND.toml` manifest. An optional `SKILL.md` provides domain expertise injected into the agent's context at activation time.

```
my-hand/
  HAND.toml     # Required: hand manifest
  SKILL.md      # Optional: domain expertise document
```

The `HAND.toml` is the single source of truth for the hand's identity, tools, requirements, settings, agent configuration, and dashboard metrics. The `SKILL.md` is a Markdown file (with optional YAML frontmatter) containing reference knowledge the agent can draw on during execution.

---

## HAND.toml manifest format

The manifest is parsed by `parse_hand_toml()` in `crates/openfang-hands/src/lib.rs`, which produces a `HandDefinition` struct. Both flat format (fields at the top level) and wrapped format (fields under a `[hand]` table) are supported.

### Top-level fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | yes | Unique hand identifier. Use lowercase kebab-case (e.g. `my-hand`). |
| `name` | string | yes | Human-readable display name (e.g. `My Hand`). |
| `description` | string | yes | One-line description of what this hand does. |
| `category` | string | yes | One of: `content`, `security`, `productivity`, `development`, `communication`, `data`, `finance`. Falls back to `other` for unrecognized values. |
| `icon` | string | no | Emoji icon for marketplace display. |
| `tools` | array of strings | no | Tool IDs the agent needs access to. Empty means all tools. |
| `skills` | array of strings | no | Skill allowlist for the spawned agent. Empty means all skills. |
| `mcp_servers` | array of strings | no | MCP server allowlist for the spawned agent. Empty means all. |

### `[[requires]]` -- requirement declarations

Each entry declares a dependency that must be satisfied before the hand can activate. Requirements are checked by the registry at activation time and via `openfang hand check-deps <id>`.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `key` | string | yes | Unique key for this requirement (e.g. `ffmpeg`, `TWITTER_BEARER_TOKEN`). |
| `label` | string | yes | Human-readable description (e.g. `FFmpeg must be installed`). |
| `requirement_type` | string | yes | One of: `binary` (check PATH), `env_var` (check environment variable), `api_key` (check env var, same as env_var). |
| `check_value` | string | yes | The value to check: binary name for `binary`, env var name for `env_var`/`api_key`. |
| `description` | string | no | Longer explanation of why this is needed. |
| `optional` | boolean | no | Default `false`. Optional requirements do not block activation; unmet optional requirements cause `degraded` status instead of `requirements not met`. |

#### `[requires.install]` -- platform-specific install instructions

| Field | Type | Description |
|-------|------|-------------|
| `macos` | string | macOS install command (e.g. `brew install ffmpeg`) |
| `windows` | string | Windows install command (e.g. `winget install Gyan.FFmpeg`) |
| `linux_apt` | string | Debian/Ubuntu command |
| `linux_dnf` | string | Fedora/RHEL command |
| `linux_pacman` | string | Arch command |
| `pip` | string | pip install command |
| `signup_url` | string | URL to sign up for a service (for API key requirements) |
| `docs_url` | string | URL to documentation |
| `env_example` | string | Example env var line (e.g. `TWITTER_BEARER_TOKEN=AAAA...`) |
| `manual_url` | string | URL to manual install page |
| `estimated_time` | string | Human-readable time estimate (e.g. `2-5 min`) |
| `steps` | array of strings | Ordered setup steps (e.g. for API key provisioning) |

### `[[settings]]` -- configurable settings

Settings are displayed in the activation modal in the dashboard. Users can override them at activation time, and the chosen values are resolved into a prompt block injected into the agent's system prompt.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `key` | string | yes | Setting identifier (used in config maps). |
| `label` | string | yes | Display label. |
| `description` | string | no | Help text shown in the UI. |
| `setting_type` | string | yes | One of: `select`, `text`, `toggle`. |
| `default` | string | no | Default value. For toggles, use `"true"` or `"false"`. |
| `env_var` | string | no | For `text` type: env var name to expose when the field has a value. |

#### `[[settings.options]]` -- options for select-type settings

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `value` | string | yes | Internal value stored in config. |
| `label` | string | yes | Display label. |
| `provider_env` | string | no | Env var to check for "Ready" badge (e.g. `GROQ_API_KEY`). |
| `binary` | string | no | Binary to check on PATH for "Ready" badge (e.g. `whisper`). |

The `provider_env` and `binary` fields drive the availability indicator in the dashboard. When checking availability, the registry verifies the env var is set and non-empty, and/or the binary exists on PATH.

### `[agent]` -- agent configuration

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | string | yes | | Agent name (e.g. `clip-hand`). |
| `description` | string | yes | | Agent description. |
| `module` | string | no | `builtin:chat` | Agent module. |
| `provider` | string | no | `anthropic` | LLM provider. Use `default` to inherit from global config. |
| `model` | string | no | `claude-sonnet-4-20250514` | Model name. Use `default` to inherit from global config. |
| `api_key_env` | string | no | | Env var name for the API key. |
| `base_url` | string | no | | Custom API base URL. |
| `max_tokens` | integer | no | `4096` | Max tokens per LLM response. |
| `temperature` | float | no | `0.7` | Sampling temperature. Lower = more deterministic. |
| `system_prompt` | string | yes | | The agent's system prompt. Use triple-quoted strings in TOML for multi-line. |
| `max_iterations` | integer | no | | Maximum tool-use iterations per conversation turn. |
| `heartbeat_interval_secs` | integer | no | | Override the kernel's default heartbeat interval (30s). Useful for hands that make long LLM calls (e.g. researcher uses 120s). |

### `[dashboard]` / `[[dashboard.metrics]]` -- dashboard metrics

Each metric maps a memory key to a display widget in the hand's dashboard card.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `label` | string | yes | Display label (e.g. `Jobs Completed`). |
| `memory_key` | string | yes | Key to read from the agent's structured memory via `memory_store`. |
| `format` | string | no | Display format: `number` (default), `duration`, `bytes`, `percentage`, `text`. |

The agent is responsible for updating these memory keys during execution using the `memory_store` tool.

---

## SKILL.md format

The `SKILL.md` file provides domain expertise injected into the agent's context alongside the system prompt. It uses standard Markdown with optional YAML frontmatter.

```markdown
---
name: my-hand-skill
version: "1.0.0"
description: "Expert knowledge for my custom hand"
author: Your Name
tags: [tag1, tag2]
tools: [tool1, tool2]
runtime: prompt_only
---

# Domain Expertise Title

## Section 1
Reference knowledge, API documentation, command examples, etc.

## Section 2
More domain-specific content the agent can draw on.
```

The frontmatter fields are metadata only -- the runtime currently uses `prompt_only` mode, meaning the entire SKILL.md content is injected as context. Keep the file focused and concise; overly long skill documents dilute the agent's attention.

---

## Settings resolution

When a hand is activated, user-provided config values are resolved against the settings schema by `resolve_settings()`. This produces:

1. A **prompt block** appended to the agent's system prompt under `## User Configuration`, summarizing each setting's chosen value.
2. A list of **env var names** the agent's subprocess should have access to, collected from `provider_env` fields of selected options and `env_var` fields of text settings.

---

## Complete example: weather-monitor hand

```toml
id = "weather-monitor"
name = "Weather Monitor Hand"
description = "Monitors weather conditions for configured locations and sends alerts"
category = "data"
icon = "W"
tools = ["web_fetch", "web_search", "memory_store", "memory_recall", "schedule_create", "schedule_list", "schedule_delete", "event_publish", "file_write", "file_read"]

[[requires]]
key = "OPENWEATHER_API_KEY"
label = "OpenWeatherMap API key"
requirement_type = "api_key"
check_value = "OPENWEATHER_API_KEY"
description = "Free API key from openweathermap.org for weather data."

[requires.install]
signup_url = "https://home.openweathermap.org/users/sign_up"
docs_url = "https://openweathermap.org/appid"
env_example = "OPENWEATHER_API_KEY=your_key_here"
estimated_time = "2-3 min"
steps = [
    "Sign up at openweathermap.org",
    "Go to API Keys in your account",
    "Copy your default key or generate a new one",
    "Set it as an environment variable",
]

[[settings]]
key = "locations"
label = "Locations"
description = "Comma-separated city names to monitor (e.g. London, Tokyo, New York)"
setting_type = "text"
default = ""

[[settings]]
key = "check_interval"
label = "Check Interval"
description = "How often to check weather conditions"
setting_type = "select"
default = "1h"

[[settings.options]]
value = "30m"
label = "Every 30 minutes"

[[settings.options]]
value = "1h"
label = "Every hour"

[[settings.options]]
value = "4h"
label = "Every 4 hours"

[[settings]]
key = "alert_on_severe"
label = "Alert on Severe Weather"
description = "Publish an event when severe weather is detected"
setting_type = "toggle"
default = "true"

[agent]
name = "weather-monitor-hand"
description = "AI weather monitor — checks conditions for configured locations and sends alerts"
module = "builtin:chat"
provider = "default"
model = "default"
max_tokens = 4096
temperature = 0.2
max_iterations = 20
system_prompt = """You are Weather Monitor Hand — an autonomous weather monitoring agent.

## Your Task
Monitor weather conditions for the locations specified in User Configuration.

## Phase 0 — Setup
1. Read User Configuration for locations and check_interval.
2. Create a schedule using schedule_create based on check_interval.
3. Recall previous state from memory.

## Phase 1 — Fetch Weather
For each location, call the OpenWeatherMap API:
```
curl -s "https://api.openweathermap.org/data/2.5/weather?q=CITY&appid=$OPENWEATHER_API_KEY&units=metric"
```

## Phase 2 — Analyze & Alert
Compare with previous readings. If alert_on_severe is enabled and severe
conditions detected, publish an event via event_publish.

## Phase 3 — Report & Persist
Save current readings to file. Update dashboard metrics via memory_store:
- `weather_hand_checks_completed` — increment
- `weather_hand_alerts_sent` — increment on alert
- `weather_hand_locations_tracked` — number of locations
"""

[dashboard]
[[dashboard.metrics]]
label = "Checks Completed"
memory_key = "weather_hand_checks_completed"
format = "number"

[[dashboard.metrics]]
label = "Alerts Sent"
memory_key = "weather_hand_alerts_sent"
format = "number"

[[dashboard.metrics]]
label = "Locations Tracked"
memory_key = "weather_hand_locations_tracked"
format = "number"
```

The corresponding `SKILL.md` would contain OpenWeatherMap API reference, common weather condition codes, and alert threshold guidance.

---

## Installing a custom hand

Place your hand directory anywhere and install it:

```bash
openfang hand install /path/to/weather-monitor/
```

The daemon:

1. Reads `HAND.toml` and optional `SKILL.md` from the directory.
2. Parses and validates the manifest via `parse_hand_toml()`.
3. Registers the definition in the in-memory `HandRegistry`.
4. Copies the directory to `~/.openfang/hands/<hand_id>/` for persistence across daemon restarts.

On daemon startup, `load_workspace_hands()` scans `~/.openfang/hands/` and re-registers all valid hand definitions found there. Invalid TOML files are logged and skipped so one bad manifest cannot take down the registry.

You can also install via the API using raw TOML + skill content:

```bash
curl -X POST http://127.0.0.1:4200/api/hands/install \
  -H "Content-Type: application/json" \
  -d '{"toml_content": "...", "skill_content": "..."}'
```

The `upsert_from_content` method is available for updating an existing definition. Active instances are not automatically restarted -- deactivate and reactivate to pick up the new definition.

---

## Publishing to FangHub

FangHub is the marketplace for sharing Hands. To publish:

1. Ensure your `HAND.toml` has a unique `id` that does not conflict with bundled hands.
2. Validate locally: `openfang hand install /path/to/hand/ && openfang hand check-deps <id>`.
3. Test the hand by activating it and verifying it operates correctly.
4. Package the hand directory (HAND.toml + SKILL.md + any supporting files).
5. Submit to FangHub following the contribution guidelines.

---

## HandDefinition struct reference

From `crates/openfang-hands/src/lib.rs`:

```rust
pub struct HandDefinition {
    pub id: String,
    pub name: String,
    pub description: String,
    pub category: HandCategory,
    pub icon: String,
    pub tools: Vec<String>,
    pub skills: Vec<String>,
    pub mcp_servers: Vec<String>,
    pub requires: Vec<HandRequirement>,
    pub settings: Vec<HandSetting>,
    pub agent: HandAgentConfig,
    pub dashboard: HandDashboard,
    pub skill_content: Option<String>,  // populated at load time, not in TOML
}
```

### HandCategory enum

```rust
pub enum HandCategory {
    Content,
    Security,
    Productivity,
    Development,
    Communication,
    Data,
    Finance,
    Other,  // fallback via #[serde(other)]
}
```

### RequirementType enum

```rust
pub enum RequirementType {
    Binary,   // binary must exist on PATH
    EnvVar,   // environment variable must be set
    ApiKey,   // API key env var must be set (same check as EnvVar)
}
```

### HandSettingType enum

```rust
pub enum HandSettingType {
    Select,
    Text,
    Toggle,
}
```

### HandStatus enum (runtime)

```rust
pub enum HandStatus {
    Active,
    Paused,
    Error(String),
    Inactive,
}
```

---

## Registry lifecycle

The `HandRegistry` (in `crates/openfang-hands/src/registry.rs`) manages definitions and active instances:

1. **Load bundled** -- `load_bundled()` reads all 9 compile-time embedded hands from `bundled.rs`.
2. **Load workspace** -- `load_workspace_hands("~/.openfang/hands/")` loads custom hands from disk.
3. **Install** -- `install_from_path()` or `install_from_content()` adds a new definition at runtime.
4. **Activate** -- `activate(hand_id, config, instance_name)` creates a `HandInstance`. The kernel then spawns the actual agent and calls `set_agent()` to link the instance to its agent ID.
5. **Pause/Resume** -- `pause(instance_id)` / `resume(instance_id)` toggle the instance status.
6. **Deactivate** -- `deactivate(instance_id)` removes the instance from the registry. The kernel kills the associated agent.
7. **Persist** -- `persist_state(path)` writes active hand state to disk; `load_state(path)` restores it on restart.

Requirement checks (`check_requirements`) verify binary existence on PATH (with special handling for `python3` and `chromium`) and env var presence. Settings availability checks (`check_settings_availability`) verify per-option readiness badges.

---

## Tips for hand development

- Use `provider = "default"` and `model = "default"` in the agent config to inherit from the user's global OpenFang configuration rather than hardcoding a specific provider.
- Keep the system prompt focused. The SKILL.md handles reference knowledge; the system prompt should describe the agent's behavior and pipeline.
- Use structured memory keys with a consistent prefix (e.g. `myhand_metric_name`) to avoid collisions with other hands.
- Declare requirements conservatively. Mark nice-to-have dependencies as `optional = true` so users can activate the hand in degraded mode.
- For hands that make long LLM calls, set `heartbeat_interval_secs` in the agent config to avoid false-positive recovery triggers from the kernel's health monitor.
- Test your hand with `openfang hand install`, `openfang hand check-deps`, and `openfang hand activate` before publishing.
