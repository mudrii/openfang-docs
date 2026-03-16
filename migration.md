# Migration from OpenClaw

OpenFang provides a one-command migration path from OpenClaw.

---

## Quick Migration

```bash
openfang migrate --from openclaw
```

This automatically:
1. Detects `~/.openclaw/` (or specified path)
2. Imports agents, memory, channels, and skills
3. Converts YAML manifests to TOML
4. Remaps tool names to OpenFang equivalents
5. Generates `migration_report.md` with a summary

---

## Options

```bash
# Preview without applying changes
openfang migrate --from openclaw --dry-run

# Custom OpenClaw directory
openfang migrate --from openclaw --source-dir /path/to/.openclaw

# Migration via skill (autonomous)
# <invoke name="openclaw_migrate">[]</invoke>
```

---

## What Gets Migrated

| Item | Migrated | Notes |
|------|----------|-------|
| Agent manifests | ✅ | YAML → TOML conversion |
| Agent personas | ✅ | system prompt preserved |
| LLM config | ✅ | provider/model mapping |
| Channel tokens | ✅ | Telegram, Discord, etc. |
| Memory files | ✅ | imported into SQLite |
| Skills | ✅ | format converted |
| Sessions | ❌ | Fresh start recommended |
| Workspace files | ❌ | Copy manually if needed |

---

## Tool Name Mappings

OpenClaw and LLM-hallucinated tool names are automatically remapped:

| OpenClaw Name | OpenFang Name |
|---------------|---------------|
| `read_file` | `file_read` |
| `write_file` | `file_write` |
| `list_files` | `file_list` |
| `execute_command` | `shell_exec` |
| `memory_save` | `memory_store` |
| `memory_load` | `memory_recall` |
| `search_web` | `web_search` |
| `fetch_url` | `web_fetch` |
| `Read` | `file_read` |
| `Write` | `file_write` |
| `Bash` | `shell_exec` |
| `Edit` | `file_write` |
| `fs-read` | `file_read` |
| `readFile` | `file_read` |
| `runCommand` | `shell_exec` |
| `shell` | `shell_exec` |
| `execute` | `shell_exec` |

---

## New Tools in OpenFang (Not in OpenClaw)

These tools are available in OpenFang but have no OpenClaw equivalent:

| Tool | Description |
|------|-------------|
| `agent_spawn` | Spawn a new agent dynamically |
| `agent_kill` | Terminate an agent |
| `agent_find` | Find agents by name or tags |
| `agent_message` | Send message to another agent |
| `task_post` | Post task to agent's queue |
| `event_publish` | Publish an event to the event bus |
| `schedule_create` | Create a cron job |
| `schedule_delete` | Delete a cron job |
| `image_analyze` | Analyze images with vision models |
| `image_generate` | Generate images (DALL-E, etc.) |
| `location_get` | Get current location |
| `tts_speak` | Text-to-speech synthesis |

---

## Config Format Differences

### OpenClaw (YAML)

```yaml
agent:
  name: my-assistant
  model: claude-3-opus-20240229
  capabilities:
    - read_files
    - execute_commands
```

### OpenFang (TOML)

```toml
[agent]
name = "my-assistant"

[model]
provider = "anthropic"
model = "claude-opus-4-6"

[capabilities]
required = [
  "FileRead(*)",
  "ShellExec",
]
```

Key differences:
- **YAML → TOML**: Config format changed
- **Explicit capabilities**: Must declare each capability explicitly (more secure)
- **Provider/model split**: Provider is now explicit, not inferred from model name
- **Tool names**: Use `snake_case` throughout

---

## Verification

After migration, verify everything works:

```bash
# Check agent list
openfang agent list

# Check channel status
openfang channel list

# Run doctor check
openfang doctor

# Review migration report
cat migration_report.md
```

---

## Manual Migration Steps

If automatic migration doesn't cover your setup:

1. **Create config.toml:**
   ```bash
   openfang init --quick
   ```

2. **Set provider API key:**
   ```bash
   openfang config set-key anthropic sk-ant-...
   ```

3. **Spawn agents manually:**
   ```bash
   openfang agent spawn agents/assistant/agent.toml --name my-assistant
   ```

4. **Reconfigure channels:**
   ```bash
   openfang channel setup telegram
   ```

5. **Install matching skills:**
   ```bash
   openfang skill install github
   openfang skill install docker
   ```
