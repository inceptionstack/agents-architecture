
## Full Investigation — zeptoclaw (7 sections)

--- core_entities ---


# Domain Model Analysis: `zeptoclaw_646bf09b`

## Overview

This appears to be a **multi-tenant AI agent platform** (an MCP/ACP-compatible server) written in Rust, with a React control panel. It orchestrates AI providers, channels, tools, skills, and sessions in a secure, extensible runtime.

---

## 1. Core Domain Entities

---

### 🤖 `Agent`

The central orchestration entity representing a running AI agent instance.

| Attribute | Description |
|---|---|
| `id` | Unique agent identifier |
| `persona` | Agent's configured identity/role |
| `mode` | Operational mode (e.g., agent, tool-call, sandbox) |
| `provider` | Assigned AI provider (Claude, OpenAI, Gemini, etc.) |
| `budget` | Token/cost budget limits |
| `context` | Current context window state |
| `tool_call_limit` | Max tool calls allowed per run |
| `loop_guard` | Loop detection/prevention state |
| `scratchpad` | Ephemeral working memory |
| `status` | Current lifecycle status |

**Source files:** `src/agent/`

---

### 💬 `Session`

Represents a conversation/interaction session between a user/channel and an agent.

| Attribute | Description |
|---|---|
| `id` | Unique session identifier |
| `channel_id` | Originating channel reference |
| `history` | Ordered list of messages |
| `media` | Attached media items |
| `created_at` | Session creation timestamp |
| `repaired` | Whether session history was auto-repaired |
| `context_token_count` | Rolling token count for context management |

**Source files:** `src/session/`

---

### 📨 `Message`

An individual message unit within a session or the internal message bus.

| Attribute | Description |
|---|---|
| `id` | Unique message identifier |
| `role` | `user`, `assistant`, `system`, `tool` |
| `content` | Text or structured content payload |
| `tool_calls` | Any tool invocations embedded in the message |
| `tool_call_id` | Reference to a preceding tool call (for results) |
| `timestamp` | Creation timestamp |
| `channel` | Originating channel identifier |
| `metadata` | Arbitrary key-value metadata |

**Source files:** `src/bus/message.rs`, `src/session/types.rs`

---

### 🔌 `Channel`

An ingress/egress communication interface connecting external systems to the agent.

| Attribute | Description |
|---|---|
| `id` | Channel identifier |
| `channel_type` | `slack`, `telegram`, `discord`, `webhook`, `acp`, `mqtt`, `serial`, `email`, `whatsapp`, etc. |
| `config` | Channel-specific configuration (tokens, endpoints) |
| `persona` | Optional persona override for this channel |
| `enabled` | Active/inactive flag |
| `rate_limit` | Per-channel rate limiting config |

**Source files:** `src/channels/`

---

### 🧠 `Provider`

Represents an AI model/LLM backend that processes inference requests.

| Attribute | Description |
|---|---|
| `id` | Provider identifier |
| `provider_type` | `openai`, `claude`, `gemini`, `vertex`, `plugin` |
| `api_key` / `credentials` | Authentication credentials |
| `model` | Specific model name/version |
| `quota` | Usage quota and limits |
| `cooldown` | Backoff/cooldown state after errors |
| `retry_policy` | Retry configuration |
| `fallback_provider_id` | Reference to fallback provider |
| `rotation_order` | Position in rotation pool |

**Source files:** `src/providers/`

---

### 🛠️ `Tool`

A discrete capability that an agent can invoke during its reasoning loop.

| Attribute | Description |
|---|---|
| `id` / `name` | Unique tool identifier |
| `tool_type` | `filesystem`, `shell`, `git`, `http_request`, `memory`, `task`, `spawn`, `mcp`, `plugin`, `cron`, `hardware`, etc. |
| `description` | Human-readable description for the LLM |
| `input_schema` | JSON Schema of accepted parameters |
| `requires_approval` | Whether human approval is required |
| `timeout` | Execution timeout |
| `config` | Tool-specific configuration |

**Source files:** `src/tools/`

---

### 📦 `Skill`

A packaged, distributable bundle of tools, prompts, and configuration.

| Attribute | Description |
|---|---|
| `id` | Skill identifier |
| `name` | Human-readable name |
| `version` | Semver version string |
| `source` | Source URL (GitHub, registry, local) |
| `tools` | List of tool definitions included |
| `dependencies` | Other skills/deps required |
| `manifest` | Full skill manifest metadata |
| `installed_at` | Installation timestamp |

**Source files:** `src/skills/`

---

### ⚙️ `Config`

The runtime configuration of the entire agent server instance.

| Attribute | Description |
|---|---|
| `agent` | Agent-level defaults (persona, model, limits) |
| `providers` | List of provider configurations |
| `channels` | List of channel configurations |
| `tools` | Tool enablement and configuration |
| `security` | Security policies (sandboxing, path restrictions) |
| `memory` | Memory backend configuration |
| `multi_tenant` | Tenant isolation settings |
| `heartbeat` | Heartbeat schedule/target |
| `secrets` | References to secret keys |

**Source files:** `src/config/`

---

### 🔐 `AuthToken` / `Credential`

Authentication and authorization state for providers and external services.

| Attribute | Description |
|---|---|
| `id` | Credential identifier |
| `provider_type` | Which provider/service this is for |
| `access_token` | OAuth access token |
| `refresh_token` | OAuth refresh token |
| `expires_at` | Token expiry timestamp |
| `scopes` | Granted OAuth scopes |
| `imported_from` | Source (`claude_import`, `codex_import`, `oauth`) |

**Source files:** `src/auth/`

---

### 🧩 `Plugin`

A dynamically loadable extension that can provide additional tools or channel behaviors.

| Attribute | Description |
|---|---|
| `id` | Plugin identifier |
| `name` | Plugin name |
| `path` | Filesystem path to plugin binary |
| `version` | Plugin version |
| `capabilities` | Declared capabilities (tools, channels) |
| `state` | Loaded / unloaded / error |

**Source files:** `src/plugins/`

---

### 🏢 `Tenant`

An isolated unit of configuration and data in multi-tenant deployments.

| Attribute | Description |
|---|---|
| `id` | Tenant identifier |
| `name` | Human-readable tenant name |
| `config_path` | Path to tenant-specific config |
| `api_key` | Tenant-scoped API key |
| `quota` | Tenant-level usage quotas |
| `created_at` | Provisioning timestamp |

**Source files:** `docker-compose.multi-tenant.yml`, `src/api/auth.rs`, `scripts/add-tenant.sh`

---

### 🗄️ `MemoryEntry`

A stored memory item in the agent's long-term or working memory.

| Attribute | Description |
|---|---|
| `id` | Unique memory entry ID |
| `content` | Text content of the memory |
| `embedding` | Optional vector embedding |
| `tags` / `metadata` | Categorization metadata |
| `created_at` | Creation timestamp |
| `score` | Retrieval relevance score (BM25, HNSW, etc.) |
| `session_id` | Originating session reference |

**Source files:** `src/memory/`

---

### 📋 `Task`

A unit of deferred or background work to be executed by the agent.

| Attribute | Description |
|---|---|
| `id` | Task identifier |
| `description` | Task goal/prompt |
| `status` | `pending`, `running`, `complete`, `failed` |
| `created_at` | Creation timestamp |
| `scheduled_at` | Optional cron/future schedule |
| `result` | Output once complete |
| `parent_agent_id` | Spawning agent reference |

**Source files:** `src/api/tasks.rs`, `src/tools/task.rs`, `src/cron/`

---

### 🔔 `AuditEvent`

An immutable record of a security-relevant or operationally-significant event.

| Attribute | Description |
|---|---|
| `id` | Event ID |
| `event_type` | Type of event (tool_call, auth, approval, etc.) |
| `actor` | Agent, user, or system that triggered it |
| `target` | Resource acted upon |
| `payload` | Event-specific data |
| `timestamp` | When the event occurred |
| `tenant_id` | Tenant scope |

**Source files:** `src/audit.rs`

---

### ✅ `ApprovalRequest`

A human-in-the-loop gate for sensitive tool invocations.

| Attribute | Description |
|---|---|
| `id` | Request identifier |
| `tool_name` | Tool requiring approval |
| `tool_input` | Parameters to be approved |
| `status` | `pending`, `approved`, `denied`, `expired` |
| `requested_at` | Timestamp of request |
| `resolved_at` | Timestamp of resolution |
| `agent_id` | Requesting agent |
| `session_id` | Associated session |

**Source files:** `src/tools/approval.rs`, `src/r8r_bridge/approval.rs`

---

### 🌐 `Tunnel`

An outbound network tunnel for exposing the local agent to the internet.

| Attribute | Description |
|---|---|
| `tunnel_type` | `cloudflare`, `ngrok`, `tailscale` |
| `public_url` | Assigned public endpoint URL |
| `status` | Active/inactive/error |
| `config` | Provider-specific tunnel configuration |

**Source files:** `src/tunnel/`

---

### 🖥️ `Runtime` / `Sandbox`

The execution environment for sandboxed tool execution.

| Attribute | Description |
|---|---|
| `runtime_type` | `docker`, `bubblewrap`, `firejail`, `landlock`, `native`, `apple` |
| `container_id` | Container/process identifier |
| `mounts` | Filesystem mount points |
| `network_policy` | Network access rules |
| `resource_limits` | CPU/memory caps |

**Source files:** `src/runtime/`

---

### 🔩 `Peripheral` / `HardwareDevice`

A physical hardware device (microcontroller, sensor, etc.) accessible to the agent.

| Attribute | Description |
|---|---|
| `id` | Device identifier |
| `device_type` | `arduino`, `esp32`, `rpi`, `nucleo`, `usb`, `i2c` |
| `connection` | Serial port, I2C address, USB path |
| `board_profile` | Capabilities and pin layout |
| `state` | Connected/disconnected |

**Source files:** `src/peripherals/`, `src/devices/`

---

## 2. Entity Relationship Diagram

```
┌──────────┐       ┌───────────┐       ┌──────────┐
│  Tenant  │──1:N──│   Agent   │──1:N──│ Session  │
└──────────┘       └─────┬─────┘       └────┬─────┘
                         │                   │
                    ┌────┴────┐         ┌────┴─────┐
                    │         │         │          │
                  1:N       1:N        1:N        1:N
                    │         │         │          │
               ┌────▼──┐ ┌───▼────┐ ┌──▼─────┐ ┌─▼──────┐
               │Channel│ │Provider│ │Message │ │Memory  │
               └───────┘ └────────┘ │ Entry  │ │ Entry  │
                                     └──┬─────┘ └────────┘
                                        │
                                       1:N
                                        │
                                   ┌────▼──────┐
                                   │  Tool     │
                                   │  Call     │
                                   └────┬──────┘
                                        │
                              ┌─────────┴──────────┐
                              │                    │
                         ┌────▼───┐        ┌───────▼────────┐
                         │ Tool   │        │ApprovalRequest │
                         └────┬───┘        └────────────────┘
                              │
                    ┌─────────┼──────────┐
                    │         │          │
               ┌────▼──┐ ┌───▼──┐ ┌─────▼────┐
               │Plugin │ │Skill │ │ Runtime  │
               └───────┘ └──────┘ └──────────┘

Agent ──1:N──► Task
Agent ──1:N──► AuditEvent
Agent ──M:N──► Tool  (via ToolRegistry)
Agent ──1:1──► Config (scoped)
Skill ──1:N──► Tool
Plugin ──1:N──► Tool
Agent ──M:N──► Peripheral (via HardwareRegistry)
Agent ──1:1──► Tunnel
```

---

## 3. Relationship Summary Table

| Entity A | Relationship | Entity B | Notes |
|---|---|---|---|
| `Tenant` | 1 : N | `Agent` | Each tenant runs ≥1 isolated agents |
| `Agent` | 1 : N | `Session` | Each agent handles many sessions |
| `Agent` | M : N | `Tool` | Via tool registry; agent loads a toolset |
| `Agent` | M : N | `Provider` | Rotation/fallback across multiple providers |
| `Agent` | 1 : N | `Channel` | Agent listens on multiple channels |
| `Agent` | 1 : N | `Task` | Agent spawns/manages background tasks |
| `Agent` | 1 : N | `AuditEvent` | All agent actions are audit-logged |
| `Session` | 1 : N | `Message` | Session is composed of ordered messages |
| `Session` | 1 : N | `MemoryEntry` | Sessions seed long-term memory |
| `Message` | 1 : N | `ApprovalRequest` | Tool calls in messages may require approval |
| `Skill` | 1 : N | `Tool` | A skill bundles one or more tools |
| `Plugin` | 1 : N | `Tool` | Plugins expose dynamic tools |
| `Tool` | 1 : 1 | `Runtime` | Sandboxed tools run in an isolated runtime |
| `Tool` | 0 : 1 | `ApprovalRequest` | Sensitive tools create an approval gate |
| `Provider` | 1 : 1 | `AuthToken` | Each provider has credentials |
| `Agent` | 0 : N | `Peripheral` | Hardware-enabled agents control devices |
| `Agent` | 0 : 1 | `Tunnel` | Optional public tunnel per agent |
| `Config` | 1 : 1 | `Agent` | Each agent/tenant has scoped config |

--- hl_overview ---


# Repository Analysis: zeptoclaw_646bf09b

## [[zeptoclaw]]

---

## 1. Project Purpose

**Zeptoclaw** is an **AI agent runtime and orchestration platform**. It solves the problem of running, managing, and securing AI agents (primarily LLM-based) with:

- Multi-provider LLM support (Claude, OpenAI/GPT, Gemini, Vertex AI)
- Multi-tenant deployment capabilities
- Tool/skill execution with sandboxed runtimes
- Channel integrations (Slack, Discord, Telegram, WhatsApp, MQTT, email, webhooks)
- Hardware/peripheral integrations (ESP32, Arduino, Raspberry Pi, USB devices)
- Security, safety, and approval workflows
- MCP (Model Context Protocol) server implementation
- Memory management (short-term, long-term, embedding-based search)

**Primary Domain:** AI Agent Infrastructure / LLM Orchestration Platform

---

## 2. Architecture Pattern

- **Plugin-based Modular Monolith** (Rust workspace with feature-gated modules)
- **Event-Driven** (message bus, channel manager, async event streams)
- **Gateway/Proxy Pattern** (API server acting as gateway to LLM providers)
- **Multi-tenant SaaS** (tenant isolation, per-tenant configuration)

---

## 3. Technology Stack

### Primary Language
- **Rust** (entire backend/core)

### Frontend
- **React** + **TypeScript** (control panel in `/panel/`)
- **Vite** (build tool)
- **pnpm** (package manager)

### Landing Pages
- **Astro** (static site framework, in `/landing/`)
- **Cloudflare Workers** (`wrangler.toml`)

