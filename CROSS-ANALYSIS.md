# Cross-Analysis: 12 AI Agent Frameworks

**Date:** 2026-04-03
**Audience:** Solutions Architects
**Scope:** CoPaw, Hermes Agent, IronClaw, Letta, MicroClaw, Moltis, NanoClaw, Nanobot, NemoClaw, OpenFang, PicoClaw, ZeptoClaw

---

## 1. Comparison Table

| Agent | Language | Architecture Pattern | Tool System | Memory/State | Security Model | LLM Integration | Unique Feature |
|-------|----------|---------------------|-------------|-------------|---------------|-----------------|----------------|
| **CoPaw** | Python (FastAPI) | Hybrid Multi-Tier + Plugin | Function-calling tools in `agents/tools/`, skill scanner pre-validates skills | SQLAlchemy ORM (Postgres/SQLite), conversation memory module, agent memory hooks | Skill scanner (static analysis), tool_guard (runtime), approval workflows, workspace isolation | 13 provider adapters (OpenAI, Anthropic, Google, local), AgentScope orchestration runtime | AgentScope runtime as core orchestration engine; Alibaba Cloud ecosystem integration (ModelScope, private base image); Ollama as default Docker deployment |
| **Hermes Agent** | Python (FastAPI) | ReAct Agent Loop + Event-Driven Gateway | Central `registry.py` maps tool names → handlers; 40+ tools including browser, MCP, delegation, RL training | SQLite sessions, Honcho memory service integration, persistent memory tool, session search | `tirith_security.py` (named policy layer), `skills_guard.py`, URL safety/SSRF checks, command guards, file read/write guards, credential redaction, PII redaction, prompt injection tests | Multi-model parsers per LLM (DeepSeek, Qwen, Kimi, GLM, Mistral, Llama, Hermes), credential pool with rotation, smart model routing | Per-model tool call parsers (12+ parsers); mixture-of-agents tool; RL training tool; CamoFox (camouflaged browser); most extensive security test suite of all 12 |
| **IronClaw** | Rust (Tokio/Axum) | Plugin (WASM) + Worker/Orchestrator | WASM-sandboxed tool plugins compiled to WebAssembly via WIT interfaces; built-in tools + MCP bridge | PostgreSQL + libSQL, vector embeddings, conversation history, document chunking | WASM sandbox isolation (Wasmtime), safety crate (`ironclaw_safety`), multi-tenant identity scope isolation, encrypted keychain secrets, Docker sandbox | Anthropic, OpenAI, Gemini, Bedrock, GitHub Copilot, NearAI, Codex; OAuth PKCE per provider, token rotation | WASM-based extension system with WIT interface types; dedicated safety crate with rule engine; fuzzing harness (`cargo-fuzz`); LLM trace fixture replay testing |
| **Letta** | Python (FastAPI) | Layered SOA + Service Manager | Tool sandbox execution (Docker, Modal, local subprocess); tool schema generator from Python functions; MCP client | PostgreSQL + pgvector (vector similarity), SQLAlchemy ORM, Alembic migrations, archival memory, memory blocks with history, summarizer pipeline | Bearer token auth, org/user scoping, sandbox isolation (Docker/Modal), encrypted BYOK columns, MCP OAuth | LiteLLM multi-provider routing, OpenAI/Anthropic/Google/Bedrock/Together/Groq/etc., OpenAI-compatible API surface | Persistent agent memory with summarization; multi-agent orchestration (round-robin, supervisor, sleeptime patterns); pgvector for archival memory search; versioned agent implementations (v1/v2/v3); ephemeral + voice agents |
| **MicroClaw** | Rust (workspace crates) | Modular Monolith + Plugin + Event-Driven | Tool trait in `microclaw-tools` crate, tool registry in `src/tools/mod.rs`, hook pipeline intercepts all tool calls | `microclaw-storage` crate (SQLite), memory service + backend, structured memory, vector embeddings | Hook-based interception (`hooks/block-bash/`, `hooks/redact-tool-output/`), tool permissions test, config validation | LLM abstraction in `src/llm.rs`, MCP client, A2A protocol, ACP subagent protocol | Hook system for AOP-style tool interception (block-bash, redact-output, filter-memory); ClawHub skill marketplace; A2A + ACP multi-agent protocols; Nix reproducible builds |
| **Moltis** | Rust (50+ crates) | Modular Monolith / Microkernel + Event-Driven | WASM plugin tools via Wasmtime + WIT interfaces, built-in tools crate | SQLite via sqlx, memory crate, sessions crate, projects crate | Vault crate (encrypted secrets), auth crate, OAuth crate, network-filter crate, Tailscale integration | Providers crate with OpenAI/Anthropic/local/GGUF support | Native iOS (Swift/Apollo GraphQL) and macOS apps via Rust↔Swift FFI bridge (cbindgen); GraphQL API (async-graphql); CalDAV integration; most crates of any framework (50+); typed broadcast events |
| **NanoClaw** | TypeScript (Node.js) | Event-Driven Multi-Channel Gateway | Skills as Claude AI capability plugins in `.claude/skills/`; container-side skills for browser, formatting | SQLite (better-sqlite3), task history in DB, group state | Container-per-task sandboxing (Docker/Apple Container), sender allowlist, mount allowlist (filesystem access control), IPC authentication | Anthropic Claude via Claude CLI inside containers; MCP configuration | Container-per-task isolation (each task gets a fresh container); Apple Container (macOS native) support; IPC auth between host daemon and container agents; launchd service integration |
| **Nanobot** | Go | Layered + Plugin via MCP | MCP servers as plugins — tools discovered via MCP protocol; built-in servers (agent, system, skills, workflows, tasks, artifacts) | SQLite via GORM, session store with migrations, conversation compaction/truncation | OAuth for MCP server auth, MCP audit logging, MCP sandboxing | OpenAI (Completions + Responses APIs) and Anthropic Claude; unified `complete.go` orchestrator | MCP-native architecture (MCP host as core design); agent-as-MCP-server pattern; YAML config with hot-reload (fswatch); GoReleaser for cross-platform releases; expression evaluation engine |
| **NemoClaw** | JavaScript/Node.js + TypeScript | CLI + Plugin + Policy Engine + Container Runtime | OpenClaw plugin system; tools via Claude inside sandbox; NIM inference images | No persistent agent memory — session-based sandbox state | **Network policy-based PBAC** (YAML policies with presets), credential exposure testing, binary restriction enforcement, Dockerfile injection prevention, path traversal prevention, DNS proxy enforcement layer | Claude AI inside sandbox; NVIDIA NIM for local GPU inference | Network policy engine with 9 presets (strict, permissive, npm, pypi, github); dual-layer network enforcement (DNS proxy + gateway); most security-focused test suite; sandbox hardening documentation; credential sanitization E2E tests |
| **OpenFang** | Rust (workspace crates) | Layered Modular Monolith + Event-Driven | Skills (60+ bundled), Hands (autonomous actions), Extensions (25 integrations); driver-based tool dispatch | Custom memory crate, wire protocol for serialization | `cargo audit` config, Tauri capability definitions for desktop permissions | Driver pattern in `openfang-runtime/src/drivers/` for Vertex AI, OpenAI, Anthropic; wire protocol | 30+ pre-built agent configurations (orchestrator, researcher, etc.); Tauri desktop app; "Hands" concept (autonomous agent actions vs. passive tools); Python + JS SDKs; LangChain agent integration |
| **PicoClaw** | Go | Plugin/Registry + Event-Driven Gateway | Tool registry pattern (`pkg/tools/registry.go`), 20+ built-in tools, MCP tools, spawn/subagent tools | JSONL-based persistent memory, JSONL session backend, BM25 text search | OAuth + PKCE auth, encrypted credential store, config security module, workspace-scoped execution | Claude, OpenAI, Bedrock, Azure, Copilot, Codex; factory pattern for provider creation | Hardware device integration (I2C, SPI, embedded devices); ASR/TTS audio subsystem (12+ ASR files); BM25 search in memory; internal event bus (`pkg/bus/`); JSONL-based persistence (no external DB) |
| **ZeptoClaw** | Rust | Plugin-Based Modular Monolith + Event-Driven | Tool registry, 30+ tool implementations, MCP tools, hardware tools, Android tools, plugin-based channel tools | BM25 + HNSW vector search, embedding-based memory, long-term storage, memory hygiene/snapshots | **6 sandbox runtimes** (Docker, Bubblewrap, Firejail, Landlock, Apple Sandbox, native), safety taint tracking, leak detection, chain alerts, input sanitization, agent mode controls (Safe/Standard/Unrestricted), AES-256-GCM credential encryption | Claude, OpenAI, Gemini, Vertex; provider rotation, fallback, retry, cooldown, quota management | Most sandbox runtime options (6); safety taint tracking + leak detection; hardware/peripheral integrations (ESP32, Arduino, RPi, USB); MQTT + serial channels; multi-tenant Docker deployment; agent budget enforcement; R8r bridge |

