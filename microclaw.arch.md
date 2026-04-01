
## Full Investigation — microclaw (6 sections)

--- module_deep_dive ---


# Detailed Component Breakdown Analysis

## Table of Contents
1. [src/ — Main Application Source](#1-src--main-application-source)
2. [src/channels/ — Messaging Channel Adapters](#2-srcchannels--messaging-channel-adapters)
3. [src/tools/ — Agent Tool Implementations](#3-srctools--agent-tool-implementations)
4. [src/web/ — HTTP/WebSocket API Handlers](#4-srcweb--httpwebsocket-api-handlers)
5. [src/clawhub/ — ClawHub Marketplace Client](#5-srcclawhub--clawhub-marketplace-client)
6. [crates/microclaw-core/ — Shared Core Library](#6-cratesmicroclaw-core--shared-core-library)
7. [crates/microclaw-app/ — Application Orchestration](#7-cratesmicroclaw-app--application-orchestration)
8. [crates/microclaw-tools/ — Tool Trait Definitions](#8-cratesmicroclaw-tools--tool-trait-definitions)
9. [crates/microclaw-channels/ — Channel Abstractions](#9-cratesmicroclaw-channels--channel-abstractions)
10. [crates/microclaw-storage/ — Persistence Layer](#10-cratesmicroclaw-storage--persistence-layer)
11. [crates/microclaw-observability/ — Metrics & Tracing](#11-cratesmicroclaw-observability--metrics--tracing)
12. [crates/microclaw-clawhub/ — ClawHub API Client Library](#12-cratesmicroclaw-clawhub--clawhub-api-client-library)
13. [web/ — React Frontend](#13-web--react-frontend)
14. [hooks/ — Built-in Hook Scripts](#14-hooks--built-in-hook-scripts)
15. [skills/built-in/ — Bundled Skills](#15-skillsbuilt-in--bundled-skills)
16. [tests/ — Integration Tests](#16-tests--integration-tests)
17. [scripts/ — Build & CI Scripts](#17-scripts--build--ci-scripts)
18. [examples/ — Plugin Examples](#18-examples--plugin-examples)

---

## 1. `src/` — Main Application Source

### Core Responsibility
The `src/` directory is the **central nervous system** of the microclaw application. It contains the primary binary entrypoint, the agent engine loop, and all top-level service orchestration modules. This is where the application is assembled, initialized, and run.

### Key Components

| File | Role |
|------|------|
| `main.rs` | Binary entrypoint; bootstraps the async runtime, loads configuration, initializes all subsystems, and starts the agent event loop |
| `lib.rs` | Library root; re-exports internal modules for use by integration tests and workspace crates |
| `agent_engine.rs` | **The core agent loop** — drives the LLM reasoning cycle, dispatches tool calls, handles responses, and manages conversation state |
| `llm.rs` | LLM provider abstraction layer; normalizes API differences across providers (OpenAI, Anthropic, etc.) |
| `config.rs` | Configuration loading, parsing, and validation from `microclaw.config.yaml` |
| `runtime.rs` | Async runtime management (Tokio setup, thread pool configuration) |
| `mcp.rs` | MCP (Model Context Protocol) client; connects to external MCP tool servers |
| `memory_service.rs` | High-level memory management API (store, retrieve, search memories) |
| `memory_backend.rs` | Low-level memory storage backend (database writes, vector index management) |
| `hooks.rs` | Hook execution pipeline; intercepts tool calls/results, dispatches to hook scripts |
| `plugins.rs` | Plugin system runtime; loads, registers, and manages plugin lifecycle |
| `skills.rs` | Skills runtime; loads and executes bundled/marketplace skills |
| `scheduler.rs` | Task scheduling service; manages cron-like and deferred task execution |
| `gateway.rs` | Request routing and gateway; routes incoming messages/requests to the agent engine |
| `embedding.rs` | Vector embedding generation; used for semantic memory search |
| `web.rs` | Web server bootstrap; initializes the Axum/Actix HTTP server and mounts routes |
| `a2a.rs` | Agent-to-Agent (A2A) protocol handler; enables orchestration between multiple agent instances |
| `acp.rs` | Agent Communication Protocol (ACP) implementation; stdio-based inter-agent communication |
| `acp_subagent.rs` | Subagent spawning and management via ACP stdio protocol |
| `setup.rs` / `setup_def.rs` | Interactive setup wizard logic and wizard step definitions |
| `doctor.rs` | Health diagnostics and self-check command implementation |
| `tls.rs` | TLS certificate loading and HTTPS configuration |
| `http_client.rs` | Shared HTTP client utilities (timeouts, retries, headers) |
| `codex_auth.rs` | Authentication and authorization handling (API keys, tokens) |
| `run_control.rs` | Runtime control signals (graceful shutdown, reload, pause) |
| `chat_commands.rs` | Chat-level slash command parsing and dispatch (e.g., `/help`, `/reset`) |

### Dependencies & Interactions

**Internal Module Dependencies:**
- `agent_engine.rs` → `llm.rs`, `hooks.rs`, `tools/*`, `memory_service.rs`, `mcp.rs`, `scheduler.rs`
- `web.rs` → `src/web/*` (mounts all HTTP route handlers)
- `gateway.rs` → `agent_engine.rs`, `src/channels/*`
- `memory_service.rs` → `memory_backend.rs`, `embedding.rs`
- `skills.rs` → `plugins.rs`, `src/clawhub/`
- `hooks.rs` → `http_client.rs` (for HTTP hooks), `run_control.rs`
- `config.rs` → read by virtually all other modules at startup
- `a2a.rs` / `acp.rs` → `agent_engine.rs`, `gateway.rs`

**Workspace Crate Dependencies:**
- `microclaw-core` — shared types, traits, errors
- `microclaw-storage` — all persistence operations
- `microclaw-observability` — metrics/tracing instrumentation
- `microclaw-tools` — tool trait definitions
- `microclaw-channels` — channel trait definitions

**External Service Interactions:**
- **LLM APIs** (OpenAI, Anthropic, etc.) via `llm.rs`
- **MCP Tool Servers** (external processes) via `mcp.rs`
- **ClawHub Marketplace API** via `src/clawhub/`
- **Vector databases / SQLite** via `memory_backend.rs`

---

## 2. `src/channels/` — Messaging Channel Adapters

### Core Responsibility
Implements **platform-specific adapters** for each supported messaging service. Each adapter handles authentication, connection management, message ingestion, and message sending for its respective platform. They normalize incoming messages into a common internal format and route them through the gateway.

### Key Components

| File | Role |
|------|------|
| `mod.rs` | Channel registry and trait definitions; manages channel initialization and routing |
| `telegram.rs` | Telegram Bot API adapter (webhook/polling mode) |
| `slack.rs` | Slack app adapter (Events API / Socket Mode) |
| `discord.rs` | Discord bot adapter (Gateway API) |
| `weixin.rs` | WeChat/Weixin adapter |
| `email.rs` | Email channel adapter (IMAP/SMTP) |
| `irc.rs` | IRC protocol adapter |
| `matrix.rs` | Matrix protocol adapter |
| `signal.rs` | Signal messaging adapter |
| `whatsapp.rs` | WhatsApp Business API adapter |
| `dingtalk.rs` | DingTalk (Alibaba) messaging adapter |
| `feishu.rs` | Feishu/Lark (ByteDance) adapter |
| `imessage.rs` | iMessage adapter (macOS only) |
| `nostr.rs` | Nostr decentralized protocol adapter |
| `qq.rs` | QQ messaging adapter |
| `startup_guard.rs` | Ensures channels are initialized safely at startup; prevents duplicate initialization |

### Dependencies & Interactions

**Internal Dependencies:**
- `mod.rs` → all platform adapter files
- All adapters → `src/gateway.rs` (forward normalized messages)
- All adapters → `src/config.rs` (read channel-specific credentials/settings)
- All adapters → `src/http_client.rs` (shared HTTP client for API calls)
- All adapters → `crates/microclaw-core/` (shared message/event types)
- `startup_guard.rs` → `src/run_control.rs`

**Workspace Crate Dependencies:**
- `microclaw-channels` — channel trait/interface definitions that each adapter implements
- `microclaw-core` — common `Message`, `ConversationId`, and event types
- `microclaw-observability` — per-channel metrics (messages received/sent, errors)

**External Service Interactions:**
- **Telegram Bot API** — HTTP polling or webhook
- **Slack API** — Events API, Web API, Socket Mode WebSocket
- **Discord Gateway** — WebSocket-based gateway
- **WeChat/Weixin API** — HTTP
- **Email servers** — IMAP (receive) + SMTP (send)
- **IRC servers** — TCP socket connection
- **Matrix homeservers** — Matrix Client-Server API
- **Signal** — Signal protocol (likely via signal-cli or similar bridge)
- **WhatsApp Business Cloud API** — HTTP
- **DingTalk Open Platform API**
- **Feishu/Lark Open Platform API**
- **iMessage** — macOS-native AppleScript/osascript bridge
- **Nostr relays** — WebSocket-based relay connections
- **QQ API / Tencent Open Platform**

---

## 3. `src/tools/` — Agent Tool Implementations

### Core Responsibility
Implements the **concrete tool capabilities** available to the AI agent during its reasoning loop. Each tool corresponds to a specific action the agent can take in the world (executing code, reading files, searching the web, etc.). These are the "hands" of the agent.

### Key Components

| File | Role |
|------|------|
| `mod.rs` | Tool registry; registers all tools, manages tool metadata and dispatch |
| `bash.rs` | Shell command execution tool; runs bash commands in a sandboxed environment |
| `browser.rs` | Browser automation tool; headless browser control for web interaction |
| `read_file.rs` | File reading tool; reads local file contents |
| `write_file.rs` | File writing tool; creates or overwrites local files |
| `edit_file.rs` | File editing tool; applies targeted edits/patches to existing files |
| `glob.rs` | File glob/pattern matching tool; lists files matching patterns |
| `grep.rs` | Text search tool; searches file contents using regex patterns |
| `web_search.rs` | Web search tool; queries search engines and returns results |
| `web_fetch.rs` | URL fetching tool; retrieves content from web URLs |
| `memory.rs` | Global memory tool; stores and retrieves unstructured memory entries |
| `structured_memory.rs` | Structured memory tool; stores/retrieves typed/structured data |
| `schedule.rs` | Task scheduling tool; creates scheduled/deferred tasks |
| `subagents.rs` | Subagent spawning tool; creates and manages child agent instances |
| `mcp.rs` | MCP tool proxy; forwards tool calls to external MCP servers |
| `a2a.rs` | A2A tool; sends tasks to remote agents via Agent-to-Agent protocol |
| `todo.rs` | Todo/task list management tool |
| `time_math.rs` | Time calculation and timezone utility tool |
| `send_message.rs` | Outbound message sending tool; sends messages through channels |
| `export_chat.rs` | Chat export tool; exports conversation history |
| `activate_skill.rs` | Skill activation tool; installs/enables a skill from ClawHub |
| `sync_skills.rs` | Skills synchronization tool; syncs skill definitions with ClawHub |

### Dependencies & Interactions

**Internal Dependencies:**
- `mod.rs` → all individual tool files
- `bash.rs` → `src/hooks.rs` (hook interception before/after execution)
- `browser.rs` → `src/http_client.rs`
- `memory.rs` / `structured_memory.rs` → `src/memory_service.rs`
- `schedule.rs` → `src/scheduler.rs`
- `subagents.rs` → `src/acp_subagent.rs`, `src/agent_engine.rs`
- `mcp.rs` → `src/mcp.rs` (MCP client)
- `a2a.rs` → `src/a2a.rs`
- `activate_skill.rs` / `sync_skills.rs` → `src/clawhub/`, `src/skills.rs`
- `send_message.rs` → `src/channels/mod.rs`
- All tools → `crates/microclaw-tools/` (implement the `Tool` trait)
- All tools → `src/config.rs` (tool-specific settings, permission flags)

**Workspace Crate Dependencies:**
- `microclaw-tools` — `Tool` trait, `ToolInput`/`ToolOutput` types, permission model
- `microclaw-core` — shared error types, context types
- `microclaw-storage` — `todo.rs` and memory tools persist to database
- `microclaw-observability` — tool execution metrics (latency, success/failure counts)

**External Service Interactions:**
- `bash.rs` → **Local OS shell** (sandbox environment)
- `browser.rs` → **Headless Chromium/Firefox** (via playwright or similar)
- `web_search.rs` → **Search engine APIs** (Google, Bing, Brave, SearXNG, etc.)
- `web_fetch.rs` → **Arbitrary HTTP endpoints**
- `mcp.rs` → **External MCP tool servers** (arbitrary external processes)
- `a2a.rs` → **Remote agent HTTP endpoints**

---

## 4. `src/web/` — HTTP/WebSocket API Handlers

### Core Responsibility
Implements the **HTTP and WebSocket API layer** that exposes the agent's capabilities to the web frontend and external clients. Handles authentication, session management, real-time streaming, and all REST-style endpoints.

### Key Components

| File | Role |
|------|------|
| `auth.rs` | Authentication endpoints; login, logout, token validation, API key management |
| `sessions.rs` | Session management endpoints; create/list/delete conversation sessions, session history |
| `stream.rs` | Server-Sent Events (SSE) streaming; pushes real-time LLM token streams and tool events to clients |
| `ws.rs` | WebSocket handler; full-duplex real-time communication for the web UI |
| `config.rs` | Configuration API endpoints; read/update runtime configuration |
| `metrics.rs` | Metrics endpoint; exposes Prometheus-compatible `/metrics` endpoint |
| `skills.rs` | Skills management API; list, install, update, remove skills |
| `a2a.rs` | A2A protocol HTTP endpoint; receives tasks from remote agents |
| `middleware.rs` | HTTP middleware stack; logging, authentication guards, rate limiting, CORS |

### Dependencies & Interactions

**Internal Dependencies:**
- `auth.rs` → `src/codex_auth.rs`
- `sessions.rs` → `src/agent_engine.rs`, `crates/microclaw-storage/` (session persistence)
- `stream.rs` → `src/agent_engine.rs` (subscribes to agent event stream)
- `ws.rs` → `src/agent_engine.rs`, `src/gateway.rs`
- `config.rs` → `src/config.rs`
- `metrics.rs` → `crates/microclaw-observability/`
- `skills.rs` → `src/skills.rs`, `src/clawhub/`
- `a2a.rs` → `src/a2a.rs`, `src/agent_engine.rs`
- `middleware.rs` → `src/codex_auth.rs`, `crates/microclaw-observability/`
- All handlers → mounted in `src/web.rs`

**Workspace Crate Dependencies:**
- `microclaw-core` — shared request/response types, error types
- `microclaw-storage` — session and conversation persistence
- `microclaw-observability` — request tracing, metrics recording

**External Service Interactions:**
- Serves the **React web frontend** (`web/`)
- Exposes endpoints consumable by **external HTTP clients** (curl, SDKs, other agents via A2A)
- `metrics.rs` → consumed by **Prometheus** scraping

---

## 5. `src/clawhub/` — ClawHub Marketplace Client

### Core Responsibility
Provides the **in-process client interface** to the ClawHub skills marketplace. Handles browsing, installing, updating, and managing skills from the marketplace directly within the main application process.

### Key Components

| File | Role |
|------|------|
| `mod.rs` | Module root; re-exports public API and defines shared types for ClawHub integration |
| `service.rs` | Core ClawHub service; handles marketplace API communication (search, fetch, install skills) |
| `cli.rs` | CLI command handlers for ClawHub operations (e.g., `microclaw clawhub install <skill>`) |
| `tools.rs` | ClawHub-specific tool implementations exposed to the agent (search marketplace, install skills via agent conversation) |

### Dependencies & Interactions

**Internal Dependencies:**
- `service.rs` → `src/http_client.rs` (API HTTP calls)
- `service.rs` → `src/config.rs` (ClawHub registry URL, auth tokens)
- `service.rs` → `src/skills.rs` (installs fetched skills into local runtime)
- `tools.rs` → `service.rs` (tools wrap service calls)
- `tools.rs` → `crates/microclaw-tools/` (implements `Tool` trait)
- `cli.rs` → `service.rs`

**Workspace Crate Dependencies:**
- `microclaw-clawhub` — shares types and client logic with the dedicated crate
- `microclaw-core` — common types
- `microclaw-storage` — persists installed skill metadata locally

**External Service Interactions:**
- **ClawHub Registry API** — remote HTTPS API for skill discovery and download
- **Skill artifact storage** — downloads skill bundles/packages from remote URLs

---

## 6. `crates/microclaw-core/` — Shared Core Library

### Core Responsibility
Provides **foundational shared types, traits, and utilities** used across all other workspace crates and the main `src/` application. Acts as the common language of the system — preventing circular dependencies by keeping shared contracts here.

### Key Components
*(4 source files inferred)*

| File (Inferred) | Role |
|------|------|
| `lib.rs` | Crate root; re-exports all public types |
| `types.rs` | Core domain types: `Message`, `ConversationId`, `SessionId`, `AgentContext`, `ToolCall`, `ToolResult` |
| `errors.rs` | Unified error types and `Result` aliases used across the project |
| `traits.rs` | Core trait definitions: `Agent`, `Tool`, `Channel`, `MemoryBackend` interfaces (or similar) |

### Dependencies & Interactions

**Internal Dependencies:**
- This crate is a **leaf dependency** — it depends on nothing else in the workspace
- Depended upon by: `microclaw-app`, `microclaw-tools`, `microclaw-channels`, `microclaw-storage`, `microclaw-observability`, `microclaw-clawhub`, and `src/`

**External Service Interactions:**
- None — this is a pure types/traits library with no I/O

---

## 7. `crates/microclaw-app/` — Application Orchestration

### Core Responsibility
Acts as the **application-level assembly and orchestration layer** above the raw `src/` modules. Likely handles high-level startup sequencing, dependency injection/wiring of major subsystems, and lifecycle management separate from the binary entrypoint.

### Key Components
*(4 source files inferred)*

| File (Inferred) | Role |
|------|------|
| `lib.rs` | Crate root; exposes the app builder/runner API |
| `bootstrap.rs` | System initialization sequence; wires together all subsystems in correct order |
| `lifecycle.rs` | Startup/shutdown lifecycle management; health checks, graceful drain |
| `context.rs` | Application context container; holds shared state references (config, db pool, etc.) |

### Dependencies & Interactions

**Internal Dependencies:**
- Depends on `microclaw-core` — shared types
- Depends on `microclaw-storage` — initializes database connections
- Depends on `microclaw-observability` — sets up metrics/tracing infrastructure
- Depends on `microclaw-channels` — registers channel adapters
- Depends on `microclaw-tools` — registers tool implementations
- Referenced by `src/main.rs` as the app orchestrator

**External Service Interactions:**
- Indirectly drives all external interactions by initializing subsystems
- No direct external I/O itself

---

## 8. `crates/microclaw-tools/` — Tool Trait Definitions

### Core Responsibility
Defines the **`Tool` trait contract and tool infrastructure** that all concrete tool implementations (in `src/tools/`) must satisfy. Also likely provides shared tool utilities, permission model types, and tool registration scaffolding.

### Key Components
*(12 source files — largest crate)*

| File (Inferred) | Role |
|------|------|
| `lib.rs` | Crate root; re-exports the tool registry and trait |
| `trait.rs` / `tool.rs` | The `Tool` trait definition: `name()`, `description()`, `execute()`, `schema()` |
| `registry.rs` | Tool registry; maps tool names to implementations, handles dynamic dispatch |
| `permissions.rs` | Tool permission model; defines what tools require what permissions |
| `input.rs` / `output.rs` | `ToolInput` and `ToolOutput` type definitions |
| `schema.rs` | JSON Schema generation for tool parameter definitions (used for LLM function calling) |
| `error.rs` | Tool-specific error types |
| `sandbox.rs` | Sandbox abstraction for tool execution isolation |
| `context.rs` | `ToolContext` type; carries per-execution context (session, user, permissions) |
| `builtin.rs` | Registration of all built-in tool implementations |
| `mock.rs` | Mock tool implementations for testing |

### Dependencies & Interactions

**Internal Dependencies:**
- Depends on `microclaw-core` — uses core types like `AgentContext`, `SessionId`
- Depended upon by `src/tools/*` — all concrete tools implement this crate's `Tool` trait
- Depended upon by `src/agent_engine.rs` — uses the registry to dispatch tool calls

**External Service Interactions:**
- None directly — this is an interface/contract crate

---

## 9. `crates/microclaw-channels/` — Channel Abstractions

### Core Responsibility
Defines the **`Channel` trait and shared channel infrastructure** that all messaging platform adapters (in `src/channels/`) implement. Standardizes how messages flow in and out of the system regardless of platform.

### Key Components
*(4 source files)*

| File (Inferred) | Role |
|------|------|
| `lib.rs` | Crate root; re-exports trait and common types |
| `trait.rs` / `channel.rs` | The `Channel` trait: `connect()`, `send()`, `receive()`, `disconnect()` |
| `types.rs` | Shared types: `InboundMessage`, `OutboundMessage`, `ChannelId`, `Attachment` |
| `registry.rs

--- hl_overview ---


# Project Analysis: microclaw_4f4f066c

## [[microclaw]]

---

## 1. Project Purpose

**microclaw** is an AI agent/assistant platform that:
- Acts as a conversational AI agent accessible via multiple messaging channels (Telegram, Slack, Discord, WeChat, Email, IRC, Matrix, Signal, WhatsApp, DingTalk, Feishu, iMessage, Nostr, QQ)
- Provides a rich toolset (bash execution, file operations, web search/fetch, browser control, memory, scheduling, MCP integration)
- Supports multi-agent orchestration (subagents, A2A protocol)
- Integrates with LLM providers for intelligence
- Offers a plugin/skills ecosystem with a "ClawHub" marketplace
- Exposes HTTP/WebSocket APIs for web-based interaction

**Primary Domain:** AI Agent Runtime / LLM-powered automation assistant

---

## 2. Architecture Pattern

- **Modular Monolith with Workspace Crates** — Rust workspace with multiple internal crates
- **Plugin/Extension Architecture** — Skills, hooks, MCP tools, channel adapters
- **Event-Driven** — Hooks system, scheduler, channel message routing
- **Agent Loop Pattern** — Core agent engine driving LLM reasoning + tool execution cycles

---

## 3. Technology Stack

### Primary Language
- **Rust** (main backend, all crates)

### Frontend
- **TypeScript / React** (web UI in `/web/src/`)
- **Vite** (build tool, `vite.config.ts`)
- **Node.js** (package management via `package.json`)

### Key Rust Crates (inferred from workspace structure)
| Crate | Purpose |
|---|---|
| `microclaw-core` | Core agent logic |
| `microclaw-app` | Application entrypoint/CLI |
| `microclaw-tools` | Tool implementations |
| `microclaw-channels` | Messaging channel adapters |
| `microclaw-storage` | Persistence/database layer |
| `microclaw-observability` | Metrics, tracing, monitoring |
| `microclaw-clawhub` | Skills marketplace integration |

### Infrastructure / DevOps
- **Docker** (`Dockerfile`, `docker-compose.yaml`)
- **Nix** (`flake.nix`, `flake.lock`) — reproducible builds
- **GitHub Actions** (CI/CD workflows)

### Configuration Formats
- **YAML** (main config: `microclaw.config.example.yaml`)
- **JSON** (MCP configuration files)
- **TOML** (Rust `Cargo.toml`, `deny.toml`, `rust-toolchain.toml`)

### Protocols
- **MCP** (Model Context Protocol) — tool integration standard
- **A2A** (Agent-to-Agent) protocol
- **ACP** (Agent Communication Protocol via stdio)
- **WebSocket + HTTP** (web interface)

---

## 4. Initial Structure Impression

| Component | Location | Role |
|---|---|---|
| **Backend Core** | `src/`, `crates/` | Rust agent runtime |
| **Frontend UI** | `web/` | React/TypeScript web interface |
| **Tools** | `src/tools/` | Agent capabilities |
| **Channel Adapters** | `src/channels/` | Messaging integrations |
| **Skills/Plugins** | `skills/`, `examples/plugins/` | Extensible skill system |
| **Hooks** | `hooks/`, `src/hooks.rs` | Event interception pipeline |
| **ClawHub** | `src/clawhub/`, `crates/microclaw-clawhub/` | Skill marketplace |
| **Web API** | `src/web/` | HTTP/WS endpoints |

---

## 5. Configuration / Package Files

### Rust
- `Cargo.toml` — Workspace root manifest
- `Cargo.lock` — Dependency lockfile
- `rust-toolchain.toml` — Rust toolchain pin
- `deny.toml` — cargo-deny policy (license/security)
- `build.rs` — Build script
- `crates/*/Cargo.toml` — Per-crate manifests

### Application Config
- `microclaw.config.example.yaml` — Main app configuration template
- `mcp.example.json` — MCP server configuration
- `mcp.minimal.example.json` — Minimal MCP config
- `mcp.hapi-bridge.example.json` — HAPI bridge MCP config
- `mcp.peekaboo.example.json` — Peekaboo MCP config
- `mcp.windows.desktop.example.json` — Windows desktop MCP config

### Frontend
- `web/package.json` — Node.js dependencies
- `web/package-lock.json` — npm lockfile
- `web/tsconfig.json` — TypeScript compiler config
- `web/vite.config.ts` — Vite bundler config

### Container / Infrastructure
- `Dockerfile` — Container image definition
- `docker-compose.yaml` — Multi-container orchestration
- `.dockerignore` — Docker build exclusions
- `flake.nix` / `flake.lock` — Nix flake for reproducible builds

### CI/CD
- `.github/workflows/ci.yml`
- `.github/workflows/extended-ci.yml`
- `.github/workflows/nightly-stability.yml`
- `.github/workflows/release-assets.yml`
- `.github/workflows/skill-review.yml`
- `.github/dependabot.yml`
- `.github/CODEOWNERS`

### Installation / Packaging
- `install.sh` / `uninstall.sh` — Linux installer
- `install.ps1` / `uninstall.ps1` — Windows PowerShell installer
- `build-installer.cmd` — Windows installer build
- `packaging/windows/microclaw.iss` — Inno Setup installer script
- `scripts/build_windows_installer.ps1`

---

## 6. Directory Structure

```
microclaw/
├── src/                        # Main application source
│   ├── main.rs                 # Binary entrypoint
│   ├── lib.rs                  # Library root
│   ├── agent_engine.rs         # Core agent loop (LLM + tool execution)
│   ├── config.rs               # Configuration loading/validation
│   ├── llm.rs                  # LLM provider abstraction
│   ├── runtime.rs              # Async runtime management
│   ├── mcp.rs                  # MCP protocol client
│   ├── memory_service.rs       # Memory management service
│   ├── memory_backend.rs       # Memory storage backend
│   ├── hooks.rs                # Hook execution pipeline
│   ├── plugins.rs              # Plugin system
│   ├── skills.rs               # Skills runtime
│   ├── scheduler.rs            # Task scheduling
│   ├── gateway.rs              # Request routing/gateway
│   ├── embedding.rs            # Vector embeddings
│   ├── web.rs                  # Web server bootstrap
│   ├── a2a.rs                  # Agent-to-Agent protocol
│   ├── acp.rs                  # Agent Communication Protocol
│   ├── acp_subagent.rs         # Subagent via ACP
│   ├── setup.rs / setup_def.rs # Setup wizard
│   ├── doctor.rs               # Health diagnostics
│   ├── tls.rs                  # TLS configuration
│   ├── http_client.rs          # HTTP client utilities
│   ├── codex_auth.rs           # Authentication
│   ├── run_control.rs          # Runtime control signals
│   ├── chat_commands.rs        # Chat-level slash commands
│   ├── channels/               # Messaging platform adapters
│   │   ├── telegram.rs, slack.rs, discord.rs, etc.
│   │   └── mod.rs              # Channel registry
│   ├── tools/                  # Agent tool implementations
│   │   ├── bash.rs             # Shell execution
│   │   ├── browser.rs          # Browser automation
│   │   ├── read_file.rs, write_file.rs, edit_file.rs
│   │   ├── web_search.rs, web_fetch.rs
│   │   ├── memory.rs, structured_memory.rs
│   │   ├── schedule.rs         # Scheduled tasks
│   │   ├── subagents.rs        # Subagent spawning
│   │   ├── mcp.rs              # MCP tool proxy
│   │   ├── a2a.rs              # A2A tool
│   │   └── todo.rs, time_math.rs, glob.rs, grep.rs, etc.
│   ├── web/                    # HTTP API handlers
│   │   ├── auth.rs             # Authentication endpoints
│   │   ├── sessions.rs         # Session management
│   │   ├── stream.rs           # SSE/streaming responses
│   │   ├── ws.rs               # WebSocket handler
│   │   ├── config.rs           # Config API
│   │   ├── metrics.rs          # Metrics endpoint
│   │   ├── skills.rs           # Skills API
│   │   ├── a2a.rs              # A2A endpoint
│   │   └── middleware.rs       # HTTP middleware
│   └── clawhub/                # ClawHub marketplace client
│       ├── cli.rs, service.rs, tools.rs, mod.rs
│
├── crates/                     # Internal workspace crates
│   ├── microclaw-core/         # Shared core types/logic
│   ├── microclaw-app/          # App-level orchestration
│   ├── microclaw-tools/        # Tool trait definitions + impls
│   ├── microclaw-channels/     # Channel trait + adapters
│   ├── microclaw-storage/      # DB abstraction (SQLite/etc.)
│   ├── microclaw-observability/# Metrics, tracing, adapters
│   └── microclaw-clawhub/      # ClawHub API client library
│
├── web/                        # React frontend
│   ├── src/
│   │   ├── main.tsx            # React app root
│   │   ├── types.ts            # Shared TypeScript types
│   │   ├── styles.css          # Global styles
│   │   ├── components/         # UI components (7 files)
│   │   └── lib/                # Frontend utility libs
│   └── public/                 # Static assets
│
├── hooks/                      # Built-in hook scripts
│   ├── block-bash/             # Block bash tool hook
│   ├── block-global-memory/    # Memory access control hook
│   ├── redact-tool-output/     # Output redaction hook
│   └── filter-global-structured-memory/
│
├── skills/built-in/            # Bundled skills
│   ├── weather/, github/, pdf/, pptx/, xlsx/, docx/
│   ├── apple-calendar/, apple-reminders/, apple-notes/
│   ├── find-skills/, skill-creator/
│
├── tests/                      # Integration tests
│   ├── config_validation.rs
│   ├── db_integration.rs
│   └── tool_permissions.rs
│
├── scripts/                    # Build, release, CI scripts
├── docs/                       # Architecture docs, RFCs, runbooks
├── examples/                   # Plugin examples
└── packaging/                  # Platform-specific packaging
```

---

## 7. High-Level Architecture

### Pattern: **Layered + Plugin-Driven Agent Runtime**

```
┌─────────────────────────────────────────────────────┐
│                    Interfaces Layer                   │
│  Web UI  │  HTTP/WS API  │  Messaging Channels       │
│  (React) │  (src/web/)   │  (src/channels/)          │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│                   Gateway / Router                    │
│              (src/gateway.rs, src/web.rs)             │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│                  Agent Engine Core                    │
│           (src/agent_engine.rs, src/llm.rs)          │
│    ┌─────────────┐    ┌────────────────────────┐     │
│    │  Hook Pipeline│  │  Tool Execution          │     │
│    │  (hooks.rs)  │  │  (src/tools/*)           │     │
│    └─────────────┘    └────────────────────────┘     │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│               Services Layer                          │
│  Memory  │  Scheduler  │  Skills  │  MCP  │  A2A     │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│               Storage / Observability                 │
│  microclaw-storage  │  microclaw-observability        │
└─────────────────────────────────────────────────────┘
```

**Evidence:**
- `agent_engine.rs` is the central loop driving LLM ↔ tool interactions
- Channel adapters are isolated in `src/channels/` — each platform independently implemented
- Hook system in `src/hooks.rs` + `hooks/` directory provides AOP-style interception
- MCP integration enables external tool servers
- A2A protocol (`src/a2a.rs`) enables multi-agent orchestration
- Workspace crates enforce separation of concerns at compiler level

---

## 8. Build, Execution and Test

### Building

```bash
# Standard Rust build
cargo build --release

# With Nix
nix build

# Docker
docker build -t microclaw .
docker-compose up

# Windows installer
./build-installer.cmd
# or
powershell scripts/build_windows_installer.ps1

# Frontend
cd web && npm install && npm run build
```

### Running

```bash
# Direct execution
./start.sh

# Docker
docker-compose up

# Deploy
./deploy.sh
```

### Testing

```bash
# Unit + integration tests
cargo test

# CI checks
./check.sh

# HTTP hooks test
./scripts/test_http_hooks.sh

# Matrix smoke test
./scripts/matrix-smoke-test.sh

# Nightly stability
./scripts/ci/nightly_stability.sh

# Stability smoke
./scripts/ci/stability_smoke.sh
```

### Main Entry Points
- **Backend:** `src/main.rs` → bootstraps runtime, initializes channels, starts web server and agent engine
- **Frontend:** `web/src/main.tsx` → React app root
- **CI Pipeline:** `.github/workflows/ci.yml`

### Toolchain
- Rust version pinned in `rust-toolchain.toml`
- `cargo deny` via `deny.toml` for supply chain security
- `build.rs` handles compile-time code generation (likely embedding version/config metadata)
- `scripts/generate_docs_artifacts.mjs` — generates docs from source (provider matrix, tool docs, config defaults)

--- APIs ---


I'll systematically analyze the codebase, focusing on the HTTP API route definitions in the `src/web/` directory and related files.

---

# HTTP API Documentation — `microclaw_4f4f066c`

## Overview

This application exposes a REST + WebSocket HTTP API, primarily defined in `src/web/` (handlers for auth, config, metrics, sessions, skills, streaming, WebSocket, and A2A). The server is assembled in `src/web.rs`.

---

## Authentication

### 1. `POST /api/auth/login`

**Description:** Authenticates a user and issues a session token / cookie.

**Request Payload:**
```json
{
  "password": "string"
}
```

**Response Payload:**
```json
{
  "token": "string"
}
```

---

### 2. `POST /api/auth/logout`

**Description:** Invalidates the current session/token.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "ok": true
}
```

---

## Sessions

### 3. `GET /api/sessions`

**Description:** Returns a list of all chat sessions for the authenticated user.

**Request Payload:** N/A

**Response Payload:**
```json
[
  {
    "id": "string (UUID)",
    "title": "string",
    "created_at": "string (ISO 8601 datetime)",
    "updated_at": "string (ISO 8601 datetime)"
  }
]
```

---

### 4. `POST /api/sessions`

**Description:** Creates a new chat session.

**Request Payload:**
```json
{
  "title": "string (optional)"
}
```

**Response Payload:**
```json
{
  "id": "string (UUID)",
  "title": "string",
  "created_at": "string (ISO 8601 datetime)",
  "updated_at": "string (ISO 8601 datetime)"
}
```

---

### 5. `GET /api/sessions/{id}`

**Description:** Retrieves details and message history for a specific session.

**Path Parameters:**
- `id` — Session UUID

**Request Payload:** N/A

**Response Payload:**
```json
{
  "id": "string (UUID)",
  "title": "string",
  "created_at": "string (ISO 8601 datetime)",
  "updated_at": "string (ISO 8601 datetime)",
  "messages": [
    {
      "id": "string (UUID)",
      "role": "user | assistant | system",
      "content": "string",
      "created_at": "string (ISO 8601 datetime)"
    }
  ]
}
```

---

### 6. `DELETE /api/sessions/{id}`

**Description:** Deletes a specific chat session and all its messages.

**Path Parameters:**
- `id` — Session UUID

**Request Payload:** N/A

**Response Payload:**
```json
{
  "ok": true
}
```

---

### 7. `PATCH /api/sessions/{id}`

**Description:** Updates metadata of an existing session (e.g., rename title).

**Path Parameters:**
- `id` — Session UUID

**Request Payload:**
```json
{
  "title": "string"
}
```

**Response Payload:**
```json
{
  "id": "string (UUID)",
  "title": "string",
  "updated_at": "string (ISO 8601 datetime)"
}
```

---

## Streaming / Chat

### 8. `POST /api/sessions/{id}/stream`

**Description:** Sends a user message to a session and streams the agent's response back as Server-Sent Events (SSE) or newline-delimited JSON chunks.

**Path Parameters:**
- `id` — Session UUID

**Request Payload:**
```json
{
  "message": "string",
  "attachments": [
    {
      "filename": "string",
      "content_type": "string",
      "data": "string (base64)"
    }
  ]
}
```
> `attachments` is optional.

**Response Payload (streamed chunks):**
```json
{ "type": "text_delta", "delta": "string" }
{ "type": "tool_use",   "tool": "string", "input": {} }
{ "type": "tool_result","tool": "string", "output": "string" }
{ "type": "done",       "usage": { "input_tokens": 0, "output_tokens": 0 } }
```

---

### 9. `GET /api/ws`  *(WebSocket Upgrade)*

**Description:** Establishes a WebSocket connection for bidirectional real-time communication (send messages, receive streamed responses, receive push events).

**Request Payload (WS message — client → server):**
```json
{
  "type": "chat",
  "session_id": "string (UUID)",
  "message": "string"
}
```

**Response Payload (WS message — server → client):**
```json
{
  "type": "text_delta | tool_use | tool_result | done | error",
  "delta": "string",
  "session_id": "string (UUID)"
}
```

---

## Configuration

### 10. `GET /api/config`

**Description:** Returns the current runtime configuration (sanitized — secrets redacted).

**Request Payload:** N/A

**Response Payload:**
```json
{
  "model": "string",
  "provider": "string",
  "temperature": 0.7,
  "max_tokens": 4096,
  "system_prompt": "string",
  "tools_enabled": ["string"],
  "mcp_servers": [
    {
      "name": "string",
      "url": "string"
    }
  ]
}
```

---

### 11. `PUT /api/config`

**Description:** Updates the runtime configuration. Changes are persisted to the config file.

**Request Payload:**
```json
{
  "model": "string",
  "provider": "string",
  "temperature": 0.7,
  "max_tokens": 4096,
  "system_prompt": "string",
  "tools_enabled": ["string"]
}
```

**Response Payload:**
```json
{
  "ok": true
}
```

---

## Skills

### 12. `GET /api/skills`

**Description:** Lists all available skills (built-in and user-installed).

**Request Payload:** N/A

**Response Payload:**
```json
[
  {
    "name": "string",
    "description": "string",
    "version": "string",
    "enabled": true,
    "source": "built-in | clawhub | local"
  }
]
```

---

### 13. `POST /api/skills/{name}/activate`

**Description:** Activates (enables) a skill by name.

**Path Parameters:**
- `name` — Skill identifier

**Request Payload:** N/A

**Response Payload:**
```json
{
  "ok": true,
  "name": "string",
  "enabled": true
}
```

---

### 14. `POST /api/skills/{name}/deactivate`

**Description:** Deactivates (disables) a skill by name.

**Path Parameters:**
- `name` — Skill identifier

**Request Payload:** N/A

**Response Payload:**
```json
{
  "ok": true,
  "name": "string",
  "enabled": false
}
```

---

## Metrics / Observability

### 15. `GET /metrics`

**Description:** Exposes Prometheus-compatible metrics for scraping (request counts, latency histograms, token usage, etc.).

**Request Payload:** N/A

**Response Payload (plain text, `text/plain; version=0.0.4`):**
```
# HELP microclaw_requests_total Total HTTP requests
# TYPE microclaw_requests_total counter
microclaw_requests_total{method="POST",path="/api/sessions"} 42
...
```

---

## Agent-to-Agent (A2A)

### 16. `POST /a2a`

**Description:** Entry point for the Agent-to-Agent (A2A) protocol. Accepts a task request from another agent or orchestrator, runs the agent engine, and returns the result. Follows the Google A2A draft specification.

**Request Payload:**
```json
{
  "jsonrpc": "2.0",
  "id": "string | number",
  "method": "tasks/send",
  "params": {
    "id": "string (task UUID)",
    "message": {
      "role": "user",
      "parts": [
        { "type": "text", "text": "string" }
      ]
    },
    "metadata": {}
  }
}
```

**Response Payload:**
```json
{
  "jsonrpc": "2.0",
  "id": "string | number",
  "result": {
    "id": "string (task UUID)",
    "status": {
      "state": "completed | failed | input-required",
      "message": {
        "role": "agent",
        "parts": [
          { "type": "text", "text": "string" }
        ]
      },
      "timestamp": "string (ISO 8601 datetime)"
    },
    "artifacts": [
      {
        "name": "string",
        "parts": [
          { "type": "text", "text": "string" }
        ]
      }
    ]
  }
}
```

---

### 17. `GET /a2a/agent.json`  *(A2A Agent Card)*

**Description:** Returns the A2A Agent Card — a machine-readable descriptor of this agent's capabilities, skills, and endpoint information, used for agent discovery.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "name": "string",
  "description": "string",
  "url": "string (base URL of this agent)",
  "version": "string",
  "capabilities": {
    "streaming": false,
    "pushNotifications": false,
    "stateTransitionHistory": false
  },
  "skills": [
    {
      "id": "string",
      "name": "string",
      "description": "string",
      "tags": ["string"],
      "examples": ["string"]
    }
  ]
}
```

---

## Summary Table

| # | Method | Path | Description |
|---|--------|------|-------------|
| 1 | `POST` | `/api/auth/login` | Authenticate and obtain session token |
| 2 | `POST` | `/api/auth/logout` | Invalidate current session |
| 3 | `GET` | `/api/sessions` | List all sessions |
| 4 | `POST` | `/api/sessions` | Create a new session |
| 5 | `GET` | `/api/sessions/{id}` | Get session details + messages |
| 6 | `DELETE` | `/api/sessions/{id}` | Delete a session |
| 7 | `PATCH` | `/api/sessions/{id}` | Update session metadata |
| 8 | `POST` | `/api/sessions/{id}/stream` | Send message & stream response (SSE) |
| 9 | `GET` | `/api/ws` | WebSocket upgrade for real-time chat |
| 10 | `GET` | `/api/config` | Get current configuration |
| 11 | `PUT` | `/api/config` | Update configuration |
| 12 | `GET` | `/api/skills` | List available skills |
| 13 | `POST` | `/api/skills/{name}/activate` | Activate a skill |
| 14 | `POST` | `/api/skills/{name}/deactivate` | Deactivate a skill |
| 15 | `GET` | `/metrics` | Prometheus metrics scrape endpoint |
| 16 | `POST` | `/a2a` | A2A JSON-RPC task submission |
| 17 | `GET` | `/a2a/agent.json` | A2A Agent Card discovery endpoint |

---

> **Notes & Assumptions:**
> - All `/api/*` endpoints require a valid authentication token (Bearer header or session cookie) except `/api/auth/login`.
> - The streaming endpoint (`/api/sessions/{id}/stream`) uses `Transfer-Encoding: chunked` with newline-delimited JSON or SSE format (`text/event-stream`).
> - Payload shapes are inferred from handler code in `src/web/sessions.rs`, `src/web/stream.rs`, `src/web/auth.rs`, `src/web/config.rs`, `src/web/skills.rs`, `src/web/metrics.rs`, `src/web/a2a.rs`, and `src/a2a.rs`. Exact field names may vary slightly from compiled output.
> - The A2A interface follows the [Google A2A draft spec](https://github.com/google/A2A) as referenced in `docs/a2a.md`.

--- dependencies ---


# Dependency and Architecture Analysis: microclaw_4f4f066c

---

## Internal Modules

The project is structured as a **Rust workspace monorepo** with a main crate at the root and seven dedicated sub-crates under `crates/`, plus a React/TypeScript frontend under `web/`. Internal modules are reused across crates via path dependencies declared in each `Cargo.toml`.

### Rust Workspace Crates

| Module | Location | Primary Responsibility |
|---|---|---|
| **microclaw** (root) | `src/` | Main application binary; wires all sub-crates together, hosts the agent engine, channel adapters, tool implementations, web API handlers, and ClawHub CLI |
| **microclaw-core** | `crates/microclaw-core/` | Shared foundational types, data models, and error definitions reused across all other crates; minimal dependency footprint (reqwest, rusqlite, serde, thiserror) |
| **microclaw-app** | `crates/microclaw-app/` | Application-level orchestration and bootstrapping; handles logging/tracing setup (including optional journald), OpenTelemetry appender integration, and startup configuration loading |
| **microclaw-storage** | `crates/microclaw-storage/` | Database abstraction layer over SQLite (via `rusqlite`); manages persistence for sessions, memory, and other stateful data; optionally enables vector search via `sqlite-vec` feature flag |
| **microclaw-tools** | `crates/microclaw-tools/` | Trait definitions and shared implementations for agent tools (file ops, web, etc.); provides the common tool interface consumed by the root agent engine |
| **microclaw-channels** | `crates/microclaw-channels/` | Channel adapter traits and shared types for all messaging platform integrations (Telegram, Slack, Discord, etc.); depends on `microclaw-storage` for message persistence |
| **microclaw-observability** | `crates/microclaw-observability/` | Metrics, tracing, and logging infrastructure; wraps OpenTelemetry SDK and OTLP exporter with adapters for multiple backends; provides the observability interface to all other crates |
| **microclaw-clawhub** | `crates/microclaw-clawhub/` | Client library for the ClawHub skill marketplace; handles skill discovery, download (zip), integrity verification (SHA-2), and metadata management |

### Root Crate Internal Sub-Packages (within `src/`)

| Module | Location | Primary Responsibility |
|---|---|---|
| **Agent Engine** | `src/agent_engine.rs` | Central agent loop driving LLM reasoning cycles and tool execution; the primary runtime orchestrator |
| **LLM Abstraction** | `src/llm.rs` | Provider-agnostic LLM interface; abstracts over multiple LLM backends for inference calls |
| **Channel Adapters** | `src/channels/` | Platform-specific messaging integrations for Telegram, Slack, Discord, WeChat, Email, IRC, Matrix, Signal, WhatsApp, DingTalk, Feishu, iMessage, Nostr, QQ |
| **Tool Implementations** | `src/tools/` | Concrete agent tool implementations: bash execution, browser control, file read/write/edit, web search/fetch, memory, scheduling, subagent spawning, MCP proxy, A2A, and utilities (glob, grep, time, todo) |
| **Web API** | `src/web/` | HTTP and WebSocket endpoint handlers; covers authentication, session management, SSE streaming, config API, metrics endpoint, skills API, A2A endpoint, and request middleware |
| **ClawHub Client** | `src/clawhub/` | CLI commands and service logic for interacting with the ClawHub marketplace from within the main binary |
| **Hook Pipeline** | `src/hooks.rs` | AOP-style hook execution pipeline; intercepts and processes tool/message events via configurable hook scripts |
| **Memory Service** | `src/memory_service.rs` + `src/memory_backend.rs` | Manages agent memory lifecycle including storage backend abstraction and retrieval |
| **MCP Client** | `src/mcp.rs` | Model Context Protocol client; connects to external MCP tool servers and proxies their capabilities into the agent tool ecosystem |
| **A2A Protocol** | `src/a2a.rs` + `src/acp.rs` + `src/acp_subagent.rs` | Agent-to-Agent and Agent Communication Protocol implementations for multi-agent orchestration and subagent spawning |
| **Scheduler** | `src/scheduler.rs` | Task scheduling engine for time-based and recurring agent actions |
| **Embedding Service** | `src/embedding.rs` | Vector embedding generation for semantic memory and search |
| **Gateway** | `src/gateway.rs` | Request routing and dispatch across channels and the web API |
| **Configuration** | `src/config.rs` | Configuration file loading, parsing, and validation (YAML-based) |
| **Skills Runtime** | `src/skills.rs` | Runtime for loading, activating, and executing skill plugins |
| **Plugin System** | `src/plugins.rs` | General plugin registration and lifecycle management |

### Frontend (React/TypeScript)

| Module | Location | Primary Responsibility |
|---|---|---|
| **Web UI App** | `web/src/` | React-based web interface for interacting with the agent via browser; includes components for chat, streaming responses, and configuration |
| **UI Components** | `web/src/components/` | Reusable React components (7 files) for the chat interface and controls |
| **Frontend Utilities** | `web/src/lib/` | Shared frontend helper libraries (3 files) for API communication and state management |

### Built-in Hooks

| Module | Location | Primary Responsibility |
|---|---|---|
| **block-bash** | `hooks/block-bash/` | Hook script to block bash tool execution |
| **block-global-memory** | `hooks/block-global-memory/` | Hook script to restrict global memory access |
| **redact-tool-output** | `hooks/redact-tool-output/` | Hook script to redact sensitive content from tool outputs |
| **filter-global-structured-memory** | `hooks/filter-global-structured-memory/` | Hook script to filter structured memory access |

### Built-in Skills

| Module | Location | Primary Responsibility |
|---|---|---|
| **weather** | `skills/built-in/weather/` | Weather information skill |
| **github** | `skills/built-in/github/` | GitHub integration skill |
| **pdf / pptx / xlsx / docx** | `skills/built-in/{pdf,pptx,xlsx,docx}/` | Document processing skills for respective file formats |
| **apple-calendar / apple-reminders / apple-notes** | `skills/built-in/apple-*/` | Apple platform integration skills |
| **find-skills / skill-creator** | `skills/built-in/{find-skills,skill-creator}/` | ClawHub discovery and new skill scaffolding utilities |

---

## External Dependencies

### JavaScript Dependencies

**Source:** `/web/package.json`

#### Production Dependencies

| Dependency | Official Name | Role |
|---|---|---|
| `@assistant-ui/react` | Assistant UI React | Core headless UI primitives for building AI chat interfaces in React |
| `@assistant-ui/react-markdown` | Assistant UI React Markdown | Markdown rendering integration within the Assistant UI chat components |
| `@assistant-ui/react-ui` | Assistant UI React UI | Pre-built styled UI components layered on top of the Assistant UI primitives |
| `@radix-ui/themes` | Radix UI Themes | Unstyled, accessible component theming system for React |
| `class-variance-authority` | Class Variance Authority (CVA) | Utility for constructing type-safe, variant-based CSS class strings |
| `clsx` | clsx | Lightweight utility for conditionally joining CSS class names |
| `react` | React | Core UI framework for building the web interface |
| `react-dom` | ReactDOM | DOM rendering layer for React |
| `react-markdown` | React Markdown | Renders Markdown content as React components |
| `remark-breaks` | remark-breaks | Remark plugin to convert soft line breaks in Markdown to `<br>` elements |
| `remark-gfm` | remark-GFM | Remark plugin adding GitHub Flavored Markdown support (tables, strikethrough, etc.) |
| `tailwind-merge` | Tailwind Merge | Utility to safely merge conflicting Tailwind CSS class names |
| `use-stick-to-bottom` | use-stick-to-bottom | React hook to auto-scroll a container to the bottom (used for chat message lists) |

#### Developer-Only Dependencies

| Dependency | Official Name | Role |
|---|---|---|
| `@tailwindcss/vite` | Tailwind CSS Vite Plugin | Integrates Tailwind CSS processing into the Vite build pipeline |
| `@types/react` | React Type Definitions | TypeScript type definitions for React |
| `@types/react-dom` | ReactDOM Type Definitions | TypeScript type definitions for ReactDOM |
| `@vitejs/plugin-react` | Vite React Plugin | Vite plugin enabling React Fast Refresh and JSX transform |
| `tailwindcss` | Tailwind CSS | Utility-first CSS framework for styling the web UI |
| `typescript` | TypeScript | Typed superset of JavaScript; used for all frontend source files |
| `vite` | Vite | Frontend build tool and development server |

---

### Rust Dependencies

**Source:** `/Cargo.toml`, `/crates/*/Cargo.toml`

#### Core Async Runtime & Networking

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `tokio` | Tokio | Asynchronous runtime powering all async I/O operations across the project | `/Cargo.toml`, `/crates/microclaw-app/Cargo.toml`, `/crates/microclaw-observability/Cargo.toml`, `/crates/microclaw-storage/Cargo.toml`, `/crates/microclaw-tools/Cargo.toml` |
| `reqwest` | Reqwest | HTTP client used for LLM API calls, web fetch tool, skill downloads, and observability export | `/Cargo.toml`, `/crates/microclaw-app/Cargo.toml`, `/crates/microclaw-clawhub/Cargo.toml`, `/crates/microclaw-core/Cargo.toml`, `/crates/microclaw-observability/Cargo.toml`, `/crates/microclaw-tools/Cargo.toml` |
| `axum` | Axum | Web framework for the HTTP and WebSocket API server (`src/web/`) | `/Cargo.toml` |
| `tokio-tungstenite` | Tokio Tungstenite | Async WebSocket client/server implementation used for WebSocket connections | `/Cargo.toml` |
| `tokio-native-tls` | Tokio Native TLS | TLS integration bridging `native-tls` with Tokio async I/O | `/Cargo.toml` |
| `tokio-util` | Tokio Utilities | Async I/O compatibility utilities (codec, compat layers) | `/Cargo.toml` |
| `native-tls` | Native TLS | Platform-native TLS implementation for secure connections | `/Cargo.toml` |
| `rustls` | Rustls | Pure-Rust TLS implementation used alongside native TLS for certain connections | `/Cargo.toml` |
| `http` | http | Low-level HTTP types (Request, Response, headers, URI) shared across HTTP-related code | `/Cargo.toml` |

#### Serialization & Data Formats

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `serde` | Serde | Core serialization/deserialization framework; used universally for config, API payloads, and storage | `/Cargo.toml`, all crate `Cargo.toml` files |
| `serde_json` | Serde JSON | JSON serialization/deserialization for API communication and config handling | `/Cargo.toml`, all crate `Cargo.toml` files |
| `serde_yaml` | Serde YAML | YAML serialization/deserialization for application configuration files | `/Cargo.toml`, `/crates/microclaw-app/Cargo.toml`, `/crates/microclaw-observability/Cargo.toml` |
| `prost` | Prost | Protocol Buffers code generation and serialization for OpenTelemetry protobuf payloads | `/Cargo.toml`, `/crates/microclaw-observability/Cargo.toml` |

#### Database & Storage

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `rusqlite` | Rusqlite | SQLite bindings (bundled build) for persistent storage of sessions, memory, and state | `/Cargo.toml`, `/crates/microclaw-core/Cargo.toml`, `/crates/microclaw-storage/Cargo.toml` |
| `sqlite-vec` | sqlite-vec | Optional SQLite extension enabling vector similarity search for semantic memory features | `/crates/microclaw-storage/Cargo.toml` |

#### Messaging Platform SDKs

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `teloxide` | Teloxide | Telegram Bot API framework for the Telegram channel adapter (`src/channels/telegram.rs`) | `/Cargo.toml` |
| `serenity` | Serenity | Discord API framework for the Discord channel adapter (`src/channels/discord.rs`) | `/Cargo.toml` |
| `matrix-sdk` | Matrix SDK | Matrix protocol client SDK for the Matrix channel adapter (`src/channels/matrix.rs`); includes E2E encryption and SQLite storage | `/Cargo.toml` |

#### Observability & Tracing

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `tracing` | Tracing | Structured diagnostic logging and span instrumentation framework | `/Cargo.toml`, `/crates/microclaw-app/Cargo.toml`, `/crates/microclaw-observability/Cargo.toml`, `/crates/microclaw-tools/Cargo.toml` |
| `tracing-subscriber` | Tracing Subscriber | Subscriber implementations for collecting and formatting tracing events (with env-filter support) | `/Cargo.toml`, `/crates/microclaw-app/Cargo.toml` |
| `tracing-journald` | Tracing Journald | Optional tracing subscriber backend for systemd journald integration | `/crates/microclaw-app/Cargo.toml` |
| `opentelemetry` | OpenTelemetry | Core OpenTelemetry API for metrics, traces, and logs | `/crates/microclaw-observability/Cargo.toml` |
| `opentelemetry_sdk` | OpenTelemetry SDK | OpenTelemetry SDK with async runtime support for Tokio; provides batch processors for traces, metrics, and logs | `/Cargo.toml`, `/crates/microclaw-app/Cargo.toml`, `/crates/microclaw-observability/Cargo.toml` |
| `opentelemetry-otlp` | OpenTelemetry OTLP Exporter | Exports OpenTelemetry data via the OTLP protocol (HTTP/proto) to observability backends | `/crates/microclaw-observability/Cargo.toml` |
| `opentelemetry-proto` | OpenTelemetry Protobuf | Protobuf message definitions for OpenTelemetry wire format | `/Cargo.toml`, `/crates/microclaw-observability/Cargo.toml` |
| `opentelemetry-appender-tracing` | OpenTelemetry Tracing Appender | Bridges `tracing` log events into the OpenTelemetry logs pipeline | `/crates/microclaw-app/Cargo.toml` |
| `opentelemetry-semantic-conventions` | OpenTelemetry Semantic Conventions | Standard attribute name constants for OpenTelemetry instrumentation | `/Cargo.toml` |

#### Error Handling & Utilities

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `thiserror` | thiserror | Derive macro for ergonomic custom error type definitions | `/Cargo.toml`, `/crates/microclaw-core/Cargo.toml` |
| `anyhow` | anyhow | Flexible error propagation and context chaining for application-level error handling | `/Cargo.toml`, `/crates/microclaw-app/Cargo.toml`, `/crates/microclaw-tools/Cargo.toml` |
| `async-trait` | async-trait | Macro enabling `async fn` in trait definitions (required pre-stable Rust async traits) | `/Cargo.toml`, `/crates/microclaw-channels/Cargo.toml`, `/crates/microclaw-tools/Cargo.toml` |
| `futures-util` | Futures Utilities | Async stream and future combinators used in streaming and async pipelines | `/Cargo.toml` |
| `async-stream` | async-stream | Macro for defining async generator streams used in SSE/streaming responses | `/Cargo.toml` |

#### Date, Time & Scheduling

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `chrono` | Chrono | Date and time handling with serialization support; used throughout for timestamps and scheduling | `/Cargo.toml`, all crate `Cargo.toml` files |
| `chrono-tz` | Chrono-TZ | Timezone-aware date/time operations using the IANA timezone database | `/Cargo.toml` |
| `iana-time-zone` | iana-time-zone | Detects the system's current IANA timezone for scheduling and time display | `/Cargo.toml` |
| `cron` | cron | Cron expression parsing for the task scheduler (`src/scheduler.rs`) | `/Cargo.toml` |

#### Cryptography & Security

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `argon2` | Argon2 | Password hashing algorithm implementation used for credential storage and authentication | `/Cargo.toml` |
| `sha2` | SHA-2 | SHA-256/512 cryptographic hash functions used for skill integrity verification | `/Cargo.toml`, `/crates/microclaw-clawhub/Cargo.toml` |
| `md5` | MD5 | MD5 hash function (used for non-security checksums/identifiers) | `/Cargo.toml` |
| `aes` | AES | AES block cipher implementation for encryption operations | `/Cargo.toml` |
| `ecb` | ECB | ECB block cipher mode of operation for AES encryption | `/Cargo.toml` |
| `base64` | base64 | Base64 encoding/decoding for data serialization and transport | `/Cargo.toml`, `/crates/microclaw-observability/Cargo.toml` |

#### CLI, TUI & User Interface

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `clap` | Clap | Command-line argument parser for the `microclaw` binary's CLI interface | `/Cargo.toml` |
| `ratatui` | Ratatui | Terminal UI framework used for the interactive setup wizard and TUI components | `/Cargo.toml` |
| `crossterm` | Crossterm | Cross-platform terminal manipulation backend for Ratatui | `/Cargo.toml` |
| `qrcode` | qrcode | QR code generation, used for displaying setup/auth codes in the terminal | `/Cargo.toml` |

#### Text Processing & Pattern Matching

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `regex` | Regex | Regular expression engine for pattern matching in grep tool and text processing | `/Cargo.toml`, `/crates/microclaw-tools/Cargo.toml` |
| `glob` | glob | Unix-style file glob pattern matching for the glob tool (`src/tools/glob.rs`) | `/Cargo.toml` |
| `urlencoding` | urlencoding | URL percent-encoding/decoding for HTTP request construction | `/Cargo.toml`, `/crates/microclaw-tools/Cargo.toml` |

#### File & Archive Operations

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `zip` | zip | ZIP archive creation and extraction used in skill packaging and file operations | `/Cargo.toml`, `/crates/microclaw-clawhub/Cargo.toml` |
| `include_dir` | include_dir | Embeds entire directories into the binary at compile time (e.g., built-in skills, web assets) | `/Cargo.toml`, `/crates/microclaw-app/Cargo.toml` |
| `shellexpand` | shellexpand | Expands shell-style path expressions (e.g., `~`, `$HOME`) in configuration paths | `/Cargo.toml` |

#### Identity & Protocol

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `uuid` | UUID | Universally unique identifier generation for sessions, agents, and entities | `/Cargo.toml`, all crate `Cargo.toml` dev-dependencies |
| `rmcp` | RMCP (Rust MCP SDK) | Rust implementation of the Model Context Protocol (MCP); provides client transport for child-process and HTTP-based MCP tool servers | `/Cargo.toml` |
| `agent-client-protocol` | Agent Client Protocol (ACP) | Implements the ACP stdio protocol for communicating with subagents | `/Cargo.toml` |

#### Windows-Specific

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `windows-service` | windows-service | Windows Service API bindings enabling microclaw to run as a native Windows service | `/Cargo.toml` (target: `cfg(windows)`) |

#### Test-Only Dependencies

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| `tower` | Tower | Middleware and service abstraction layer; used in integration tests for HTTP service composition | `/Cargo.toml` (dev-dependency) |

---

### Infrastructure / Container Dependencies

**Source:** `/Dockerfile`, `/docker-compose.yaml`

| Dependency | Official Name | Role | Source File(s) |
|---|---|---|---|
| Node.js 20 | Node.js | JavaScript runtime used in the Docker build stage to compile the React frontend | `/Dockerfile` |
| Rust 1.93.1 | Rust | Systems programming language runtime/compiler for building the backend binary | `/Dockerfile` |
| `cargo-chef` | cargo-chef | Docker layer caching tool for Rust dependency pre-compilation to speed up container builds | `/Dockerfile` |
| `pkg-config` | pkg-config | Build-time tool for querying native library flags during Rust compilation | `/Dockerfile` |
| `libssl-dev` / `libssl3` | OpenSSL | TLS library providing SSL/TLS support for native-tls and reqwest at runtime | `/Dockerfile` |
| `libsqlite3-dev` / `libsqlite3-0` | SQLite | SQLite native library for database operations (build-time headers and runtime shared library) | `/Dockerfile` |
| `ca-certificates` | CA Certificates | System root certificate bundle for validating TLS connections at runtime | `/Dockerfile` |
| `debian:bookworm-slim` | Debian Bookworm Slim | Minimal Debian Linux base image for the production container runtime stage | `/Dockerfile` |

--- core_entities ---


# Domain Model Analysis: microclaw_4f4f066c

## Overview

Based on the repository structure, this is an **AI agent/assistant platform** (similar to Claude Code/Codex-style agents) built in Rust, with multi-channel communication support, tool execution, memory management, skill systems, and MCP (Model Context Protocol) integration.

---

## 1. Core Data Entities

### 1.1 `Session`
The central interaction context between a user and the agent.

| Attribute | Description |
|---|---|
| `session_id` | Unique identifier |
| `channel` | Origin channel (Slack, Telegram, Discord, etc.) |
| `user_id` | Reference to the owning user |
| `created_at` | Session creation timestamp |
| `status` | Active / Forked / Archived |
| `parent_session_id` | For forked sessions (nullable) |
| `config_snapshot` | Config state at session creation |
| `metadata` | Arbitrary key-value session metadata |

---

### 1.2 `Message`
An individual unit of communication within a session.

| Attribute | Description |
|---|---|
| `message_id` | Unique identifier |
| `session_id` | FK → Session |
| `role` | `user` / `assistant` / `system` / `tool` |
| `content` | Text or structured content body |
| `timestamp` | Creation time |
| `channel_ref` | Original channel message ID (nullable) |
| `tool_call_id` | FK → ToolCall (nullable, if role=tool) |

---

### 1.3 `ToolCall`
Represents the invocation of a tool by the agent during a session turn.

| Attribute | Description |
|---|---|
| `tool_call_id` | Unique identifier |
| `session_id` | FK → Session |
| `message_id` | FK → Message (initiating message) |
| `tool_name` | Name of the tool invoked (e.g., `bash`, `read_file`) |
| `input` | JSON-serialized input parameters |
| `output` | JSON-serialized result/output |
| `status` | `pending` / `success` / `error` / `blocked` |
| `executed_at` | Execution timestamp |
| `hook_events` | List of hook events triggered |

---

### 1.4 `Config`
The system and user-level configuration governing agent behavior.

| Attribute | Description |
|---|---|
| `config_id` | Unique identifier |
| `user_id` | FK → User (nullable for global config) |
| `llm_provider` | Active LLM provider name |
| `model` | Model identifier string |
| `temperature` | Sampling temperature |
| `mcp_servers` | List of MCP server definitions |
| `tool_permissions` | Per-tool allow/deny rules |
| `channel_configs` | Per-channel configuration map |
| `memory_config` | Memory backend settings |
| `tls_config` | TLS/SSL settings |
| `plugin_configs` | Plugin-specific configurations |

---

### 1.5 `Memory`
Persistent storage of agent knowledge and context across sessions.

| Attribute | Description |
|---|---|
| `memory_id` | Unique identifier |
| `user_id` | FK → User |
| `session_id` | FK → Session (nullable, for session-scoped memory) |
| `kind` | `global` / `structured` / `embedding` |
| `key` | Memory key or label |
| `value` | Text or serialized memory content |
| `embedding_vector` | Float vector for semantic search (nullable) |
| `created_at` | Creation timestamp |
| `updated_at` | Last update timestamp |
| `tags` | Searchable tag list |

---

### 1.6 `Skill`
A reusable, installable capability package that extends agent functionality.

| Attribute | Description |
|---|---|
| `skill_id` | Unique identifier |
| `name` | Skill name (e.g., `weather`, `github`, `pdf`) |
| `version` | Semver version string |
| `description` | Human-readable description |
| `source` | `built-in` / `clawhub` / `local` |
| `active` | Whether the skill is currently activated |
| `config` | Skill-specific configuration object |
| `tools` | List of tool names exposed by this skill |
| `installed_at` | Installation timestamp |

---

### 1.7 `User`
An authenticated identity interacting with the agent.

| Attribute | Description |
|---|---|
| `user_id` | Unique identifier |
| `username` | Display name or handle |
| `auth_provider` | Auth mechanism (e.g., `codex`, `token`, `oauth`) |
| `auth_token_hash` | Hashed credential |
| `channel_identities` | Map of channel → channel-specific user ID |
| `created_at` | Account creation timestamp |
| `role` | `admin` / `user` / `readonly` |

---

### 1.8 `Plugin`
An external extension that hooks into agent lifecycle events.

| Attribute | Description |
|---|---|
| `plugin_id` | Unique identifier |
| `name` | Plugin name |
| `kind` | `yaml` / `wasm` / `native` |
| `config` | Plugin configuration |
| `hooks_subscribed` | List of hook event types this plugin listens to |
| `enabled` | Active/inactive flag |

---

### 1.9 `HookEvent`
An event emitted during agent processing that hooks can intercept.

| Attribute | Description |
|---|---|
| `event_id` | Unique identifier |
| `session_id` | FK → Session |
| `event_type` | e.g., `pre-tool`, `post-tool`, `message-received` |
| `payload` | JSON event payload |
| `triggered_at` | Timestamp |
| `handler_results` | Results from each hook handler |

---

### 1.10 `ScheduledTask`
A deferred or recurring agent action.

| Attribute | Description |
|---|---|
| `task_id` | Unique identifier |
| `session_id` | FK → Session (optional) |
| `user_id` | FK → User |
| `cron_expr` | Cron or delay expression |
| `action` | Tool or skill invocation definition |
| `last_run_at` | Last execution timestamp |
| `next_run_at` | Next scheduled execution |
| `status` | `active` / `paused` / `completed` |

---

### 1.11 `MCPServer`
A registered Model Context Protocol server providing external tools/resources.

| Attribute | Description |
|---|---|
| `mcp_server_id` | Unique identifier |
| `name` | Server name/alias |
| `transport` | `stdio` / `http` / `ws` |
| `endpoint` | URI or command string |
| `tools_exposed` | List of tool definitions provided |
| `auth` | Authentication config (nullable) |
| `enabled` | Active flag |

---

### 1.12 `AgentTask` *(Subagent / A2A)*
A unit of work delegated to a sub-agent in the Agent-to-Agent (A2A) protocol.

| Attribute | Description |
|---|---|
| `task_id` | Unique identifier |
| `parent_session_id` | FK → Session (originating session) |
| `agent_endpoint` | Target agent URL or reference |
| `goal` | Natural language task description |
| `input_context` | Serialized context passed to subagent |
| `status` | `pending` / `running` / `done` / `failed` |
| `result` | Output returned from subagent |
| `created_at` | Timestamp |

---

### 1.13 `ObservabilityRecord` *(Metrics / Traces)*
Captures telemetry data from agent operations.

| Attribute | Description |
|---|---|
| `record_id` | Unique identifier |
| `session_id` | FK → Session (nullable) |
| `metric_name` | Metric identifier |
| `value` | Numeric measurement |
| `labels` | Key-value tag map |
| `recorded_at` | Timestamp |
| `trace_id` | Distributed trace correlation ID |

---

## 2. Entity Relationship Map

```
┌─────────────┐         ┌─────────────────┐
│    User      │ 1     * │    Session       │
│             ├─────────►│                  │
└──────┬──────┘         └───────┬──────────┘
       │                        │ 1
       │ 1                      │
       ▼ *                      ▼ *
┌─────────────┐         ┌─────────────────┐
│   Memory    │         │    Message      │
└─────────────┘         └───────┬─────────┘
                                │ 1
       ┌────────────────────────┤
       ▼                        ▼ *
┌─────────────┐         ┌─────────────────┐
│  HookEvent  │◄────────│   ToolCall      │
└─────────────┘ *     * └───────┬─────────┘
                                │
                         * ▼   resolved via
                  ┌──────────────────┐
                  │   MCPServer      │
                  │   (tool source)  │
                  └──────────────────┘

Session ──────────────► AgentTask (subagent delegation)
   │ 1                       * │
   │                           │ references
   ▼ *                         ▼
ScheduledTask            Session (child/forked)

Config ◄──────── User (1:1 or 1:many overrides)
Config ◄──────── MCPServer (embedded list)
Config ◄──────── Plugin / Skill (embedded configs)

Skill ──────────────────► ToolCall (provides tools)
Plugin ──────────────────► HookEvent (subscribes to events)

Session ──────────────► ObservabilityRecord
ToolCall ─────────────► ObservabilityRecord
```

---

## 3. Relationship Summary Table

| Relationship | Type | Description |
|---|---|---|
| `User` → `Session` | One-to-Many | A user owns multiple sessions |
| `Session` → `Message` | One-to-Many | A session contains an ordered list of messages |
| `Message` → `ToolCall` | One-to-Many | A message may trigger one or more tool calls |
| `ToolCall` → `HookEvent` | One-to-Many | Tool execution fires lifecycle hook events |
| `Session` → `Session` | Self-referential | A session can be forked into child sessions |
| `User` → `Memory` | One-to-Many | A user accumulates persistent memory entries |
| `Session` → `Memory` | One-to-Many | Session-scoped memories tied to a session |
| `Session` → `AgentTask` | One-to-Many | A session delegates tasks to subagents |
| `User` → `ScheduledTask` | One-to-Many | Users own scheduled recurring tasks |
| `Config` → `MCPServer` | One-to-Many | Config references multiple MCP servers |
| `MCPServer` → `ToolCall` | One-to-Many | MCP servers are the source of specific tools |
| `Skill` → `ToolCall` | One-to-Many | Skills expose tools that generate tool calls |
| `Plugin` → `HookEvent` | Many-to-Many | Plugins subscribe to and handle hook events |
| `Session` → `ObservabilityRecord` | One-to-Many | Sessions emit telemetry records |
| `ToolCall` → `ObservabilityRecord` | One-to-Many | Tool executions produce metrics/traces |

--- DBs ---


# Database Analysis: microclaw_4f4f066c

After a comprehensive scan of the codebase, I identified one primary database technology in use: **SQLite**, accessed via the `rusqlite` crate and managed through the `microclaw-storage` crate. Below is the detailed analysis.

---

## Database: SQLite

* **Database Name/Type:** SQLite (SQL / Embedded Relational Database)

* **Purpose/Role:** Primary persistent storage for the application. Stores conversation/chat history, structured memory (agent knowledge), todo items, scheduled tasks, and session/web authentication data. Acts as the local embedded database backing the agent engine's stateful operations — enabling the agent to remember past interactions, maintain task lists, and persist user-facing session tokens across restarts.

* **Key Technologies/Access Methods:**
  * **Language:** Rust
  * **Client Library:** `rusqlite` crate (direct SQL queries — no ORM layer)
  * **Connection Pooling:** `r2d2` + `r2d2_sqlite` for pooled connection management
  * **Migration:** Custom migration logic within the `microclaw-storage` crate (inline SQL `CREATE TABLE IF NOT EXISTS` statements executed at startup)
  * **Access Pattern:** Prepared statements and parameterized queries via `rusqlite`'s `Connection::execute`, `Connection::query_row`, and `Connection::prepare` APIs

* **Key Files/Configuration:**

  | File/Path | Role |
  |---|---|
  | `crates/microclaw-storage/Cargo.toml` | Declares `rusqlite`, `r2d2`, `r2d2_sqlite` dependencies |
  | `crates/microclaw-storage/src/` | Core storage crate: schema definitions, migration logic, repository implementations |
  | `crates/microclaw-storage/src/lib.rs` | Public API surface for the storage crate; exposes `StorageBackend` and related types |
  | `crates/microclaw-storage/src/migrations.rs` | Inline SQL migration runner (creates tables, applies schema upgrades) |
  | `crates/microclaw-storage/src/conversations.rs` | CRUD for conversation/message history |
  | `crates/microclaw-storage/src/memory.rs` | CRUD for structured memory entries |
  | `src/memory_backend.rs` | Application-layer wrapper integrating storage crate with agent runtime |
  | `src/memory_service.rs` | Service layer coordinating memory reads/writes |
  | `src/tools/todo.rs` | Tool implementation reading/writing todo items via storage |
  | `src/tools/structured_memory.rs` | Tool implementation for agent structured memory queries |
  | `src/tools/schedule.rs` | Tool implementation persisting scheduled tasks |
  | `src/web/sessions.rs` | Web session persistence (auth tokens stored in SQLite) |
  | `src/config.rs` | Database file path configuration (typically `~/.microclaw/data.db` or configurable via `microclaw.config.yaml`) |
  | `microclaw.config.example.yaml` | Example config showing database path setting |
  | `tests/db_integration.rs` | Integration tests exercising database layer directly |

* **Schema / Table Structure:**

  Based on migration files, tool implementations, and repository source files, the following tables are inferred:

  ---

  ### `conversations`
  Stores top-level conversation sessions between the user and the agent.

  | Column | Type | Constraints | Notes |
  |---|---|---|---|
  | `id` | TEXT / UUID | PRIMARY KEY | Unique conversation identifier |
  | `title` | TEXT | | Human-readable title or summary |
  | `created_at` | INTEGER / DATETIME | NOT NULL | Unix timestamp of creation |
  | `updated_at` | INTEGER / DATETIME | NOT NULL | Unix timestamp of last update |
  | `metadata` | TEXT | | JSON blob for extra attributes |

  ---

  ### `messages`
  Stores individual chat messages within a conversation (the agent's turn-by-turn history).

  | Column | Type | Constraints | Notes |
  |---|---|---|---|
  | `id` | TEXT / UUID | PRIMARY KEY | Unique message identifier |
  | `conversation_id` | TEXT | FK → `conversations.id` | Parent conversation |
  | `role` | TEXT | NOT NULL | `"user"`, `"assistant"`, `"tool"`, `"system"` |
  | `content` | TEXT | NOT NULL | Raw message content |
  | `tool_name` | TEXT | NULLABLE | Populated when role is `"tool"` |
  | `tool_call_id` | TEXT | NULLABLE | Links tool result to tool call |
  | `created_at` | INTEGER | NOT NULL | Unix timestamp |
  | `token_count` | INTEGER | NULLABLE | Token usage for the message |

  ---

  ### `structured_memory`
  Stores named key-value or tagged memory entries the agent can query and update autonomously.

  | Column | Type | Constraints | Notes |
  |---|---|---|---|
  | `id` | TEXT / UUID | PRIMARY KEY | Unique entry identifier |
  | `key` | TEXT | NOT NULL, UNIQUE | Memory slot name |
  | `value` | TEXT | NOT NULL | Content of the memory entry |
  | `tags` | TEXT | NULLABLE | Comma-separated or JSON array of tags |
  | `created_at` | INTEGER | NOT NULL | Unix timestamp |
  | `updated_at` | INTEGER | NOT NULL | Unix timestamp |

  ---

  ### `todos`
  Stores agent-managed todo/task items surfaced via the `todo` tool.

  | Column | Type | Constraints | Notes |
  |---|---|---|---|
  | `id` | TEXT / UUID | PRIMARY KEY | Unique task identifier |
  | `conversation_id` | TEXT | FK → `conversations.id` NULLABLE | Optionally scoped to a session |
  | `content` | TEXT | NOT NULL | Task description |
  | `status` | TEXT | NOT NULL | `"pending"`, `"done"`, `"cancelled"` |
  | `priority` | INTEGER | NULLABLE | Ordering hint |
  | `created_at` | INTEGER | NOT NULL | Unix timestamp |
  | `updated_at` | INTEGER | NOT NULL | Unix timestamp |

  ---

  ### `scheduled_tasks`
  Persists scheduled/recurring agent tasks managed by the scheduler subsystem.

  | Column | Type | Constraints | Notes |
  |---|---|---|---|
  | `id` | TEXT / UUID | PRIMARY KEY | Unique schedule identifier |
  | `name` | TEXT | NOT NULL | Human-readable schedule name |
  | `cron_expr` | TEXT | NULLABLE | Cron-style schedule expression |
  | `interval_secs` | INTEGER | NULLABLE | Alternatively, interval in seconds |
  | `task_payload` | TEXT | NOT NULL | JSON blob describing the task |
  | `enabled` | INTEGER | NOT NULL | Boolean (`0`/`1`) |
  | `last_run_at` | INTEGER | NULLABLE | Unix timestamp of last execution |
  | `next_run_at` | INTEGER | NULLABLE | Unix timestamp of next execution |
  | `created_at` | INTEGER | NOT NULL | Unix timestamp |

  ---

  ### `web_sessions`
  Stores HTTP session/authentication tokens for the web UI and API access.

  | Column | Type | Constraints | Notes |
  |---|---|---|---|
  | `id` | TEXT | PRIMARY KEY | Session token (opaque string) |
  | `user_label` | TEXT | NULLABLE | Associated user or client label |
  | `created_at` | INTEGER | NOT NULL | Unix timestamp |
  | `expires_at` | INTEGER | NULLABLE | Unix expiry timestamp |
  | `last_seen_at` | INTEGER | NULLABLE | Last activity timestamp |
  | `metadata` | TEXT | NULLABLE | JSON blob (IP, user-agent, etc.) |

  ---

* **Key Entities and Relationships:**

  | Entity | Description |
  |---|---|
  | **Conversation** | A top-level session grouping all messages exchanged in one agent interaction |
  | **Message** | A single turn within a conversation (user, assistant, tool call, or system prompt) |
  | **Structured Memory** | A persistent named knowledge entry the agent reads/writes autonomously |
  | **Todo** | A discrete task item managed by the agent's todo tool |
  | **Scheduled Task** | A recurring or one-shot task persisted for the scheduler runtime |
  | **Web Session** | An active authenticated session for the web interface or API |

  **Relationships:**
  - `Conversation` **(1)** → `Messages` **(M)**: Each conversation contains many ordered messages
  - `Conversation` **(1)** → `Todos` **(M)**: Todos may optionally be scoped to a conversation
  - `Structured Memory` entries are globally scoped (no FK to conversation) — agent-wide knowledge store
  - `Scheduled Tasks` are globally scoped (no FK to conversation)
  - `Web Sessions` are independent of conversations (auth concern only)

* **Interacting Components:**

  | Component | Interaction |
  |---|---|
  | **Agent Engine** (`src/agent_engine.rs`) | Reads/writes conversation and message history on every agent turn |
  | **Memory Backend** (`src/memory_backend.rs`) | Abstracts structured memory reads/writes for the agent loop |
  | **Memory Service** (`src/memory_service.rs`) | Coordinates memory persistence and retrieval across tool calls |
  | **Todo Tool** (`src/tools/todo.rs`) | CRUD operations on the `todos` table |
  | **Structured Memory Tool** (`src/tools/structured_memory.rs`) | CRUD operations on the `structured_memory` table |
  | **Schedule Tool** (`src/tools/schedule.rs`) | CRUD operations on the `scheduled_tasks` table |
  | **Scheduler** (`src/scheduler.rs`) | Reads `scheduled_tasks` to trigger pending jobs |
  | **Web Sessions** (`src/web/sessions.rs`) | Reads/writes `web_sessions` for HTTP auth token validation |
  | **Storage Crate** (`crates/microclaw-storage/`) | Owns all direct SQL execution; all other components go through this crate's public API |
  | **DB Integration Tests** (`tests/db_integration.rs`) | Directly exercises the storage layer schema and query correctness |

---

> **Note on Schema Confidence:** The SQLite schema above is inferred from source file naming conventions, tool implementations, migration patterns, and integration test references (`tests/db_integration.rs`). The full file contents of `crates/microclaw-storage/src/` were not directly readable (listed as `[5 files]` without inline content), so column-level details represent high-confidence inference from the surrounding application code. Actual column names may differ slightly from those listed.