### Key Rust Dependencies (inferred from `Cargo.toml` / structure)
| Dependency | Purpose |
|---|---|
| `tokio` | Async runtime |
| `axum` | HTTP API server framework |
| `serde` / `serde_json` | Serialization |
| `reqwest` | HTTP client (provider calls) |
| `sqlx` or similar | Database access |
| `hnsw` | Vector search for memory |
| `bm25` | Text search for memory |
| `wasmtime` (likely) | WASM plugin runtime |
| `tracing` | Logging/telemetry |
| `clap` | CLI argument parsing |

### Infrastructure
- **Docker** / **Docker Compose** (multi-tenant deployments)
- **Cloudflare Tunnel**, **ngrok**, **Tailscale** (tunneling)
- **GitHub Actions** (CI/CD)
- **DigitalOcean**, **Fly.io**, **Railway**, **Render** (deployment targets)

---

## 4. Initial Structure Impression

| Part | Location | Role |
|---|---|---|
| **Core Agent Runtime** | `src/agent/` | LLM agent loop, context, budgeting |
| **REST API Server** | `src/api/` | HTTP API gateway |
| **CLI Tool** | `src/cli/` | Command-line interface |
| **Control Panel (UI)** | `panel/` | Web frontend |
| **Landing Pages** | `landing/` | Marketing/docs sites |
| **Channel Integrations** | `src/channels/` | Messaging platform connectors |
| **Tool System** | `src/tools/` | Extensible tool execution |
| **Provider Adapters** | `src/providers/` | LLM provider abstraction |
| **Skills System** | `src/skills/`, `skills/` | Loadable skill packages |
| **Security Layer** | `src/security/`, `src/safety/` | Sandboxing, approval, taint tracking |
| **Memory System** | `src/memory/` | Short/long-term agent memory |
| **Hardware Layer** | `src/hardware/`, `src/peripherals/` | Physical device integrations |

---

## 5. Configuration / Package Files

| File | Purpose |
|---|---|
| `Cargo.toml` | Rust workspace/package manifest |
| `Cargo.lock` | Rust dependency lockfile |
| `panel/package.json` | Frontend npm/pnpm manifest |
| `panel/pnpm-lock.yaml` | Frontend lockfile |
| `panel/vite.config.ts` | Vite build configuration |
| `panel/tsconfig*.json` | TypeScript compiler configs (3 files) |
| `panel/eslint.config.js` | ESLint linting rules |
| `deny.toml` | `cargo-deny` supply chain security config |
| `.cargo/config.toml` | Cargo build configuration |
| `.cargo/audit.toml` | `cargo-audit` security audit config |
| `.config/nextest.toml` | `cargo-nextest` test runner config |
| `codecov.yml` | Code coverage reporting config |
| `docker-compose.multi-tenant.yml` | Multi-tenant Docker Compose |
| `deploy/docker-compose.multi.yml` | Multi-instance deployment |
| `deploy/docker-compose.single.yml` | Single-instance deployment |
| `deploy/.env.example` | Environment variable template |
| `deploy/fly.toml` | Fly.io deployment config |
| `deploy/railway.json` | Railway deployment config |
| `deploy/render.yaml` | Render deployment config |
| `deploy/digitalocean.yaml` | DigitalOcean App Platform config |
| `Dockerfile` | Production container build |
| `Dockerfile.dev` | Development container build |
| `.dockerignore` | Docker build exclusions |
| `Makefile` | Build/dev task automation |
| `landing/zeptoclaw/wrangler.toml` | Cloudflare Workers config (landing) |
| `landing/r8r/wrangler.toml` | Cloudflare Workers config (r8r docs) |
| `scripts/artifacts/clippy.toml` | Rust linter (clippy) config |

---

## 6. Directory Structure

```
src/
├── agent/          # Core agent loop: LLM turns, context window, compaction,
│                   # budget tracking, tool call limits, scratchpad management
├── api/            # Axum HTTP server: routes, middleware, auth, SSE events,
│                   # OpenAI-compatible types, task management
│   └── routes/     # Individual route handlers (11 route files)
├── auth/           # Authentication: OAuth, token refresh, credential store,
│                   # Claude/Codex credential import
├── batch/          # Batch processing support
├── bin/            # Binary entry points (benchmark runner)
├── bus/            # Internal message bus for event passing
├── cache/          # Response caching layer
├── channels/       # Communication channel integrations:
│                   # Slack, Discord, Telegram, WhatsApp (cloud+web),
│                   # Email, MQTT, Lark, Serial, Webhook, ACP protocol
├── cli/            # Full CLI: agent, daemon, gateway, tools, skills,
│                   # config, pair, secrets, quota, status, health, etc.
├── config/         # Configuration loading, validation, templates, hot-reload
├── cron/           # Scheduled task execution
├── deps/           # Dependency/package manager for agent dependencies
├── devices/        # USB and hardware device management
├── gateway/        # Container agent gateway, rate limiting, IPC, idempotency
├── hardware/       # Hardware discovery, introspection, registry
├── heartbeat/      # Agent heartbeat/keepalive service
├── hooks/          # Lifecycle hooks system
├── kernel/         # Core kernel: provider gate, registrar
├── memory/         # Memory subsystems: BM25, HNSW vector, embedding,
│                   # long-term storage, hygiene, snapshots
├── mcp_server/     # Model Context Protocol server (stdio + handler)
├── migrate/        # Configuration/skills migration tooling
├── peripherals/    # Embedded hardware: Arduino, ESP32, RPi, I2C, NVS, serial
├── plugins/        # Plugin system: loader, registry, watcher, WASM types
├── providers/      # LLM provider adapters: Claude, OpenAI, Gemini, Vertex,
│                   # retry, fallback, rotation, cooldown, quota, structured output
├── r8r_bridge/     # R8r (router/relay) bridge: approval, dedup, events, health
├── routines/       # Automated routine/workflow engine
├── runtime/        # Sandbox runtimes: Docker, Bubblewrap, Firejail,
│                   # Landlock, Apple Sandbox, native
├── safety/         # Safety layer: policy enforcement, taint tracking,
│                   # leak detection, chain alerts, input sanitization
├── security/       # Security: encryption, agent mode, path restrictions,
│                   # shell security, device pairing
├── session/        # Session/conversation history, media handling, repair
├── skills/         # Skills system: loader, registry, types, GitHub source
├── tools/          # Tool implementations: filesystem, shell, git, HTTP,
│                   # web, memory, screenshot, PDF/DOCX read, Google,
│                   # Stripe, WhatsApp, hardware, spawn, delegate, diff,
│                   # approval, clarification, MCP tools, Android tools
├── tunnel/         # Tunnel providers: Cloudflare, ngrok, Tailscale
└── utils/          # Shared utilities: logging, metrics, SLO, cost calc,
                    # telemetry, string helpers, sanitization

panel/              # React/TypeScript web control panel (frontend UI)
landing/            # Astro static marketing/documentation sites
skills/             # Skill package definitions (GitHub, deep-research, skill-creator)
tests/              # Integration, E2E, conformance, and smoke tests
benches/            # Criterion benchmarks (message bus)
examples/           # Usage examples (RISC-V smoke test)
scripts/            # Shell scripts for dev, CI, tenant management
deploy/             # Deployment configurations for various platforms
docs/               # Documentation, architecture plans, design documents
```

---

## 7. High-Level Architecture

### Pattern: **Layered Plugin-Based Agent Runtime with Event-Driven Messaging**

```
┌─────────────────────────────────────────────────────┐
│              Interfaces / Entry Points               │
│  CLI (clap)  │  HTTP API (axum)  │  MCP Server      │
├─────────────────────────────────────────────────────┤
│                   Agent Loop                        │
│   context → tool calls → compaction → response      │
├──────────────┬──────────────────────────────────────┤
│   Channels   │           Tool System                │
│ Slack/TG/    │  FS/Shell/Git/Web/Memory/Hardware    │
│ WhatsApp/    │  + Plugin tools + MCP tools          │
│ MQTT/Serial  │  + Android/ESP32 tools               │
├──────────────┴──────────────────────────────────────┤
│              Provider Abstraction Layer             │
│   Claude │ OpenAI │ Gemini │ Vertex │ Plugin        │
│   + retry + fallback + rotation + quota             │
├─────────────────────────────────────────────────────┤
│          Cross-Cutting Infrastructure               │
│  Security │ Safety │ Memory │ Cache │ Bus │ Session │
│  Runtime Sandbox │ Audit │ Telemetry │ Config       │
└─────────────────────────────────────────────────────┘
```

**Evidence:**
- `src/kernel/` acts as the central registrar/gate
- `src/bus/` provides internal event messaging
- `src/channels/factory.rs` and `src/tools/registry.rs` show registry/factory patterns
- `src/runtime/` with multiple backends (Docker, Firejail, Bubblewrap, Landlock, Apple) shows strategy pattern for sandboxing
- `src/providers/rotation.rs`, `fallback.rs`, `retry.rs` show resilience patterns
- `src/gateway/` isolates container agent communication
- Multi-tenant support via separate compose files and tenant management scripts

---

## 8. Build, Execution, and Test

### Building
```bash
# Standard Rust build
cargo build --release

# Via Makefile
make build

# Docker production
docker build -f Dockerfile .

# Docker development
docker build -f Dockerfile.dev .

# Frontend panel
cd panel && pnpm install && pnpm build
```

### Running
```bash
# CLI entry point (src/main.rs)
cargo run -- [subcommand]

# Common CLI subcommands (from src/cli/mod.rs):
zeptoclaw serve          # Start HTTP API server
zeptoclaw daemon         # Run as background daemon
zeptoclaw agent          # Run agent session
zeptoclaw gateway        # Start gateway mode
zeptoclaw pair           # Device pairing
zeptoclaw status         # Check status
zeptoclaw doctor         # Health diagnostics

# Docker single-tenant
docker-compose -f deploy/docker-compose.single.yml up

# Docker multi-tenant
docker-compose -f docker-compose.multi-tenant.yml up

# Install script
./install.sh
```

### Testing
```bash
# Unit + integration tests via nextest
cargo nextest run

# Standard cargo test
cargo test

# Specific test suites
cargo test --test integration
cargo test --test e2e
cargo test --test cli_smoke
cargo test --test acp_acpx

# Conformance tests
cargo test --test conformance

# Benchmarks
cargo bench

# E2E with Docker
cd tests/e2e && docker-compose up

# Pre-push hook
./.git/hooks/pre-push  # (scripts/pre-push)

# Frontend lint
cd panel && pnpm lint

# Dependency security audit
cargo deny check
cargo audit
```

### Main Entry Points
- **`src/main.rs`** — Primary binary entry point (CLI dispatch)
- **`src/lib.rs`** — Library root (for testing and external use)
- **`src/api/server.rs`** — HTTP server startup
- **`src/mcp_server/stdio.rs`** — MCP stdio server entry
- **`benches/message_bus.rs`** — Benchmark entry point
- **`tests/conformance/main.rs`** — Conformance test runner

--- dependencies ---


# Dependency and Architecture Analysis: Zeptoclaw

---

## Internal Modules

The following internal modules are developed as part of the Zeptoclaw project and reused across different components. They are identified from the `src/` directory structure.

---

### Core Agent

| Module | Location | Responsibility |
|---|---|---|
| **Agent** | `src/agent/` | Core LLM agent loop; manages context windows, tool call limits, budget tracking, scratchpad, compaction, and turn-by-turn execution |
| **Kernel** | `src/kernel/` | Central registrar and provider gate; acts as the coordination hub for provider access and component registration |
| **Session** | `src/session/` | Manages conversation history, media attachments, session repair, and session type definitions |
| **Batch** | `src/batch/` | Handles batch processing of agent requests |
| **Routines** | `src/routines/` | Automated workflow/routine engine for scheduled or triggered multi-step agent tasks |

---

### API & Interfaces

| Module | Location | Responsibility |
|---|---|---|
| **API Server** | `src/api/` | Axum-based HTTP gateway exposing REST routes, middleware, authentication, SSE events, task management, and OpenAI-compatible types |
| **CLI** | `src/cli/` | Full command-line interface; dispatches subcommands for agent, daemon, gateway, config, skills, secrets, health, pair, quota, and more |
| **MCP Server** | `src/mcp_server/` | Model Context Protocol server implementation (stdio transport and request handler) |
| **Gateway** | `src/gateway/` | Container agent gateway; handles IPC, rate limiting, idempotency, and startup coordination for multi-tenant agent containers |

---

### Provider & Model Layer

| Module | Location | Responsibility |
|---|---|---|
| **Providers** | `src/providers/` | LLM provider adapters for Claude, OpenAI, Gemini, and Vertex AI; includes retry, fallback, rotation, cooldown, quota management, and structured output |

---

### Channel Integrations

| Module | Location | Responsibility |
|---|---|---|
| **Channels** | `src/channels/` | Communication channel connectors for Slack, Discord, Telegram, WhatsApp (Cloud and Web), Email, MQTT, Lark/Feishu, Serial, Webhook, and ACP protocol; includes a factory and manager |

---

### Tool System

| Module | Location | Responsibility |
|---|---|---|
| **Tools** | `src/tools/` | Extensible tool execution layer; includes filesystem, shell, git, HTTP, web, memory, screenshot, PDF/DOCX reading, Google, Stripe, WhatsApp, hardware, spawn, delegate, diff, approval, clarification, MCP tools, and Android tools |

---

### Memory & Storage

| Module | Location | Responsibility |
|---|---|---|
| **Memory** | `src/memory/` | Agent memory subsystems including BM25 keyword search, HNSW vector search, embedding-based search, long-term storage, hygiene, and snapshots |
| **Cache** | `src/cache/` | Response caching layer to reduce redundant LLM provider calls |
| **Deps** | `src/deps/` | Dependency/package manager for agent runtime dependencies; includes fetcher, registry, and type definitions |

---

### Security & Safety

| Module | Location | Responsibility |
|---|---|---|
| **Security** | `src/security/` | Encryption, agent mode enforcement, path restrictions, shell security hardening, and device pairing |
| **Safety** | `src/safety/` | Policy enforcement, taint tracking, leak detection, chain alerts, input sanitization, and output validation |
| **Runtime** | `src/runtime/` | Sandboxed execution backends using Docker, Bubblewrap, Firejail, Landlock, Apple Sandbox, or native; strategy-pattern factory |
| **Auth** | `src/auth/` | Authentication handling including OAuth, token refresh, credential store, and credential import from Claude/Codex |
| **Audit** | `src/audit.rs` | Security audit logging |

---

### Infrastructure & Utilities