---

## 2. Top 10 Most Interesting Architectural Insights

### 1. WASM-Based Tool Sandboxing (IronClaw, Moltis)
IronClaw and Moltis use **WebAssembly** (via Wasmtime) with **WIT (WebAssembly Interface Types)** to sandbox tool and channel plugins. This is architecturally significant because WASM provides:
- **Memory isolation** — each plugin gets its own linear memory space
- **Capability-based security** — plugins can only access what the WIT interface exposes
- **Language agnosticism** — tools can be written in any WASM-targeting language
- **No container overhead** — microsecond startup vs. seconds for Docker

IronClaw has a dedicated database migration (`V2__wasm_secure_api.sql`) specifically for WASM security state, proving this was designed-in rather than bolted on.

### 2. Per-Model Tool Call Parsers (Hermes Agent)
Hermes Agent maintains **12+ dedicated parsers** for extracting tool calls from different LLM outputs (DeepSeek V3, DeepSeek V3.1, Qwen, Qwen3-Coder, Kimi K2, Mistral, Llama, GLM-4.5, GLM-4.7, Hermes/NousResearch, Longcat). This solves a real production problem: each model has subtly different tool call formatting, and a single generic parser would silently fail on edge cases. This is the most model-diverse tool call parsing system across all 12 frameworks.

