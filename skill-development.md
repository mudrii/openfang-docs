# Skill Development

Skills are pluggable tool bundles that extend agent capabilities in OpenFang. A skill packages one or more tools with their implementation, letting agents do things that built-in tools do not cover.

---

## Overview

A skill consists of:

1. A **manifest** (`skill.toml`) that declares metadata, runtime type, provided tools, and requirements.
2. An **entry point** (Python script, Node.js module, WASM module, shell script, or prompt-only Markdown) that implements the tool logic.
3. An optional **SKILL.md** file with expert knowledge injected into the LLM system prompt.

Skills are installed to `~/.openfang/skills/` and registered via the skill registry. For v0.5.7, OpenFang ships 61 bundled prompt-only skills compiled into the binary.

---

## Supported Runtimes

| Runtime | Language | Sandboxed | Notes |
|---------|----------|-----------|-------|
| `promptonly` | Markdown | N/A | Expert knowledge injected into system prompt. No code execution. Default runtime. |
| `python` | Python 3.8+ | Subprocess with `env_clear()` | JSON stdin/stdout protocol. Easiest to write. |
| `node` | JavaScript | Subprocess with `env_clear()` | OpenClaw compatibility. Same protocol as Python. |
| `wasm` | Rust, C, Go | Wasmtime dual metering | Fully sandboxed. Fuel limit + memory limit + capability restriction. |
| `shell` | Bash/sh | Subprocess with `env_clear()` | Shell script execution. Added in v0.5.0. |
| `builtin` | Rust | N/A | Compiled into the binary. For core tools only. |

---

## SKILL.md Format

SKILL.md files use YAML frontmatter and a Markdown body. The body becomes `prompt_context` -- expert knowledge injected into the LLM's system prompt when the skill is active.

```markdown
---
name: my-domain-expert
description: Expert knowledge for my domain
metadata:
  openclaw:
    emoji: "🔧"
    requires:
      bins:
        - git
      env:
        - API_TOKEN
    commands:
      - name: analyze_code
        description: Analyze code for patterns
---

# My Domain Expert

## Key Principles
- Always validate inputs before processing
- Use established patterns when available
- Document edge cases explicitly

## Common Patterns
...

## Pitfalls to Avoid
...
```

### Frontmatter Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Skill display name. Falls back to directory name if omitted. |
| `description` | string | Short description shown in skill listings. |
| `metadata.openclaw.emoji` | string | Emoji icon for the skill. |
| `metadata.openclaw.requires.bins` | array | Required system binaries (e.g., `["git", "gh"]`). |
| `metadata.openclaw.requires.env` | array | Required environment variables. |
| `metadata.openclaw.commands` | array | Tool definitions with `name`, `description`, and optional `dispatch` config. |

SKILL.md files placed in `~/.openfang/skills/<name>/` are automatically detected and converted to OpenFang format (a `skill.toml` is generated alongside). All SKILL.md content passes through the prompt injection scanner before activation.

---

## Skill Manifest (skill.toml)

### Directory Structure

```
my-skill/
  skill.toml          # Manifest (required)
  SKILL.md            # Prompt context (optional, for prompt-only skills)
  src/
    main.py           # Entry point (for Python skills)
```

### Full Manifest Example

```toml
[skill]
name = "web-summarizer"
version = "0.1.0"
description = "Summarizes any web page into bullet points"
author = "openfang-community"
license = "MIT"
tags = ["web", "summarizer", "research"]

[runtime]
type = "python"
entry = "src/main.py"

[[tools.provided]]
name = "summarize_url"
description = "Fetch a URL and return a concise bullet-point summary"
input_schema = { type = "object", properties = { url = { type = "string", description = "The URL to summarize" } }, required = ["url"] }

[[tools.provided]]
name = "extract_links"
description = "Extract all links from a web page"
input_schema = { type = "object", properties = { url = { type = "string" } }, required = ["url"] }

[requirements]
tools = ["web_fetch"]
capabilities = ["NetConnect(*)"]
```

### Manifest Sections