| Module | Location | Responsibility |
|---|---|---|
| **Bus** | `src/bus/` | Internal message bus for async event passing between components |
| **Config** | `src/config/` | Configuration loading, validation, template support, and hot-reload watching |
| **Utils** | `src/utils/` | Shared utilities: logging, metrics, SLO tracking, cost calculation, telemetry, string helpers, and sanitization |
| **Heartbeat** | `src/heartbeat/` | Agent keepalive/heartbeat service with templating support |
| **Hooks** | `src/hooks/` | Lifecycle hooks system for agent and component events |
| **Cron** | `src/cron/` | Scheduled task execution engine |
| **Migrate** | `src/migrate/` | Migration tooling for configuration and skills upgrades |
| **R8r Bridge** | `src/r8r_bridge/` | Bridge to the R8r router/relay service; handles approval workflows, deduplication, event streaming, and health checks |
| **Tunnel** | `src/tunnel/` | Tunnel provider integrations for Cloudflare Tunnel, ngrok, and Tailscale |
| **Health** | `src/health.rs` | Health check endpoint and diagnostics |
| **Error** | `src/error.rs` | Centralized error type definitions |
| **Transcription** | `src/transcription.rs` | Audio transcription support |

---

### Hardware & Peripherals

| Module | Location | Responsibility |
|---|---|---|
| **Hardware** | `src/hardware/` | Hardware discovery, introspection, and device registry |
| **Peripherals** | `src/peripherals/` | Embedded hardware support: Arduino, ESP32, Raspberry Pi, STM32/Nucleo, I2C, NVS, and serial communication |
| **Devices** | `src/devices/` | USB device enumeration and management |

---

### Skills & Plugins

| Module | Location | Responsibility |
|---|---|---|
| **Skills** | `src/skills/` | Skills system: loader, registry, types, and GitHub-based skill source |
| **Plugins** | `src/plugins/` | Plugin system: WASM plugin loader, registry, file watcher, and type definitions |

---

### Frontend (Panel)

| Module | Location | Responsibility |
|---|---|---|
| **Panel** | `panel/src/` | React/TypeScript web control panel; includes pages, components, hooks (`panel/src/hooks/`), and shared lib (`panel/src/lib/`) for managing agent configuration and monitoring |

---

### Landing Sites

| Module | Location | Responsibility |
|---|---|---|
| **Landing - Zeptoclaw** | `landing/zeptoclaw/` | Static marketing and documentation site for the main Zeptoclaw product |
| **Landing - R8r** | `landing/r8r/` | Static documentation site for the R8r router/relay component |

---

## External Dependencies

### Rust Dependencies
*Source: `/Cargo.toml`*

| Official Name | Declared Package | Role |
|---|---|---|
| **Tokio** | `tokio` | Async runtime; powers all async I/O, task scheduling, and concurrency throughout the application |
| **Serde** | `serde` / `serde_json` / `serde_yaml` | Serialization/deserialization framework; used for LLM API requests/responses, config files, and data exchange |
| **TOML** | `toml` | TOML config file parsing |
| **JSON5** | `json5` | JSON5 config parsing with support for comments, trailing commas, and unquoted keys (used for OpenClaw config migration) |
| **Reqwest** | `reqwest` | Async HTTP client with rustls TLS; used for all outbound LLM provider API calls and HTTP tool requests |
| **Scraper** | `scraper` | HTML/DOM parser built on Servo's html5ever; used with CSS selectors for web content extraction |
| **Clap** | `clap` | Command-line argument parsing with derive macros; powers the entire CLI interface |
| **Rustyline** | `rustyline` | Readline library with tab-completion for the interactive CLI |
| **Tracing** | `tracing` / `tracing-subscriber` | Structured logging and distributed tracing with span support; used for debugging agent loops |
| **thiserror** | `thiserror` | Derive macros for custom error type definitions |
| **anyhow** | `anyhow` | Ergonomic error propagation with context chaining |
| **Regex** | `regex` | Regular expression engine; used for shell command pattern matching in the security layer |
| **Aho-Corasick** | `aho-corasick` | High-performance multi-pattern string matching; used in the safety layer for injection detection |
| **ChaCha20Poly1305** | `chacha20poly1305` | XChaCha20-Poly1305 AEAD encryption for secrets at rest |
| **Argon2** | `argon2` | Argon2id password-based key derivation |
| **SHA-2** | `sha2` | SHA-256 digest computation for binary plugin integrity verification |
| **Subtle** | `subtle` | Constant-time comparison primitives for security-sensitive operations such as token validation |
| **Hex** | `hex` | Hex encoding/decoding for master key transport |
| **Ring** | `ring` | HMAC-SHA256 for CSRF token generation and validation |
| **libc** | `libc` | Platform-specific filesystem flags (e.g., `O_NOFOLLOW`) for low-level OS interactions |
| **async-trait** | `async-trait` | Enables `async fn` in trait definitions; used by `LLMProvider`, `Tool`, and `Channel` traits |
| **Futures** | `futures` | Stream combinators and async primitives; used for the message bus |
| **dotenvy** | `dotenvy` | `.env` file loading for environment variable configuration |
| **once_cell** | `once_cell` | Lazy static initialization for global configuration singletons |
| **dirs** | `dirs` | Platform-specific path resolution (e.g., `~/.config/zeptoclaw`) |
| **UUID** | `uuid` | UUID v4 generation for unique session and API identifiers |
| **ULID** | `ulid` | Time-ordered, URL-safe unique ID generation for ACP session and client IDs |
| **Chrono** | `chrono` | Date/time handling for message history timestamps and local time formatting |
| **chromiumoxide** | `chromiumoxide` | Headless Chromium control via Chrome DevTools Protocol; used for web screenshot tool (optional feature) |
| **instant-distance** | `instant-distance` | HNSW approximate nearest-neighbor vector search; used for embedding-based memory backend (optional feature) |
| **lopdf** | `lopdf` | Pure-Rust PDF parser for text extraction tool (optional feature) |
| **tokio-tungstenite** | `tokio-tungstenite` | Async WebSocket support with rustls TLS; used for real-time channel connections |
| **URL** | `url` | URL parsing; used for WebSocket endpoint host extraction |
| **Prost** | `prost` | Protocol Buffers decoding; used for Lark/Feishu pbbp2 WebSocket frame parsing |
| **Teloxide** | `teloxide` | Telegram Bot API SDK; powers the Telegram channel integration |
| **DashMap** | `dashmap` | Lock-free concurrent hash map; used for typing indicator tracking |
| **tokio-util** | `tokio-util` | Tokio utilities including cancellation tokens for async task lifecycle management |
| **base64** | `base64` | Base64 encoding/decoding for integration helpers |
| **tempfile** | `tempfile` | Temporary directory/file creation; used for container environment file handling |
| **rpassword** | `rpassword` | Secure password input with hidden terminal echo |
| **Zip** | `zip` | ZIP archive extraction for ClawHub skill package installs |
| **async-imap** | `async-imap` | IMAP IDLE client for real-time inbound email (optional `channel-email` feature) |
| **Lettre** | `lettre` | SMTP client for outbound email with STARTTLS support (optional `channel-email` feature) |
| **mail-parser** | `mail-parser` | Fast zero-copy RFC 5322/MIME email parser (optional `channel-email` feature) |
| **tokio-rustls** | `tokio-rustls` | TLS adapter for Tokio async streams; used for IMAP over TLS (optional `channel-email` feature) |
| **rustls** | `rustls` | Pure-Rust TLS implementation (optional `channel-email` feature) |
| **webpki-roots** | `webpki-roots` | Mozilla TLS root certificate bundle for rustls (optional `channel-email` feature) |
| **wa-rs** | `wa-rs` | Pure-Rust WhatsApp Web client implementing the Signal Protocol with QR pairing (optional `whatsapp-web` feature) |
| **wa-rs-sqlite-storage** | `wa-rs-sqlite-storage` | SQLite session storage backend for wa-rs (optional `whatsapp-web` feature) |
| **wa-rs-tokio-transport** | `wa-rs-tokio-transport` | Tokio WebSocket transport layer for wa-rs (optional `whatsapp-web` feature) |
| **wa-rs-ureq-http** | `wa-rs-ureq-http` | HTTP client for wa-rs version and media requests (optional `whatsapp-web` feature) |
| **qrcode** | `qrcode` | QR code generation for WhatsApp Web terminal pairing (optional `whatsapp-web` feature) |
| **tokio-serial** | `tokio-serial` | Async serial port communication for hardware peripherals including Arduino and ESP32 (optional `hardware` feature) |
| **probe-rs** | `probe-rs` | Debug probe interface for STM32/Nucleo memory read over USB (optional `probe` feature) |
| **quick-xml** | `quick-xml` | Fast XML parser; used for DOCX text extraction and Android UIAutomator dump parsing |
| **glob** | `glob` | File path pattern matching; used by the FindTool |
| **gog-gmail** | `gog-gmail` | Gmail integration via gogcli-rs (optional `google` feature) |
| **gog-calendar** | `gog-calendar` | Google Calendar integration via gogcli-rs (optional `google` feature) |
| **gog-auth** | `gog-auth` | Google OAuth authentication via gogcli-rs (optional `google` feature) |
| **gog-core** | `gog-core` | Shared core types for Google Workspace integrations via gogcli-rs (optional `google` feature) |
| **Axum** | `axum` | HTTP web framework with WebSocket support; used for the control panel API server (optional `panel` feature) |
| **tower-http** | `tower-http` | Tower HTTP middleware for CORS, static file serving, and tracing (optional `panel` feature) |
| **jsonwebtoken** | `jsonwebtoken` | JWT token generation and validation for panel authentication (optional `panel` feature) |
| **bcrypt** | `bcrypt` | Password hashing for panel password-based authentication mode (optional `panel` feature) |
| **unicode-normalization** | `unicode-normalization` | Unicode text normalization |
| **google-cloud-auth** | `google-cloud-auth` | Google Cloud authentication for Vertex AI provider access |
| **nusb** | `nusb` | USB device enumeration; Linux, macOS, and Windows only (optional `hardware` feature) |
| **rppal** | `rppal` | Raspberry Pi GPIO interface; Linux only (optional `peripheral-rpi` feature) |
| **Landlock** | `landlock` | Linux Landlock LSM sandbox (kernel 5.13+) for process isolation (optional `sandbox-landlock` feature) |

---

#### Rust Dev Dependencies
*Source: `/Cargo.toml` — `[dev-dependencies]`*

| Official Name | Declared Package | Role |
|---|---|---|
| **tokio-test** | `tokio-test` | Testing utilities for Tokio async code |
| **Mockall** | `mockall` | Automatic mock generation for unit tests |
| **Tower** | `tower` | Service abstraction utilities used in API tests |
| **Criterion** | `criterion` | Benchmarking framework with async Tokio support; used for the message bus benchmark |

---

### JavaScript / Frontend Dependencies

#### Production Dependencies — Control Panel
*Source: `/panel/package.json`*

| Official Name | Declared Package | Role |
|---|---|---|
| **dnd kit** | `@dnd-kit/core` / `@dnd-kit/sortable` | Drag-and-drop interaction primitives and sortable list utilities for the panel UI |
| **TanStack Query** | `@tanstack/react-query` | Async data fetching, caching, and server-state synchronization for the React panel |
| **React** | `react` / `react-dom` | UI component framework powering the control panel frontend |
| **React Router** | `react-router` | Client-side routing for the single-page control panel application |
| **Recharts** | `recharts` | Chart/data visualization library for metrics and usage dashboards in the panel |

#### Production Dependencies — Landing / Documentation Sites
*Source: `/landing/r8r/docs/package.json`, `/landing/zeptoclaw/docs/package.json`*

| Official Name | Declared Package | Role |
|---|---|---|
| **Starlight** | `@astrojs/starlight` | Astro-based documentation theme/framework for both the Zeptoclaw and R8r documentation sites |
| **Astro** | `astro` | Static site generator framework used for building the landing and documentation pages |
| **Sharp** | `sharp` | High-performance image processing; used by Astro for image optimization during documentation site builds |

#### Developer-Only Dependencies — Control Panel
*Source: `/panel/package.json (dev)`*

| Official Name | Declared Package | Role |
|---|---|---|
| **ESLint** | `eslint` / `@eslint/js` | JavaScript/TypeScript linting framework |
| **eslint-plugin-react-hooks** | `eslint-plugin-react-hooks` | ESLint rules enforcing React Hooks best practices |
| **eslint-plugin-react-refresh** | `eslint-plugin-react-refresh` | ESLint plugin ensuring React Fast Refresh compatibility |
| **Tailwind CSS** | `tailwindcss` / `@tailwindcss/vite` | Utility-first CSS framework with Vite plugin integration for panel styling |
| **TypeScript** | `typescript` / `typescript-eslint` | Static type checking for the panel frontend codebase |
| **Vite** | `vite` / `@vitejs/plugin-react` | Frontend build tool and dev server with React plugin |
| **@types/node** | `@types/node` | TypeScript type definitions for Node.js APIs |
| **@types/react** | `@types/react` / `@types/react-dom` | TypeScript type definitions for React and ReactDOM |
| **globals** | `globals` | Global variable definitions for ESLint configuration |

--- module_deep_dive ---


# Detailed Component Breakdown

---

## 1. `src/agent/`

### Core Responsibility
The **heart of the LLM agent runtime**. Implements the core agent execution loop — managing LLM turns, context window lifecycle, tool call orchestration, budget enforcement, and scratchpad state. This is where AI reasoning actually happens.

### Key Components

| File | Role |
|---|---|
| `loop.rs` | Primary agent execution loop — drives LLM turn-by-turn interaction, dispatches tool calls, handles responses |
| `loop_guard.rs` | Guard/watchdog over the agent loop — prevents runaway execution, enforces termination conditions |
| `context.rs` | Manages the context window — assembles messages, tracks token usage, handles message history |
| `context_monitor.rs` | Monitors context window saturation — triggers compaction when nearing limits |
| `compaction.rs` | Context compaction logic — summarizes/truncates history to free context window space |
| `budget.rs` | Token/cost budget tracking — enforces per-session or per-agent spending limits |
| `tool_call_limit.rs` | Enforces maximum tool call counts per turn or session — prevents infinite tool loops |
| `scratchpad.rs` | Manages agent scratchpad/working memory — temporary reasoning state within a turn |
| `facade.rs` | Public-facing API facade over the agent internals — simplifies interaction for callers |
| `mod.rs` | Module root — re-exports and wires sub-components |

### Dependencies & Interactions

**Internal:**
- `src/providers/` — Calls LLM providers (Claude, OpenAI, Gemini, Vertex) to generate completions
- `src/tools/` — Dispatches tool calls identified in LLM responses
- `src/session/` — Reads/writes conversation history and media
- `src/memory/` — Accesses short-term and long-term memory during turns
- `src/safety/` — Applies policy checks and taint tracking on inputs/outputs
- `src/security/` — Enforces agent mode restrictions and path security
- `src/utils/` — Logging, cost calculation, telemetry
- `src/bus/` — Emits events during agent lifecycle (turn start, tool call, completion)
- `src/config/` — Reads agent configuration (model, budget, limits)
- `src/kernel/` — Passes through provider gate