### 3. Six Sandbox Runtimes in One System (ZeptoClaw)
ZeptoClaw implements **six distinct sandbox runtimes**: Docker, Bubblewrap, Firejail, Landlock, Apple Sandbox, and native. Each uses different OS-level isolation primitives:
- **Docker** — container-level isolation (heaviest)
- **Bubblewrap** — lightweight Linux namespaces
- **Firejail** — SUID sandbox with seccomp-bpf
- **Landlock** — Linux kernel LSM for filesystem restrictions
- **Apple Sandbox** — macOS sandbox profiles
- **Native** — direct process execution (least isolated)

This strategy pattern lets operators choose the security/performance tradeoff appropriate for their deployment.

### 4. Network Policy Engine with DNS Proxy Enforcement (NemoClaw)
NemoClaw implements a **dual-layer network enforcement** system: YAML-based network policies define allowed egress destinations, enforced at both the **DNS resolution layer** (DNS proxy blocks resolution of non-allowlisted domains) and the **network gateway layer** (TCP connections blocked). This defense-in-depth approach means even if one layer is bypassed, the other catches it. The 9 policy presets (strict, permissive, npm-registry, pypi, github, etc.) make it operationally practical.

### 5. Safety Taint Tracking (ZeptoClaw)
ZeptoClaw's `src/safety/` module implements **taint tracking** — data flowing through the agent is tagged with provenance (user input, tool output, LLM response, external API), and the safety layer can detect when tainted data flows to sensitive sinks (credentials, shell execution, network egress). This is a technique borrowed from web application security (and compiler analysis) applied to LLM agent safety. It also includes **leak detection** and **chain alerts** — monitoring for multi-step attack patterns.

### 6. Persistent Agent Memory with Summarization Pipeline (Letta)
Letta's architecture treats memory as a first-class concern with three tiers:
- **In-context memory blocks** (actively in the prompt)
- **Conversational memory** (recent messages)
- **Archival memory** (vector-indexed in pgvector)

When the context window fills, the **summarizer pipeline** automatically compresses older messages while preserving key information. Memory blocks have **history tracking** (`block_history.py`), enabling temporal queries ("what did the agent know at time T?"). This is the most sophisticated memory system across all 12 frameworks.

### 7. Container-Per-Task Isolation with IPC Auth (NanoClaw)
NanoClaw runs each AI task in a **fresh Docker or Apple Container**, communicating via authenticated IPC. The host daemon and container agent are separate processes connected by an IPC channel with explicit authentication (`ipc-auth.ts`). This provides:
- **Blast radius containment** — a compromised task can't affect others
- **Clean-slate execution** — no state leakage between tasks
- **Platform abstraction** — Docker on Linux, Apple Container on macOS

The sender allowlist and mount allowlist add additional access control layers.

### 8. Hook System for AOP-Style Tool Interception (MicroClaw)
MicroClaw's hook system (`src/hooks.rs` + `hooks/` directory) lets operators install script-based interceptors that run before/after every tool call. Built-in hooks include:
- `block-bash/` — prevents bash tool execution entirely
- `redact-tool-output/` — sanitizes sensitive data from tool output
- `filter-global-structured-memory/` — controls memory access

This is essentially **Aspect-Oriented Programming** for agent tool calls, enabling security policies to be expressed as composable, testable scripts rather than embedded in tool code.

### 9. Rust↔Swift FFI Bridge for Native iOS/macOS (Moltis)
Moltis bridges its Rust backend to native Swift iOS/macOS apps via `cbindgen` (C bindings generated from Rust). The iOS app uses Apollo GraphQL client talking to the Rust GraphQL API layer (async-graphql). This is architecturally unique among the 12 — no other framework provides native mobile apps backed by a systems language runtime. The approach avoids the Electron/WebView compromise while keeping a single Rust core.

### 10. Agent-as-MCP-Server Pattern (Nanobot)
Nanobot uses MCP as its **fundamental architectural primitive** — agents themselves can be exposed as MCP servers (`pkg/servers/agent/`), making the system recursively composable. An agent can call another agent through the same MCP protocol used for external tools. Combined with hot-reloadable YAML configuration (`pkg/fswatch/`) and the expression evaluation engine (`pkg/expr/`), this creates a highly dynamic system where agent capabilities can be reconfigured at runtime without restart.

---

## 3. Common Patterns

### Shared by All or Nearly All (10+/12)

