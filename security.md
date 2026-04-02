# Security

OpenFang implements 17 independent security layers in a defense-in-depth architecture. Each layer operates independently so that failure of one does not compromise others.

**Security contact:** jaber@rightnowai.co (48-hour response SLA)

---

## The 17 Security Layers

### 1. Capability-Based Access Control

Every agent declares required capabilities in its `agent.toml`. The kernel enforces these at execution time — agents can only perform actions they have been explicitly granted.

**23 capability types across 9 categories:**

```toml
[capabilities]
required = [
  "FileRead(./workspace/**)",   # Read files matching glob
  "FileWrite(./output/*.md)",   # Write to specific paths
  "NetConnect(api.github.com)", # Connect to specific host
  "ToolInvoke(web_fetch)",      # Use specific tool
  "MemoryRead",                 # Read own memory
  "MemoryWrite",                # Write to own memory
]
```

**Privilege escalation prevention:** Child agents cannot inherit capabilities beyond what the parent agent possesses.

### 2. WASM Dual Metering

Skills and untrusted code run inside Wasmtime with two independent interrupt mechanisms:

- **Fuel metering:** Instruction-level budget. Exhausting fuel → immediate halt.
- **Epoch interruption:** Wall-clock timeout enforced by a dedicated watchdog thread.

Both mechanisms must be bypassed for a WASM escape — independently improbable.

### 3. Merkle Hash Chain Audit Trail

Every agent action is recorded in a cryptographic audit log stored in SQLite:

- Each entry includes: timestamp, agent_id, action, hash of previous entry (SHA256)
- Tamper detection: Modifying any entry breaks the chain
- Verification: `GET /api/audit/verify` checks entire chain integrity

```bash
# Verify audit trail integrity
curl http://127.0.0.1:4200/api/audit/verify \
  -H "Authorization: Bearer <api_key>"
```

### 4. Information Flow Taint Tracking

Data is labeled with taint labels as it flows through the system:

| Label | Applied To |
|-------|-----------|
| `ExternalNetwork` | Data fetched from the internet |
| `UserInput` | Data from user messages |
| `Pii` | Personally identifiable information |
| `Secret` | Vault secrets |
| `UntrustedAgent` | Output from untrusted agents |

**Pre-configured sinks (blocked combinations):**

| Sink | Blocked Labels |
|------|---------------|
| `shell_exec` | ExternalNetwork, UntrustedAgent, UserInput |
| `net_fetch` | Secret, Pii |
| `agent_message` | Secret |

This prevents prompt injection (ExternalNetwork → shell_exec blocked) and data exfiltration (Secret → net_fetch blocked).

### 5. Ed25519 Signed Agent Manifests

Agent manifests can be signed with Ed25519:

```rust
// sign_manifest() produces SignedManifest with:
// - manifest hash (SHA256)
// - Ed25519 signature
// - public key identifier

// verify() checks:
// - Signature validity
// - Hash matches content
// - Rejects modified manifests
// - Rejects wrong-key signatures
```

This provides supply chain security — manifests from untrusted sources can be verified before spawning.

### 6. SSRF Protection

All outbound HTTP requests are validated against:

- **Private IP ranges:** 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 127.0.0.0/8
- **Cloud metadata endpoints:** 169.254.169.254 (AWS/GCP/Azure IMDS)
- **DNS rebinding protection:** Resolved IP is checked, not just the hostname

Blocks agents from accessing internal infrastructure via HTTP tools.

### 7. Secret Zeroization

All API keys and secrets use `Zeroizing<String>` — memory is overwritten with zeros when the value is dropped.

```rust
use zeroize::Zeroizing;

struct ProviderConfig {
    api_key: Zeroizing<String>,  // Auto-wiped on drop
}
```

Secrets are also redacted from logs and health endpoint responses.

### 8. OFP Mutual Authentication

The OpenFang Wire Protocol (peer-to-peer TCP) uses HMAC-SHA256 with nonces:

- **Handshake:** Both peers prove identity by signing a challenge with the shared secret
- **Replay prevention:** Nonce tracker maintains a 5-minute window of seen nonces
- **Per-message HMAC:** Each wire message is individually authenticated (added v0.3.30)

### 9. Security Headers