#### [skill] -- Metadata

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Unique skill name (used as directory name and registry key) |
| `version` | string | No | Semantic version (default: `"0.1.0"`) |
| `description` | string | No | Human-readable description |
| `author` | string | No | Author name or organization |
| `license` | string | No | License identifier (e.g., `"MIT"`, `"Apache-2.0"`) |
| `tags` | array | No | Tags for discovery on ClawHub/FangHub |

#### [runtime] -- Execution Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | Yes | `"promptonly"`, `"python"`, `"node"`, `"wasm"`, `"shell"`, or `"builtin"` |
| `entry` | string | Conditional | Relative path to entry point. Required for all runtimes except `promptonly` and `builtin`. |

#### [[tools.provided]] -- Tool Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Tool name (must be unique across all loaded tools) |
| `description` | string | Yes | Description shown to the LLM for tool selection |
| `input_schema` | object | Yes | JSON Schema defining the tool's input parameters |

#### [requirements] -- Host Requirements

| Field | Type | Description |
|-------|------|-------------|
| `tools` | array | Built-in tools this skill needs the host to provide |
| `capabilities` | array | Capability strings the agent must have (e.g., `"NetConnect(*)"`, `"ShellExec(*)"`) |

---

## Python Skills

Python skills run as subprocesses and communicate via JSON over stdin/stdout. The environment is isolated (`env_clear()`) -- only `PATH`, `HOME`, and `PYTHONIOENCODING=utf-8` are forwarded.

### Protocol

OpenFang sends a JSON payload to stdin:

```json
{
  "tool": "summarize_url",
  "input": {
    "url": "https://example.com"
  }
}
```

The script writes a JSON result to stdout:

```json
{
  "result": "- Point one\n- Point two\n- Point three"
}
```

On error, return an error object:

```json
{
  "error": "Failed to fetch URL: connection refused"
}
```

If stdout is not valid JSON, OpenFang wraps the raw text in `{"result": "<stdout>"}`.

### Example: Web Summarizer

```python
#!/usr/bin/env python3
import json
import sys
import urllib.request


def summarize_url(url: str) -> str:
    req = urllib.request.Request(url, headers={"User-Agent": "OpenFang-Skill/1.0"})
    with urllib.request.urlopen(req, timeout=30) as resp:
        content = resp.read().decode("utf-8", errors="replace")
    text = content[:500].strip()
    return f"Summary of {url}:\n{text}..."


def main():
    payload = json.loads(sys.stdin.read())
    tool_name = payload["tool"]
    input_data = payload["input"]

    try:
        if tool_name == "summarize_url":
            result = summarize_url(input_data["url"])
        else:
            print(json.dumps({"error": f"Unknown tool: {tool_name}"}))
            return
        print(json.dumps({"result": result}))
    except Exception as e:
        print(json.dumps({"error": str(e)}))


if __name__ == "__main__":
    main()
```

### Python SDK

For more advanced skills, use the SDK (`sdk/python/openfang_sdk.py`):

```python
#!/usr/bin/env python3
from openfang_sdk import SkillHandler

handler = SkillHandler()

@handler.tool("summarize_url")
def summarize_url(url: str) -> str:
    return "Summary..."

if __name__ == "__main__":
    handler.run()
```

---

## Node.js Skills

Node.js skills use the same JSON stdin/stdout protocol as Python. The environment is isolated with `env_clear()` and `NODE_NO_WARNINGS=1`.

Requires Node.js 18+.

```javascript
const fs = require('fs');

const input = JSON.parse(fs.readFileSync('/dev/stdin', 'utf8'));
const { tool, input: params } = input;

if (tool === 'my_action') {
  console.log(JSON.stringify({ result: `Processed: ${params.input}` }));
} else {
  console.log(JSON.stringify({ error: `Unknown tool: ${tool}` }));
}
```

---

## WASM Skills

WASM skills run inside a sandboxed Wasmtime environment with resource limits.

### Building

1. Write your skill in Rust (or any language that targets `wasm32-wasi`):

