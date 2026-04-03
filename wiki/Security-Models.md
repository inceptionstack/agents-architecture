# Security Models

Security is one of the most differentiated dimensions across the 12 frameworks. This page provides a deep dive into sandboxing approaches, permission models, taint tracking, and network policies — and ranks all 12 from most to least secure.

Related: [Tool Systems](Tool-Systems.md), [Architecture Patterns](Architecture-Patterns.md), [Unique Innovations](Unique-Innovations.md).

---

## Security Dimensions

Each framework is evaluated on six dimensions:

1. **Sandbox isolation** — how tool/plugin code is isolated
2. **Credential protection** — how API keys and secrets are stored
3. **Network policy** — egress control
4. **Input validation** — injection and traversal protection
5. **Security testing** — depth of the security test suite
6. **Design philosophy** — secure by default vs. secure by configuration

---

## Sandbox Runtimes

| Sandbox Type | Mechanism | Overhead | Agent |
|-------------|-----------|----------|-------|
| **WASM (Wasmtime)** | Memory-isolated plugin with WIT capability interface | Microseconds | IronClaw, Moltis |
| **Docker container** | Linux namespace + cgroup isolation | Seconds startup | NanoClaw, Letta, ZeptoClaw, NemoClaw |
| **Apple Container** | macOS native container runtime | Low | NanoClaw, ZeptoClaw |
| **Bubblewrap** | Lightweight Linux namespaces (used by Flatpak) | Milliseconds | ZeptoClaw |
| **Firejail** | SUID sandbox with seccomp-bpf | Milliseconds | ZeptoClaw |
| **Landlock** | Linux kernel LSM for filesystem restrictions (kernel 5.13+) | Negligible | ZeptoClaw |
| **Apple Sandbox** | macOS sandbox profiles | Negligible | ZeptoClaw |
| **Native** | Direct OS process execution | None | ZeptoClaw, most others |

**ZeptoClaw's 6-runtime strategy** is unique: operators choose the isolation level appropriate for their deployment context. Production multi-tenant? Use Docker. Edge device with no container runtime? Use Landlock. macOS developer? Use Apple Sandbox.

**WASM** is the best isolation-to-overhead tradeoff for plugin code: no container spin-up, no privileged operations, formal capability interface. IronClaw even has a dedicated DB migration (`V2__wasm_secure_api.sql`) proving WASM security was designed-in from the start.

---

## Security Ranking: Most to Least Secure

### Tier 1: Security-First Design

#### 1. NemoClaw — Most Focused on Network Security
NemoClaw's entire security model is built around network access control with defense in depth:

- **Dual-layer network enforcement**: YAML policies enforced at both DNS resolution (DNS proxy) and TCP connection (gateway) layers
- **9 policy presets**: `strict`, `permissive`, `npm-registry`, `pypi`, `github`, etc. — least-privilege defaults
- **Credential exposure unit tests** AND end-to-end sanitization tests
- **Dedicated security test categories**: binary restriction, Dockerfile injection (C2 attack vectors), manifest path traversal (C4), method wildcard abuse
- **Sandbox hardening documentation** with step-by-step guides
- **Telegram injection prevention tests**
- **Coverage ratchet** in CI: test coverage cannot decrease

*Limitation:* Security is narrowly focused on network and credential domains. Tool execution security relies on Docker container isolation rather than fine-grained sandboxing.

#### 2. ZeptoClaw — Most Comprehensive Security Stack
ZeptoClaw has the broadest security coverage:

- **6 sandbox runtimes** (see above)
- **Safety taint tracking** — data tagged with provenance (user input, tool output, LLM response, external API); monitors flows to sensitive sinks
- **Leak detection** — detects when sensitive data is about to leave the system
- **Chain alerts** — monitors multi-step attack patterns (e.g., prompt injection → credential access → exfiltration)
- **Agent mode controls**: Safe / Standard / Unrestricted with explicit escalation required
- **AES-256-GCM credential encryption** with per-operation random nonces
- **Path restriction enforcement** with symlink validation
- **Shell security module** (`src/security/`)
- **Input sanitization** at safety layer boundary
- **Rate limiting** at gateway
- **Constant-time token comparison** (prevents timing attacks)
- **Agent budget enforcement** — per-session token/cost hard limits

