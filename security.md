# Security

OpenFang implements 16 documented security layers in a defense-in-depth architecture. Each layer operates independently so that failure of one does not compromise others.

**Security contact:** jaber@rightnowai.co (48-hour response SLA)

---

## Supported Versions

| Version | Support |
|---------|---------|
| 0.5.x | Active security updates |
| 0.4.x | Critical fixes only |
| < 0.4.0 | End of life |

---

## The 16 Security Layers

### 1. Capability-Based Access Control

Every agent declares required capabilities in its `agent.toml`. The kernel enforces these at execution time via `CapabilityManager` (`crates/openfang-kernel/src/capabilities.rs`). Agents can only perform actions they have been explicitly granted.

**21 capability types across 7 categories:**

| Category | Capabilities |
|----------|-------------|
| Filesystem | `FileRead(glob)`, `FileWrite(glob)` |
| Network | `NetConnect(host)`, `NetListen(port)` |
| Tools | `ToolInvoke(name)`, `ToolAll` |
| LLM | `LlmQuery(pattern)`, `LlmMaxTokens(n)` |
| Agent interaction | `AgentSpawn`, `AgentMessage(pattern)`, `AgentKill(pattern)` |
| Memory | `MemoryRead(scope)`, `MemoryWrite(scope)` |
| Shell | `ShellExec(pattern)`, `EnvRead(pattern)` |
| OFP | `OfpDiscover`, `OfpConnect(pattern)`, `OfpAdvertise` |
| Economic | `EconSpend(usd)`, `EconEarn`, `EconTransfer(pattern)` |

```toml
[capabilities]
required = [
  "FileRead(./workspace/**)",
  "FileWrite(./output/*.md)",
  "NetConnect(api.github.com)",
  "ToolInvoke(web_fetch)",
  "MemoryRead",
  "MemoryWrite",
]
```

The capability check (`CapabilityManager::check`) uses pattern matching via `capability_matches()` from `openfang-types`. Child agents cannot inherit capabilities beyond what the parent agent possesses.

At the tool execution layer (`tool_runner.rs`), an `allowed_tools` list enforces that the LLM cannot hallucinate tool names outside the agent's capability grants.

### 2. WASM Dual-Metered Sandbox

Skills and untrusted code run inside Wasmtime with two independent interrupt mechanisms (`crates/openfang-runtime/src/sandbox.rs`):

- **Fuel metering:** Instruction-level budget. Default: 1,000,000 fuel units. Exhausting fuel triggers `Trap::OutOfFuel` and immediate halt.
- **Epoch interruption:** Wall-clock timeout enforced by a dedicated watchdog thread. Default: 30 seconds. A background thread calls `engine.increment_epoch()` after the deadline, triggering `Trap::Interrupt`.

Both mechanisms must be bypassed for a WASM escape -- independently improbable.

The sandbox enforces deny-by-default permissions: no filesystem, network, or credential access unless explicitly granted through capabilities checked in `host_functions::dispatch()`. The guest ABI exposes only `host_call` (capability-checked RPC) and `host_log` (no capability required).

### 3. Merkle Hash-Chain Audit Trail

Every agent action is recorded in a cryptographic audit log (`crates/openfang-runtime/src/audit.rs`). Entries are stored in SQLite (`audit_entries` table, schema V8) and survive daemon restarts.

Each entry contains:

| Field | Description |
|-------|-------------|
| `seq` | Monotonically increasing sequence number |
| `timestamp` | ISO-8601 timestamp |
| `agent_id` | Agent that triggered the action |
| `action` | Category: ToolInvoke, ShellExec, AgentSpawn, AuthAttempt, etc. |
| `detail` | Free-form action detail |
| `outcome` | Result (ok, denied, error message) |
| `prev_hash` | SHA-256 hash of the previous entry |
| `hash` | SHA-256 of this entry's fields concatenated with `prev_hash` |

12 auditable action categories: `ToolInvoke`, `CapabilityCheck`, `AgentSpawn`, `AgentKill`, `AgentMessage`, `MemoryAccess`, `FileAccess`, `NetworkAccess`, `ShellExec`, `AuthAttempt`, `WireConnect`, `ConfigChange`.