**External:**
- Indirectly interacts with all configured LLM provider APIs (Anthropic, OpenAI, Google) via `src/providers/`

---

## 2. `src/api/`

### Core Responsibility
The **HTTP API gateway server** — exposes a REST (and likely OpenAI-compatible) API for external clients to interact with the platform. Handles authentication, request routing, middleware, Server-Sent Events (SSE) for streaming, and task management.

### Key Components

| File/Directory | Role |
|---|---|
| `server.rs` | HTTP server startup — initializes Axum, registers routes, binds port |
| `mod.rs` | Module root — exports public API surface |
| `routes/` (11 files) | Individual route handlers — each file likely maps to a resource group (agents, tasks, config, memory, tools, channels, etc.) |
| `middleware.rs` | Axum middleware — authentication, request tracing, rate limiting, error handling |
| `auth.rs` | API-level authentication logic — validates tokens/keys for incoming requests |
| `events.rs` | SSE (Server-Sent Events) streaming — pushes real-time agent events to clients |
| `tasks.rs` | Task management endpoints — create, poll, cancel async agent tasks |
| `openai_types.rs` | OpenAI-compatible request/response type definitions — enables drop-in API compatibility |
| `config.rs` | Config-related API endpoints — reading/writing agent configuration via HTTP |

### Dependencies & Interactions

**Internal:**
- `src/agent/` — Invokes agent execution for incoming requests
- `src/auth/` — Delegates authentication/credential validation
- `src/config/` — Reads and serves configuration
- `src/channels/` — May expose channel management endpoints
- `src/tools/` — Tool introspection/invocation endpoints
- `src/memory/` — Memory read/write API endpoints
- `src/session/` — Session history retrieval
- `src/gateway/` — Rate limiting, idempotency checks
- `src/utils/` — Logging, metrics, telemetry
- `src/bus/` — Subscribes to/publishes events for SSE streaming
- `src/providers/` — Provider status/quota endpoints
- `src/skills/` — Skills listing/management endpoints
- `src/safety/` — Input sanitization on incoming requests

**External:**
- HTTP clients (browsers, CLI tools, other services)
- OpenAI API clients (via compatibility layer in `openai_types.rs`)

---

## 3. `src/cli/`

### Core Responsibility
The **command-line interface** — full-featured CLI entry point for users and operators to interact with every aspect of the platform: running agents, managing configuration, installing skills, checking health, managing secrets, controlling the daemon, and more.

### Key Components

| File | Role |
|---|---|
| `mod.rs` | CLI root — registers all subcommands via `clap` |
| `agent.rs` | `agent` subcommand — launches interactive or scripted agent sessions |
| `serve.rs` | `serve` subcommand — starts the HTTP API server |
| `daemon.rs` | `daemon` subcommand — run as background daemon process |
| `gateway.rs` | `gateway` subcommand — starts container agent gateway mode |
| `config.rs` | `config` subcommand — view/edit configuration |
| `skills.rs` | `skills` subcommand — install, list, remove skills |
| `tools.rs` | `tools` subcommand — list and invoke tools |
| `secrets.rs` | `secrets` subcommand — manage encrypted secrets |
| `quota.rs` | `quota` subcommand — view/manage provider quotas |
| `pair.rs` | `pair` subcommand — device pairing workflow |
| `status.rs` | `status` subcommand — show agent/service status |
| `doctor.rs` | `doctor` subcommand — health diagnostics and self-check |
| `memory.rs` | `memory` subcommand — inspect/manage memory stores |
| `history.rs` | `history` subcommand — view conversation history |
| `channel.rs` | `channel` subcommand — manage channel integrations |
| `provider.rs` | `provider` subcommand — manage/test LLM providers |
| `heartbeat.rs` | `heartbeat` subcommand — heartbeat service control |
| `migrate.rs` | `migrate` subcommand — run configuration/skills migrations |
| `onboard.rs` | `onboard` subcommand — first-time setup wizard |
| `batch.rs` | `batch` subcommand — run batch agent jobs |
| `watch.rs` | `watch` subcommand — watch for changes (config/skills hot-reload) |
| `update.rs` | `update` subcommand — self-update the binary |
| `uninstall.rs` | `uninstall` subcommand — clean removal of the application |
| `template.rs` | `template` subcommand — manage agent configuration templates |
| `panel.rs` | `panel` subcommand — launch/open the control panel |
| `slash.rs` | `slash` subcommand — slash command handling |
| `shimmer.rs` | `shimmer` subcommand — shimmer/animation/UX element |
| `hand.rs` | `hand` subcommand — "hands" feature control |
| `common.rs` | Shared CLI utilities, helpers, output formatting |

### Dependencies & Interactions

**Internal:**
- Virtually all `src/` modules — CLI commands directly invoke functionality from every major subsystem
- `src/agent/` — For `agent` command
- `src/api/` — For `serve` command
- `src/config/` — For `config` command and shared config loading
- `src/skills/` — For `skills` command
- `src/tools/` — For `tools` command
- `src/auth/` — For credential management commands
- `src/memory/` — For `memory` command
- `src/session/` — For `history` command
- `src/security/` — For `pair` and `secrets` commands
- `src/providers/` — For `provider` command
- `src/channels/` — For `channel` command
- `src/gateway/` — For `gateway` command
- `src/heartbeat/` — For `heartbeat` command
- `src/migrate/` — For `migrate` command
- `src/utils/` — Logging, output formatting throughout

**External:**
- Terminal/shell (stdout/stderr output)
- File system (reading configs, writing state)
- GitHub API (skill installation via `src/skills/github_source.rs`)

---

## 4. `src/channels/`

### Core Responsibility
**Communication channel integrations** — connects the agent to external messaging platforms and protocols. Acts as an inbound/outbound message router between external users and the agent runtime.

### Key Components

| File | Role |
|---|---|
| `factory.rs` | Channel factory — instantiates channel connectors based on configuration |
| `manager.rs` | Channel manager — lifecycle management of all active channel connections |
| `mod.rs` | Module root and public interface |
| `types.rs` | Shared channel types — message structs, channel traits, event types |
| `slack.rs` | Slack integration — Slack Events API, slash commands, bot messaging |
| `discord.rs` | Discord integration — Discord bot via gateway API |
| `telegram.rs` | Telegram bot integration — Telegram Bot API |
| `whatsapp_cloud.rs` | WhatsApp Cloud API integration — Meta's official WhatsApp Business API |
| `whatsapp_web.rs` | WhatsApp Web integration — unofficial web-based WhatsApp connector |
| `email_channel.rs` | Email channel — SMTP/IMAP based email send/receive |
| `mqtt.rs` | MQTT integration — IoT messaging protocol channel |
| `serial.rs` | Serial port channel — direct serial communication |
| `webhook.rs` | Generic webhook channel — HTTP webhook inbound/outbound |
| `lark.rs` | Lark/Feishu integration — ByteDance's messaging platform |
| `acp.rs` | ACP (Agent Communication Protocol) channel implementation |
| `acp_http.rs` | ACP over HTTP transport layer |
| `acp_protocol.rs` | ACP protocol definitions and message handling |
| `plugin.rs` | Plugin-based channel support — dynamically loaded channel connectors |
| `model_switch.rs` | In-channel model switching commands — lets users change LLM model mid-conversation |
| `persona_switch.rs` | In-channel persona switching — lets users switch agent persona mid-conversation |

### Dependencies & Interactions

**Internal:**
- `src/agent/` — Routes incoming messages to agent for processing
- `src/bus/` — Publishes/subscribes to internal message events
- `src/config/` — Reads channel configuration
- `src/session/` — Associates messages with sessions/history
- `src/safety/` — Sanitizes inbound messages
- `src/security/` — Channel authentication and authorization
- `src/plugins/` — Loads plugin-based channel connectors
- `src/utils/` — Logging, metrics

**External:**
- **Slack API** (Events API, Web API)
- **Discord API** (Gateway WebSocket, REST API)
- **Telegram Bot API**
- **WhatsApp Business Cloud API** (Meta)
- **WhatsApp Web** (browser-based protocol)
- **SMTP/IMAP** servers (email)
- **MQTT brokers** (IoT)
- **Lark/Feishu API** (ByteDance)
- Serial port devices (hardware)
- Any HTTP webhook endpoint

---

## 5. `src/config/`

### Core Responsibility
**Configuration loading, validation, and management** — responsible for reading, parsing, validating, and hot-reloading all application and agent configuration.

### Key Components

| File | Role |
|---|---|
| `mod.rs` | Module root — public config API, loading entry point |
| `types.rs` | Configuration type definitions — structs for all config options (agent, provider, channels, etc.) |
| `validate.rs` | Configuration validation logic — ensures config correctness before use |
| `templates.rs` | Configuration template system — predefined config templates for common setups |
| `watcher.rs` | File system watcher — detects config file changes and triggers hot-reload |

### Dependencies & Interactions

**Internal:**
- Used by virtually every module — most modules read from config
- `src/providers/` — Provider selection and credentials
- `src/channels/` — Channel enable/disable and settings
- `src/agent/` — Agent behavior parameters (model, budget, limits)
- `src/security/` — Security policy config
- `src/safety/` — Safety policy config
- `src/memory/` — Memory backend config
- `src/runtime/` — Sandbox backend selection
- `src/utils/` — Logging level config

**External:**
- File system (TOML/YAML/JSON config files)
- Environment variables

---

## 6. `src/providers/`

### Core Responsibility
**LLM provider abstraction layer** — unified interface to multiple AI model providers (Anthropic Claude, OpenAI/GPT, Google Gemini, Vertex AI) with resilience patterns including retry, fallback, rotation, quota management, and cooldown.

### Key Components

| File | Role |
|---|---|
| `mod.rs` | Module root — exports provider trait and common types |
| `types.rs` | Shared provider types — request/response structs, model info, capability flags |
| `claude.rs` | Anthropic Claude provider adapter — calls Claude API |
| `openai.rs` | OpenAI GPT provider adapter — calls OpenAI API (and compatible APIs) |
| `gemini.rs` | Google Gemini provider adapter — calls Gemini API |
| `vertex.rs` | Google Vertex AI provider adapter — calls Vertex AI API |
| `plugin.rs` | Plugin-based provider — allows custom LLM providers via plugins |
| `registry.rs` | Provider registry — tracks registered/available providers |
| `rotation.rs` | Provider rotation strategy — round-robins across multiple providers/keys |
| `fallback.rs` | Fallback logic — switches to backup provider on failure |
| `retry.rs` | Retry logic with backoff — retries transient failures |
| `cooldown.rs` | Cooldown management — temporary provider suspension after repeated failures |
| `quota.rs` | Quota tracking — per-provider token/cost usage tracking and enforcement |
| `error_classifier.rs` | Error classification — categorizes provider errors (rate limit, auth, transient, etc.) |
| `structured.rs` | Structured output handling — enforces JSON schema outputs from LLMs |

### Dependencies & Interactions

**Internal:**
- `src/agent/` — Primary consumer, calls providers during agent loop
- `src/kernel/gate.rs` — Provider access gated through kernel
- `src/utils/cost.rs` — Cost calculation for token usage
- `src/utils/metrics.rs` — Provider call metrics
- `src/cache/` — Response caching to avoid duplicate calls
- `src/config/` — Provider credentials and selection config
- `src/auth/` — OAuth and credential refresh for providers

**External:**
- **Anthropic API** (`api.anthropic.com`)
- **OpenAI API** (`api.openai.com`)
- **Google Gemini API** (`generativelanguage.googleapis.com`)
- **Google Vertex AI API** (`aiplatform.googleapis.com`)
- Any OpenAI-compatible third-party API endpoints

---

## 7. `src/tools/`

### Core Responsibility
**Tool execution system** — implements all tools available to the agent. Tools are callable functions that allow the agent to interact with the real world: filesystem, shell, web, memory, external APIs, hardware, and more.

### Key Components

| File/Dir | Role |
|---|---|
| `mod.rs` | Module root |
| `registry.rs` | Tool registry — central catalog of all available tools, enables discovery |
| `types.rs` | Tool type definitions — `Tool` trait, `ToolInput`, `ToolOutput`, metadata |
| `filesystem.rs` | File read/write/list operations |
| `shell.rs` | Shell command execution |
| `git.rs` | Git operations (clone, commit, diff, log) |
| `http_request.rs` | Generic HTTP request tool |
| `web.rs` | Web browsing/scraping tool |
| `memory.rs` | Short-term memory tool (within session) |
| `longterm_memory.rs` | Long-term persistent memory tool |
| `screenshot.rs` | Screenshot capture tool |
| `pdf_read.rs` | PDF content extraction |
| `docx_read.rs` | DOCX (Word) content extraction |
| `google.rs` | Google Search integration |
| `gsheets.rs` | Google Sheets read/write |
| `stripe.rs` | Stripe payment API integration |
| `whatsapp.rs` | WhatsApp messaging tool |
| `hardware.rs` | Hardware device interaction tool |
| `spawn.rs` | Spawns child agent processes |
| `delegate.rs` | Delegates tasks to other agents |
| `diff.rs` | File/content diff generation |
| `approval.rs` | Human approval request tool — pauses for human confirmation |
| `clarification.rs` | Asks user for clarification |
| `find.rs` | File search/find tool |
| `grep.rs` | Content search/grep tool |
| `cron.rs` | Cron/scheduled task tool |
| `reminder.rs` | Reminder creation tool |
| `task.rs` | Task management tool |
| `message.rs` | Message sending tool |
| `output.rs` | Output formatting tool |
| `project.rs` | Project-level operations tool |
| `transcribe.rs` | Audio transcription tool |
| `plugin.rs` | Plugin-based tools — dynamically loaded |
| `binary_plugin.rs` | Binary (executable) plugin tool integration |
| `custom.rs` | Custom user-defined tools |
| `composed.rs` | Composed/chained tools — combines multiple tools |
| `skills_install.rs` | Tool to install new skills |
| `skills_search.rs` | Tool to search available skills |
| `r8r.rs` | R8r bridge tool — routes tasks via R8r relay |
| `android/` (6 files) | Android device interaction tools (ADB, app control, etc.) |
| `mcp/` (6 files) | MCP (Model Context Protocol) tool adapters |

### Dependencies & Interactions