*Limitation:* React control panel uses localStorage for tokens (XSS risk). No HSM or OS keychain integration.

#### 3. IronClaw — Best Isolation Architecture
- **WASM plugin sandboxing** via Wasmtime — each plugin is memory-isolated
- **Dedicated `ironclaw_safety` crate** — compiled Rust safety rule engine (not a script)
- **Multi-tenant identity scope isolation** with database-enforced scope boundaries
- **OS keychain integration** (`secrets/keychain.rs`) for encrypted secret storage
- **TLS for database connections**
- **`cargo-fuzz` harness** — fuzzing for edge cases in tool parameter parsing
- **WASM Secure API migration** — WASM security state in DB schema
- **LLM trace fixture replay** — deterministic testing without live LLM calls

*Limitation:* WASM host functions must be carefully scoped; overly broad WIT interfaces still allow data exfiltration through allowed channels.

---

### Tier 2: Strong Security with Gaps

#### 4. Hermes Agent — Best Security Test Coverage
- **18+ dedicated security test files**: SQL injection, SSRF, command injection, path traversal, symlink attacks, prompt injection, PII redaction, credential redaction, `force_dangerous_override`, `yolo_mode`, write deny, cron prompt injection
- **`tirith_security.py`** — named security policy enforcement layer
- **Skills guard** (`skills_guard.py`) — validates skills before execution
- **URL safety + SSRF checks**
- **Command guards** and **file read/write guards**
- **Credential redaction** in logs and outputs
- **Env var blocklist** for local environment protection
- **Supply chain audit** GitHub Action
- **Nix reproducible builds** — deterministic build environment

*Limitation:* Local credentials stored unencrypted in YAML files. No OS keychain integration. `.envrc` creates credential leak risk in dev environments. "Yolo mode" exists as an explicit safety bypass.

#### 5. MicroClaw — Good Operational Security
- **Hook-based tool interception** — `block-bash` hook can completely disable shell access; `redact-tool-output` sanitizes sensitive data
- **`tool_permissions.rs`** integration test
- **Config validation** tests
- **`cargo-deny`** for supply chain security
- **Nix reproducible builds**

*Limitation:* Security is opt-in via hooks rather than default-secure. No dedicated encryption module visible. Hook scripts (shell scripts) can themselves be modified if the filesystem is writable.

#### 6. Moltis — Solid Foundation
- **Dedicated `moltis-vault` crate** with encryption
- **`moltis-auth` + `moltis-oauth`** as separate crates (clean separation)
- **`moltis-network-filter` crate** for egress control
- **Tailscale integration** for network-level security
- **`zizmor`** GitHub Actions security scanner

*Limitation:* 50+ crates increases attack surface. Security assessment limited by architectural complexity. Comprehensive audit would require crate-by-crate review.

---

### Tier 3: Adequate Security

#### 7. NanoClaw — Good Container Isolation
- **Container-per-task** — each task gets a fresh container; no state leakage
- **Sender allowlist** — only approved channel identities can trigger tasks
- **Mount allowlist** — filesystem access controlled via mount configuration
- **IPC authentication** between host daemon and container agents

*Limitation:* TypeScript/Node.js lacks Rust's compile-time memory safety. Mount allowlist bypass possible with misconfigured Docker. No encryption for stored credentials.

#### 8. Letta — Enterprise Features but Historical Weaknesses
- **Org/user scoping** for multi-tenancy
- **BYOK encrypted columns** for credential storage (post-backfill migration)
- **Docker/Modal sandbox** for tool execution
- **MCP OAuth** with encrypted storage

*Limitation:* TLS private key committed to repo (`certs/localhost-key.pem`). Historical plaintext credential storage (pre-migration). API tokens table dropped in OSS version. Simple single-password auth model.

#### 9. PicoClaw — Reasonable but Limited
- **OAuth + PKCE** for provider authentication
- **`pkg/credential/`** encrypted credential store
- **Config security module**
- **Workspace-scoped execution**