1. **Multi-Channel Messaging Support** — Every framework supports at minimum Telegram, Slack, and Discord. Most support 10+ channels including WhatsApp, DingTalk, Feishu/Lark, Email, Matrix, and more. This is clearly a table-stakes requirement for the domain.

2. **MCP (Model Context Protocol) Integration** — 11/12 frameworks implement MCP client or server capabilities. MCP has become the de facto standard for agent tool extensibility.

3. **Multi-Provider LLM Abstraction** — All 12 support multiple LLM providers behind an abstraction layer. Anthropic Claude and OpenAI are universally supported. The provider abstraction follows either a Strategy pattern (Letta, IronClaw) or a factory pattern (PicoClaw, OpenFang).

4. **Skills/Plugin System** — All 12 have an extensible capability system (called "skills", "tools", "plugins", or "extensions"). These are typically self-contained directories with manifests, scripts, and reference documentation.

5. **CLI Entry Point** — Every framework provides a command-line interface as a primary entry point. Most also provide web UIs and API servers.

6. **Docker Containerization** — All 12 use Docker for deployment. Docker Compose is the standard for local development orchestration.

7. **Tool Call / Function Calling Pattern** — All frameworks implement the LLM tool-calling loop (prompt → LLM → tool call → execute → feed result back → repeat).

8. **Approval/Confirmation Gates** — 10/12 have some form of human-in-the-loop approval before executing sensitive operations (CoPaw's `approvals/`, Hermes's `approval.py`, IronClaw's safety layer, ZeptoClaw's approval tools, etc.).

9. **Credential/Secret Management** — All frameworks handle API keys for LLM providers. The more mature ones (IronClaw, Moltis, ZeptoClaw) use encrypted stores; others rely on environment variables.

10. **Session/Conversation Persistence** — All persist conversation history in some form (SQLite being the most common local option, PostgreSQL for production deployments).

### Shared by Most (7-9/12)

- **Cron/Scheduled Task Execution** — CoPaw, Hermes, MicroClaw, Moltis, PicoClaw, ZeptoClaw, NanoClaw all support scheduled agent execution.
- **Memory/Context Compression** — Letta, Nanobot, Hermes, PicoClaw, ZeptoClaw implement automatic context window management (summarization, compaction, truncation).
- **OAuth 2.0 + PKCE for Provider Auth** — IronClaw, Hermes, Moltis, PicoClaw, ZeptoClaw, Letta implement PKCE-protected OAuth flows for LLM provider authentication.
- **Sub-Agent/Delegation** — Hermes, MicroClaw, Letta, PicoClaw, ZeptoClaw, NanoClaw, Nanobot support spawning sub-agents for task decomposition.

### Language Split

- **Rust**: IronClaw, MicroClaw, Moltis, OpenFang, ZeptoClaw (5/12 — 42%)
- **Python**: CoPaw, Hermes Agent, Letta (3/12 — 25%)
- **Go**: Nanobot, PicoClaw (2/12 — 17%)
- **TypeScript/Node.js**: NanoClaw, NemoClaw (2/12 — 17%)

The Rust dominance is notable — it suggests the community values memory safety, performance, and strong type systems for agent infrastructure. The Rust frameworks also tend to have the most sophisticated security models.

---

## 4. Unique Approaches by Agent

### CoPaw
- **AgentScope Runtime dependency** — uniquely builds on a third-party agent orchestration runtime (`agentscope==1.0.18`) rather than building its own agent loop. This is a double-edged sword: faster time-to-market but vendor lock-in.
- **Alibaba Cloud ecosystem** — base Docker image from `agentscope-registry.ap-southeast-1.cr.aliyuncs.com`, ModelScope integration, Chinese font support, WeChat/QQ/DingTalk channels. Most China-ecosystem-integrated of all 12.
- **Desktop app via NSIS** — Windows installer build alongside Docker, targeting a consumer/prosumer audience.

### Hermes Agent
- **Mixture-of-Agents Tool** (`mixture_of_agents_tool.py`) — orchestrates multiple LLMs simultaneously for a single query, aggregating their outputs.
- **CamoFox** — a camouflaged Firefox browser instance (`browser_camofox.py`) with persistent state for stealthy web automation. Unique across all 12.
- **RL Training Tool** — agents can trigger reinforcement learning training runs, making this the only framework with ML training integration.
- **Nix-based reproducible builds** (`flake.nix`) alongside standard Python packaging.
- **Most extensive security test suite** — 18+ dedicated security test files covering SQL injection, SSRF, command injection, path traversal, symlink attacks, prompt injection, PII redaction, and more.

### IronClaw
- **WASM extension system with WIT** — the most architecturally mature plugin sandbox. Tools and channels compile to WebAssembly with formally-defined interfaces.
- **LLM trace fixture replay** — recorded LLM interactions stored as fixtures in `tests/fixtures/llm_traces/` and replayed in tests. This enables deterministic testing of agent behavior without live LLM calls.
- **Fuzzing harness** — `cargo-fuzz` integration for discovering edge cases in tool parameter parsing.
- **OpenClaw import** — data migration from another agent framework, suggesting a competitive ecosystem.