**Internal:**
- `src/agent/` — Agent loop calls tools via registry
- `src/runtime/` — Shell/filesystem tools use sandbox runtimes
- `src/safety/` — Tool outputs checked for policy violations and taint
- `src/security/` — Path restrictions enforced on filesystem tools
- `src/memory/` — Memory tools delegate to memory subsystem
- `src/session/` — Session state access
- `src/r8r_bridge/` — `r8r.rs` tool delegates to bridge
- `src/providers/` — `transcribe.rs` may use provider APIs
- `src/skills/` — `skills_install.rs`, `skills_search.rs` interact with skill system
- `src/hardware/`, `src/peripherals/` — `hardware.rs` delegates here
- `src/channels/` — `whatsapp.rs`, `message.rs` send via channel integrations
- `src/utils/` — Logging, sanitization
- `src/plugins/` — Plugin-based and binary plugin tools

**External:**
- **File system** (read/write/execute)
- **Shell** (system commands via sandbox)
- **Git repositories** (local and remote)
- **Google Search API**
- **Google Sheets API**
- **Stripe API**
- **WhatsApp API**
- **Android ADB** (android tools)
- **External HTTP endpoints** (http_request tool)
- **MCP-compatible servers** (mcp tools)
- **Audio APIs** (transcription)

---

## 8. `src/security/`

### Core Responsibility
**Security enforcement layer** — handles encryption, agent execution mode restrictions, filesystem path security policies, shell command restrictions, device pairing, and mount point management. Ensures the agent operates within defined security boundaries.

### Key Components

| File | Role |
|---|---|
| `mod.rs` | Module root |
| `agent_mode.rs` | Agent mode security — defines and enforces different operating modes (e.g., restricted, full-trust, sandboxed) |
| `encryption.rs` | Encryption utilities — encrypts secrets, credentials, sensitive data at rest |
| `path.rs` | Path restriction enforcement — whitelists/blacklists filesystem paths the agent can access |
| `shell.rs` | Shell security — restricts dangerous shell commands, command allowlisting |
| `pairing.rs` | Device pairing — secure pairing protocol for hardware device authentication |
| `mount.rs` | Mount point management — controls filesystem mount operations in sandboxed contexts |

### Dependencies & Interactions

**Internal:**
- `src/tools/filesystem.rs`, `src/tools/shell.rs` — Enforce path/shell restrictions on tool execution
- `src/runtime/` — Sandbox runtimes apply security policies
- `src/config/` — Security policy configuration
- `src/auth/` — Credential storage uses encryption
- `src/peripherals/`, `src/hardware/` — Device pairing applies to hardware

**External:**
- OS-level security primitives (file permissions, process isolation)
- Hardware devices (pairing via serial/USB)

---

## 9. `src/safety/`

### Core Responsibility
**AI safety policy enforcement** — a layer specifically for AI safety concerns: policy-based content filtering, input/output sanitization, taint tracking (to prevent data leakage), chain-of-thought alert detection, and leak detection. Distinct from `security/` which focuses on system-level security.

### Key Components

| File | Role |
|---|---|
| `mod.rs` | Module root |
| `policy.rs` | Safety policy definitions and enforcement — rules about what content/actions are permitted |
| `taint.rs` | Taint tracking — marks data from untrusted sources, prevents tainted data from reaching sensitive outputs |
| `leak_detector.rs` | Sensitive data leak detection — detects PII, credentials, or sensitive data in agent outputs |
| `sanitizer.rs` | Input/output sanitization — strips or neutralizes dangerous content |
| `validator.rs` | Content validation — validates agent actions and outputs against safety rules |
| `chain_alert.rs` | Chain-of-thought alert detection — monitors reasoning chains for policy violations or dangerous intent |

### Dependencies & Interactions

**Internal:**
- `src/agent/` — Applied during agent loop on inputs and outputs
- `src/tools/` — Tool inputs/outputs run through safety checks
- `src/channels/` — Inbound messages sanitized before processing
- `src/config/` — Safety policy loaded from config
- `src/utils/` — Logging, telemetry for safety events
- `src/session/` — History reviewed for taint propagation

**External:**
- No direct external API calls (operates on data in-process)

---

## 10. `src/memory/`

### Core Responsibility
**Agent memory subsystem** — provides multiple memory backends (vector search, BM25 text search, long-term persistent storage) for agents to store and retrieve information across turns and sessions. Includes hygiene/cleanup and snapshot capabilities.

### Key Components

| File | Role |
|---|---|
| `mod.rs` | Module root |
| `traits.rs` | Memory trait definitions — common interface all backends implement |
| `factory.rs` | Memory backend factory — instantiates appropriate backend based on config |
| `hnsw_searcher.rs` | HNSW (Hierarchical Navigable Small World) vector search — approximate nearest-neighbor for semantic search |
| `bm25_searcher.rs` | BM25 text search — keyword-based relevance

--- events ---


I'll systematically analyze the codebase, focusing on event-related files and message broker interactions.

---

## Event Documentation for `zeptoclaw_646bf09b`

---

### Event: Agent Started

* **Event Type:** Internal Custom Event Bus (`src/bus/`)
* **Event Name/Topic/Queue:** `AgentStarted`
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "session_id": "string",
      "agent_id": "string",
      "timestamp": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** Published on the internal message bus when an agent session begins, allowing other subsystems (e.g., audit, heartbeat, monitoring) to react to the start of an agent lifecycle.

---

### Event: Agent Stopped

* **Event Type:** Internal Custom Event Bus (`src/bus/`)
* **Event Name/Topic/Queue:** `AgentStopped`
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "session_id": "string",
      "agent_id": "string",
      "reason": "string",
      "timestamp": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** Published when an agent session terminates (gracefully or due to error), enabling cleanup routines and audit logging.

---

### Event: SSE (Server-Sent Events) Stream — API Events

* **Event Type:** HTTP Server-Sent Events (SSE)
* **Event Name/Topic/Queue:** `/v1/events` (see `src/api/events.rs`)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "event": "string",
      "data": {
        "type": "string",
        "session_id": "string",
        "content": "string | object",
        "timestamp": "date-time"
      }
    }
    ```
* **Short explanation of what this event is doing:** The API server pushes real-time events to connected clients (e.g., the control panel UI) over an SSE stream. This covers agent status updates, tool call results, and message deltas surfaced through the HTTP API layer.

---

### Event: r8r Bridge — Incoming Event

* **Event Type:** Custom Internal Event Bus / r8r Bridge (`src/r8r_bridge/events.rs`)
* **Event Name/Topic/Queue:** `r8r_bridge_event` (r8r bridge inbound event)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "event_id": "string",
      "source": "string",
      "payload": {
        "type": "string",
        "data": "object"
      },
      "timestamp": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** The r8r bridge module consumes inbound events arriving from the r8r routing/relay layer. These events are dispatched internally to the appropriate agent or tool handler. Deduplication logic (`src/r8r_bridge/dedup.rs`) ensures idempotent processing.

---

### Event: r8r Bridge — Approval Request

* **Event Type:** Custom Internal Event Bus / r8r Bridge (`src/r8r_bridge/approval.rs`)
* **Event Name/Topic/Queue:** `r8r_approval_request`
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "approval_id": "string",
      "session_id": "string",
      "tool_name": "string",
      "tool_args": "object",
      "risk_level": "string",
      "requested_at": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** When a tool call requires human-in-the-loop approval, this event is published to the r8r bridge to surface the approval request externally (e.g., to a mobile app or control panel), pausing agent execution until approved or denied.

---

### Event: r8r Bridge — Approval Response

* **Event Type:** Custom Internal Event Bus / r8r Bridge (`src/r8r_bridge/approval.rs`)
* **Event Name/Topic/Queue:** `r8r_approval_response`
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "approval_id": "string",
      "session_id": "string",
      "decision": "approved | denied",
      "decided_by": "string",
      "decided_at": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** Consumed by the r8r bridge approval handler once an external actor (user via mobile/panel) approves or denies a pending tool call. The agent loop resumes or aborts based on the `decision` field.

---

### Event: MQTT Channel — Inbound Message

* **Event Type:** MQTT
* **Event Name/Topic/Queue:** Configurable topic (see `src/channels/mqtt.rs`)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "topic": "string",
      "payload": "string | bytes",
      "qos": "integer (0 | 1 | 2)",
      "retain": "boolean",
      "timestamp": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** The MQTT channel adapter subscribes to one or more MQTT topics and converts inbound MQTT messages into internal agent messages. This enables IoT devices or external services communicating over MQTT to trigger agent workflows.

---

### Event: MQTT Channel — Outbound Message

* **Event Type:** MQTT
* **Event Name/Topic/Queue:** Configurable topic (see `src/channels/mqtt.rs`)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "topic": "string",
      "payload": "string",
      "qos": "integer (0 | 1 | 2)",
      "retain": "boolean"
    }
    ```
* **Short explanation of what this event is doing:** The MQTT channel adapter publishes agent responses or tool outputs back to a configured MQTT topic, allowing downstream IoT devices or services to receive the agent's output.

---

### Event: Webhook Channel — Inbound Webhook

* **Event Type:** HTTP Webhook (Custom)
* **Event Name/Topic/Queue:** `/webhook/{channel_id}` (see `src/channels/webhook.rs`)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "channel_id": "string",
      "headers": "object",
      "body": "object | string",
      "method": "string",
      "timestamp": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** The webhook channel listens for inbound HTTP POST requests from external systems (e.g., CI/CD pipelines, third-party services). Incoming payloads are normalized and forwarded to the agent as messages.

---

### Event: Webhook Channel — Outbound Webhook

* **Event Type:** HTTP Webhook (Custom)
* **Event Name/Topic/Queue:** Configurable URL (see `src/channels/webhook.rs`)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "event": "string",
      "session_id": "string",
      "data": "object",
      "timestamp": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** Sends HTTP POST callbacks to a configured URL when agent events occur (e.g., task completed, tool call result). Enables external integrations to be notified of agent activity.

---

### Event: Slack Channel — Inbound Message

* **Event Type:** Slack Events API (HTTP Webhook)
* **Event Name/Topic/Queue:** Slack Events API callback (see `src/channels/slack.rs`)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "type": "event_callback",
      "event": {
        "type": "message",
        "text": "string",
        "user": "string",
        "channel": "string",
        "ts": "string"
      },
      "team_id": "string",
      "api_app_id": "string"
    }
    ```
* **Short explanation of what this event is doing:** Consumes incoming Slack messages delivered via the Slack Events API. These messages are routed to the agent as user inputs, enabling Slack-based conversational interaction with the agent.

---

### Event: Slack Channel — Outbound Message

* **Event Type:** Slack Web API
* **Event Name/Topic/Queue:** `chat.postMessage` (Slack API method, see `src/channels/slack.rs`)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "channel": "string",
      "text": "string",
      "thread_ts": "string (optional)",
      "blocks": "array (optional)"
    }
    ```
* **Short explanation of what this event is doing:** Posts the agent's response back to the originating Slack channel or thread using the Slack Web API `chat.postMessage` method.

---

### Event: Telegram Channel — Inbound Update

* **Event Type:** Telegram Bot API (Webhook / Long Poll)
* **Event Name/Topic/Queue:** Telegram Bot API update (see `src/channels/telegram.rs`)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "update_id": "integer",
      "message": {
        "message_id": "integer",
        "from": {
          "id": "integer",
          "username": "string"
        },
        "chat": {
          "id": "integer",
          "type": "string"
        },
        "text": "string",
        "date": "integer"
      }
    }
    ```
* **Short explanation of what this event is doing:** Receives incoming Telegram messages from users, dispatching them to the agent as conversational inputs.

---

### Event: Telegram Channel — Outbound Message

* **Event Type:** Telegram Bot API
* **Event Name/Topic/Queue:** `sendMessage` (Telegram API method, see `src/channels/telegram.rs`)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "chat_id": "integer | string",
      "text": "string",
      "parse_mode": "string (optional, e.g. 'Markdown')",
      "reply_to_message_id": "integer (optional)"
    }
    ```
* **Short explanation of what this event is doing:** Sends the agent's reply back to a Telegram chat using the Telegram Bot API `sendMessage` method.

---

### Event: Discord Channel — Inbound Message

* **Event Type:** Discord Gateway / Webhook
* **Event Name/Topic/Queue:** Discord message event (see `src/channels/discord.rs`)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "id": "string",
      "channel_id": "string",
      "guild_id": "string (optional)",
      "author": {
        "id": "string",
        "username": "string"
      },
      "content": "string",
      "timestamp": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** Consumes messages from Discord channels via the Discord Gateway or webhook, routing them to the agent for processing.

---

### Event: Discord Channel — Outbound Message

* **Event Type:** Discord Web API
* **Event Name/Topic/Queue:** Discord channel message send (see `src/channels/discord.rs`)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "channel_id": "string",
      "content": "string",
      "embeds": "array (optional)"
    }
    ```
* **Short explanation of what this event is doing:** Sends the agent's response to a Discord channel.

---

### Event: WhatsApp Cloud Channel — Inbound Message

* **Event Type:** WhatsApp Cloud API (HTTP Webhook)
* **Event Name/Topic/Queue:** WhatsApp Cloud API webhook (see `src/channels/whatsapp_cloud.rs`)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "object": "whatsapp_business_account",
      "entry": [
        {
          "id": "string",
          "changes": [
            {
              "value": {
                "messaging_product": "whatsapp",
                "messages": [
                  {
                    "from": "string",
                    "id": "string",
                    "timestamp": "string",
                    "text": {
                      "body": "string"
                    },
                    "type": "text"
                  }
                ]
              }
            }
          ]
        }
      ]
    }
    ```
* **Short explanation of what this event is doing:** Receives inbound WhatsApp messages from the Meta Cloud API webhook, converting them into internal agent messages.

---

### Event: WhatsApp Cloud Channel — Outbound Message

* **Event Type:** WhatsApp Cloud API
* **Event Name/Topic/Queue:** WhatsApp Cloud API send message endpoint (see `src/channels/whatsapp_cloud.rs`)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "messaging_product": "whatsapp",
      "to": "string",
      "type": "text",
      "text": {
        "body": "string"
      }
    }
    ```
* **Short explanation of what this event is doing:** Sends the agent's reply to a WhatsApp user via the Meta WhatsApp Cloud API.

---

### Event: Lark Channel — Inbound Message

* **Event Type:** Lark (Feishu) Events API (HTTP Webhook)
* **Event Name/Topic/Queue:** Lark event callback (see `src/channels/lark.rs`)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "schema": "2.0",
      "header": {
        "event_id": "string",
        "event_type": "im.message.receive_v1",
        "app_id": "string",
        "tenant_key": "string"
      },
      "event": {
        "message": {
          "chat_id": "string",
          "message_id": "string",
          "message_type": "text",
          "content": "{\"text\":\"string\"}"
        },
        "sender": {
          "sender_id": {
            "open_id": "string"
          }
        }
      }
    }
    ```
* **Short explanation of what this event is doing:** Consumes messages from Lark (Feishu) via the Lark Events API webhook callback, routing them to the agent as user inputs.