*Limitation:* JSONL persistence has no built-in access control. No sandbox isolation for tool execution (all in-process). No dedicated security test suite.

---

### Tier 4: Minimal Security

#### 10. Nanobot — Basic Auth Only
- OAuth for MCP server authentication
- MCP audit logging (`pkg/mcp/auditlogs/`)
- MCP sandboxing module

*Limitation:* Minimal visible security infrastructure. No dedicated security crate or test suite. Go standard library without extensive hardening.

#### 11. OpenFang — Audit Config but Limited Implementation
- **`cargo audit` + `.cargo/audit.toml`** for dependency scanning
- **Tauri capability definitions** for desktop app permissions

*Limitation:* Minimal security testing. No dedicated security module. 48-file channels crate with only 1 test file. Significant untested surface area.

#### 12. CoPaw — Most Vulnerable
- Skill scanner + tool_guard (positive)
- Approval workflows (positive)

**Critical issues:**
- `COPAW_AUTH_ENABLED` **commented out** in docker-compose — deployed instances are publicly accessible by default
- Private base image from Alibaba Cloud Registry (`agentscope-registry.ap-southeast-1.cr.aliyuncs.com`) — not independently auditable
- Chromium `--no-sandbox` in container
- No rate limiting, no RBAC, no MFA, no session management, no CORS config visible
- No token revocation mechanism

---

## Permission Models

| Agent | Permission Model | Granularity |
|-------|-----------------|-------------|
| ZeptoClaw | Agent modes (Safe/Standard/Unrestricted) | Per-agent mode |
| NemoClaw | YAML network policies | Per-domain/protocol |
| IronClaw | Identity scope + safety rule engine | Per-tenant |
| Letta | Org/user scoping | Per-org |
| NanoClaw | Sender + mount allowlists | Per-sender, per-path |
| MicroClaw | Hook-based (block-bash, redact) | Per-tool |
| Hermes Agent | Skills guard + command guards | Per-tool |
| Moltis | Vault + auth + network-filter | Per-crate feature |
| Nanobot | MCP OAuth | Per-MCP-server |
| PicoClaw | OAuth + PKCE | Per-provider |
| OpenFang | Tauri capabilities | Per-desktop-permission |
| CoPaw | Skill scanner (no auth by default) | Pre-execution only |

**No framework implements full RBAC.** Authorization is typically workspace/org scoping or bearer token checks. This is a significant gap for production multi-tenant deployments where different users should have different tool access levels.

---

## Taint Tracking (ZeptoClaw)

ZeptoClaw's `src/safety/` module implements compiler-inspired taint tracking:

1. All data entering the agent is tagged with its **source label**: `UserInput`, `ToolOutput`, `LLMResponse`, `ExternalAPI`
2. As data flows through the pipeline, taint propagates (if tainted data is used to construct a shell command, the command is tainted)
3. The safety layer checks taint at **sensitive sinks**: shell execution, credential access, network egress
4. **Leak detection** monitors for tainted data reaching output channels
5. **Chain alerts** fire when a sequence of operations matches a known attack pattern (e.g., jailbreak attempt → credential tool call)

This technique is borrowed from compiler security analysis (LLVM's taint analysis) and web security (OWASP taint analysis). It's the most sophisticated runtime safety mechanism across all 12 frameworks.

---

## Surprising Security Findings

1. **CoPaw auth disabled by default** — deployed instances are public. Combined with Chromium `--no-sandbox`, this is a critical risk.
2. **Letta TLS private key in git** — `certs/localhost-key.pem` should never be committed.
3. **Hermes "yolo mode"** — explicit bypass of all safety guardrails. `test_yolo_mode.py` documents this exists.
4. **No RBAC in any framework** — authorization gaps for multi-user production deployments.
5. **NemoClaw's coverage ratchet** — only framework that prevents test coverage regression in CI.

---

*Back to [Home](Home.md) | See also: [Tool Systems](Tool-Systems.md) | [Unique Innovations](Unique-Innovations.md)*