### Letta
- **Multi-agent orchestration patterns** — round-robin, supervisor, and sleeptime (background memory consolidation) patterns in `letta/groups/`. The "sleeptime" pattern is novel: agents consolidate memory during idle periods.
- **Versioned agent implementations** (v1, v2, v3) — maintains backward-compatible agent versions, enabling gradual migration.
- **Ephemeral agents** — stateless agents that don't persist memory, useful for one-shot tasks without storage overhead.
- **Voice agents** — dedicated voice agent variants optimized for audio I/O.
- **Credit verification service** — built-in usage quota/credit tracking, suggesting SaaS deployment intent.
- **pgvector for archival memory** — production-grade vector similarity search for long-term agent memory.

### MicroClaw
- **Hook-based tool interception** — the most developer-friendly approach to tool security policy. Hooks are shell scripts, not Rust code, making them accessible to operators without Rust expertise.
- **A2A + ACP protocols** — supports both Agent-to-Agent (HTTP-based) and Agent Communication Protocol (stdio-based) for inter-agent communication. Most protocol-diverse multi-agent system.
- **Nostr channel** — decentralized social protocol adapter. Unique among all 12.
- **ClawHub marketplace** — centralized skill marketplace with CLI and API integration.

### Moltis
- **50+ Rust crates** — the most modular architecture. Each feature is an independent crate with its own `Cargo.toml`, enabling precise dependency management and compile-time feature gating.
- **GraphQL API** (async-graphql) — the only framework using GraphQL instead of REST. Includes subscriptions for real-time streaming.
- **CalDAV integration** — calendar protocol support for scheduling-aware agents.
- **Native iOS/macOS apps** via Rust→Swift FFI bridge.
- **Typed broadcast events** — event bus with compile-time typed event enums (per design doc `plans/2026-02-15-typed-broadcast-event-enum.md`).
- **Most deployment targets** — Docker, Fly.io, Railway, Render, DigitalOcean, Flatpak, Snap, Homebrew, AUR, RPM, Nix.

### NanoClaw
- **Container-per-task model** — each Claude task gets its own fresh container. Simplest and most isolated execution model.
- **Apple Container support** — native macOS containerization alongside Docker.
- **launchd service integration** — runs as a macOS daemon via `.plist`, targeting the macOS developer audience.
- **Claude-first design** — unlike others that abstract across providers, NanoClaw is designed specifically around Claude CLI inside containers.
- **Group-based routing** — messages are routed to groups (not just agents), each with their own CLAUDE.md configuration.

### Nanobot
- **MCP-native architecture** — MCP is not a bolt-on integration but the fundamental design primitive. The entire plugin system is built on MCP servers.
- **Agent-as-MCP-server** — agents expose themselves as MCP servers, enabling recursive agent composition.
- **YAML+frontmatter configuration** — agent configurations are YAML files with optional Markdown frontmatter.
- **Expression evaluation engine** (`pkg/expr/`) — custom expression language for dynamic configuration.
- **Go implementation** — one of only two Go-based frameworks, optimized for small binary size and fast startup.

### NemoClaw
- **Network policy engine** — the only framework where network access control is the primary security primitive (rather than process/file isolation).
- **Policy presets** (9 YAML files) — pre-approved network access profiles composable by operators.
- **DNS proxy enforcement** — network policies enforced at the DNS resolution layer as a second enforcement point.
- **Dedicated security test categories** — binaries restriction, Dockerfile injection (C2), manifest traversal (C4), method wildcard abuse.
- **AI agent skill definitions** (`.agents/skills/`) — guidance documents that teach AI agents how to use NemoClaw itself.

### OpenFang
- **"Hands" concept** — autonomous agent actions (`openfang-hands/`) distinct from passive tools. Hands can include browser automation, trading, and other autonomous behaviors.
- **30+ pre-built agent configurations** — the largest library of ready-to-use agent personas (orchestrator, researcher, hello-world, etc.).
- **Tauri desktop app** — native desktop GUI alongside CLI, API, and TUI.
- **Python + JavaScript SDKs** — dual client SDK support.
- **Wire protocol crate** — dedicated serialization format for inter-component communication.

### PicoClaw
- **Hardware device integration** — I2C, SPI, embedded device support via `pkg/devices/`. The only framework targeting IoT/embedded scenarios.
- **ASR/TTS subsystem** — 12+ ASR files, 6 TTS files with multiple provider backends. Most comprehensive audio pipeline.
- **JSONL-based persistence** — uses JSONL files (not SQLite) for memory and session storage. Zero-dependency persistence.
- **BM25 search in Go** — custom BM25 implementation for text search over agent memory.
- **Event bus architecture** — `pkg/bus/` provides application-wide pub/sub, with per-agent event buses (`pkg/agent/eventbus.go`).
- **Launcher TUI** — separate terminal UI application for configuration and launch (`cmd/picoclaw-launcher-tui/`).