Tamper detection: modifying any entry breaks the chain. The `verify_integrity()` method walks the entire chain recomputing every hash. Chain integrity is verified on boot when loading from the database.

```bash
curl http://127.0.0.1:4200/api/audit/verify \
  -H "Authorization: Bearer <api_key>"
```

### 4. Information Flow Taint Tracking

Data is labeled with taint labels as it flows through the system (`crates/openfang-types/src/taint.rs`). The system implements a lattice-based taint propagation model that prevents tainted values from flowing into sensitive sinks without explicit declassification.

**Labels:**

| Label | Applied To |
|-------|-----------|
| `ExternalNetwork` | Data fetched from the internet |
| `UserInput` | Data from user messages |
| `Pii` | Personally identifiable information |
| `Secret` | Vault secrets, API keys, tokens |
| `UntrustedAgent` | Output from untrusted/sandboxed agents |

**Pre-configured sinks (blocked combinations):**

| Sink | Blocked Labels | Purpose |
|------|---------------|---------|
| `shell_exec` | ExternalNetwork, UntrustedAgent, UserInput | Prevents prompt injection into shell |
| `net_fetch` | Secret, Pii | Prevents data exfiltration |
| `agent_message` | Secret | Prevents secret leakage between agents |

In `tool_runner.rs`, taint checks are applied before tool execution:
- `check_taint_shell_exec()` blocks shell metacharacters and heuristic patterns (curl, wget, base64 -d, eval) with ExternalNetwork taint.
- `check_taint_net_fetch()` blocks URLs containing credential patterns (api_key=, token=, password=) with Secret taint.

Values can be explicitly declassified via `TaintedValue::declassify()` when the caller asserts the data has been sanitized.

### 5. Ed25519 Signed Agent Manifests

Agent manifests can be cryptographically signed with Ed25519 (`crates/openfang-types/src/manifest_signing.rs`):

1. Compute SHA-256 of the manifest content
2. Sign the hash with Ed25519 (via `ed25519-dalek`)
3. Bundle the signature, public key, and content hash into a `SignedManifest` envelope

Verification recomputes the hash and checks the Ed25519 signature against the embedded public key. Rejects modified manifests, wrong-key signatures, and hash mismatches.

This provides supply chain security -- manifests from untrusted sources (ClawHub marketplace, peer networks) can be verified before spawning.

### 6. SSRF Protection

All outbound HTTP requests pass through `check_ssrf()` in `crates/openfang-runtime/src/web_fetch.rs` BEFORE any network I/O.

**Blocked targets:**

| Category | Addresses |
|----------|-----------|
| Private IPs | 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 169.254.0.0/16 |
| Cloud metadata | 169.254.169.254 (AWS/GCP/Azure), 100.100.100.200 (Alibaba), 192.0.0.192 (Azure alt) |
| Hostname blocklist | localhost, ip6-localhost, metadata.google.internal, metadata.aws.internal, instance-data, 0.0.0.0, ::1 |
| Protocol | Only http:// and https:// allowed (blocks file://, ftp://, gopher://) |

**DNS rebinding protection:** The resolved IP is checked, not just the hostname. A domain resolving to 169.254.169.254 is blocked even if the hostname is not on the blocklist.

**SSRF allowlist (v0.5.6):** Self-hosted deployments can bypass the private-IP check for specific hosts:

```toml
[web.fetch]
ssrf_allowed_hosts = ["n8n.local", "*.olares.com", "10.0.0.0/8"]
```

Entries can be exact hostnames, wildcard domains, or CIDR ranges. **Cloud metadata endpoints are NEVER allowed regardless of the allowlist.** Even `169.254.0.0/16` in the allowlist will not permit access to 169.254.169.254.

### 7. Secret Zeroization

All API keys and secrets use `Zeroizing<String>` from the `zeroize` crate. Memory is overwritten with zeros when the value is dropped.

Used extensively in:
- `openfang-extensions/src/vault.rs` -- encrypted vault uses `Zeroizing<String>` for all stored entries, `Zeroizing<[u8; 32]>` for master keys, and `Zeroizing<String>` for key derivation intermediaries.
- `openfang-kernel/src/kernel.rs` -- API key resolution from vault returns `Zeroizing<String>`.
- `openfang-cli/src/main.rs` -- vault set operations use `Zeroizing<String>`.