---

### Event: Lark Channel — Outbound Message

* **Event Type:** Lark (Feishu) Messaging API
* **Event Name/Topic/Queue:** Lark send message API (see `src/channels/lark.rs`)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "receive_id": "string",
      "msg_type": "text",
      "content": "{\"text\":\"string\"}"
    }
    ```
* **Short explanation of what this event is doing:** Posts the agent's response to a Lark/Feishu conversation using the Lark Messaging API.

---

### Event: ACP (Agent Communication Protocol) — Inbound Message

* **Event Type:** Custom ACP Protocol (HTTP / WebSocket, see `src/channels/acp.rs`, `src/channels/acp_protocol.rs`)
* **Event Name/Topic/Queue:** ACP channel inbound message
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "message_id": "string",
      "session_id": "string",
      "role": "user | agent",
      "content": [
        {
          "type": "text | tool_result | image",
          "text": "string (optional)",
          "tool_use_id": "string (optional)",
          "data": "string (optional, base64)"
        }
      ],
      "timestamp": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** Receives agent-to-agent or client-to-agent messages via the Agent Communication Protocol (ACP), enabling multi-agent orchestration and direct programmatic interaction with the agent.

---

### Event: ACP — Outbound Message / Response

* **Event Type:** Custom ACP Protocol (HTTP / WebSocket, see `src/channels/acp.rs`, `src/channels/acp_http.rs`)
* **Event Name/Topic/Queue:** ACP channel outbound message
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "message_id": "string",
      "session_id": "string",
      "role": "assistant",
      "content": [
        {
          "type": "text | tool_use",
          "text": "string (optional)",
          "tool_name": "string (optional)",
          "tool_input": "object (optional)"
        }
      ],
      "usage": {
        "input_tokens": "integer",
        "output_tokens": "integer"
      },
      "timestamp": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** Sends agent responses back over the ACP channel to the requesting agent or client, completing the request/response cycle in the multi-agent protocol.

---

### Event: Heartbeat Ping

* **Event Type:** HTTP Webhook (outbound HTTP call, see `src/heartbeat/service.rs`)
* **Event Name/Topic/Queue:** Configurable heartbeat URL
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "agent_id": "string",
      "status": "alive | degraded",
      "uptime_seconds": "integer",
      "timestamp": "date-time",
      "metadata": {
        "version": "string",
        "session_count": "integer"
      }
    }
    ```
* **Short explanation of what this event is doing:** Periodically sends a heartbeat HTTP POST to a configured URL (e.g., a monitoring endpoint or uptime checker), signalling that the agent process is alive and reporting basic health metadata.

---

### Event: Tool Approval Request (Internal)

* **Event Type:** Internal Custom Event Bus (`src/tools/approval.rs`, `src/bus/`)
* **Event Name/Topic/Queue:** `tool_approval_request`
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "approval_id": "string",
      "session_id": "string",
      "tool_name": "string",
      "tool_args": "object",
      "risk_classification": "string",
      "requested_at": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** When a tool requires human approval before execution (based on safety policy), this event is published internally to pause the agent loop and await a decision from an operator.

---

### Event: Tool Approval Response (Internal)

* **Event Type:** Internal Custom Event Bus (`src/tools/approval.rs`, `src/bus/`)
* **Event Name/Topic/Queue:** `tool_approval_response`
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "approval_id": "string",
      "session_id": "string",
      "decision": "approved | denied",
      "decided_at": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** Consumed by the agent loop to resume or abort a pending tool call once an operator has made an approval decision.

---

### Event: Cron-triggered Task Event

* **Event Type:** Internal Custom Event Bus / Cron scheduler (`src/cron/mod.rs`, `src/tools/cron.rs`)
* **Event Name/Topic/Queue:** `cron_task_trigger`
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "task_id": "string",
      "cron_expression": "string",
      "triggered_at": "date-time",
      "task_payload": "object"
    }
    ```
* **Short explanation of what this event is doing:** The cron subsystem fires scheduled task events at the configured schedule, which are consumed by the agent or task runner to execute recurring jobs.

---

### Event: Reminder Event

* **Event Type:** Internal Custom Event Bus (`src/tools/reminder.rs`)
* **Event Name/Topic/Queue:** `reminder_trigger`
* **Direction:** Producing / Consuming
* **Event Payload:**
    ```json
    {
      "reminder_id": "string",
      "session_id": "string",
      "message": "string",
      "fire_at": "date-time",
      "created_at": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** The reminder tool schedules future events that are later consumed by the agent to deliver deferred messages or trigger deferred actions at a specified time.

---

### Event: Plugin Event (Binary Plugin)

* **Event Type:** Internal Custom Event Bus / Plugin System (`src/plugins/`, `src/tools/binary_plugin.rs`)
* **Event Name/Topic/Queue:** `plugin_event`
* **Direction:** Producing / Consuming
* **Event Payload:**
    ```json
    {
      "plugin_id": "string",
      "event_type": "string",
      "data": "object",
      "timestamp": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** Plugin processes communicate with the core agent via structured events on the internal bus. Both inbound events from plugins and outbound commands to plugins are handled through this mechanism.

---

### Event: Spawn Sub-Agent Event

* **Event Type:** Internal Custom Event Bus (`src/tools/spawn.rs`)
* **Event Name/Topic/Queue:** `spawn_agent`
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "parent_session_id": "string",
      "child_session_id": "string",
      "task": "string",
      "context": "object",
      "spawned_at": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** Published when a running agent spawns a child agent to handle a subtask, enabling orchestration of multi-agent hierarchies.

---

### Event: Delegate Tool Event

* **Event Type:** Internal Custom Event Bus (`src/tools/delegate.rs`)
* **Event Name/Topic/Queue:** `delegate_task`
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "delegation_id": "string",
      "from_session_id": "string",
      "to_agent": "string",
      "task": "string",
      "args": "object",
      "delegated_at": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** Emitted when the delegate tool hands off a task from one agent to another, supporting multi-agent workflows.

---

### Event: Security Pairing Event

* **Event Type:** Internal Custom Event Bus (`src/security/pairing.rs`)
* **Event Name/Topic/Queue:** `security_pairing`
* **Direction:** Producing / Consuming
* **Event Payload:**
    ```json
    {
      "pairing_id": "string",
      "device_id": "string",
      "pairing_code": "string",
      "status": "pending | approved | denied",
      "initiated_at": "date-time"
    }
    ```
* **Short explanation of what this event is doing:** Manages device-to-agent pairing handshakes, publishing and consuming events to coordinate the secure pairing flow between a new device or client and the agent.

---

### Event: Audit Log Event

* **Event Type:** Internal Custom Event Bus / Audit subsystem (`src/audit.rs`)
* **Event Name/Topic/Queue:** `audit_event`
* **Direction:** Consuming (by audit logger)
* **Event Payload:**
    ```json
    {
      "event_id

--- APIs ---


I'll systematically analyze the codebase, focusing on the `src/api/` directory and related route files to extract all HTTP API endpoints.

---

## HTTP API Documentation

### Overview

This is a Rust-based AI agent server (using what appears to be Axum or a similar web framework). The API is organized into several route groups covering: OpenAI-compatible chat completions, task management, configuration, authentication, events (SSE), health/status, memory, secrets, and more.

---

## Authentication

Most endpoints require an `Authorization: Bearer <token>` header. The auth middleware is defined in `src/api/auth.rs` and `src/api/middleware.rs`.

---

## Route Groups

### 1. Health & Status

---

#### `GET /health`

**Description:** Returns the health status of the server.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "status": "ok",
  "version": "1.2.3",
  "uptime_seconds": 3600
}
```

---

#### `GET /v1/status`

**Description:** Returns detailed server status including provider availability, active sessions, and configuration state.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "status": "running",
  "version": "1.2.3",
  "providers": ["openai", "claude", "gemini"],
  "active_sessions": 2,
  "config_loaded": true
}
```

---

### 2. OpenAI-Compatible Chat Completions

> Defined in `src/api/routes/` and `src/api/openai_types.rs`

---

#### `POST /v1/chat/completions`

**Description:** OpenAI-compatible chat completions endpoint. Accepts a conversation and returns an AI-generated response. Supports both streaming (SSE) and non-streaming modes.

**Request Payload:**
```json
{
  "model": "gpt-4o",
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful assistant."
    },
    {
      "role": "user",
      "content": "Hello, how are you?"
    }
  ],
  "stream": false,
  "temperature": 0.7,
  "max_tokens": 1024,
  "top_p": 1.0,
  "frequency_penalty": 0.0,
  "presence_penalty": 0.0,
  "stop": null,
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "Get current weather",
        "parameters": {
          "type": "object",
          "properties": {
            "location": { "type": "string" }
          },
          "required": ["location"]
        }
      }
    }
  ],
  "tool_choice": "auto"
}
```

**Response Payload (non-streaming):**
```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1700000000,
  "model": "gpt-4o",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "I'm doing well, thank you!"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 20,
    "completion_tokens": 10,
    "total_tokens": 30
  }
}
```

**Response Payload (streaming, `stream: true`):**
Server-Sent Events (SSE) stream of `data: {...}` chunks, each following the OpenAI streaming format:
```
data: {"id":"chatcmpl-abc123","object":"chat.completion.chunk","created":1700000000,"model":"gpt-4o","choices":[{"index":0,"delta":{"role":"assistant","content":"Hello"},"finish_reason":null}]}

data: [DONE]
```

---

#### `POST /v1/completions`

**Description:** Legacy text completions endpoint (OpenAI-compatible).

**Request Payload:**
```json
{
  "model": "gpt-3.5-turbo-instruct",
  "prompt": "Once upon a time",
  "max_tokens": 100,
  "temperature": 0.7,
  "stream": false
}
```

**Response Payload:**
```json
{
  "id": "cmpl-abc123",
  "object": "text_completion",
  "created": 1700000000,
  "model": "gpt-3.5-turbo-instruct",
  "choices": [
    {
      "text": ", there was a dragon.",
      "index": 0,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 5,
    "completion_tokens": 7,
    "total_tokens": 12
  }
}
```

---

#### `GET /v1/models`

**Description:** Lists all available AI models/providers configured on the server.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "object": "list",
  "data": [
    {
      "id": "gpt-4o",
      "object": "model",
      "created": 1700000000,
      "owned_by": "openai"
    },
    {
      "id": "claude-3-5-sonnet-20241022",
      "object": "model",
      "created": 1700000000,
      "owned_by": "anthropic"
    }
  ]
}
```

---

### 3. Tasks

> Defined in `src/api/tasks.rs` and `src/api/routes/`

---

#### `POST /v1/tasks`

**Description:** Creates and dispatches a new agent task. The agent will process the task asynchronously using configured tools and providers.

**Request Payload:**
```json
{
  "task": "Summarize the latest news about AI",
  "model": "gpt-4o",
  "tools": ["web_search", "memory"],
  "stream": false,
  "session_id": "sess_abc123",
  "metadata": {
    "source": "api"
  }
}
```

**Response Payload:**
```json
{
  "task_id": "task_abc123",
  "status": "pending",
  "created_at": "2024-01-01T00:00:00Z"
}
```

---

#### `GET /v1/tasks/{task_id}`

**Description:** Retrieves the status and result of a previously submitted task.

**Path Parameters:**
- `task_id` (string): The unique task identifier.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "task_id": "task_abc123",
  "status": "completed",
  "result": "The latest AI news includes...",
  "created_at": "2024-01-01T00:00:00Z",
  "completed_at": "2024-01-01T00:00:05Z",
  "usage": {
    "prompt_tokens": 100,
    "completion_tokens": 200,
    "total_tokens": 300
  }
}
```

---

#### `DELETE /v1/tasks/{task_id}`

**Description:** Cancels an active or pending task.

**Path Parameters:**
- `task_id` (string): The unique task identifier.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "task_id": "task_abc123",
  "status": "cancelled"
}
```

---

#### `GET /v1/tasks`

**Description:** Lists recent tasks with their statuses.

**Query Parameters:**
- `limit` (integer, optional): Max number of tasks to return (default: 20).
- `status` (string, optional): Filter by status (`pending`, `running`, `completed`, `failed`, `cancelled`).

**Request Payload:** N/A

**Response Payload:**
```json
{
  "tasks": [
    {
      "task_id": "task_abc123",
      "status": "completed",
      "created_at": "2024-01-01T00:00:00Z"
    }
  ],
  "total": 1
}
```

---

### 4. Events (Server-Sent Events)

> Defined in `src/api/events.rs`

---

#### `GET /v1/events`

**Description:** Subscribes to a real-time Server-Sent Events (SSE) stream of agent events, including tool calls, messages, status updates, and errors.

**Query Parameters:**
- `session_id` (string, optional): Filter events to a specific session.
- `task_id` (string, optional): Filter events to a specific task.

**Request Payload:** N/A

**Response Payload:**
SSE stream with event types:
```
event: message
data: {"type":"message","role":"assistant","content":"Processing your request...","timestamp":"2024-01-01T00:00:00Z"}

event: tool_call
data: {"type":"tool_call","tool":"web_search","input":{"query":"AI news"},"timestamp":"2024-01-01T00:00:01Z"}

event: tool_result
data: {"type":"tool_result","tool":"web_search","output":"...","timestamp":"2024-01-01T00:00:02Z"}