### ZeptoClaw
- **Safety taint tracking** — data provenance tracking through the agent pipeline, detecting when tainted inputs reach sensitive sinks.
- **6 sandbox runtimes** — Docker, Bubblewrap, Firejail, Landlock, Apple Sandbox, native. Most options of any framework.
- **Agent budget enforcement** — per-session and per-agent token/cost budgets with hard enforcement.
- **R8r bridge** — a relay/router bridge component for approval, deduplication, events, and health monitoring.
- **Android tools** — direct Android device interaction tools.
- **MQTT + Serial channels** — IoT protocol and serial port communication channels.
- **Memory hygiene + snapshots** — automated memory cleanup and point-in-time snapshots.
- **Multi-tenant Docker deployments** — dedicated compose files for multi-tenant SaaS operation.

---

## 5. Security Model Deep-Dive

### Ranking: Most to Least Secure

#### Tier 1: Security-First Design

**1. NemoClaw** — Most Security-Focused
- Network policy-based access control with DNS proxy enforcement (dual-layer)
- Dedicated security test categories: binary restriction, Dockerfile injection (C2), manifest traversal (C4), method wildcards
- Credential exposure unit tests AND E2E sanitization tests
- Sandbox hardening documentation
- Telegram injection prevention tests
- Policy presets for least-privilege network access
- **Limitation:** Focused narrowly on network/credential security; limited tool execution security beyond container isolation.

**2. ZeptoClaw** — Most Comprehensive Security Stack
- 6 sandbox runtimes (Docker, Bubblewrap, Firejail, Landlock, Apple Sandbox, native)
- Safety taint tracking + leak detection + chain alerts
- Agent mode controls (Safe/Standard/Unrestricted) with explicit escalation
- AES-256-GCM encrypted credential store with per-operation random nonces
- Path restriction enforcement with symlink validation
- Shell security module
- Input sanitization at safety layer
- Rate limiting at gateway layer
- Constant-time token comparison
- **Limitation:** localStorage for control panel tokens (XSS risk); no HSM/OS keychain integration.

**3. IronClaw** — Best Isolation Architecture
- WASM sandboxing via Wasmtime provides memory-safe plugin isolation without container overhead
- Dedicated `ironclaw_safety` crate as a compiled safety rule engine
- Multi-tenant identity scope isolation with database-enforced scope boundaries
- Encrypted secrets via OS keychain integration (`secrets/keychain.rs`)
- TLS for database connections
- Fuzzing harness for input validation
- WASM Secure API database migration
- **Limitation:** WASM extensions could still exfiltrate data through allowed host functions if WIT interfaces are too broad.

#### Tier 2: Strong Security with Gaps

**4. Hermes Agent** — Best Security Test Coverage
- 18+ dedicated security test files (SQL injection, SSRF, command injection, path traversal, symlink attacks, prompt injection, PII redaction, force dangerous override, yolo mode, write deny, cron prompt injection)
- `tirith_security.py` — named security policy enforcement
- Skills guard, URL safety, website policy, command guards, file read/write guards
- Credential redaction in logs
- Env var blocklist for local environments
- Supply chain audit GitHub Action
- **Limitation:** Unencrypted local credential storage (YAML files); no OS keychain integration; `.envrc` credential leak risk.

**5. MicroClaw** — Good Operational Security
- Hook-based tool interception enables runtime security policies
- `tool_permissions.rs` integration test
- Config validation tests
- `cargo-deny` for supply chain security
- Redact-tool-output hook
- Block-bash hook (can completely disable shell access)
- **Limitation:** Security is hook-based (opt-in) rather than default-secure; no dedicated encryption module visible.

**6. Moltis** — Solid Foundation
- Dedicated vault crate with encryption
- Auth crate + OAuth crate (separate concerns)
- Network-filter crate for egress control
- Tailscale integration for network security
- `zizmor` GitHub Actions security scanner
- **Limitation:** Complex architecture (50+ crates) increases attack surface; security assessment limited by architectural complexity.

#### Tier 3: Adequate Security

**7. NanoClaw** — Good Container Isolation
- Container-per-task model limits blast radius
- Sender allowlist + mount allowlist
- IPC authentication between host and container
- Fresh container per task (no state leakage)
- **Limitation:** TypeScript/Node.js has weaker memory safety than Rust; mount allowlist bypass possible if Docker is misconfigured.

**8. Letta** — Enterprise Features but Historical Weaknesses
- Organization/user scoping for multi-tenancy
- Encrypted BYOK columns (migrated from plaintext — `8149a781ac1b` backfill)
- Docker/Modal sandbox for tool execution
- MCP OAuth with encrypted storage
- **Limitation:** API tokens table dropped in OSS version; historical plaintext credential storage; TLS private key committed to repo (`certs/localhost-key.pem`); simple single-password auth model.