The vault encrypts secrets at rest with AES-256-GCM using a master key derived via HKDF from a keyring-stored key or environment variable. The master key itself is stored as `Zeroizing<[u8; 32]>`.

Secrets are also redacted from logs and health endpoint responses.

### 8. OFP Mutual Authentication

The OpenFang Wire Protocol (`crates/openfang-wire/src/peer.rs`) uses HMAC-SHA256 with nonces for mutual authentication over TCP:

**Handshake flow:**
1. Client sends `Handshake` with `node_id`, a fresh UUID nonce, and `HMAC-SHA256(shared_secret, nonce || node_id)`
2. Server verifies HMAC, checks nonce against the replay tracker, sends `HandshakeAck` with its own nonce and HMAC
3. Both sides derive a per-session key: `HMAC-SHA256(shared_secret, our_nonce || their_nonce)`

**Security properties:**
- **Replay prevention:** `NonceTracker` maintains a 5-minute window of seen nonces with garbage collection. Replayed nonces are rejected.
- **Per-message HMAC:** After handshake, all messages use authenticated read/write with the session key. Frame format: `[4-byte length][JSON body][64-byte hex HMAC]`.
- **Constant-time verification:** HMAC comparison uses `subtle::ConstantTimeEq` to prevent timing attacks.
- **Mandatory authentication:** Non-Handshake messages before authentication are rejected with HTTP 401.
- **Message size limits:** Maximum 16 MB per message (`MAX_MESSAGE_SIZE`).

OFP refuses to start without a shared secret (`PeerNode::start` returns error if `shared_secret` is empty).

### 9. Security Headers

All HTTP responses include security headers via `middleware::security_headers()` in `crates/openfang-api/src/middleware.rs`:

```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'none'; frame-ancestors 'none'
Referrer-Policy: strict-origin-when-cross-origin
Cache-Control: no-store, no-cache, must-revalidate
Strict-Transport-Security: max-age=63072000; includeSubDomains
```

The dashboard page sets its own nonce-based CSP (`script-src 'self' 'nonce-...'`). The middleware preserves that header when present, applying the strict API default only to non-dashboard responses.

### 10. GCRA Rate Limiter

Per-IP rate limiting using Governor's GCRA (Generic Cell Rate Algorithm) in `crates/openfang-api/src/rate_limiter.rs`:

**Budget:** 500 tokens per minute per IP address.

**Cost-aware operations:**

| Cost | Endpoints |
|------|-----------|
| 1 | `/api/health`, `/api/status`, `/api/version`, `/api/tools` |
| 2 | `/api/agents`, `/api/skills`, `/api/peers`, `/api/config` |
| 3 | `/api/usage` |
| 5 | `/api/audit/*`, default |
| 10 | `/api/marketplace/*`, PUT updates |
| 30 | POST `*/message` |
| 50 | POST `/api/agents`, POST `/api/skills/install` |
| 100 | POST `*/run`, POST `/api/migrate` |

Returns HTTP 429 when the client's token budget is exhausted.

### 11. Path Traversal Prevention

All file operations go through `validate_path()` and `resolve_file_path()` in `tool_runner.rs`:

1. Reject `..` path components (`Component::ParentDir` check)
2. When a workspace root is set, resolve through `workspace_sandbox::resolve_sandbox_path()` which canonicalizes paths and verifies they remain within the workspace
3. Executable paths are validated separately via `subprocess_sandbox::validate_executable_path()`

This applies to `file_read`, `file_write`, `file_list`, `apply_patch`, and skill loading. Test coverage includes explicit traversal attack vectors (`../../etc/passwd`, `/foo/../../etc`).

### 12. Subprocess Sandbox

All subprocess executions use environment isolation (`crates/openfang-runtime/src/subprocess_sandbox.rs`):

- `env_clear()` strips ALL inherited environment variables
- Re-adds only safe vars: `PATH`, `HOME`, `TMPDIR`, `LANG`, `LC_ALL`, `TERM`
- On Windows: also `USERPROFILE`, `SYSTEMROOT`, `COMSPEC`, `PATHEXT`, etc.
- Plus caller-specified allowed vars (e.g., API keys for Hands)