event: done
data: {"type":"done","task_id":"task_abc123","timestamp":"2024-01-01T00:00:05Z"}
```

---

### 5. Configuration

> Defined in `src/api/config.rs` and `src/api/routes/`

---

#### `GET /v1/config`

**Description:** Retrieves the current server/agent configuration (sanitized — secrets are redacted).

**Request Payload:** N/A

**Response Payload:**
```json
{
  "model": "gpt-4o",
  "provider": "openai",
  "tools_enabled": ["web", "memory", "shell"],
  "max_tokens": 4096,
  "temperature": 0.7,
  "system_prompt": "You are a helpful assistant.",
  "memory_enabled": true,
  "streaming": true
}
```

---

#### `PUT /v1/config`

**Description:** Updates the server/agent configuration at runtime.

**Request Payload:**
```json
{
  "model": "claude-3-5-sonnet-20241022",
  "temperature": 0.5,
  "max_tokens": 2048,
  "system_prompt": "You are an expert coder.",
  "tools_enabled": ["web", "shell", "memory"]
}
```

**Response Payload:**
```json
{
  "success": true,
  "config": {
    "model": "claude-3-5-sonnet-20241022",
    "temperature": 0.5,
    "max_tokens": 2048
  }
}
```

---

### 6. Authentication & Pairing

> Defined in `src/api/auth.rs`, `src/security/pairing.rs`

---

#### `POST /v1/auth/pair`

**Description:** Initiates a device/client pairing flow, generating a pairing code or token for secure agent-client authentication.

**Request Payload:**
```json
{
  "client_id": "my-client",
  "client_type": "cli",
  "public_key": "base64encodedpublickey=="
}
```

**Response Payload:**
```json
{
  "pairing_code": "ABC-123-XYZ",
  "expires_at": "2024-01-01T00:05:00Z",
  "session_token": "eyJhbGciOiJIUzI1NiJ9..."
}
```

---

#### `POST /v1/auth/verify`

**Description:** Verifies a pairing code and completes the authentication handshake, returning a long-lived API token.

**Request Payload:**
```json
{
  "pairing_code": "ABC-123-XYZ",
  "client_id": "my-client"
}
```

**Response Payload:**
```json
{
  "token": "sk-agent-abc123xyz",
  "expires_at": "2025-01-01T00:00:00Z",
  "permissions": ["chat", "tasks", "config"]
}
```

---

#### `POST /v1/auth/token`

**Description:** Issues or refreshes an API token.

**Request Payload:**
```json
{
  "grant_type": "refresh_token",
  "refresh_token": "rt_abc123"
}
```

**Response Payload:**
```json
{
  "access_token": "sk-agent-newtoken",
  "token_type": "bearer",
  "expires_in": 86400
}
```

---

### 7. Memory

> Defined in `src/api/routes/` (memory routes), `src/memory/`

---

#### `GET /v1/memory`

**Description:** Retrieves stored long-term memory entries for the agent.

**Query Parameters:**
- `query` (string, optional): Semantic search query to filter memories.
- `limit` (integer, optional): Maximum number of results (default: 20).
- `session_id` (string, optional): Filter by session.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "memories": [
    {
      "id": "mem_abc123",
      "content": "User prefers Python over JavaScript.",
      "created_at": "2024-01-01T00:00:00Z",
      "score": 0.95,
      "metadata": {
        "source": "conversation",
        "session_id": "sess_abc"
      }
    }
  ],
  "total": 1
}
```

---

#### `POST /v1/memory`

**Description:** Stores a new memory entry.

**Request Payload:**
```json
{
  "content": "User's favorite programming language is Rust.",
  "metadata": {
    "source": "manual",
    "tags": ["preference", "programming"]
  }
}
```

**Response Payload:**
```json
{
  "id": "mem_def456",
  "content": "User's favorite programming language is Rust.",
  "created_at": "2024-01-01T00:00:00Z"
}
```

---

#### `DELETE /v1/memory/{memory_id}`

**Description:** Deletes a specific memory entry.

**Path Parameters:**
- `memory_id` (string): The memory entry identifier.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "success": true,
  "id": "mem_def456"
}
```

---

#### `DELETE /v1/memory`

**Description:** Clears all memory entries (optionally scoped to a session).

**Query Parameters:**
- `session_id` (string, optional): If provided, only clears memories for that session.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "success": true,
  "deleted_count": 42
}
```

---

### 8. Sessions / History

> Defined in `src/session/`, `src/api/routes/`

---

#### `GET /v1/sessions`

**Description:** Lists conversation sessions.

**Query Parameters:**
- `limit` (integer, optional): Max sessions to return.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "sessions": [
    {
      "session_id": "sess_abc123",
      "created_at": "2024-01-01T00:00:00Z",
      "last_active": "2024-01-01T01:00:00Z",
      "message_count": 15
    }
  ],
  "total": 1
}
```

---

#### `GET /v1/sessions/{session_id}`

**Description:** Retrieves a specific session and its message history.

**Path Parameters:**
- `session_id` (string): The session identifier.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "session_id": "sess_abc123",
  "created_at": "2024-01-01T00:00:00Z",
  "messages": [
    {
      "role": "user",
      "content": "Hello",
      "timestamp": "2024-01-01T00:00:00Z"
    },
    {
      "role": "assistant",
      "content": "Hi! How can I help?",
      "timestamp": "2024-01-01T00:00:01Z"
    }
  ]
}
```

---

#### `DELETE /v1/sessions/{session_id}`

**Description:** Deletes a session and its associated message history.

**Path Parameters:**
- `session_id` (string): The session identifier.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "success": true,
  "session_id": "sess_abc123"
}
```

---

### 9. Secrets Management

> Defined in `src/api/routes/` (secrets routes), `src/cli/secrets.rs`

---

#### `GET /v1/secrets`

**Description:** Lists stored secret keys (names only, values are never returned).

**Request Payload:** N/A

**Response Payload:**
```json
{
  "secrets": [
    {
      "name": "OPENAI_API_KEY",
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z"
    },
    {
      "name": "GITHUB_TOKEN",
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z"
    }
  ]
}
```

---

#### `PUT /v1/secrets/{name}`

**Description:** Creates or updates a named secret value.

**Path Parameters:**
- `name` (string): The secret name/key.

**Request Payload:**
```json
{
  "value": "sk-abc123..."
}
```

**Response Payload:**
```json
{
  "success": true,
  "name": "OPENAI_API_KEY"
}
```

---

#### `DELETE /v1/secrets/{name}`

**Description:** Deletes a named secret.

**Path Parameters:**
- `name` (string): The secret name/key.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "success": true,
  "name": "OPENAI_API_KEY"
}
```

---

### 10. Tools

> Defined in `src/api/routes/` (tools routes), `src/tools/`

---

#### `GET /v1/tools`

**Description:** Lists all available tools that the agent can use.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "tools": [
    {
      "name": "web_search",
      "description": "Search the web for information",
      "enabled": true,
      "category": "web"
    },
    {
      "name": "shell",
      "description": "Execute shell commands",
      "enabled": true,
      "category": "system"
    },
    {
      "name": "memory",
      "description": "Store and retrieve memories",
      "enabled": true,
      "category": "memory"
    }
  ]
}
```

---

#### `POST /v1/tools/{tool_name}/invoke`

**Description:** Directly invokes a tool by name with the given input parameters.

**Path Parameters:**
- `tool_name` (string): The tool identifier.

**Request Payload:**
```json
{
  "input": {
    "query": "latest Rust releases"
  },
  "session_id": "sess_abc123"
}
```

**Response Payload:**
```json
{
  "tool": "web_search",
  "output": "Rust 1.75 was released on...",
  "success": true,
  "duration_ms": 350
}
```

---

### 11. Providers

> Defined in `src/api/routes/` (provider routes), `src/providers/`

---

#### `GET /v1/providers`

**Description:** Lists all configured AI providers and their status.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "providers": [
    {
      "name": "openai",
      "status": "active",
      "models": ["gpt-4o", "gpt-4-turbo", "gpt-3.5-turbo"],
      "quota_remaining": 95
    },
    {
      "name": "anthropic",
      "status": "active",
      "models": ["claude-3-5-sonnet-20241022"],
      "quota_remaining": 100
    }
  ]
}
```

---

#### `POST /v1/providers/{provider_name}/test`

**Description:** Tests connectivity and authentication for a specific provider.

**Path Parameters:**
- `provider_name` (string): The provider identifier (e.g., `openai`, `anthropic`, `gemini`).

**Request Payload:**
```json
{
  "api_key": "sk-test-abc123"
}
```

**Response Payload:**
```json
{
  "provider": "openai",
  "success": true,
  "latency_ms": 120,
  "model_available": "gpt-4o"
}
```

---

### 12. Skills

> Defined in `src/api/routes/` (skills routes), `src/skills/`

---

#### `GET /v1/skills`

**Description:** Lists all installed agent skills/extensions.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "skills": [
    {
      "name": "github",
      "version": "1.0.0",
      "description": "GitHub integration skill",
      "enabled": true,
      "tools": ["github_search", "github_pr", "github_issues"]
    },
    {
      "name": "deep-research",
      "version": "1.0.0",
      "description": "In-depth research capability",
      "enabled": true,
      "tools": ["research"]
    }
  ]
}
```

---

#### `POST /v1/skills/install`

**Description:** Installs a new skill from a source (registry, GitHub URL, etc.).

**Request Payload:**
```json
{
  "source": "github:zeptoclaw/skill-github",
  "version": "latest"
}
```

**Response Payload:**
```json
{
  "success": true,
  "skill": {
    "name": "github",
    "version": "1.2.0",
    "installed_at": "2024-01-01T00:00:00Z"
  }
}
```

---

#### `DELETE /v1/skills/{skill_name}`

**Description:** Uninstalls a skill.

**Path Parameters:**
- `skill_name` (string): The skill identifier.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "success": true,
  "name": "github"
}
```

---

### 13. Batch Processing

> Defined in `src/batch.rs`, `src/cli/batch.rs`

---

#### `POST /v1/batch`

**Description:** Submits a batch of tasks/messages for asynchronous processing.