All HTTP responses include:

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-...'; ...
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000; includeSubDomains
Referrer-Policy: strict-origin-when-cross-origin
```

### 10. GCRA Rate Limiter

Per-IP rate limiting using Governor's GCRA (Generalized Cell Rate Algorithm):

- **Cost-aware:** Expensive operations consume more tokens
- **Token bucket:** Configurable burst size and refill rate
- **Per-endpoint:** Different limits for different API endpoints

```toml
[rate_limiting]
requests_per_minute = 60
burst_size = 10
```

### 11. Path Traversal Prevention

All file operations go through canonical path resolution:

1. Resolve symlinks
2. Check resolved path starts with allowed prefix
3. Reject `..` components that escape the workspace
4. Reject absolute paths to sensitive system directories

This applies to both agent tools (`file_read`, `file_write`) and skill loading.

### 12. Subprocess Sandbox

All subprocess executions (shell tools, Python skills, Node skills) use:

```rust
Command::new(program)
    .env_clear()                              // Clear ALL environment variables
    .env("SPECIFIC_VAR", allowed_value)       // Inject only approved vars
    .current_dir(workspace_dir)              // Restrict working directory
```

Since v0.3.30, `shell_exec` uses direct `execve()` — no shell interpreter — eliminating an entire class of shell injection vulnerabilities.

### 13. Prompt Injection Scanner

All skill SKILL.md files are scanned for known prompt injection patterns before installation:

- Override instructions (`"ignore previous instructions"`)
- Data exfiltration requests (`"send the contents of"`, `"transmit to"`)
- Capability escalation (`"you now have access to"`)
- Role confusion (`"act as if you are"`)

The scanner runs automatically on `openfang skill install`.

### 14. Loop Guard

Prevents infinite tool-call loops:

- **SHA256 dedup:** Tracks hash of recent tool call sequences
- **Circuit breaker:** Halts after detecting repeated identical sequences
- **Max iterations:** Hard limit on LLM-tool iterations per request (configurable)
- **Guidance injection:** When halted, injects explanation into LLM context

### 15. Session Repair (7-Phase Validation)

On startup and before each request, the session message history is validated:

1. Check for orphaned tool results (no matching tool call)
2. Verify message alternation (user→assistant→user→...)
3. Remove empty content blocks
4. Fix malformed tool call IDs
5. Repair truncated messages
6. Remove duplicate messages
7. Validate role sequences per provider requirements

This prevents cryptic LLM API errors from corrupted session state.

### 16. Argon2id Password Hashing

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

This prompts for a password and outputs an Argon2id PHC string to paste into `config.toml`.

**Breaking change (v0.5.0):** Prior versions used unsalted SHA256 for dashboard passwords. Existing `password_hash` values must be regenerated with `openfang auth hash-password`. SHA256 hex hashes are no longer accepted.

### 17. Health Endpoint Redaction

Public health endpoint returns minimal information:
```json
GET /api/health
{"status": "ok"}
```

Authenticated health endpoint returns full diagnostics:
```json
GET /api/health (with Authorization header)
{
  "status": "ok",
  "version": "0.4.4",
  "uptime_secs": 3600,
  "agent_count": 5,
  "memory_usage_mb": 42,
  "providers": [...],
  ...
}
```

---

## Security Configuration

```toml
# ~/.openfang/config.toml

[security]
# Require authentication for all endpoints (default: loopback bypass)
require_auth = true

# API key for Bearer token authentication
api_key = "${OPENFANG_API_KEY}"

# CORS allowed origins
cors_origins = ["http://localhost:3000"]

# Rate limiting
[security.rate_limit]
requests_per_minute = 60
burst_size = 10

# Approval policy (global default)
[security.approval]
require_approval = ["shell_exec"]
approval_timeout_secs = 60
auto_approve_autonomous = false
```

---

## Security-Critical Dependencies

These packages are pinned and audited (`.cargo/audit.toml`):

| Crate | Purpose |
|-------|---------|
| `ed25519-dalek 2` | Manifest signing |
| `sha2` | Hash functions |
| `hmac` | HMAC authentication |
| `subtle` | Constant-time comparisons |
| `zeroize` | Secret memory wiping |
| `rand` | Cryptographic randomness |
| `governor 0.8` | GCRA rate limiting |
| `argon2` | Password hashing |
| `aes-gcm` | Vault encryption |

---

## Security Advisories

### RUSTSEC-2026-0049 (rustls-webpki)

Fixed in v0.5.4. The `rustls-webpki` dependency was updated to address a certificate validation issue. No user action required -- the fix is included in the binary.

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

- [ ] Set `require_auth = true` if exposing externally
- [ ] Use a strong, random `api_key`
- [ ] Bind to `127.0.0.1` (not `0.0.0.0`) unless remote access is needed
- [ ] Review agent capability declarations (`agent.toml`)
- [ ] Enable approval gates for `shell_exec` and `file_write`
- [ ] Pin to a specific commit/tag for reproducibility
- [ ] Run `cargo audit` before building from source
- [ ] Use `RUST_LOG=warn` in production (avoid `debug` leaking secrets)
- [ ] Restrict `daemon.json` permissions (auto-set to 0600 on Unix)
- [ ] Use WASM runtime for untrusted agent code

See `docs/production-checklist.md` in the repository for the full deployment checklist.