**9. PicoClaw** — Reasonable but Limited
- OAuth + PKCE for provider auth
- Encrypted credential store (`pkg/credential/`)
- Config security module
- Workspace-scoped execution
- **Limitation:** JSONL-based persistence has no built-in access control; no sandbox isolation for tool execution.

#### Tier 4: Minimal Security

**10. Nanobot** — Basic Auth Only
- OAuth for MCP server auth
- MCP audit logging
- MCP sandboxing module
- **Limitation:** Minimal visible security infrastructure; Go standard library without extensive hardening; no dedicated security crate or test suite.

**11. OpenFang** — Audit Config but Limited Implementation
- `cargo audit` + `.cargo/audit.toml` for dependency scanning
- Tauri capability definitions for desktop permissions
- **Limitation:** Minimal visible security testing; no dedicated security module; channels crate (48 files) with only 1 test file.

**12. CoPaw** — Most Vulnerable
- Skill scanner + tool guard (positive)
- Approval workflows (positive)
- **However:** Authentication disabled by default (`COPAW_AUTH_ENABLED` commented out); private base image from Alibaba Cloud Registry (supply chain risk); Chromium `--no-sandbox` in container; no rate limiting; no RBAC; no MFA; no session management; no password policy; no CORS configuration visible; no token revocation mechanism; tunnel authentication unknown.

---

## 6. "Best of Breed" Agent Design

If we could combine the best architectural ideas from each framework, the ideal agent would look like:

### Core Runtime
- **Rust workspace** (from IronClaw/MicroClaw/ZeptoClaw) — memory safety, strong typing, fearless concurrency
- **Agent loop** from Hermes (ReAct pattern with per-model tool call parsers) — production-proven with 12+ model formats
- **Persistent memory architecture** from Letta — three-tier memory (in-context, conversational, archival with pgvector) with automatic summarization

### Extension System
- **WASM plugin sandbox** from IronClaw/Moltis — memory-isolated, capability-based, language-agnostic plugins via WIT interfaces
- **Hook pipeline** from MicroClaw — AOP-style interception for security policies as composable scripts
- **MCP-native tool integration** from Nanobot — MCP as the standard protocol for all tool extensions
- **ClawHub marketplace** from MicroClaw — centralized skill discovery, review, and installation

### Security
- **6 sandbox runtimes** from ZeptoClaw — Docker, Bubblewrap, Firejail, Landlock, Apple Sandbox, native (operator chooses the tradeoff)
- **Network policy engine** from NemoClaw — dual-layer (DNS proxy + gateway) network enforcement with preset library
- **Safety taint tracking** from ZeptoClaw — data provenance through the entire pipeline
- **Agent mode controls** from ZeptoClaw (Safe/Standard/Unrestricted) with explicit escalation
- **Encrypted vault** from Moltis/IronClaw — OS keychain integration + AES-256-GCM
- **Security test suite** from Hermes — 18+ categories (SQL injection, SSRF, prompt injection, symlink, path traversal, etc.)
- **WASM sandbox** from IronClaw — for plugin isolation without container overhead

### LLM Integration
- **Multi-provider routing** from Letta (via LiteLLM) — broadest provider coverage
- **Smart model routing** from Hermes — automatic model selection based on task type
- **Credential pool with rotation** from Hermes — multi-key load balancing
- **Provider resilience** from ZeptoClaw — rotation, fallback, retry, cooldown, quota management
- **Context compaction** from Letta — automatic summarization when context window fills
- **Token budget enforcement** from ZeptoClaw — per-session and per-agent cost limits

### Multi-Agent
- **Orchestration patterns** from Letta — round-robin, supervisor, sleeptime
- **A2A + ACP protocols** from MicroClaw — HTTP and stdio inter-agent communication
- **Agent-as-MCP-server** from Nanobot — recursive agent composition
- **Delegation tool** from Hermes — spawn sub-agents with context passing

### Communication
- **Channel abstraction** from MicroClaw/PicoClaw — registry pattern with 15+ platforms
- **Event bus** from PicoClaw — decoupled async communication between components
- **GraphQL API** from Moltis — flexible query language for client apps
- **Native mobile apps** from Moltis — Rust→Swift FFI bridge for iOS/macOS

### Operations
- **6 deployment targets** from Moltis — Docker, Fly.io, Railway, Render, DigitalOcean, Nix
- **Config hot-reload** from Nanobot (fswatch) — zero-downtime configuration changes
- **OpenTelemetry** from Letta/ZeptoClaw — distributed tracing and metrics
- **LLM trace replay** from IronClaw — deterministic testing without live LLM calls
- **Coverage ratchet** from NemoClaw — CI prevents test coverage from decreasing