```rust
use std::io::{self, Read};

#[no_mangle]
pub extern "C" fn _start() {
    let mut input = String::new();
    io::stdin().read_to_string(&mut input).unwrap();

    let payload: serde_json::Value = serde_json::from_str(&input).unwrap();
    let tool = payload["tool"].as_str().unwrap_or("");
    let input_data = &payload["input"];

    let result = match tool {
        "my_tool" => {
            let param = input_data["param"].as_str().unwrap_or("");
            format!("Processed: {param}")
        }
        _ => format!("Unknown tool: {tool}"),
    };

    println!("{}", serde_json::json!({"result": result}));
}
```

2. Compile:

```bash
cargo build --target wasm32-wasi --release
```

3. Reference in manifest:

```toml
[runtime]
type = "wasm"
entry = "target/wasm32-wasi/release/my_skill.wasm"
```

### Sandbox Limits

- **Fuel limit** -- maximum computation steps (prevents infinite loops)
- **Memory limit** -- maximum memory allocation
- **Capabilities** -- only the capabilities granted to the agent apply

---

## Shell Skills

Shell skills run Bash or sh scripts via subprocess. Added in v0.5.0.

Same JSON stdin/stdout protocol. The environment is fully isolated via `env_clear()`.

```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.tool')
PARAM=$(echo "$INPUT" | jq -r '.input.query')

case "$TOOL" in
  my_search)
    RESULT=$(grep -r "$PARAM" /some/path 2>/dev/null || echo "No matches")
    echo "{\"result\": \"$RESULT\"}"
    ;;
  *)
    echo "{\"error\": \"Unknown tool: $TOOL\"}"
    ;;
esac
```

---

## SHA256 Verification

OpenFang computes SHA256 checksums for all downloaded skills. The `SkillVerifier` module provides:

- `sha256_hex(data)` -- compute the hex-encoded SHA256 hash
- `verify_checksum(data, expected_sha256)` -- verify data integrity against a known hash

When installing from ClawHub, the download content is hashed immediately after receipt and logged for auditability. Checksum comparison is case-insensitive.

---

## Prompt Injection Scanning

All SKILL.md content (bundled, user-installed, workspace, and ClawHub-downloaded) passes through an automated prompt injection scanner. This was implemented after 341 malicious skills were discovered on ClawHub in February 2026.

### Detected Patterns

**Critical** (blocks skill loading):
- "ignore previous instructions"
- "ignore all previous"
- "disregard previous"
- "forget your instructions"
- "you are now"
- "new instructions:"
- "system prompt override"
- "ignore the above"
- "do not follow"
- "override system"

**Warning** (logged, skill still loads):
- Data exfiltration patterns: "send to http/https", "post to http/https", "exfiltrate", "forward all", "send all data", "base64 encode and send", "upload to"
- Shell command references: `rm -rf`, `chmod`, `sudo`

**Info**:
- Content exceeding 50,000 bytes

### Manifest Security Scan

Beyond prompt content, the manifest itself is scanned:
- Node.js runtime triggers a "broad filesystem and network access" warning
- `ShellExec` or `shell_exec` capability requests are flagged as critical
- `NetConnect(*)` is flagged as a warning
- `shell_exec` or `bash` tool requirements are critical
- `file_write` or `file_delete` tool requirements are warnings
- More than 10 tool requirements triggers an info notice

---

## Complete Example: Creating a Custom Skill

This walkthrough creates a `url-checker` skill that verifies whether URLs are reachable.

### 1. Create the directory

```bash
mkdir -p ~/.openfang/skills/url-checker
cd ~/.openfang/skills/url-checker
```

### 2. Write skill.toml

```toml
[skill]
name = "url-checker"
version = "0.1.0"
description = "Checks whether URLs are reachable and reports HTTP status codes"
author = "yourname"
license = "MIT"
tags = ["web", "monitoring", "health-check"]

[runtime]
type = "python"
entry = "main.py"

[[tools.provided]]
name = "check_url"
description = "Send an HTTP HEAD request to a URL and report its status code and response time"
input_schema = { type = "object", properties = { url = { type = "string", description = "The URL to check" } }, required = ["url"] }

[[tools.provided]]
name = "check_urls_batch"
description = "Check multiple URLs and return a status report"
input_schema = { type = "object", properties = { urls = { type = "array", items = { type = "string" }, description = "List of URLs to check" } }, required = ["urls"] }

[requirements]
capabilities = ["NetConnect(*)"]
```