**Exec policy enforcement** (three modes):
- **Deny:** All subprocess execution blocked.
- **Allowlist** (default): Commands validated against `safe_bins` and `allowed_commands`. Uses direct argv splitting via shlex -- no shell interpreter. Eliminates shell injection via encoding tricks, `$IFS`, glob expansion.
- **Full:** User explicitly opted into unrestricted shell access; uses `sh -c`.

**Shell metacharacter blocking:** `contains_shell_metacharacters()` blocks backticks, `$(`, `${`, semicolons, pipes, redirects, braces, newlines, null bytes, and ampersands in ALL modes including Full. This is a defense-in-depth layer that runs before allowlist validation.

**v0.5.7 security fix (#919):** `process_start` previously spawned subprocesses with NO exec policy enforcement. An LLM in Allowlist mode could call `process_start` with `command="rm"` to bypass `allowed_commands`. Fixed by adding metacharacter rejection and `validate_command_allowlist()` checks to `tool_process_start()`. Both `shell_exec` and `process_start` are now treated as shell tools for approval gating via `is_shell_tool()`.

### 13. Prompt Injection Scanner

All skill `SKILL.md` files are scanned before installation via `SkillVerifier::scan_prompt_content()` in `crates/openfang-skills/src/verify.rs`.

**Critical patterns (block installation):**
- Override instructions: "ignore previous instructions", "disregard previous", "forget your instructions", "you are now", "new instructions:", "system prompt override", "override system", "do not follow"
- Role confusion: "ignore the above", "ignore all previous"

**Warning patterns (flag for review):**
- Data exfiltration: "send to http", "exfiltrate", "forward all", "base64 encode and send", "upload to"
- Shell commands: "rm -rf", "chmod", "sudo"

**Manifest scanning** (`security_scan()`): flags dangerous runtime types (Node.js), dangerous capabilities (ShellExec, NetConnect(*)), dangerous tool requirements (shell_exec, file_write), and unusually high tool counts (>10).

The session repair module (`session_repair.rs`) also strips known prompt injection markers from tool results: `<|system|>`, `<|im_start|>`, `<<SYS>>`, `IGNORE PREVIOUS INSTRUCTIONS`, etc.

### 14. Loop Guard

Prevents infinite tool-call loops (`crates/openfang-runtime/src/loop_guard.rs`):

- **SHA-256 dedup:** Tracks hash of `(tool_name, serialized_params)` for each tool call.
- **Graduated response:** Warn at 3 identical calls, block at 5. Global circuit breaker at 30 total calls.
- **Outcome-aware detection:** Tracks `(call + result)` hashes. Identical call+result pairs escalate faster (warn at 2, block at 3).
- **Ping-pong detection:** Identifies A-B-A-B or A-B-C-A-B-C alternating patterns that evade single-hash counting. Blocks after 3 full pattern repeats.
- **Poll tool handling:** Relaxed thresholds (3x multiplier) for tools expected to poll repeatedly (e.g., `shell_exec` status checks with "status", "poll", "wait", "docker ps" patterns).
- **Backoff schedule:** Suggests increasing wait times for polling (5s, 10s, 30s, 60s).
- **Warning bucket:** Prevents warn spam by upgrading to Block after `max_warnings_per_call` (default 3) warnings for the same call hash.

### 15. Session Repair

On startup and before each request, the session message history is validated and repaired (`crates/openfang-runtime/src/session_repair.rs`):

1. **Drop orphaned ToolResult blocks** that have no matching ToolUse ID
2. **Drop empty messages** (empty text or empty block lists)
3. **Reorder misplaced ToolResults** to follow their matching ToolUse assistant message
4. **Deduplicate ToolResults** with the same `tool_use_id` (keeps first occurrence)
5. **Insert synthetic error results** for unmatched ToolUse blocks (prevents API 400s)
6. **Remove aborted assistant messages** (empty content blocks followed by user messages)
7. **Merge consecutive same-role messages** (Anthropic API requires alternation)

Phase ordering matters: dedup runs before synthetic insertion to correctly handle providers like Moonshot that reuse `tool_use_id` values across turns.

Additionally, `strip_tool_result_details()` sanitizes tool output before feeding it back to the LLM: truncates to 10K chars, strips base64 blobs >1000 chars, and removes prompt injection markers.

`prune_heartbeat_turns()` removes NO_REPLY assistant turns and their preceding triggers from session history to prevent context bloat.

### 16. Health Endpoint Redaction

Public health endpoint returns minimal information:

```
GET /api/health -> {"status": "ok"}
```

Authenticated health detail endpoint returns full diagnostics:

```
GET /api/health/detail -> {
  "status": "ok",
  "version": "v0.5.7",
  "uptime_secs": 3600,
  "agent_count": 5,
  "memory_usage_mb": 42,
  "providers": [...],
  ...
}
```

---

## Dashboard Auth (Argon2id)

Dashboard login passwords are hashed using Argon2id with random salts, stored in PHC string format.

```toml
[auth]
enabled = true
username = "admin"
password_hash = "$argon2id$v=19$m=19456,t=2,p=1$..."
```

Generate a password hash:
```bash
openfang auth hash-password
```

**Breaking change in v0.5.7:** Prior versions used unsalted SHA256 for dashboard passwords. Existing `password_hash` values must be regenerated with `openfang auth hash-password`. SHA256 hex hashes are no longer accepted.

---

## Security Configuration

```toml
# ~/.openfang/config.toml

# API Bearer token for general API access
api_key = "replace-me"

[auth]
enabled = true
username = "admin"
password_hash = "$argon2id$v=19$m=19456,t=2,p=1$..."

[web.fetch]
ssrf_allowed_hosts = []  # SSRF allowlist for self-hosted environments
```

---

## Security-Critical Dependencies

These packages are pinned and audited (`.cargo/audit.toml`):

| Crate | Purpose |
|-------|---------|
| `ed25519-dalek 2` | Manifest signing |
| `sha2` | Hash functions (audit trail, taint, loop guard) |
| `hmac` | HMAC authentication (OFP wire protocol) |
| `subtle` | Constant-time comparisons (OFP HMAC verify) |
| `zeroize` | Secret memory wiping |
| `rand` | Cryptographic randomness |
| `governor 0.10` | GCRA rate limiting |
| `argon2` | Password hashing |
| `aes-gcm` | Vault encryption |
| `wasmtime` | WASM sandbox runtime |

---

## Security Advisories

### v0.5.7 -- Allowlist Bypass via process_start (#919)

`process_start` spawned subprocesses without exec policy enforcement. An LLM in Allowlist mode could use `process_start` with `command="rm"` to bypass `allowed_commands`. Fixed by applying the same metacharacter rejection and `validate_command_allowlist()` checks as `shell_exec`.

### RUSTSEC-2026-0049 (rustls-webpki)

Fixed in v0.5.4. The `rustls-webpki` dependency was updated to address a certificate validation issue. No user action required.

---

## Reporting Vulnerabilities

**Email:** jaber@rightnowai.co
**Response SLA:** 48 hours for acknowledgment, 90 days for fix

**In scope:**
- Authentication bypass
- Remote code execution (RCE)
- Path traversal and SSRF
- Privilege escalation
- Information disclosure
- Denial of service
- Supply chain attacks
- WASM sandbox escapes

**Out of scope:**
- Social engineering
- Attacks requiring physical access
- Third-party service vulnerabilities

---

## Security Checklist

For production deployments:

- [ ] Set `api_key` to a strong, random bearer token
- [ ] Bind to `127.0.0.1` (not `0.0.0.0`) unless remote access is needed
- [ ] Enable `[auth]` with Argon2id password hash for the dashboard
- [ ] Review agent capability declarations (`agent.toml`)
- [ ] Enable approval gates for `shell_exec` and `file_write`
- [ ] Keep `exec_policy.mode = "allowlist"` (default) for agent shell access
- [ ] Pin to a specific commit/tag for reproducibility
- [ ] Run `cargo audit` before building from source
- [ ] Use `log_level = "warn"` in production (avoid `debug` leaking secrets)
- [ ] Restrict `daemon.json` permissions (auto-set to 0600 on Unix)
- [ ] Use WASM runtime for untrusted agent code
- [ ] Review `ssrf_allowed_hosts` -- only add hosts you control