### Novel Additions
- **Hardware integration** from PicoClaw/ZeptoClaw — I2C, SPI, MQTT, serial for IoT agents
- **Audio pipeline** from PicoClaw — ASR/TTS with multiple providers
- **Container-per-task** from NanoClaw — for highest-isolation workloads
- **Tauri desktop** from OpenFang — native desktop experience

---

## 7. Surprising Findings

### 1. Authentication Disabled by Default (CoPaw) ⚠️
CoPaw's `COPAW_AUTH_ENABLED` is **commented out** in docker-compose, meaning deployed instances are publicly accessible by default. For a platform that executes arbitrary tools on behalf of users, this is a critical design flaw. Combined with Chromium's `--no-sandbox` mode, a publicly accessible CoPaw instance represents a significant attack surface.

### 2. Private Base Image from Alibaba Cloud (CoPaw) ⚠️
CoPaw's Docker base image is pulled from `agentscope-registry.ap-southeast-1.cr.aliyuncs.com` — an Alibaba Cloud private registry. This is a **supply chain risk**: the base image is not independently auditable, and a compromise of this registry could inject malicious code into all CoPaw deployments. No other framework depends on a private, non-public registry.

### 3. TLS Private Key Committed to Git (Letta) ⚠️
Letta's repository contains `certs/localhost-key.pem` — a TLS private key committed to source control. While it's a localhost development certificate, the pattern is dangerous: any contributor or fork inherits the key, and the `.gitignore` evidently doesn't exclude it.

### 4. Historical Plaintext Credential Storage (Letta)
Letta's migration history (`8149a781ac1b_backfill_encrypted_columns_for_`) reveals that LLM provider API keys were stored in plaintext before being backfilled to encrypted columns. Any database backup taken before this migration contains plaintext API keys.

### 5. Rust Dominates Agent Infrastructure (5/12)
42% of these frameworks are written in Rust — a language traditionally associated with systems programming, not web services. This suggests the community has concluded that agent infrastructure requires the safety guarantees (memory safety, thread safety, type safety) that Rust provides. The Rust frameworks also consistently have the most sophisticated security models.

### 6. No Framework Implements Full RBAC
Despite most frameworks supporting multi-tenancy and multi-user scenarios, **none implement a full Role-Based Access Control system**. Authorization is typically workspace/org scoping or simple bearer token checks. Even IronClaw's multi-tenant isolation is scope-based rather than role-based. This is a significant gap given that agents execute tools with real-world side effects.

### 7. Everyone Builds Their Own Memory System
All 12 frameworks implement their own conversation memory and state persistence. There's no shared library or standard protocol for agent memory (unlike MCP for tools). Memory approaches range from JSONL files (PicoClaw) to SQLite (NanoClaw, PicoClaw, Moltis) to PostgreSQL + pgvector (Letta, IronClaw). This fragmentation suggests the community hasn't converged on a memory standard yet.

### 8. Hardware/IoT Integration is Emerging
Both PicoClaw and ZeptoClaw include hardware integration (I2C, SPI, ESP32, Arduino, RPi, USB, serial, MQTT). This signals that LLM agents are expanding beyond software tasks into physical-world automation. ZeptoClaw even includes Android device tools. This is an underexplored but potentially transformative use case.

### 9. "Yolo Mode" Exists (Hermes Agent)
Hermes Agent has a `test_yolo_mode.py` test, implying a mode where all safety guardrails are disabled. The existence of both `force_dangerous_override` and `yolo_mode` tests suggests operators explicitly request bypassing security — and the framework accommodates this. This tension between safety and usability is a recurring theme.

### 10. Multi-Agent Protocols Are Divergent
Three distinct multi-agent protocols exist across these frameworks:
- **A2A (Agent-to-Agent)** — HTTP-based, used by MicroClaw and ZeptoClaw
- **ACP (Agent Communication Protocol)** — stdio-based, used by MicroClaw, ZeptoClaw, and PicoClaw
- **MCP** — used for tool extensions but repurposed for agent communication by Nanobot

There is no convergence on a single agent-to-agent communication standard. This fragmentation will become a significant interoperability problem as multi-agent systems scale.

### 11. China Ecosystem vs. Western Ecosystem Split
CoPaw integrates heavily with the Chinese tech ecosystem (Alibaba Cloud, ModelScope, WeChat, QQ, DingTalk, Feishu, Chinese fonts), while others integrate with the Western ecosystem (GitHub, Slack, Discord, WhatsApp). MicroClaw, PicoClaw, and IronClaw support both, but the ecosystem split is architecturally visible in code dependencies and channel adapters.

### 12. Container-per-Task is Rare
Despite being the most secure execution model, only NanoClaw implements true container-per-task isolation. Most frameworks run all tasks in the same process or at best the same container. The overhead (startup latency, resource consumption) apparently outweighs the security benefit for most use cases — but this may change as container startup times decrease.

---

*Analysis based on repository structure, dependency manifests, directory organization, and documentation. Source code content was not directly inspected for most files; findings are architectural-level and should be validated with code-level review for production security decisions.*