### 3. Write main.py

```python
#!/usr/bin/env python3
import json
import sys
import time
import urllib.request


def check_url(url: str) -> dict:
    try:
        start = time.time()
        req = urllib.request.Request(url, method="HEAD",
                                     headers={"User-Agent": "OpenFang-URLChecker/1.0"})
        with urllib.request.urlopen(req, timeout=10) as resp:
            elapsed = round((time.time() - start) * 1000)
            return {"url": url, "status": resp.status, "time_ms": elapsed, "ok": True}
    except Exception as e:
        return {"url": url, "status": 0, "error": str(e), "ok": False}


def main():
    payload = json.loads(sys.stdin.read())
    tool = payload["tool"]
    params = payload["input"]

    try:
        if tool == "check_url":
            result = check_url(params["url"])
        elif tool == "check_urls_batch":
            result = [check_url(u) for u in params["urls"]]
        else:
            print(json.dumps({"error": f"Unknown tool: {tool}"}))
            return
        print(json.dumps({"result": result}))
    except Exception as e:
        print(json.dumps({"error": str(e)}))


if __name__ == "__main__":
    main()
```

### 4. Test locally

```bash
echo '{"tool":"check_url","input":{"url":"https://example.com"}}' | python3 main.py
```

Expected output:
```json
{"result": {"url": "https://example.com", "status": 200, "time_ms": 123, "ok": true}}
```

### 5. Install

```bash
openfang skill install ~/.openfang/skills/url-checker
```

### 6. Assign to an agent

Add to your agent's configuration:

```toml
skills = ["url-checker"]
```

Or via the API:

```bash
curl -X PUT http://127.0.0.1:4200/api/agents/<id>/skills \
  -H "Content-Type: application/json" \
  -d '{"skills": ["url-checker"]}'
```

---

## OpenClaw Compatibility

OpenFang can install and run OpenClaw-format skills. The installer auto-detects both formats:

**SKILL.md format** -- Detected by the presence of a `SKILL.md` file with YAML frontmatter. Converted to a `promptonly` skill with the Markdown body as prompt context.

**Node.js format** -- Detected by `package.json` + `index.js`/`index.ts`/`dist/index.js`. Converted to a `node` runtime skill. TypeScript skills must be compiled first (`npm run build`).

Tool name translation is automatic. OpenClaw tool names are mapped to OpenFang equivalents (e.g., `Bash` becomes `shell_exec`, `Read` becomes `file_read`).

Skills imported via `openfang migrate --from openclaw` are scanned and reported in the migration report.

---

## Publishing to ClawHub

```bash
# Validate and package
openfang skill publish
```

Ensure your `skill.toml` has complete metadata: `name`, `version`, `description`, `author`, `license`, and `tags`.

---

## CLI Commands

```bash
openfang skill install <source>     # Install from local dir, ClawHub slug, or git URL
openfang skill list                 # List all installed skills
openfang skill remove <name>        # Remove an installed skill
openfang skill search <query>       # Search ClawHub marketplace
openfang skill info <slug>          # View ClawHub skill details
openfang skill create               # Scaffold a new skill (interactive)
openfang skill publish              # Publish to ClawHub
```

---

## Skill Provenance

Every skill tracks its origin via the `source` field:

| Source | Description |
|--------|-------------|
| `Bundled` | Compiled into the OpenFang binary at build time |
| `Native` | Manually installed or locally developed |
| `OpenClaw` | Converted from OpenClaw SKILL.md or package.json format |
| `ClawHub { slug, version }` | Downloaded from the ClawHub marketplace |

---

## Best Practices

1. **Keep skills focused** -- one skill should do one thing well.
2. **Declare minimal requirements** -- only request the tools and capabilities your skill actually needs.
3. **Use descriptive tool names** -- the LLM reads the tool name and description to decide when to use it.
4. **Provide clear input schemas** -- include descriptions for every parameter.
5. **Handle errors gracefully** -- always return a JSON error object rather than crashing.
6. **Version carefully** -- use semantic versioning; breaking changes require a major version bump.
7. **Test with multiple agents** -- verify your skill works with different agent templates and providers.