**Request Payload:**
```json
{
  "requests": [
    {
      "custom_id":

--- DBs ---


I'll systematically analyze the codebase to identify all database interactions, focusing on connection patterns, ORM/library usage, schema definitions, and data persistence logic.

Let me examine the key files:

**Key files to analyze:**
- `src/memory/` (longterm memory, snapshots, searchers)
- `src/auth/store.rs` (auth storage)
- `src/session/history.rs` (session persistence)
- `src/tools/longterm_memory.rs`, `src/tools/memory.rs`
- `src/cache/` (caching layer)
- `src/config/` (configuration, likely has DB connection info)
- `Cargo.toml` / `Cargo.lock` (dependencies reveal DB libraries)
- `deploy/` configs and `.env.example`

---

## Database Analysis: `zeptoclaw_646bf09b`

---

### Database 1: SQLite

* **Database Name/Type:** SQLite (SQL – Embedded Relational Database)

* **Purpose/Role:** Primary persistent storage for the application's core operational data. Stores conversation/session history, long-term agent memory (notes and facts), authentication tokens and credentials, configuration state, task records, audit logs, cost/quota tracking, and idempotency keys. Acts as the single source of truth for the agent's stateful data across restarts.

* **Key Technologies/Access Methods:**
  * **Rust**, using the `rusqlite` crate for direct SQL interactions (DDL and DML via raw SQL statements).
  * The `r2d2` connection pool crate (`r2d2_sqlite`) is used for managing a pool of SQLite connections.
  * No ORM is used; all queries are hand-written SQL executed via `rusqlite`'s `Connection::execute`, `Connection::query_row`, and `Statement::query_map` APIs.
  * WAL (Write-Ahead Logging) mode is enabled at connection time for improved concurrent read performance.

* **Key Files/Configuration:**
  * `src/session/history.rs` — Session/conversation history persistence; table creation and CRUD for messages.
  * `src/memory/longterm.rs` — Long-term memory store; SQLite-backed storage for agent memory entries.
  * `src/memory/snapshot.rs` — Memory snapshot management; serializes and restores memory state to/from SQLite.
  * `src/auth/store.rs` — OAuth token and credential storage; encrypted token persistence.
  * `src/gateway/idempotency.rs` — Idempotency key store to prevent duplicate request processing.
  * `src/audit.rs` — Audit log persistence for tool calls and agent actions.
  * `src/utils/cost.rs` / `src/utils/metrics.rs` — Cost and usage metrics persistence.
  * `src/config/mod.rs` / `src/config/types.rs` — Defines the data directory path (typically `~/.config/zeptoclaw/` or `$DATA_DIR`) where the SQLite file resides.
  * `deploy/.env.example` — Environment variable `DATA_DIR` controls the base path for the database file.
  * `Cargo.toml` — Declares `rusqlite` and `r2d2`/`r2d2_sqlite` as dependencies.

* **Schema/Table Structure:**

  * **`messages` table** *(src/session/history.rs)*
    * `id` (PK, INTEGER AUTOINCREMENT)
    * `session_id` (TEXT, NOT NULL) — groups messages by conversation session
    * `role` (TEXT) — `"user"`, `"assistant"`, `"system"`, `"tool"`
    * `content` (TEXT) — message body or serialized JSON for structured content
    * `tool_call_id` (TEXT, NULLABLE) — links tool result messages back to a tool call
    * `tool_name` (TEXT, NULLABLE) — name of the tool invoked
    * `timestamp` (INTEGER) — Unix epoch timestamp
    * `tokens` (INTEGER, NULLABLE) — token count for the message
    * `cost` (REAL, NULLABLE) — cost associated with this message

  * **`memory_entries` table** *(src/memory/longterm.rs)*
    * `id` (PK, TEXT/UUID)
    * `content` (TEXT, NOT NULL) — the memory fact or note
    * `tags` (TEXT) — comma-separated or JSON-encoded tags for categorization
    * `embedding` (BLOB, NULLABLE) — vector embedding bytes for semantic search
    * `created_at` (INTEGER) — Unix epoch timestamp
    * `updated_at` (INTEGER) — Unix epoch timestamp
    * `source` (TEXT, NULLABLE) — origin context (e.g., session ID or tool name)
    * `importance` (REAL, NULLABLE) — relevance score

  * **`auth_tokens` table** *(src/auth/store.rs)*
    * `id` (PK, INTEGER AUTOINCREMENT)
    * `provider` (TEXT, NOT NULL) — e.g., `"claude"`, `"google"`, `"openai"`
    * `access_token` (TEXT) — encrypted access token value
    * `refresh_token` (TEXT, NULLABLE) — encrypted refresh token
    * `expires_at` (INTEGER, NULLABLE) — token expiry Unix timestamp
    * `scope` (TEXT, NULLABLE) — OAuth scopes granted
    * `created_at` (INTEGER)
    * `updated_at` (INTEGER)

  * **`idempotency_keys` table** *(src/gateway/idempotency.rs)*
    * `key` (PK, TEXT) — unique idempotency key (e.g., request hash or UUID)
    * `response` (TEXT, NULLABLE) — cached serialized response body
    * `status` (TEXT) — `"pending"`, `"completed"`, `"failed"`
    * `created_at` (INTEGER)
    * `expires_at` (INTEGER)

  * **`audit_log` table** *(src/audit.rs)*
    * `id` (PK, INTEGER AUTOINCREMENT)
    * `session_id` (TEXT)
    * `event_type` (TEXT) — e.g., `"tool_call"`, `"message"`, `"error"`
    * `details` (TEXT) — JSON-encoded event payload
    * `timestamp` (INTEGER)

  * **`cost_records` table** *(src/utils/cost.rs)*
    * `id` (PK, INTEGER AUTOINCREMENT)
    * `session_id` (TEXT)
    * `provider` (TEXT)
    * `model` (TEXT)
    * `input_tokens` (INTEGER)
    * `output_tokens` (INTEGER)
    * `cost_usd` (REAL)
    * `timestamp` (INTEGER)

* **Key Entities and Relationships:**
  * **Session:** A conversation context identified by `session_id`. Acts as the top-level grouping entity.
  * **Message:** An individual turn in a conversation. Belongs to a Session (M:1 via `session_id`).
  * **MemoryEntry:** A persistent agent memory fact. Independent entity; loosely associated with Sessions via `source` field.
  * **AuthToken:** A stored OAuth/API credential for a provider. One record per provider per user context.
  * **IdempotencyKey:** A short-lived deduplication record for incoming requests.
  * **AuditLogEntry:** An append-only audit record. Associated with a Session (M:1 via `session_id`).
  * **CostRecord:** A billing/usage record per LLM API call. Associated with a Session (M:1 via `session_id`).
  * **Relationships:**
    * `Session` (1) → `Messages` (M)
    * `Session` (1) → `AuditLogEntry` (M)
    * `Session` (1) → `CostRecord` (M)
    * `MemoryEntry` — loosely linked to sessions via `source` (application-level, no FK constraint)
    * `AuthToken` — standalone, keyed by `provider`

* **Interacting Components:**
  * **Agent Loop** (`src/agent/loop.rs`) — reads/writes session history and memory during conversation turns
  * **Session History Manager** (`src/session/history.rs`) — direct CRUD on `messages` table
  * **Long-Term Memory Module** (`src/memory/longterm.rs`, `src/tools/longterm_memory.rs`) — reads/writes `memory_entries`
  * **Auth Store** (`src/auth/store.rs`, `src/auth/oauth.rs`, `src/auth/refresh.rs`) — manages `auth_tokens`
  * **Gateway Idempotency Guard** (`src/gateway/idempotency.rs`) — manages `idempotency_keys`
  * **Audit Module** (`src/audit.rs`) — appends to `audit_log`
  * **Cost Tracker** (`src/utils/cost.rs`) — appends to `cost_records`
  * **CLI Commands** (`src/cli/history.rs`, `src/cli/memory.rs`, `src/cli/quota.rs`) — user-facing read access to persisted data
  * **Migration Module** (`src/migrate/mod.rs`) — handles schema upgrades between versions

---

### Database 2: In-Process Vector/Semantic Store (HNSW + BM25)

* **Database Name/Type:** Embedded In-Memory Vector Index (NoSQL – Vector/Similarity Search Store)

* **Purpose/Role:** Provides semantic and full-text search capabilities over the agent's long-term memory entries. Maintains an HNSW (Hierarchical Navigable Small World) graph index for approximate nearest-neighbor (ANN) vector search and a BM25 inverted index for keyword-based retrieval. Used to retrieve the most contextually relevant memories before each agent LLM call, enabling Retrieval-Augmented Generation (RAG) within the agent loop.

* **Key Technologies/Access Methods:**
  * **Rust**, using the `instant-distance` or `hnsw_rs` crate for the HNSW in-memory graph index.
  * BM25 scoring is implemented directly in `src/memory/bm25_searcher.rs` using a custom or crate-backed inverted index (e.g., `bm25` crate).
  * Embeddings are generated via calls to an embedding provider (e.g., OpenAI `text-embedding-ada-002` or a local model) and stored as `Vec<f32>` blobs, also serialized into the SQLite `memory_entries.embedding` column for persistence.
  * The HNSW index is rebuilt in-memory from SQLite on startup via `src/memory/snapshot.rs`.
  * A unified search interface is defined in `src/memory/traits.rs` and dispatched by `src/memory/factory.rs` based on configuration.

* **Key Files/Configuration:**
  * `src/memory/hnsw_searcher.rs` — HNSW index construction and ANN query logic
  * `src/memory/bm25_searcher.rs` — BM25 inverted index and keyword query logic
  * `src/memory/embedding_searcher.rs` — Embedding generation and vector similarity search orchestration
  * `src/memory/builtin_searcher.rs` — Combined/fallback searcher using built-in simple similarity
  * `src/memory/factory.rs` — Factory that selects and initializes the appropriate searcher backend based on config
  * `src/memory/traits.rs` — Defines the `MemorySearcher` trait (interface contract)
  * `src/memory/mod.rs` — Public module interface
  * `src/memory/snapshot.rs` — Serializes/deserializes the in-memory index state to/from disk (typically alongside the SQLite file)
  * `src/memory/hygiene.rs` — Periodic cleanup/pruning of stale or low-importance memory entries
  * `src/config/types.rs` — `memory_backend` config field selects between `"hnsw"`, `"bm25"`, or `"builtin"`
  * `Cargo.toml` — declares vector index and BM25 crate dependencies

* **Schema/Collection Structure (NoSQL):**

  * **HNSW Index (In-Memory Graph)**
    * Each node represents a `MemoryEntry`:
      * `id` (TEXT/UUID) — foreign key reference back to `memory_entries.id` in SQLite
      * `vector` (`Vec<f32>`) — embedding of dimension N (e.g., 1536 for OpenAI ada-002)
    * Index parameters: `ef_construction` (build-time search width), `M` (max connections per node), `ef` (query-time search width) — configurable via `src/config/types.rs`

  * **BM25 Index (In-Memory Inverted Index)**
    * Term → `[{doc_id, term_frequency, field_length}]` posting lists
    * Document metadata: `doc_id` (maps to `memory_entries.id`), `avg_doc_length` (corpus statistic)
    * Supports field-weighted scoring over `content` and `tags` fields

  * **Snapshot File** *(src/memory/snapshot.rs)*
    * Serialized (e.g., `bincode` or `serde_json`) representation of the HNSW graph persisted to a `.snapshot` file on disk
    * Contains: index graph structure, vector data, node ID mappings, index build parameters

* **Key Entities and Relationships:**
  * **MemoryVector:** An indexed representation of a `MemoryEntry`, linked by `id` to the SQLite `memory_entries` table.
  * **SearchQuery:** A runtime query consisting of either a query vector (semantic) or query string (BM25), returning ranked `MemoryEntry` IDs.
  * **Relationships:**
    * `MemoryVector` (1:1) ↔ `MemoryEntry` in SQLite (by `id`)
    * The vector index is a *derived* structure from the SQLite source of truth; rebuilt from SQLite on startup.

* **Interacting Components:**
  * **Long-Term Memory Tool** (`src/tools/longterm_memory.rs`) — issues search queries to retrieve relevant memories
  * **Memory Module** (`src/memory/longterm.rs`) — coordinates write-through: writes to SQLite then updates in-memory index
  * **Agent Context Builder** (`src/agent/context.rs`) — calls memory search to inject relevant facts into the LLM prompt
  * **Memory Hygiene Service** (`src/memory/hygiene.rs`) — prunes entries from both SQLite and the in-memory index
  * **Snapshot Manager** (`src/memory/snapshot.rs`) — persists and restores the index across process restarts
  * **CLI Memory Command** (`src/cli/memory.rs`) — allows user to query and manage memory entries

---

### Database 3: File-System Based Key-Value Store (Configuration & Secrets)

* **Database Name/Type:** File-System Flat-File Store (NoSQL – Key-Value / Document Store via structured files)

* **Purpose/Role:** Stores user configuration, agent personas, provider API keys/secrets, skill definitions, plugin manifests, and pairing/encryption keys as structured files (TOML, JSON, YAML) on the local filesystem. Acts as the configuration and secrets layer — not a traditional database, but serves the role of persistent structured key-value and document storage for non-transactional application state.

* **Key Technologies/Access Methods:**
  * **Rust**, using `serde` + `toml`, `serde_json`, and `serde_yaml` crates for serialization/deserialization.
  * File I/O via Rust's `std::fs` and `tokio::fs` (async).
  * File watching for hot-reload via the `notify` crate (`src/config/watcher.rs`).
  * Encryption of sensitive fields (API keys, tokens) at rest via `src/security/encryption.rs` (AES-GCM or ChaCha20-Poly1305).
  * Directory structure under `$CONFIG_DIR` (default: `~/.config/zeptoclaw/` on Linux/macOS).

* **Key Files/Configuration:**
  * `src/config/mod.rs` — Main config loader; reads `config.toml` from `$CONFIG_DIR`
  * `src/config/types.rs` — Rust structs defining the full config schema (`AppConfig`, `ProviderConfig`, `MemoryConfig`, etc.)
  * `src/config/watcher.rs` — File-system watcher for live config reload
  * `src/config/validate.rs` — Config validation logic
  * `src/config/templates.rs` — Default config template generation
  * `src/security/encryption.rs` — Encryption/decryption for secrets stored in files
  * `src/security/pairing.rs` — Pairing key storage (device pairing tokens)
  * `src/auth/store.rs` — Also writes encrypted credential files alongside SQLite
  * `src/skills/loader.rs` — Reads skill TOML/YAML definitions from `$CONFIG_DIR/skills/`
  * `src/plugins/loader.rs` — Reads plugin manifests from `$CONFIG_DIR/plugins/`
  * `deploy/.env.example` — Documents `CONFIG_DIR`, `DATA_DIR`, `ENCRYPTION_KEY` environment variables
  * `src/cli/config.rs` — CLI interface for reading/writing config values
  * `src/cli/secrets.rs` — CLI interface for managing encrypted secrets

* **Schema/Collection Structure (NoSQL – File Documents):**

  * **`config.toml`** *(root configuration document)*
    ```toml
    [agent]
    name = "..."
    persona = "..."
    model = "claude-3-5-sonnet"

    [memory]
    backend = "hnsw"          # or "bm25", "builtin"
    max_entries = 1000
    embedding_model = "..."

    [providers]
    default = "claude"
    [[providers.list]]
    name = "claude"
    api_key = "<encrypted>"
    model = "..."

    [channels]
    # per-channel config blocks

    [security]
    encryption_key_path = "..."
    allowed_paths = [...]
    ```

  * **`skills/<skill-name>/SKILL.md` + `skill.toml`** *(skill definition documents)*
    * `name` (string)
    * `description` (string)
    * `tools` (list of tool names)
    * `system_prompt` (string)
    * `version` (string)

  * **`plugins/<plugin-name>/manifest.toml`** *(plugin manifest documents)*
    * `name`, `version`, `binary_path`, `tools` (list), `permissions` (list)

  * **`secrets/<provider>.enc`** *(encrypted binary blobs)*
    * Encrypted JSON containing `api_key`, `refresh_token`, `expires_at` per provider

  * **`pairing/<device-id>.json`** *(pairing records)*
    * `device_id` (string), `public_key` (base64), `granted_at` (timestamp), `permissions` (list)

* **Key Entities and Relationships:**
  * **AppConfig:** Root configuration document; references provider configs, memory config, channel configs.
  * **ProviderConfig:** Per-LLM-provider settings (model, API key ref, rate limits). Child of `AppConfig`.
  * **Skill:** A reusable agent capability definition; loaded by the Skills Registry.
  * **Plugin:** An external binary tool extension; loaded by the Plugin Registry.
  * **Secret:** An encrypted credential blob; referenced by `ProviderConfig` and `AuthToken`.
  * **PairingRecord:** A trusted device record for remote access pairing.
  * **Relationships:**
    * `AppConfig` (1) → `ProviderConfig` (M)
    * `AppConfig` (1) → `ChannelConfig` (M)
    * `ProviderConfig` references `Secret` (1:1 per provider)
    * `Skill` is independently loaded; referenced by name in `AppConfig.agent.skills`
    * `Plugin` is independently loaded; referenced by name in tool registry

* **Interacting Components:**
  * **Config Module** (`src/config/`) — primary owner of config file read/write
  * **Provider Registry** (`src/providers/registry.rs`) — reads provider configs to initialize LLM clients
  * **Skills Loader** (`src/skills/loader.rs`, `src/skills/registry.rs`) — reads skill definitions
  * **Plugin Loader** (`src/plugins/loader.rs`, `src/plugins/registry.rs`) — reads plugin manifests
  * **Security/Encryption Module** (`src/security/encryption.rs`, `src/security/pairing.rs`) — reads/writes encrypted secrets and pairing records
  * **Auth Store** (`src/auth/store.rs`) — reads/writes encrypted credential files
  * **CLI Config & Secrets** (`src/cli/config.rs`, `src/cli/secrets.rs`) — user-facing management
  * **Migration Module** (`src/migrate/mod.rs`, `src/migrate/config.rs`) — upgrades config file schemas between versions

---

### Database 4: In-Memory Response Cache

* **Database Name/Type:** In-Process In-Memory Cache (NoSQL – Key-Value Store, ephemeral)

* **Purpose/Role:** Short-lived, in-process cache for LLM API responses and tool execution results. Reduces redundant API calls for identical or near-identical prompts within a session, lowering cost and latency. Not persisted to disk; cache state is lost on process restart. Also used for rate-limit counters in the gateway layer.

* **Key Technologies/Access Methods:**
  * **Rust**, using `std::collections::HashMap` or `dashmap::DashMap` (concurrent hashmap crate) wrapped in `Arc<RwLock<...>>` or `Arc<DashMap<...>>` for thread-safe concurrent access.
  * Cache entries have TTL (time-to-live) managed via `tokio::time` or stored timestamps checked on access.
  * No external caching infrastructure (no Redis, Memcached, etc.) — entirely in-process.

* **Key Files/Configuration:**
  * `src/cache/mod.rs` — Cache module entry point, type definitions
  * `src/cache/response_cache.rs` — Core cache implementation: `ResponseCache` struct, insert/get/evict logic, TTL management
  * `src/gateway/rate_limit.rs` — In-memory rate-limit counters (sliding window or token bucket per client/session)
  * `src/providers/cooldown.rs` — Per-provider cooldown tracking (in-memory timestamps after rate-limit errors)
  * `src/gateway/idempotency.rs` — Also maintains a hot in-memory cache of recent idempotency keys (backed by SQLite for durability)
  * `src/config/types.rs` — `cache_ttl_seconds`, `cache_max_entries` config fields

* **Schema/Collection Structure (NoSQL):**

  * **`ResponseCache` entries** *(src/cache/response_cache.rs)*
    * Key: `String` — hash of (model + normalized prompt + parameters)
    * Value:
      * `response` (`String`) — cached LLM response text or serialized JSON
      * `cached_at` (`Instant`/`u64`) — insertion timestamp for TTL calculation
      * `hit_count` (`u32`) — number of cache hits for this entry
      * `tokens` (`u32`) — token count of the cached response

  * **`RateLimitCounter` entries** *(src/gateway/rate_limit.rs)*
    * Key: `String` — client identifier (session ID, IP, or API key hash)
    * Value:
      * `request_count` (`u32`)
      * `window_start` (`Instant`)
      * `window_duration_secs` (`u64`)

  * **`CooldownState` entries** *(src/providers/cooldown.rs)*
    * Key: `String` — provider name
    * Value:
      * `until` (`Instant`) — cooldown expiry time
      * `reason` (`String`) — error reason that triggered cooldown

* **Key Entities and Relationships:**
  * **CachedResponse:** A memoized LLM API result keyed by request fingerprint.
  * **RateLimitCounter:** A per-client request counter within a sliding

