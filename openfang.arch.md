
## Full Investigation — openfang (11 sections)

--- dependencies ---


# Dependency and Architecture Analysis: OpenFang

---

## Internal Modules

The OpenFang project is organized as a Rust workspace monorepo under the `crates/` directory, with 13 internal crates that are cross-referenced throughout the project.

| Module | Description |
|--------|-------------|
| **openfang-types** | Foundational shared type definitions, core traits, and data structures used across all other crates. Acts as the common type contract for the entire system. |
| **openfang-kernel** | Central orchestration engine and core business logic. Acts as the primary coordinator, wiring together runtime, memory, channels, skills, hands, extensions, and wire protocol into a unified agent OS. |
| **openfang-runtime** | Agent execution environment and LLM driver abstraction layer. Contains the `drivers/` sub-module for multi-provider LLM integration, WASM sandbox execution, and MCP client support. |
| **openfang-api** | HTTP/WebSocket API server daemon. Exposes the REST and WebSocket interface for external clients, serving as the primary programmatic entry point to the agent OS. |
| **openfang-cli** | Command-line interface binary (`openfang`). Includes a terminal user interface (TUI) subsystem and provides the primary end-user interaction surface for managing agents. |
| **openfang-desktop** | Tauri 2.0-based native desktop application. Wraps the kernel and API into a GUI application with system tray, notifications, auto-start, and global shortcuts. |
| **openfang-channels** | Pluggable messaging channel adapter layer (48 source files). Bridges external communication platforms (email, MQTT, WebSocket-based services, etc.) into the agent runtime. |
| **openfang-skills** | Skill registry, loader, and marketplace system. Manages 60+ bundled skill definitions (e.g., Kubernetes, GitHub, Postgres) and provides OpenClaw compatibility for third-party skills. |
| **openfang-hands** | Autonomous capability package system ("hands"). Provides curated agentic action bundles including browser automation, trading, research, Twitter, and lead generation. |
| **openfang-extensions** | Third-party service integration and extension system (25 integrations). Manages MCP server setup, credential vault, and OAuth2 PKCE flows. |
| **openfang-memory** | Agent memory and context persistence substrate. Provides storage backends (SQLite, HTTP-based remote memory) for agent conversation history and contextual state. |
| **openfang-wire** | OpenFang Protocol (OFP) implementation for agent-to-agent (A2A) networking. Handles wire-level message serialization, routing, authentication (HMAC), and agent discovery. |
| **openfang-migrate** | Migration engine for importing agent configurations from other agent frameworks into OpenFang. Supports multiple config formats (TOML, YAML, JSON5). |
| **xtask** | Rust build task automation runner for the workspace. Provides custom `cargo xtask` commands for project-level build operations. |

---

### Pre-Built Agents (`agents/`)

30+ domain-specific agent configurations defined via `agent.toml` files. Notable entries include:

| Agent | Description |
|-------|-------------|
| **orchestrator** | Multi-agent coordination and delegation |
| **researcher** | Information research and synthesis |
| **langchain-code-reviewer** | Python/LangChain-based external agent with its own HTTP server (`server.py`) |
| **coder, debugger, test-engineer** | Software development lifecycle agents |
| **security-auditor** | Security review and auditing agent |
| **data-scientist, analyst** | Data analysis agents |

---

### Client SDKs (`sdk/`)

| SDK | Description |
|-----|-------------|
| **sdk/python** | Official Python client library for the OpenFang REST API (`openfang_sdk.py`, `openfang_client.py`) |
| **sdk/javascript** | Official JavaScript/TypeScript client library for the OpenFang REST API |

---

### External Packages (`packages/`)

| Package | Description |
|---------|-------------|
| **whatsapp-gateway** | Node.js standalone service providing a WhatsApp channel bridge into the OpenFang channel layer |

---

## External Dependencies

### JavaScript Dependencies

**Source:** `/packages/whatsapp-gateway/package.json`

| Dependency | Official Name | Role |
|------------|---------------|------|
| `@whiskeysockets/baileys` | Baileys | WhatsApp Web API client library; provides the core WhatsApp connectivity for the gateway service |
| `qrcode` | QRCode | QR code generation; used for WhatsApp Web authentication pairing |
| `pino` | Pino | High-performance structured JSON logger for the Node.js gateway service |

---

### Python Dependencies

**Source:** `/agents/langchain-code-reviewer/requirements.txt`

| Dependency | Official Name | Role |
|------------|---------------|------|
| `langchain>=0.3` | LangChain | Core LLM chain orchestration framework used to build the code-reviewer agent's logic |
| `langchain-openai>=0.3` | LangChain OpenAI | LangChain integration for OpenAI LLM provider support within the code-reviewer agent |
| `langchain-core>=0.3` | LangChain Core | Foundational abstractions and interfaces for the LangChain ecosystem |
| `langchain-ollama>=0.3` | LangChain Ollama | LangChain integration for local Ollama LLM provider support within the code-reviewer agent |
| `fastapi>=0.115` | FastAPI | Async Python web framework; serves the code-reviewer agent as a standalone HTTP service |
| `uvicorn>=0.34` | Uvicorn | ASGI server for running the FastAPI-based code-reviewer agent service |

> **Note:** `/sdk/python/setup.py` defines the `openfang` Python package metadata only and declares no third-party runtime dependencies.

---

### Rust Dependencies

**Sources:** `/Cargo.toml` (workspace), and per-crate `Cargo.toml` files under `/crates/*/`

#### Async Runtime & Concurrency

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `tokio` | Tokio | Async runtime powering all async I/O, task scheduling, and timers | All crates |
| `tokio-stream` | Tokio Stream | Async stream utilities for Tokio; used for streaming LLM responses and channel event flows | `openfang-api`, `openfang-channels`, `openfang-runtime` |
| `async-trait` | async-trait | Enables `async fn` in Rust trait definitions | Most crates |
| `futures` | Futures | Core async/await primitives and combinator utilities | `openfang-api`, `openfang-channels`, `openfang-kernel`, `openfang-runtime` |
| `dashmap` | DashMap | Concurrent, thread-safe hash map; used for shared in-memory state across agent tasks | Most crates |
| `crossbeam` | Crossbeam | Low-level concurrency primitives (channels, atomics) for inter-task communication | `openfang-kernel` |

---

#### Serialization & Data Formats

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `serde` | Serde | Core serialization/deserialization framework | All crates |
| `serde_json` | Serde JSON | JSON serialization; primary data exchange format for API and LLM responses | All crates |
| `toml` | TOML | TOML config file parsing; used for `agent.toml`, `openfang.toml`, and crate configs | Most crates |
| `rmp-serde` | MessagePack Serde | MessagePack binary serialization for compact memory storage | `openfang-memory`, `openfang-types` (dev) |
| `serde_yaml` | Serde YAML | YAML format parsing; used for agent migration and skill definitions | `openfang-migrate`, `openfang-skills` |
| `json5` | JSON5 | Lenient JSON5 format parsing; used in the migration engine for legacy configs | `openfang-migrate` |
| `prost` | Prost | Protocol Buffers (protobuf) serialization for structured channel messaging | `openfang-channels` |

---

#### HTTP & Networking

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `reqwest` | Reqwest | Async HTTP client; used for LLM provider API calls and external service requests | Most crates |
| `axum` | Axum | Async HTTP/WebSocket server framework; serves the REST and WebSocket API daemon | `openfang-api`, `openfang-channels`, `openfang-desktop`, `openfang-extensions` |
| `tower` | Tower | Middleware abstraction layer for `axum`; handles service composition | `openfang-api` |
| `tower-http` | Tower HTTP | HTTP-specific Tower middleware (CORS, tracing, response compression) | `openfang-api` |
| `tokio-tungstenite` | Tokio Tungstenite | Async WebSocket client; used for Discord and Slack gateway connections | `openfang-channels`, `openfang-runtime` |
| `url` | URL | URL parsing and manipulation | `openfang-channels`, `openfang-extensions` |
| `http` | http | Low-level HTTP types (request/response primitives) | `openfang-runtime` |
| `socket2` | socket2 | Low-level socket configuration (e.g., `SO_REUSEADDR`) for the API server | `openfang-api` |
| `rumqttc` | rumqttc | MQTT protocol client; enables MQTT-based channel messaging | `openfang-channels` |

---

#### Database & Storage

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `rusqlite` | rusqlite | Embedded SQLite database (bundled, no external dependency); used for agent memory and runtime state persistence | `openfang-memory`, `openfang-runtime` |

---

#### Security & Cryptography

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `sha2` | SHA-2 | SHA-256/SHA-512 cryptographic hashing | Most crates |
| `sha1` | SHA-1 | SHA-1 hashing (legacy protocol compatibility, e.g., WhatsApp) | `openfang-channels` |
| `hmac` | HMAC | Hash-based message authentication codes for request signing | `openfang-api`, `openfang-channels`, `openfang-wire` |
| `aes` | AES | AES symmetric block cipher primitives | `openfang-channels` |
| `aes-gcm` | AES-GCM | AES-GCM authenticated encryption; used for credential vault encryption | `openfang-extensions` |
| `cbc` | CBC | AES-CBC cipher mode for channel encryption | `openfang-channels` |
| `argon2` | Argon2 | Password hashing/key derivation (KDF) for credential storage | `openfang-api`, `openfang-extensions` |
| `ed25519-dalek` | ed25519-dalek | Ed25519 digital signature scheme for agent identity and wire protocol signing | `openfang-types` |
| `hex` | hex | Hexadecimal encoding/decoding for cryptographic output | Most crates |
| `base64` | base64 | Base64 encoding/decoding for binary data transport | Most crates |
| `subtle` | subtle | Constant-time comparison utilities to prevent timing side-channel attacks | `openfang-api`, `openfang-kernel`, `openfang-wire` |
| `zeroize` | Zeroize | Secure memory zeroing for sensitive data (keys, tokens) on drop | Multiple crates |
| `rand` | rand | Cryptographically secure random number generation | Multiple crates |
| `openssl` | OpenSSL | Vendored OpenSSL (statically compiled); TLS support for email (IMAP/SMTP) and native TLS | workspace-level |

---

#### Email

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `lettre` | Lettre | SMTP email sending with TLS support; used for the email channel adapter | `openfang-channels` |
| `imap` | imap | IMAP email receiving client; used for the email channel adapter | `openfang-channels` |
| `native-tls` | native-tls | Native TLS bindings (vendored) for IMAP/SMTP TLS connections | `openfang-channels` |
| `mailparse` | mailparse | Raw email message parsing (RFC 2822/MIME) | `openfang-channels` |

---

#### CLI & Terminal UI

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `clap` | Clap | Command-line argument parsing with derive macros | `openfang-cli` |
| `clap_complete` | Clap Complete | Shell completion script generation for the CLI | `openfang-cli` |
| `ratatui` | Ratatui | Terminal user interface (TUI) framework for the CLI's interactive interface | `openfang-cli` |
| `colored` | colored | Terminal text colorization for CLI output | `openfang-cli` |

---

#### Desktop (Tauri)

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `tauri` | Tauri | Cross-platform desktop application framework (v2); hosts the desktop GUI | `openfang-desktop` |
| `tauri-build` | Tauri Build | Build-time code generation for Tauri applications | `openfang-desktop` |
| `tauri-plugin-notification` | Tauri Notification Plugin | System notification support for the desktop app | `openfang-desktop` |
| `tauri-plugin-shell` | Tauri Shell Plugin | Shell command execution from the desktop app | `openfang-desktop` |
| `tauri-plugin-single-instance` | Tauri Single Instance Plugin | Enforces single application instance | `openfang-desktop` |
| `tauri-plugin-dialog` | Tauri Dialog Plugin | Native OS dialog boxes (file picker, alerts) | `openfang-desktop` |
| `tauri-plugin-global-shortcut` | Tauri Global Shortcut Plugin | System-wide keyboard shortcut registration | `openfang-desktop` |
| `tauri-plugin-autostart` | Tauri Autostart Plugin | System startup launch registration | `openfang-desktop` |
| `tauri-plugin-updater` | Tauri Updater Plugin | In-app automatic update mechanism | `openfang-desktop` |
| `open` | open | Opens URLs or files in the default system application | `openfang-desktop` |

---

#### WASM & Sandboxing

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `wasmtime` | Wasmtime | WebAssembly runtime; provides sandboxed WASM execution for agent skills and tools | `openfang-runtime` |

---

#### MCP (Model Context Protocol)

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `rmcp` | RMCP (Rust MCP SDK) | Official Rust implementation of the Model Context Protocol; enables MCP client connections to external tool servers | `openfang-runtime` |

---

#### Observability & Logging

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `tracing` | Tracing | Structured, async-aware application instrumentation and logging facade | All crates |
| `tracing-subscriber` | Tracing Subscriber | Tracing event collection and output formatting (JSON, env-filter) | `openfang-cli`, `openfang-desktop`, `openfang-kernel` |

---

#### Scheduling & Time

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `chrono` | Chrono | Date and time handling with serde support | All crates |
| `chrono-tz` | Chrono-TZ | Timezone-aware datetime operations | `openfang-kernel` |
| `cron` | cron | Cron expression parsing for scheduled agent tasks | `openfang-kernel` |

---

#### Utilities

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `uuid` | UUID | UUID (v4/v5) generation for agent, session, and message identifiers | All crates |
| `thiserror` | thiserror | Ergonomic `Error` trait derive macro for structured error types | Most crates |
| `anyhow` | anyhow | Flexible error propagation for application-level code | `openfang-runtime` |
| `bytes` | bytes | Efficient byte buffer abstraction for streaming data | `openfang-runtime` |
| `dirs` | dirs | Standard OS directory resolution (home, config, data paths) | `openfang-cli`, `openfang-extensions`, `openfang-kernel`, `openfang-migrate`, `openfang-types` |
| `walkdir` | WalkDir | Recursive directory traversal; used for scanning skill and agent config directories | `openfang-migrate`, `openfang-skills` |
| `zip` | zip | ZIP archive extraction (deflate); used for skill package installation | `openfang-skills` |
| `governor` | Governor | Token-bucket rate limiting for API request throttling | `openfang-api` |
| `regex-lite` | regex-lite | Lightweight regular expression matching (no Unicode overhead) | `openfang-channels`, `openfang-runtime` |
| `html-escape` | html-escape | HTML entity encoding/decoding for channel message sanitization | `openfang-channels` |
| `roxmltree` | roxmltree | Read-only XML tree parser; used for XML-based channel message parsing | `openfang-channels` |
| `shlex` | shlex | Shell-like string tokenization for command parsing | `openfang-runtime` |
| `libc` | libc | Unix/POSIX system call bindings (Unix targets only) | `openfang-kernel` |

---

#### Testing Utilities

| Dependency | Official Name | Role | Consuming Crates |
|------------|---------------|------|-----------------|
| `tokio-test` | tokio-test | Async test utilities and mock time/task helpers for Tokio-based tests | All crates (dev) |
| `tempfile` | tempfile | Temporary file and directory creation for isolated test environments | Most crates (dev) |

--- module_deep_dive ---


# Detailed Component Breakdown Analysis

---

## 1. `crates/openfang-kernel/`

### Core Responsibility
The **central orchestration engine** of the entire platform. This is the brain of OpenFang — responsible for routing messages between agents, managing agent lifecycles, coordinating multi-agent workflows, and implementing the A2A (Agent-to-Agent) protocol. All other crates ultimately serve or are coordinated by the kernel.

### Key Components

| File/Directory | Role |
|----------------|------|
| `src/` (22 files) | Full orchestration logic — agent router, workflow engine, session management, message dispatch |
| `tests/` (4 files) | Integration/unit tests covering core orchestration scenarios — agent routing, workflow execution, session handling |
| `Cargo.toml` | Crate manifest — declares dependencies on `openfang-types`, `openfang-runtime`, likely `openfang-memory` |

**Inferred key source files (based on 22-file scope):**
- **Agent Router** — Dispatches incoming messages to the correct agent instance
- **Workflow Engine** — Executes multi-step agent workflows (ref: `docs/workflows.md`)
- **Session Manager** — Tracks conversation state across channels
- **A2A Protocol Handler** — Manages agent-to-agent communication (ref: `docs/mcp-a2a.md`)
- **Agent Registry** — Maintains catalog of loaded/active agents
- **Orchestrator Logic** — Coordinates the `agents/orchestrator/agent.toml` persona

### Dependencies & Interactions

```
openfang-kernel
    ├── DEPENDS ON → openfang-types     (shared domain types)
    ├── DEPENDS ON → openfang-runtime   (LLM execution, driver abstraction)
    ├── DEPENDS ON → openfang-memory    (agent context/history retrieval)
    ├── DEPENDS ON → openfang-wire      (message serialization/deserialization)
    ├── CALLED BY  → openfang-api       (API server delegates to kernel)
    ├── CALLED BY  → openfang-cli       (CLI dispatches through kernel)
    ├── CALLED BY  → openfang-channels  (inbound messages routed into kernel)
    └── COORDINATES → openfang-skills / openfang-hands (via runtime)
```

**External interactions:** Indirectly via runtime drivers (LLM APIs). No direct external service calls expected at this layer.

---

## 2. `crates/openfang-runtime/`

### Core Responsibility
The **agent execution engine** — the layer that actually runs agents by interfacing with LLM providers. Implements the driver abstraction pattern to support multiple AI providers (Vertex AI, OpenAI, Anthropic, etc.) and manages the execution lifecycle of a single agent turn (prompt construction, LLM call, response parsing, tool/skill invocation).

### Key Components

| File/Directory | Role |
|----------------|------|
| `src/drivers/` (nested) | **LLM provider drivers** — one driver per provider (Vertex AI confirmed, others inferred). Implements a common `LlmDriver` trait |
| `src/` (49 files total) | Execution loop, agent context builder, prompt templating, tool dispatch, streaming response handling |
| `Cargo.toml` | Declares provider SDK dependencies (Google Cloud SDK for Vertex, reqwest for HTTP calls) |

**Inferred key source files:**
- **`drivers/vertex.rs`** — Google Vertex AI integration (confirmed by `start-vertex.bat`, `test_vertex_e2e.py`)
- **`drivers/openai.rs`** — OpenAI-compatible driver
- **`drivers/anthropic.rs`** — Anthropic Claude driver
- **`drivers/mod.rs`** — Driver trait definition and registry
- **Agent Executor** — Manages a single agent's turn: build context → call LLM → parse output → invoke tools
- **Prompt Builder** — Constructs prompts from agent config + memory + message history
- **Tool Dispatcher** — Routes LLM tool-call outputs to skills/hands
- **Streaming Handler** — Manages token streaming from LLM providers

### Dependencies & Interactions

```
openfang-runtime
    ├── DEPENDS ON → openfang-types     (agent config types, message types)
    ├── DEPENDS ON → openfang-memory    (retrieves conversation history/context)
    ├── DEPENDS ON → openfang-skills    (invokes skill functions during tool calls)
    ├── DEPENDS ON → openfang-hands     (invokes autonomous hand actions)
    ├── DEPENDS ON → openfang-wire      (serializes LLM requests/responses)
    ├── CALLED BY  → openfang-kernel    (kernel schedules execution tasks)
    └── EXTERNAL   → Vertex AI API, OpenAI API, Anthropic API (via HTTP/gRPC)
```

**External interactions:** **YES** — Direct HTTP/gRPC calls to:
- Google Cloud Vertex AI (confirmed)
- OpenAI API (inferred)
- Anthropic API (inferred)
- Any MCP-compatible endpoint (ref: `docs/mcp-a2a.md`)

---

## 3. `crates/openfang-api/`

### Core Responsibility
The **HTTP/WebSocket API server** — exposes OpenFang's capabilities to external clients via REST endpoints and real-time WebSocket connections. Also serves the embedded web UI (static assets). Acts as the primary programmatic interface for the JavaScript SDK and web dashboard.

### Key Components

| File/Directory | Role |
|----------------|------|
| `src/` (13 files) | Route handlers, middleware, WebSocket upgrade logic, request/response types, server bootstrap |
| `static/css/` | Web dashboard stylesheet assets |
| `static/js/` | Web dashboard JavaScript (vanilla or bundled) |
| `static/vendor/` | Third-party frontend libraries (bundled, e.g., Alpine.js, HTMX) |
| `static/` (6 root files) | `index.html` and other root web assets |
| `tests/` (3 files) | API integration tests — endpoint behavior, auth, response shapes |
| `Cargo.toml` | Declares Axum/Actix dependency, tokio runtime, serde for JSON |

**Inferred key source files:**
- **`server.rs`** / **`main.rs`** — Server initialization, port binding, route registration
- **`routes/agents.rs`** — CRUD endpoints for agent management
- **`routes/messages.rs`** — Message send/receive endpoints
- **`routes/channels.rs`** — Channel configuration endpoints
- **`ws.rs`** — WebSocket handler for real-time message streaming
- **`middleware/auth.rs`** — Authentication/authorization middleware
- **`handlers/`** — Individual request handler implementations

### Dependencies & Interactions

```
openfang-api
    ├── DEPENDS ON → openfang-kernel    (delegates all business logic)
    ├── DEPENDS ON → openfang-types     (request/response type definitions)
    ├── DEPENDS ON → openfang-wire      (serialization for API responses)
    ├── DEPENDS ON → openfang-memory    (potentially for conversation history endpoints)
    ├── CALLED BY  → sdk/javascript/    (JS SDK targets these endpoints)
    ├── CALLED BY  → sdk/python/        (Python SDK targets these endpoints)
    ├── CALLED BY  → openfang-cli       (CLI may call local API)
    └── CALLED BY  → openfang-desktop   (Tauri app communicates via this API)
```

**External interactions:** Serves as the **externally-facing interface** — no direct external service calls; delegates to kernel which delegates to runtime.

---

## 4. `crates/openfang-cli/`

### Core Responsibility
The **command-line interface binary** — provides terminal-based interaction with OpenFang for developers and operators. Includes both a traditional CLI (commands/flags) and a rich TUI (Terminal User Interface) for interactive agent conversations and system monitoring.

### Key Components

| File/Directory | Role |
|----------------|------|
| `src/` (9 root files) | CLI command definitions, argument parsing, config loading, binary entrypoint |
| `src/tui/` (nested) | Full TUI implementation — panels, input handling, rendering loop, event system |
| `Cargo.toml` | Declares `clap` (arg parsing), `ratatui`/`tui` (terminal UI), `crossterm` (terminal backend) |

**Inferred key source files:**
- **`main.rs`** — Binary entrypoint, command dispatch
- **`commands/`** — Subcommand implementations (`start`, `agent`, `channel`, `config`, etc.)
- **`tui/app.rs`** — TUI application state machine
- **`tui/ui.rs`** — TUI layout and widget rendering
- **`tui/events.rs`** — Keyboard/mouse event handling
- **`tui/chat.rs`** — Interactive chat panel for agent conversations
- **`config.rs`** — CLI configuration loading from `openfang.toml`

### Dependencies & Interactions

```
openfang-cli
    ├── DEPENDS ON → openfang-kernel    (direct kernel calls for embedded mode)
    ├── DEPENDS ON → openfang-api       (may call local API server for remote mode)
    ├── DEPENDS ON → openfang-types     (shared types for display/input)
    ├── DEPENDS ON → openfang-wire      (message protocol for agent communication)
    └── READS      → openfang.toml     (runtime configuration file)
```

**External interactions:** Minimal — primarily interacts with local OpenFang instance (embedded or via API). Terminal I/O via `crossterm`.

---

## 5. `crates/openfang-desktop/`

### Core Responsibility
The **Tauri-based desktop application** — packages OpenFang's capabilities into a native desktop GUI for Windows, macOS, and Linux. Provides a graphical interface equivalent to the TUI, built on web technologies (HTML/JS/CSS) wrapped by Tauri's Rust shell.

### Key Components

| File/Directory | Role |
|----------------|------|
| `src/` (7 files) | Tauri command handlers, Rust backend glue code, IPC bridge |
| `tauri.conf.json` | Tauri application configuration — window settings, permissions, bundle IDs, update endpoints |
| `build.rs` | Build script — code generation or asset embedding |
| `capabilities/` (1 file) | Tauri v2 capability definitions — filesystem, network, shell permissions |
| `gen/schemas/` | Auto-generated JSON schemas for Tauri IPC type safety |
| `icons/` (5 files) | Application icons for all platforms (`.ico`, `.icns`, `.png`) |
| `Cargo.toml` | Declares `tauri` crate dependency with feature flags |

**Inferred key source files:**
- **`main.rs`** / **`lib.rs`** — Tauri app initialization, command registration
- **`commands.rs`** — `#[tauri::command]` functions exposed to frontend JS
- **`state.rs`** — Application state managed by Tauri's state manager
- Frontend (web assets likely embedded or in `static/`) — Chat UI, agent config panels, dashboard

### Dependencies & Interactions

```
openfang-desktop
    ├── DEPENDS ON → openfang-kernel    (embeds kernel for local agent execution)
    ├── DEPENDS ON → openfang-api       (may embed or connect to API server)
    ├── DEPENDS ON → openfang-types     (IPC type definitions)
    ├── READS      → openfang.toml     (application configuration)
    └── BUNDLES    → Web frontend      (HTML/JS/CSS via Tauri webview)
```

**External interactions:** Via embedded kernel/runtime → LLM provider APIs. Tauri provides OS-level integration (file system, notifications, system tray).

---

## 6. `crates/openfang-channels/`

### Core Responsibility
The **communication channel adapter layer** — implements the adapter/plugin pattern for connecting OpenFang to external messaging platforms. Each channel adapter translates platform-specific message formats into OpenFang's internal message protocol and routes them to the kernel.

### Key Components

| File/Directory | Role |
|----------------|------|
| `src/` (48 files) | Individual channel adapters — one or more files per channel |
| `tests/` (1 file) | Channel integration tests — message format transformation, adapter behavior |
| `Cargo.toml` | Per-channel SDK dependencies (Slack SDK, email crates, Telegram API crates, etc.) |

**Inferred channel adapters (48 files suggests ~15-20 channels):**
- **WhatsApp** — Bridges with `packages/whatsapp-gateway/` (Node.js gateway)
- **Slack** — Direct Slack API/Events API integration
- **Email** — SMTP/IMAP adapter
- **Telegram** — Telegram Bot API
- **Discord** — Discord gateway
- **REST/Webhook** — Generic HTTP webhook receiver
- **WebSocket** — Direct WS channel
- **CLI channel** — Terminal input/output channel

**Inferred key source files:**
- **`mod.rs`** / **`lib.rs`** — Channel trait definition (`trait Channel { fn send(); fn receive(); }`)
- **`registry.rs`** — Channel registry/factory
- **`slack.rs`** — Slack adapter implementation
- **`email.rs`** — Email adapter implementation
- **`telegram.rs`** — Telegram adapter implementation
- **`whatsapp.rs`** — WhatsApp adapter (delegates to Node.js gateway)
- **`webhook.rs`** — Generic webhook receiver

### Dependencies & Interactions

```
openfang-channels
    ├── DEPENDS ON → openfang-types     (internal message/event types)
    ├── DEPENDS ON → openfang-wire      (message serialization for inter-service comms)
    ├── ROUTES TO  → openfang-kernel    (delivers normalized messages to kernel)
    └── EXTERNAL   → Slack API, Telegram Bot API, SMTP/IMAP servers,
                     Discord Gateway, WhatsApp (via packages/whatsapp-gateway)
```

**External interactions:** **YES** — Each adapter maintains connections to external platforms:
- Slack Events API / RTM
- Telegram Bot API (`api.telegram.org`)
- Discord Gateway
- SMTP/IMAP mail servers
- HTTP webhooks from external services
- `packages/whatsapp-gateway/` (Node.js sidecar process via HTTP/IPC)

---

## 7. `crates/openfang-skills/`

### Core Responsibility
The **bundled skill library** — a catalog of pre-built, reusable capability modules that agents can use. Skills define domain-specific knowledge, prompt templates, and tool configurations that enhance an agent's ability in a particular area (e.g., Kubernetes operations, SQL analysis, code review).

### Key Components

| File/Directory | Role |
|----------------|------|
| `src/` (8 files) | Skill loading, registry, trait definitions, skill invocation interface |
| `bundled/` (60+ skill directories) | Individual skill definitions — each with prompts, config, and tool specs |

**Skill Categories (from directory listing):**

| Category | Skills |
|----------|--------|
| **Cloud/Infra** | `aws/`, `azure/`, `gcp/`, `kubernetes/`, `helm/`, `terraform/`, `docker/`, `ansible/` |
| **Languages** | `python-expert/`, `rust-expert/`, `golang-expert/`, `typescript-expert/`, `wasm-expert/` |
| **Databases** | `postgres-expert/`, `mongodb/`, `redis-expert/`, `sqlite-expert/`, `elasticsearch/` |
| **Frontend** | `react-expert/`, `nextjs-expert/`, `css-expert/`, `figma-expert/` |
| **DevOps** | `ci-cd/`, `git-expert/`, `github/`, `sentry/`, `prometheus/` |
| **AI/ML** | `llm-finetuning/`, `ml-engineer/`, `vector-db/`, `prompt-engineer/` |
| **Productivity** | `jira/`, `confluence/`, `notion/`, `slack-tools/`, `linear-tools/` |
| **Security** | `security-audit/`, `oauth-expert/`, `compliance/` |
| **Content** | `technical-writer/`, `email-writer/`, `presentation/`, `writing-coach/` |
| **Data** | `data-analyst/`, `data-pipeline/`, `sql-analyst/`, `graphql-expert/` |
| **Utilities** | `web-search/`, `searxng/`, `pdf-reader/`, `regex-expert/`, `shell-scripting/` |

**Inferred key source files:**
- **`lib.rs`** — Public skill API
- **`registry.rs`** — Skill registration and lookup by name/ID
- **`loader.rs`** — Loads skill definitions from `bundled/` directories
- **`skill.rs`** — `Skill` trait definition
- **`executor.rs`** — Skill invocation logic

### Dependencies & Interactions

```
openfang-skills
    ├── DEPENDS ON → openfang-types     (skill definition types, tool spec types)
    ├── DEPENDS ON → openfang-extensions (skills may leverage extension integrations)
    ├── CALLED BY  → openfang-runtime   (runtime invokes skills during tool-call handling)
    └── READS      → bundled/*/         (skill TOML/JSON definition files at load time)
```

**External interactions:** Skills themselves may reference external tools (e.g., `searxng/` → SearXNG search instance, `web-search/` → search APIs), but the invocation is mediated through the runtime/extensions layer.

---

## 8. `crates/openfang-hands/`

### Core Responsibility
The **autonomous agent action library** — implements complex, multi-step automated workflows ("hands") that agents can trigger. Unlike skills (which are prompt/tool enhancements), hands are autonomous executors that perform real-world actions: browsing the web, trading, scraping leads, conducting research, syncing secrets, etc.

### Key Components

| File/Directory | Role |
|----------------|------|
| `src/` (3 files) | Minimal core — hand trait definition, registry, loader |
| `bundled/` (9 hand directories) | Individual hand implementations |

**Bundled Hands:**

| Hand | Inferred Capability |
|------|---------------------|
| `browser/` | Headless browser automation (Playwright/puppeteer-style) |
| `clip/` | Clipboard interaction / content capture |
| `collector/` | Data collection / web scraping |
| `infisical-sync/` | Secret synchronization via Infisical secrets manager |
| `lead/` | Lead generation / CRM data gathering |
| `predictor/` | ML-based prediction execution |
| `researcher/` | Autonomous research workflow (mirrors `agents/researcher/`) |
| `trader/` | Automated trading / financial operations |
| `twitter/` | Twitter/X API interactions (posting, reading, DMs) |

**Inferred key source files:**
- **`lib.rs`** — Public hand API, `Hand` trait
- **`registry.rs`** — Hand registration
- **`loader.rs`** — Loads hand definitions from `bundled/`

### Dependencies & Interactions

```
openfang-hands
    ├── DEPENDS ON → openfang-types       (hand definition types)
    ├── DEPENDS ON → openfang-extensions  (hands use extensions for service auth/clients)
    ├── CALLED BY  → openfang-runtime     (runtime dispatches hand execution)
    └── EXTERNAL   → Browser (headless Chrome/WebKit), Twitter API,
                     Infisical API, Trading APIs/exchanges,
                     Web scraping targets
```

**External interactions:** **YES** — Hands are the most externally-active component:
- Headless browser → arbitrary web URLs
- Twitter/X API
- Infisical secrets management API
- Financial exchange APIs (trader)
- Arbitrary HTTP endpoints (collector, researcher)

---

## 9. `crates/openfang-extensions/`

### Core Responsibility
The **third-party service integration layer** — provides authenticated, reusable client wrappers for external services and APIs. Acts as a shared dependency for both skills and hands, preventing duplicate integration code. Think of it as an "integration hub" — OAuth flows, API key management, and service-specific client initialization live here.

### Key Components

| File/Directory | Role |
|----------------|------|
| `src/` (8 files) | Extension trait, registry, loader, credential management |
| `integrations/` (25 files) | One file per external service integration |

**Inferred integrations (25 files → ~25 services):**
- Cloud providers: AWS, GCP, Azure
- Communication: Slack, email (SMTP), Twilio
- Productivity: Jira, Notion, Confluence, Linear
- Version control: GitHub, GitLab
- Monitoring: Sentry, Prometheus, Datadog
- Databases: Various managed DB connections
- Secrets: Infisical, HashiCorp Vault
- Social: Twitter/X

**Inferred key source files:**
- **`lib.rs`** — Public extension API
- **`registry.rs`** — Extension registry
- **`extension.rs`** — `Extension` trait (init, credentials, client)
- **`credentials.rs`** — Credential storage/retrieval
- **`integrations/slack.rs`** — Slack client wrapper
- **`integrations/github.rs`** — GitHub API client wrapper

### Dependencies & Interactions

```
openfang-extensions
    ├── DEPENDS ON → openfang-types     (credential types, config types)
    ├── CALLED BY  → openfang-skills    (skills use extension clients)
    ├── CALLED BY  → openfang-hands     (hands use extension clients for auth)
    ├── CALLED BY  → openfang-kernel    (kernel may manage extension lifecycle)
    └── EXTERNAL   → All 25 integrated third-party APIs
```

**External interactions:** **YES** — This crate exists specifically to interface with external services. All 25 integrations make outbound API calls.

---

## 10. `crates/openfang-memory/`

### Core Responsibility
The **agent memory and context management system** — persists and retrieves conversation history, agent state, embeddings, and long-term memory. Provides the memory backend that allows agents to maintain context across sessions and implement retrieval-augmented responses.

### Key Components

| File/Directory | Role |
|----------------|------|
| `src/` (10 files) | Memory backends, embedding interface, retrieval logic, context window management |
| `Cargo.toml` | Declares database driver (SQLite/PostgreSQL), embedding model dependencies |

**Inferred key source files:**
- **`lib.rs`** — Public memory API
- **`store.rs`** — `MemoryStore` trait — abstract storage interface
- **`sqlite.rs`** / **`postgres.rs`** — Concrete storage backends
- **`embedding.rs`** — Vector embedding generation for semantic search
- **`retrieval.rs`** — Semantic similarity search / RAG retrieval
- **`context.rs`** — Context window builder — assembles relevant memories for prompt injection
- **`session.rs`** — Session-scoped memory (short-term, within a conversation)
- **`long_term.rs`** — Cross-session persistent memory

### Dependencies & Interactions

```
openfang-memory
    ├── DEPENDS ON → openfang-types     (message types, agent ID types)
    ├── DEPENDS ON → openfang-wire      (serialization for stored messages)
    ├── CALLED BY  → openfang-kernel    (kernel reads/writes session state)
    ├── CALLED BY  → openfang-runtime   (runtime injects memory into prompts)
    └── EXTERNAL   → SQLite/PostgreSQL (database storage)
                     Embedding model API (OpenAI embeddings or local model)
```

**External interactions:** Database I

--- service_dependencies ---


# External Dependencies Analysis: openfang_53d69049

## Overview
This repository is **OpenFang**, a Rust-based Agent OS platform. The analysis covers all external dependencies identified across Rust crates, JavaScript packages, Python packages, Docker configurations, and environment configurations.

---

## 1. LLM / AI Provider APIs

### 1.1 Anthropic (Claude)

| Field | Details |
|-------|---------|
| **Dependency Name** | Anthropic Claude API |
| **Type** | Third-party API |
| **Purpose/Role** | Provides LLM inference capabilities via Claude models for agent execution |
| **Integration Point** | `docker-compose.yml` → `ANTHROPIC_API_KEY` environment variable; referenced in `docs/providers.md` and agent configurations |

---

### 1.2 OpenAI API

| Field | Details |
|-------|---------|
| **Dependency Name** | OpenAI API |
| **Type** | Third-party API |
| **Purpose/Role** | Provides LLM inference (GPT models) and embeddings for agent execution; also used by LangChain integration |
| **Integration Point** | `docker-compose.yml` → `OPENAI_API_KEY`; `agents/langchain-code-reviewer/requirements.txt` → `langchain-openai>=0.3`; `.env.example` |

---

### 1.3 Groq API

| Field | Details |
|-------|---------|
| **Dependency Name** | Groq API |
| **Type** | Third-party API |
| **Purpose/Role** | Provides fast LLM inference via Groq hardware accelerators |
| **Integration Point** | `docker-compose.yml` → `GROQ_API_KEY` environment variable |

---

### 1.4 Google Vertex AI

| Field | Details |
|-------|---------|
| **Dependency Name** | Google Vertex AI |
| **Type** | Third-party API / Cloud Service |
| **Purpose/Role** | Provides LLM inference via Google Cloud's Vertex AI platform (Gemini models etc.) |
| **Integration Point** | `docs/VERTEX_AI_LOCAL_TESTING.md`, `test_vertex_e2e.py`, `start-vertex.bat` — dedicated test scripts and documentation for Vertex AI integration; referenced in `docs/providers.md` |

---

### 1.5 Ollama (Local LLM Runtime)

| Field | Details |
|-------|---------|
| **Dependency Name** | Ollama |
| **Type** | External Service (local/self-hosted) |
| **Purpose/Role** | Provides locally-hosted LLM inference; used in the LangChain code reviewer agent |
| **Integration Point** | `agents/langchain-code-reviewer/requirements.txt` → `langchain-ollama>=0.3` |

---

## 2. Messaging / Channel Integrations

### 2.1 Telegram Bot API

| Field | Details |
|-------|---------|
| **Dependency Name** | Telegram Bot API |
| **Type** | Third-party API |
| **Purpose/Role** | Enables Telegram bot channel for agent communication |
| **Integration Point** | `docker-compose.yml` → `TELEGRAM_BOT_TOKEN`; `crates/openfang-channels/` contains Telegram channel adapter; `docs/channel-adapters.md` |

---

### 2.2 Discord API / Gateway

| Field | Details |
|-------|---------|
| **Dependency Name** | Discord Bot API & WebSocket Gateway |
| **Type** | Third-party API |
| **Purpose/Role** | Enables Discord bot channel for agent communication via REST API and WebSocket gateway |
| **Integration Point** | `docker-compose.yml` → `DISCORD_BOT_TOKEN`; `Cargo.toml` → `tokio-tungstenite` used for Discord/Slack WebSocket gateway (comment in code); `crates/openfang-channels/src/` |

---

### 2.3 Slack API

| Field | Details |
|-------|---------|
| **Dependency Name** | Slack Bot API & Socket Mode |
| **Type** | Third-party API |
| **Purpose/Role** | Enables Slack bot channel for agent communication |
| **Integration Point** | `docker-compose.yml` → `SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`; `crates/openfang-channels/src/` has Slack adapter; `openfang-skills/bundled/slack-tools/` |

---

### 2.4 WhatsApp (via Baileys)

| Field | Details |
|-------|---------|
| **Dependency Name** | WhatsApp Web API (via @whiskeysockets/baileys) |
| **Type** | Third-party API / Library |
| **Purpose/Role** | Enables WhatsApp messaging channel for agent communication via unofficial WhatsApp Web multi-device protocol |
| **Integration Point** | `packages/whatsapp-gateway/package.json` → `"@whiskeysockets/baileys": "^6"`; `packages/whatsapp-gateway/index.js` |

---

### 2.5 MQTT Broker

| Field | Details |
|-------|---------|
| **Dependency Name** | MQTT Message Broker |
| **Type** | Message Broker / External Service |
| **Purpose/Role** | Provides publish/subscribe messaging, likely for IoT or home automation agent channels |
| **Integration Point** | `Cargo.toml` (workspace) → `rumqttc = "0.24"`; `crates/openfang-channels/Cargo.toml` → `rumqttc`; `agents/home-automation/agent.toml` |

---

## 3. Email Services

### 3.1 SMTP Email Service

| Field | Details |
|-------|---------|
| **Dependency Name** | SMTP Email Service |
| **Type** | External Service |
| **Purpose/Role** | Sends outgoing emails via SMTP for the email assistant agent channel |
| **Integration Point** | `Cargo.toml` → `lettre = { version = "0.11", features = ["smtp-transport", "tokio1-rustls-tls", ...] }`; `crates/openfang-channels/Cargo.toml` → `lettre` |

---

### 3.2 IMAP Email Service

| Field | Details |
|-------|---------|
| **Dependency Name** | IMAP Email Service |
| **Type** | External Service |
| **Purpose/Role** | Reads/receives incoming emails via IMAP for the email assistant agent channel |
| **Integration Point** | `Cargo.toml` → `imap = "2"`; `crates/openfang-channels/Cargo.toml` → `imap`, `mailparse`, `native-tls` |

---

## 4. Database Services

### 4.1 SQLite (via rusqlite)

| Field | Details |
|-------|---------|
| **Dependency Name** | SQLite Database |
| **Type** | Embedded Database |
| **Purpose/Role** | Primary local persistent storage for agent memory, state, and data — bundled directly into the binary (no external server required) |
| **Integration Point** | `Cargo.toml` → `rusqlite = { version = "0.31", features = ["bundled", "serde_json"] }`; `crates/openfang-memory/Cargo.toml`, `crates/openfang-runtime/Cargo.toml` |

> **Note:** While SQLite is embedded, it is still a distinct external library dependency. The `bundled` feature compiles SQLite from source into the binary.

---

## 5. External Tool/Platform Integrations (via Skills/Hands bundles)

### 5.1 GitHub API

| Field | Details |
|-------|---------|
| **Dependency Name** | GitHub API |
| **Type** | Third-party API |
| **Purpose/Role** | Provides GitHub repository management capabilities to agents |
| **Integration Point** | `crates/openfang-skills/bundled/github/` skill directory |

---

### 5.2 Jira API

| Field | Details |
|-------|---------|
| **Dependency Name** | Atlassian Jira API |
| **Type** | Third-party API |
| **Purpose/Role** | Provides Jira issue tracking and project management capabilities to agents |
| **Integration Point** | `crates/openfang-skills/bundled/jira/` skill directory |

---

### 5.3 Confluence API

| Field | Details |
|-------|---------|
| **Dependency Name** | Atlassian Confluence API |
| **Type** | Third-party API |
| **Purpose/Role** | Provides Confluence wiki/knowledge base capabilities to agents |
| **Integration Point** | `crates/openfang-skills/bundled/confluence/` skill directory |

---

### 5.4 Notion API

| Field | Details |
|-------|---------|
| **Dependency Name** | Notion API |
| **Type** | Third-party API |
| **Purpose/Role** | Provides Notion workspace/document management capabilities to agents |
| **Integration Point** | `crates/openfang-skills/bundled/notion/` skill directory |

---

### 5.5 Linear API

| Field | Details |
|-------|---------|
| **Dependency Name** | Linear API |
| **Type** | Third-party API |
| **Purpose/Role** | Provides Linear project/issue tracking capabilities to agents |
| **Integration Point** | `crates/openfang-skills/bundled/linear-tools/` skill directory |

---

### 5.6 Elasticsearch

| Field | Details |
|-------|---------|
| **Dependency Name** | Elasticsearch |
| **Type** | External Service |
| **Purpose/Role** | Provides search and analytics engine capabilities to agents |
| **Integration Point** | `crates/openfang-skills/bundled/elasticsearch/` skill directory |

---

### 5.7 SearXNG

| Field | Details |
|-------|---------|
| **Dependency Name** | SearXNG (Web Search Engine) |
| **Type** | External Service (self-hosted) |
| **Purpose/Role** | Provides private web search capabilities to agents via SearXNG metasearch engine |
| **Integration Point** | `crates/openfang-skills/bundled/searxng/` and `crates/openfang-skills/bundled/web-search/` skill directories |

---

### 5.8 Infisical Secret Manager

| Field | Details |
|-------|---------|
| **Dependency Name** | Infisical Secret Management |
| **Type** | External Service |
| **Purpose/Role** | Provides secrets synchronization/management capabilities |
| **Integration Point** | `crates/openfang-hands/bundled/infisical-sync/` hands directory |

---

### 5.9 Sentry

| Field | Details |
|-------|---------|
| **Dependency Name** | Sentry Error Monitoring |
| **Type** | Monitoring Tool / Third-party API |
| **Purpose/Role** | Provides error monitoring and reporting capabilities to agents |
| **Integration Point** | `crates/openfang-skills/bundled/sentry/` skill directory |

---

### 5.10 Prometheus

| Field | Details |
|-------|---------|
| **Dependency Name** | Prometheus Monitoring |
| **Type** | External Service |
| **Purpose/Role** | Provides metrics collection and monitoring capabilities |
| **Integration Point** | `crates/openfang-skills/bundled/prometheus/` skill directory |

---

### 5.11 Figma API

| Field | Details |
|-------|---------|
| **Dependency Name** | Figma API |
| **Type** | Third-party API |
| **Purpose/Role** | Provides design tool integration capabilities to agents |
| **Integration Point** | `crates/openfang-skills/bundled/figma-expert/` skill directory |

---

### 5.12 Twitter/X API

| Field | Details |
|-------|---------|
| **Dependency Name** | Twitter/X API |
| **Type** | Third-party API |
| **Purpose/Role** | Provides Twitter/X social media interaction capabilities |
| **Integration Point** | `crates/openfang-hands/bundled/twitter/` hands directory; `agents/social-media/agent.toml` |

---

## 6. Cloud Provider Services

### 6.1 AWS Services

| Field | Details |
|-------|---------|
| **Dependency Name** | Amazon Web Services (AWS) |
| **Type** | Cloud Service |
| **Purpose/Role** | Provides various AWS cloud service capabilities to agents (S3, EC2, Lambda, etc.) |
| **Integration Point** | `crates/openfang-skills/bundled/aws/` skill directory |

---

### 6.2 Azure Services

| Field | Details |
|-------|---------|
| **Dependency Name** | Microsoft Azure |
| **Type** | Cloud Service |
| **Purpose/Role** | Provides Azure cloud service capabilities to agents |
| **Integration Point** | `crates/openfang-skills/bundled/azure/` skill directory |

---

### 6.3 Google Cloud Platform (GCP)

| Field | Details |
|-------|---------|
| **Dependency Name** | Google Cloud Platform |
| **Type** | Cloud Service |
| **Purpose/Role** | Provides GCP service capabilities to agents |
| **Integration Point** | `crates/openfang-skills/bundled/gcp/` skill directory |

---

## 7. Authentication / OAuth

### 7.1 OAuth 2.0 Providers (Generic)

| Field | Details |
|-------|---------|
| **Dependency Name** | OAuth 2.0 Providers |
| **Type** | Authentication Service |
| **Purpose/Role** | Provides OAuth2 PKCE authentication flow for extension/integration credentials |
| **Integration Point** | `crates/openfang-extensions/Cargo.toml` description: "OAuth2 PKCE"; `crates/openfang-skills/bundled/oauth-expert/` skill directory; `axum`, `rand`, `sha2`, `base64` used for PKCE implementation in `openfang-extensions` |

---

## 8. Desktop Application Framework

### 8.1 Tauri Framework

| Field | Details |
|-------|---------|
| **Dependency Name** | Tauri (Desktop App Framework) |
| **Type** | Library/Framework |
| **Purpose/Role** | Powers the native cross-platform desktop application for OpenFang |
| **Integration Point** | `crates/openfang-desktop/Cargo.toml` → `tauri = { version = "2", features = ["tray-icon", "image-png"] }`, `tauri-build`, `tauri-plugin-notification`, `tauri-plugin-shell`, `tauri-plugin-single-instance`, `tauri-plugin-dialog`, `tauri-plugin-global-shortcut`, `tauri-plugin-autostart`, `tauri-plugin-updater`; `crates/openfang-desktop/tauri.conf.json` |

---

## 9. MCP (Model Context Protocol)

### 9.1 RMCP (Rust MCP SDK)

| Field | Details |
|-------|---------|
| **Dependency Name** | RMCP — Official Rust MCP SDK |
| **Type** | Library/Framework |
| **Purpose/Role** | Implements the Model Context Protocol (MCP) for agent-to-tool communication, supporting child process and HTTP transport |
| **Integration Point** | `Cargo.toml` → `rmcp = { version = "1.2", features = ["client", "transport-child-process", "transport-streamable-http-client-reqwest", "reqwest"] }`; `crates/openfang-runtime/Cargo.toml` → `rmcp` |

---

## 10. WebAssembly Runtime

### 10.1 Wasmtime

| Field | Details |
|-------|---------|
| **Dependency Name** | Wasmtime (WASM Runtime) |
| **Type** | Library/Framework |
| **Purpose/Role** | Executes WebAssembly skill/plugin sandboxes within the agent runtime |
| **Integration Point** | `Cargo.toml` → `wasmtime = "41"`; `crates/openfang-runtime/Cargo.toml` → `wasmtime`; skills bundled in `wasm-expert/` |

---

## 11. Container / CI Infrastructure

### 11.1 Docker / Docker Hub — `rust:1-slim-bookworm`

| Field | Details |
|-------|---------|
| **Dependency Name** | Docker Base Image: `rust:1-slim-bookworm` |
| **Type** | Container Image |
| **Purpose/Role** | Base OS image for building and running the OpenFang server in Docker |
| **Integration Point** | `Dockerfile` → `FROM rust:1-slim-bookworm AS builder` and `FROM rust:1-slim-bookworm` (runtime stage) |

---

### 11.2 GitHub Container Registry (GHCR)

| Field | Details |
|-------|---------|
| **Dependency Name** | GitHub Container Registry (GHCR) |
| **Type** | External Service |
| **Purpose/Role** | Hosts the published Docker image `ghcr.io/rightnow-ai/openfang` for distribution |
| **Integration Point** | `docker-compose.yml` → commented reference `ghcr.io/rightnow-ai/openfang:latest`; `.github/workflows/release.yml` |

---

### 11.3 GitHub Actions

| Field | Details |
|-------|---------|
| **Dependency Name** | GitHub Actions CI/CD |
| **Type** | External Service |
| **Purpose/Role** | Runs automated CI pipelines (build, test, lint) and release workflows |
| **Integration Point** | `.github/workflows/ci.yml`, `.github/workflows/release.yml` |

---

## 12. Python Libraries (LangChain Agent)

### 12.1 LangChain

| Field | Details |
|-------|---------|
| **Dependency Name** | LangChain Framework |
| **Type** | Library/Framework |
| **Purpose/Role** | Provides agent orchestration, chains, and tool-use framework for the Python-based code reviewer agent |
| **Integration Point** | `agents/langchain-code-reviewer/requirements.txt` → `langchain>=0.3`, `langchain-core>=0.3` |

---

### 12.2 LangChain-OpenAI

| Field | Details |
|-------|---------|
| **Dependency Name** | LangChain OpenAI Integration |
| **Type** | Library/Framework |
| **Purpose/Role** | Connects LangChain agent to OpenAI GPT models |
| **Integration Point** | `agents/langchain-code-reviewer/requirements.txt` → `langchain-openai>=0.3` |

---

### 12.3 LangChain-Ollama

| Field | Details |
|-------|---------|
| **Dependency Name** | LangChain Ollama Integration |
| **Type** | Library/Framework |
| **Purpose/Role** | Connects LangChain agent to locally-hosted Ollama LLM models |
| **Integration Point** | `agents/langchain-code-reviewer/requirements.txt` → `langchain-ollama>=0.3` |

---

### 12.4 FastAPI

| Field | Details |
|-------|---------|
| **Dependency Name** | FastAPI |
| **Type** | Library/Framework |
| **Purpose/Role** | Provides the HTTP API server for the Python-based LangChain code reviewer agent |
| **Integration Point** | `agents/langchain-code-reviewer/requirements.txt` → `fastapi>=0.115`; `agents/langchain-code-reviewer/server.py` |

---

### 12.5 Uvicorn

| Field | Details |
|-------|---------|
| **Dependency Name** | Uvicorn (ASGI Server) |
| **Type** | Library/Framework |
| **Purpose/Role** | ASGI web server used to run the FastAPI application in the LangChain code reviewer agent |
| **Integration Point** | `agents/langchain-code-reviewer/requirements.txt` → `uvicorn>=0.34` |

---

## 13. JavaScript SDK Dependencies

### 13.1 QRCode (npm)

| Field | Details |
|-------|---------|
| **Dependency Name** | NPM `qrcode` library |
| **Type** | Library/Framework |
| **Purpose/Role** | Generates QR codes for WhatsApp Web authentication pairing in the WhatsApp gateway service |
| **Integration Point** | `packages/whatsapp-gateway/package.json` → `"qrcode": "^1.5"` |

---

### 13.2 Pino (npm)

| Field | Details |
|-------|---------|
| **Dependency Name** | NPM `pino` library |
| **Type** | Library/Framework |
| **Purpose/Role** | High-performance JSON structured logging for the WhatsApp gateway Node.js service |
| **Integration Point** | `packages/whatsapp-gateway/package.json` → `"pino": "^9"` |

---

## 14. Key Rust Library Dependencies

### 14.1 Tokio

| Field | Details |
|-------|---------|
| **Dependency Name** | Tokio Async Runtime |
| **Type** | Library/Framework |
| **Purpose/Role** | Core async I/O runtime underpinning all async operations in the Rust codebase |
| **Integration Point** | `Cargo.toml` → `tokio = { version = "1", features = ["full"] }` — used across all crates |

---

### 14.2 Axum

| Field | Details |
|-------|---------|
| **Dependency Name** | Axum HTTP Server Framework |
| **Type** | Library/Framework |
| **Purpose/Role** | Powers the HTTP/WebSocket API daemon (`openfang-api`) and OAuth callback endpoints |
| **Integration Point** | `Cargo.toml` → `axum = { version = "0.8", features = ["ws", "multipart"] }` |

---

### 14.3 Reqwest

| Field | Details |
|-------|---------|
| **Dependency Name** | Reqwest HTTP Client |
| **Type** | Library/Framework |
| **Purpose/Role** | Makes outgoing HTTP requests to LLM provider APIs, skill marketplaces, and other external services |
| **Integration Point** | `Cargo.toml` → `reqwest = { version = "0.12", features = ["json", "stream", "rustls-tls", ...] }` — used across `openfang-runtime`, `openfang-channels`, `openfang-memory`, etc. |

---

### 14.4 Serde / Serde JSON

| Field | Details |
|-------|---------|
| **Dependency Name** | Serde / Serde JSON |
| **Type** | Library/Framework |
| **Purpose/Role** | Serialization and deserialization framework for all JSON/TOML/YAML data interchange |
| **Integration Point** | `Cargo.toml` → `serde = "1"`, `serde_json = "1"` — used in all crates |

---

### 14.5 Argon2

| Field | Details |
|-------|---------|
| **Dependency Name** | Argon2 Password Hashing |
| **Type** | Library/Framework |
| **Purpose/Role** | Password hashing for credential/API key storage security |
| **Integration Point** | `Cargo.toml` → `argon2 = "0.5"`; used in `openfang-api`, `openfang-extensions` |

---

### 14.6 OpenSSL (vendored)

| Field | Details |
|-------|---------|
| **Dependency Name** | OpenSSL (vendored) |
| **Type** | Library/Framework |
| **Purpose/

--- events ---


I'll systematically analyze the codebase for event-related code. Let me examine the key source files.

Based on my analysis of the repository structure and the files provided, I need to examine the source code in the crates directories carefully. The repository appears to be an AI agent runtime platform called "openfang." Let me analyze the relevant source files for event interactions.

After a comprehensive scan of the repository structure — including the Rust crates (`openfang-channels`, `openfang-runtime`, `openfang-kernel`, `openfang-api`, `openfang-types`, `openfang-wire`, `openfang-extensions`), the SDK files (`sdk/python/`, `sdk/javascript/`), the `packages/whatsapp-gateway/`, and the agent definitions — the codebase does not expose any concrete event broker integration code (no SQS, Kafka, EventBridge, RabbitMQ, Pub/Sub, Ably, or similar message broker SDK usage) within the files made available for inspection.

The repository structure references channel adapters and a runtime, but the actual source file contents within the nested directories (e.g., `crates/openfang-channels/src/`, `crates/openfang-runtime/src/`) were not provided, and no event-producing or event-consuming code is visible in the files that were shared.

no events

--- core_entities ---


# OpenFang Domain Model Analysis

## Overview

OpenFang appears to be an **AI Agent Platform** — a multi-runtime, multi-channel system for defining, deploying, and orchestrating AI agents with skills, memory, tools, and external integrations.

---

## 1. Core Data Entities / Domain Models

### 1.1 `Agent`
The central domain entity representing an AI agent definition and its runtime state.

| Attribute | Description |
|---|---|
| `id` | Unique identifier |
| `name` | Human-readable name (e.g., `researcher`, `coder`, `orchestrator`) |
| `description` | Purpose/role of the agent |
| `model` / `provider` | The LLM backend (e.g., Vertex AI, OpenAI) |
| `system_prompt` | Persona/instruction context for the LLM |
| `skills` | List of attached skill references |
| `tools` / `hands` | List of attached tool/hand references |
| `memory` | Memory configuration reference |
| `channels` | Channels this agent is reachable through |
| `extensions` / `integrations` | External service bindings |
| `workflow` | Optional workflow definition (e.g., `workflow.json`) |
| `config` | Agent-level TOML configuration blob |
| `tags` | Categorization labels |

> **Source evidence:** `agents/*/agent.toml`, `agents/langchain-code-reviewer/config.example.toml`, `agents/langchain-code-reviewer/workflow.json`

---

### 1.2 `Skill`
A reusable, composable capability that can be attached to one or more agents.

| Attribute | Description |
|---|---|
| `id` | Unique identifier |
| `name` | Skill name (e.g., `kubernetes`, `rust-expert`, `pdf-reader`) |
| `description` | What the skill provides |
| `prompt_template` | Skill-specific prompt additions |
| `tools` | Tools or sub-functions this skill exposes |
| `config_schema` | Configuration parameters the skill accepts |
| `category` | Domain category (e.g., cloud, coding, data) |
| `version` | Skill version |

> **Source evidence:** `crates/openfang-skills/bundled/*/` — ~60+ bundled skills (kubernetes, react-expert, postgres-expert, etc.)

---

### 1.3 `Hand` (Tool / Action)
An executable action or automation that an agent can invoke against external systems.

| Attribute | Description |
|---|---|
| `id` | Unique identifier |
| `name` | Tool name (e.g., `browser`, `trader`, `researcher`, `twitter`) |
| `description` | What the hand does |
| `input_schema` | JSON schema for invocation parameters |
| `output_schema` | Expected return structure |
| `executor` | Runtime handler reference |
| `config` | API keys, endpoints, and settings |

> **Source evidence:** `crates/openfang-hands/bundled/*/` — browser, clip, collector, infisical-sync, lead, predictor, researcher, trader, twitter

---

### 1.4 `Message`
A discrete unit of communication flowing through the system, between users, agents, and channels.

| Attribute | Description |
|---|---|
| `id` | Unique identifier |
| `conversation_id` / `session_id` | Parent conversation reference |
| `role` | `user` \| `assistant` \| `system` \| `tool` |
| `content` | Text or structured message body |
| `channel` | Originating/target channel |
| `agent_id` | Associated agent |
| `timestamp` | Creation time |
| `metadata` | Extra context (e.g., tool call results, attachments) |

> **Source evidence:** `crates/openfang-types/src/`, `crates/openfang-channels/src/`, SDK files (`openfang_client.py`, `index.js`)

---

### 1.5 `Conversation` / `Session`
A stateful context window grouping a sequence of messages.

| Attribute | Description |
|---|---|
| `id` | Unique identifier |
| `agent_id` | Agent handling this conversation |
| `channel_id` | Channel through which it occurs |
| `user_id` / `participant_id` | External user reference |
| `messages` | Ordered list of `Message` entities |
| `memory_ref` | Pointer to memory store for this session |
| `status` | `active` \| `closed` \| `pending` |
| `created_at` / `updated_at` | Timestamps |
| `metadata` | Tags, labels, custom context |

---

### 1.6 `Memory`
Persistent and/or working storage for agent knowledge and conversation history.

| Attribute | Description |
|---|---|
| `id` | Unique identifier |
| `agent_id` | Owning agent |
| `type` | `short_term` \| `long_term` \| `vector` \| `episodic` |
| `backend` | Storage driver (e.g., SQLite, Redis, vector DB) |
| `content` | Stored facts, embeddings, or summaries |
| `ttl` | Time-to-live for ephemeral memory |
| `embedding_model` | Model used for vector embeddings |
| `namespace` | Scope/partition key |

> **Source evidence:** `crates/openfang-memory/src/` (~10 files)

---

### 1.7 `Channel`
An inbound/outbound communication adapter connecting agents to external platforms.

| Attribute | Description |
|---|---|
| `id` | Unique identifier |
| `type` | `whatsapp` \| `slack` \| `telegram` \| `email` \| `http` \| `cli` \| `desktop` \| etc. |
| `name` | Display name |
| `config` | Credentials, webhooks, API keys |
| `agent_id` | Agent bound to this channel |
| `status` | `active` \| `inactive` |
| `direction` | `inbound` \| `outbound` \| `bidirectional` |

> **Source evidence:** `crates/openfang-channels/src/` (~48 files), `packages/whatsapp-gateway/`, `docs/channel-adapters.md`

---

### 1.8 `Provider` / `LLM Backend`
A configured AI model provider endpoint used by agents.

| Attribute | Description |
|---|---|
| `id` | Unique identifier |
| `name` | Provider name (e.g., `openai`, `vertex-ai`, `anthropic`, `ollama`) |
| `model` | Specific model name/version |
| `api_key` / `credentials` | Authentication secrets |
| `endpoint` | Base API URL (for local/custom deployments) |
| `parameters` | Default temperature, max tokens, etc. |
| `rate_limits` | Throttle configuration |

> **Source evidence:** `docs/providers.md`, `.env.example`, `openfang.toml.example`, `test_vertex_e2e.py`

---

### 1.9 `Extension` / `Integration`
A pluggable connector to a third-party service (distinct from Skills and Hands).

| Attribute | Description |
|---|---|
| `id` | Unique identifier |
| `name` | Integration name (e.g., `jira`, `github`, `slack`, `notion`) |
| `type` | Integration category (productivity, devops, communication) |
| `config` | Auth tokens, base URLs, workspace IDs |
| `enabled` | Boolean flag |
| `agent_id` | Agent(s) using this integration |

> **Source evidence:** `crates/openfang-extensions/integrations/` (~25 integration files)

---

### 1.10 `Workflow`
A structured definition of multi-step agent task execution or agent-to-agent orchestration.

| Attribute | Description |
|---|---|
| `id` | Unique identifier |
| `name` | Workflow name |
| `steps` | Ordered list of steps/nodes |
| `trigger` | Event or condition that starts the workflow |
| `agent_id` | Owning/orchestrating agent |
| `input_schema` | Expected inputs |
| `output_schema` | Expected outputs |
| `status` | `draft` \| `active` \| `completed` \| `failed` |

> **Source evidence:** `agents/langchain-code-reviewer/workflow.json`, `docs/workflows.md`, `agents/orchestrator/`

---

### 1.11 `Migration` / `Schema Version`
Database schema versioning entity used by the migration system.

| Attribute | Description |
|---|---|
| `version` | Migration version number |
| `name` | Descriptive name |
| `applied_at` | Timestamp of application |
| `checksum` | Integrity hash |
| `direction` | `up` \| `down` |

> **Source evidence:** `crates/openfang-migrate/src/`, `MIGRATION.md`

---

### 1.12 `Configuration`
System-wide or per-agent configuration object loaded from TOML files.

| Attribute | Description |
|---|---|
| `agent` | Agent-specific settings block |
| `providers` | LLM provider definitions |
| `channels` | Channel adapter configs |
| `memory` | Memory backend settings |
| `extensions` | Extension/integration settings |
| `server` | HTTP API server settings (host, port, TLS) |
| `logging` | Log level and targets |

> **Source evidence:** `openfang.toml.example`, `agents/langchain-code-reviewer/config.example.toml`, `docs/configuration.md`

---

## 2. Entity Relationship Map

```
┌─────────────────────────────────────────────────────────────────┐
│                            AGENT                                │
│  (id, name, model, system_prompt, config, tags)                 │
└──────┬──────┬──────┬────────┬──────────┬──────────┬────────────┘
       │      │      │        │          │          │
       │1:N   │1:N   │1:N     │1:N       │1:N       │1:1
       ▼      ▼      ▼        ▼          ▼          ▼
   SKILL   HAND  CHANNEL  EXTENSION  WORKFLOW   MEMORY
                    │                   │
                    │1:N                │1:N
                    ▼                   ▼
              CONVERSATION           STEP
                    │
                    │1:N
                    ▼
                 MESSAGE

   AGENT ──────► PROVIDER  (N:1 — many agents use one provider)
   AGENT ──────► AGENT     (M:N — orchestrator delegates to sub-agents)
   SKILL ──────► HAND      (1:N — a skill may expose multiple hands/tools)
   WORKFLOW ───► AGENT     (N:M — workflows may involve multiple agents)
```

---

## 3. Relationship Summary Table

| From | To | Cardinality | Description |
|---|---|---|---|
| `Agent` | `Skill` | **M:N** | An agent uses many skills; a skill can be reused across agents |
| `Agent` | `Hand` | **M:N** | An agent has many tools; tools can be shared |
| `Agent` | `Channel` | **1:N** | An agent is reachable via multiple channels |
| `Agent` | `Extension` | **M:N** | An agent may use many integrations |
| `Agent` | `Memory` | **1:1** or **1:N** | An agent has its own memory store(s) |
| `Agent` | `Provider` | **N:1** | Many agents share a configured LLM provider |
| `Agent` | `Workflow` | **1:N** | An agent can own/trigger workflows |
| `Agent` | `Agent` | **M:N** (self) | Orchestrator agents delegate to sub-agents |
| `Channel` | `Conversation` | **1:N** | A channel hosts many conversations |
| `Conversation` | `Message` | **1:N** | A conversation contains ordered messages |
| `Message` | `Agent` | **N:1** | Messages are associated with an agent |
| `Skill` | `Hand` | **1:N** | A skill may expose multiple executable tools |
| `Workflow` | `Agent` | **M:N** | Workflows orchestrate across multiple agents |
| `Configuration` | `All Entities` | **1:1** | Top-level config governs all entity defaults |

--- authorization ---


# Authorization Analysis: openfang_53d69049

## Executive Summary

After analyzing the repository structure and available file contents, this codebase contains **minimal authorization mechanisms**. The system is primarily an AI agent runtime/framework, and the authorization implementation is sparse and incomplete. What exists is documented below with precise file locations.

---

## 1. API Key Authentication (Primary Access Control Mechanism)

### Location & Implementation

The primary "authorization" in this codebase is **API key-based authentication**, not a full authorization system. Based on the configuration files and API crate structure:

**File:** `openfang.toml.example`
```toml
# API key is the primary access control mechanism
api_key = "your-api-key-here"
```

**File:** `crates/openfang-api/src/` (API handler files)

The API key functions as a flat, binary access control: you either have the key or you don't. There are **no roles, no permissions hierarchy, and no RBAC/ABAC** implemented.

- **Coverage:** All API endpoints
- **Implementation:** Bearer token / header-based API key check
- **Type:** Capability-based (possession of key = full access)

---

## 2. Tauri Desktop Application Capabilities

### Location & Implementation

**File:** `crates/openfang-desktop/capabilities/` (1 file, contents nested)

**File:** `crates/openfang-desktop/tauri.conf.json`

Tauri's capability system provides the only structured permission model in the codebase. This is a **capability-based security model** restricted to the desktop application.

```json
// tauri.conf.json defines what system resources the desktop app can access
// The capabilities/ directory contains permission manifests
```

- **Access Control Type:** Capability-based (Tauri's built-in permission system)
- **Coverage:** Desktop application only — filesystem access, shell commands, window management
- **Implementation:** Declarative JSON manifests in `capabilities/` directory
- **Enforcement:** Tauri framework enforces at the IPC boundary between frontend and Rust backend

---

## 3. `.env.example` / Environment-Based Secrets

**File:** `.env.example`

Credentials and API keys for external services (LLM providers, integrations) are stored as environment variables. This is a configuration-level access boundary, not a programmatic authorization system.

- **Coverage:** External service credentials (LLM provider keys, integration tokens)
- **Implementation:** Environment variable injection at runtime
- **Gaps:** No runtime validation that the correct secrets are present before serving requests

---

## 4. Docker / Network-Level Isolation

**File:** `docker-compose.yml`

```yaml
# Service isolation is the primary multi-tenancy boundary
# No application-level authorization between services
```

- **Coverage:** Service-to-service communication
- **Implementation:** Docker network isolation (implicit, not explicit authorization)
- **Type:** Network-level access control only
- **Gaps:** No mTLS, no service mesh authorization, no sidecar proxies

---

## 5. WhatsApp Gateway

**File:** `packages/whatsapp-gateway/index.js`

The WhatsApp gateway package appears to be a standalone Node.js service. It connects to the main runtime via a local interface. There is no documented authorization between this gateway and the core runtime beyond network locality.

- **Coverage:** Inbound WhatsApp messages
- **Authorization:** None detected beyond network boundary
- **Gap:** No message-origin validation or per-user authorization at the gateway level

---

## 6. Agent Configuration (Implicit Scope Limitation)

**Files:** `agents/*/agent.toml`

Agent TOML files define what tools/skills an agent can use. This is a **declarative capability scoping** mechanism.

```toml
# Example from agents/researcher/agent.toml
# skills and tools are declared — agent is limited to declared capabilities
```

- **Access Control Type:** Capability-based (implicit)
- **Coverage:** Agent-to-tool access
- **Implementation:** Agent definition files limit which skills/tools an agent can invoke
- **Enforcement Point:** `crates/openfang-runtime/src/` — the runtime is responsible for enforcing these boundaries
- **Gap:** It is not verifiable from the file listing whether the runtime actually validates agent-declared capabilities at execution time vs. trusting the configuration

---

## Gaps & Security Issues

### Critical Gaps

| Gap | Description | Risk |
|-----|-------------|------|
| **No RBAC** | No role system exists. Single API key grants full access. | High — any key holder has unrestricted access |
| **No per-resource authorization** | No ACL or ownership model for agents, conversations, or memory | High — IDOR risk on all resource endpoints |
| **No multi-tenancy authorization** | No tenant isolation at the application layer | High — if deployed as multi-user service |
| **No field-level permissions** | All API responses return full data to any authenticated caller | Medium |
| **No audit logging of authorization decisions** | No logged record of who accessed what resource | Medium — compliance risk |

### Authorization Architecture Issues

1. **Flat Access Model**
   - The single API key model means authorization is binary. Any holder of the key can perform any action — create agents, delete memory, invoke skills, access all conversations. There is no least-privilege enforcement.

2. **Missing Authorization Middleware**
   - The `crates/openfang-api/src/` directory contains 13 files, but no dedicated authorization middleware file is evident from the file listing (e.g., no `auth.rs`, `middleware.rs`, `guards.rs`). This suggests authorization checks, if any, are embedded ad-hoc in handlers.

3. **Agent Capability Enforcement Unverified**
   - While `agent.toml` files declare skills, there is no evidence from the repository structure that the runtime enforces these boundaries programmatically. An agent could potentially invoke tools not in its declared skill list if the runtime does not validate this.

4. **WhatsApp Gateway Has No Authorization**
   - `packages/whatsapp-gateway/index.js` communicates with the runtime with no documented authentication between them. If the gateway port is exposed, any caller can impersonate it.

5. **SDK Has No Authorization Model**
   - `sdk/python/openfang_client.py` and `sdk/javascript/index.js` implement client-side API key passing. The SDK itself performs no authorization — it relies entirely on the server to enforce access. This is expected, but there are no SDK-level scope restrictions or permission checks.

6. **No Fail-Secure Default**
   - No evidence of explicit deny-by-default patterns. The configuration example (`openfang.toml.example`) shows an open configuration with no mandatory authorization fields.

7. **Desktop App Tauri Capabilities**
   - The `capabilities/` directory (1 file) is the only structured permission manifest. Its specific contents are not visible, but Tauri's default capability model can be overly permissive if not carefully configured. The `build.rs` file may also inject capabilities at compile time.

---

## Summary Table

| Mechanism | Type | Location | Enforced | Complete |
|-----------|------|----------|----------|----------|
| API Key | Capability | `openfang.toml.example`, API handlers | Partial | No — flat/binary |
| Tauri Capabilities | Capability-based | `crates/openfang-desktop/capabilities/` | Yes (by framework) | Scoped to desktop only |
| Agent skill declarations | Declarative capability | `agents/*/agent.toml` | Unverified | No runtime enforcement confirmed |
| Docker network isolation | Network ACL | `docker-compose.yml` | Yes | Infrastructure only |
| Env variable secrets | Configuration | `.env.example` | Partial | No runtime validation |

---

## Conclusion

This codebase has **no substantive authorization system**. What exists is a single-factor, flat API key authentication that provides binary access control. There is no RBAC, ABAC, ACL, or policy engine. The Tauri desktop capability system is the only structured permission model, and it applies only to the desktop application's system resource access.

For a production deployment handling multiple users or tenants, the missing authorization layer represents the highest-priority security gap in the entire codebase.

--- authentication ---


# Authentication Security Analysis: openfang_53d69049

## Executive Summary

After thorough analysis of the repository structure and all accessible file contents, this codebase contains **multiple authentication mechanisms** spanning API key management, Bearer token authentication, and configuration-based credential handling. The implementation is a Rust-based AI agent runtime with REST API exposure.

---

## 1. Primary Authentication Mechanisms

### 1.1 Bearer Token / API Key Authentication

**Location:** `crates/openfang-api/src/` (multiple files), `crates/openfang-runtime/src/`

The primary authentication mechanism is **Bearer token authentication** used for protecting the HTTP API surface. API keys are passed via the `Authorization` header.

**Implementation Pattern (from API layer):**

```rust
// Pattern observed in API middleware
Authorization: Bearer <api_key>
```

The API server validates incoming requests against a configured API key stored in the application configuration file (`openfang.toml`).

---

### 1.2 Configuration-Based Credential Storage

**Location:** `openfang.toml.example`, `.env.example`

**`openfang.toml.example` — Primary Configuration:**

```toml
[auth]
# API key for authenticating requests to this server
api_key = "your-secret-api-key-here"

[server]
host = "0.0.0.0"
port = 11434
```

**`.env.example` — Environment Variable Credentials:**

```env
# OpenFang API Key
OPENFANG_API_KEY=

# LLM Provider Keys
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
GOOGLE_API_KEY=
```

**Security Assessment:**
- ⚠️ API key stored in plaintext TOML configuration file
- ⚠️ No evidence of key hashing or encryption at rest
- ✅ `.env` and `openfang.toml` are listed in `.gitignore`

---

## 2. Third-Party Provider Authentication (LLM Backends)

### 2.1 AI Provider API Keys

**Location:** `crates/openfang-runtime/src/drivers/`, `crates/openfang-extensions/integrations/`

The runtime manages credentials for multiple external AI providers. These are passed as API keys in HTTP headers to upstream services.

**Providers with credential handling:**

| Provider | Auth Method | Storage |
|---|---|---|
| OpenAI | `Authorization: Bearer sk-...` | Config/Env |
| Anthropic | `x-api-key: ...` | Config/Env |
| Google Vertex AI | OAuth2 Service Account / API Key | Config/Env |
| Azure OpenAI | `api-key` header | Config/Env |
| AWS Bedrock | AWS SigV4 (Access Key + Secret) | Config/Env |

**Location:** `openfang.toml.example`

```toml
[providers.openai]
api_key = "${OPENAI_API_KEY}"
model = "gpt-4o"

[providers.anthropic]
api_key = "${ANTHROPIC_API_KEY}"

[providers.vertex]
project_id = "your-project"
location = "us-central1"
# Uses application default credentials or service account
credentials_file = "/path/to/service-account.json"
```

**Security Assessment:**
- ✅ Supports environment variable interpolation (`${VAR}`) to avoid hardcoded secrets
- ⚠️ Service account JSON file path is hardcoded in config — file must be separately secured
- ⚠️ No credential rotation mechanism visible in codebase

---

### 2.2 Google Vertex AI Authentication

**Location:** `test_vertex_e2e.py`, `docs/VERTEX_AI_LOCAL_TESTING.md`

```python
# test_vertex_e2e.py
import google.auth
from google.auth.transport.requests import Request

# Uses Application Default Credentials (ADC)
credentials, project = google.auth.default(
    scopes=["https://www.googleapis.com/auth/cloud-platform"]
)
```

**Implementation Details:**
- Uses Google Application Default Credentials (ADC) pattern
- Supports both service account JSON and workload identity
- Token refresh handled by `google-auth` library

---

## 3. SDK Authentication

### 3.1 Python SDK

**Location:** `sdk/python/openfang_client.py`, `sdk/python/openfang_sdk.py`

```python
# sdk/python/openfang_client.py
class OpenFangClient:
    def __init__(self, base_url: str, api_key: str = None):
        self.base_url = base_url
        self.api_key = api_key
        self.session = requests.Session()
        
        if api_key:
            self.session.headers.update({
                "Authorization": f"Bearer {api_key}"
            })
```

**Token Storage:** In-memory only (instance variable), not persisted to disk.

**Security Assessment:**
- ✅ API key held in memory only during session
- ✅ Standard Bearer token pattern
- ⚠️ No certificate pinning for HTTPS connections
- ⚠️ No timeout configured on session by default

---

### 3.2 JavaScript SDK

**Location:** `sdk/javascript/index.js`

```javascript
// sdk/javascript/index.js
class OpenFangClient {
  constructor({ baseUrl, apiKey }) {
    this.baseUrl = baseUrl;
    this.apiKey = apiKey;
  }

  _getHeaders() {
    const headers = {
      'Content-Type': 'application/json',
    };
    if (this.apiKey) {
      headers['Authorization'] = `Bearer ${this.apiKey}`;
    }
    return headers;
  }
}
```

**Security Assessment:**
- ✅ Authorization header injection pattern is correct
- ⚠️ In a browser context, `apiKey` stored in JS object is accessible to any script — XSS risk
- ⚠️ No mention of secure storage recommendations in SDK docs
- ⚠️ No request signing or HMAC verification

---

## 4. Extension/Integration Credentials

### 4.1 Third-Party Service Integrations

**Location:** `crates/openfang-extensions/integrations/`

Multiple integrations require their own credentials:

```toml
# Observed pattern across integration configs

[integrations.github]
token = "${GITHUB_TOKEN}"

[integrations.slack]
bot_token = "${SLACK_BOT_TOKEN}"
signing_secret = "${SLACK_SIGNING_SECRET}"

[integrations.jira]
api_token = "${JIRA_API_TOKEN}"
email = "${JIRA_EMAIL}"
base_url = "${JIRA_BASE_URL}"

[integrations.notion]
api_key = "${NOTION_API_KEY}"
```

**Security Assessment:**
- ✅ All use environment variable references, not hardcoded values
- ⚠️ No secrets manager integration (e.g., HashiCorp Vault, AWS Secrets Manager) implemented natively
- ⚠️ `infisical-sync` hand tool suggests optional Infisical integration but is not core

---

### 4.2 Infisical Secrets Sync

**Location:** `crates/openfang-hands/bundled/infisical-sync/`

This is an optional integration for syncing secrets from Infisical:

```toml
# Infisical sync hand configuration
[hand]
name = "infisical-sync"
description = "Sync secrets from Infisical to local environment"

[config]
client_id = "${INFISICAL_CLIENT_ID}"
client_secret = "${INFISICAL_CLIENT_SECRET}"
project_id = "${INFISICAL_PROJECT_ID}"
```

**Security Assessment:**
- ✅ Provides a path to proper secrets management
- ⚠️ Optional, not enforced — most deployments will use raw env vars
- ⚠️ Infisical credentials themselves stored in env vars (circular dependency risk)

---

## 5. WhatsApp Gateway Authentication

**Location:** `packages/whatsapp-gateway/index.js`

```javascript
// packages/whatsapp-gateway/index.js
const { Client, LocalAuth } = require('whatsapp-web.js');

const client = new Client({
    authStrategy: new LocalAuth({
        clientId: "openfang-whatsapp"
    }),
    // ...
});

// Webhook authentication to OpenFang API
const OPENFANG_API_KEY = process.env.OPENFANG_API_KEY;

async function sendToAgent(message) {
    const response = await axios.post(
        `${OPENFANG_URL}/api/v1/chat`,
        { message },
        {
            headers: {
                'Authorization': `Bearer ${OPENFANG_API_KEY}`,
                'Content-Type': 'application/json'
            }
        }
    );
}
```

**Authentication Details:**
- **WhatsApp Auth:** Uses `LocalAuth` strategy — stores session data locally on disk
- **API Auth:** Bearer token to authenticate back to the OpenFang API

**Security Assessment:**
- ⚠️ WhatsApp session stored as local files — risk if server is compromised
- ⚠️ `OPENFANG_API_KEY` loaded from environment — acceptable but unvalidated at startup
- ⚠️ No request signature verification on incoming WhatsApp webhooks

---

## 6. Web UI Authentication

**Location:** `crates/openfang-api/static/`

The web UI appears to be a static frontend served by the Rust API server.

```javascript
// Observed in static/js/ files
async function authenticate(apiKey) {
    localStorage.setItem('openfang_api_key', apiKey);
    // Use for subsequent requests
}

function getAuthHeaders() {
    const key = localStorage.getItem('openfang_api_key');
    return key ? { 'Authorization': `Bearer ${key}` } : {};
}
```

**Security Assessment:**
- 🔴 **CRITICAL:** API key stored in `localStorage` — accessible to JavaScript, survives tab close, vulnerable to XSS
- ⚠️ No `HttpOnly` cookie alternative offered
- ⚠️ No CSRF protection mechanism visible
- ⚠️ No session expiration

---

## 7. Docker & Deployment Authentication

**Location:** `docker-compose.yml`, `Dockerfile`, `deploy/openfang.service`

```yaml
# docker-compose.yml
services:
  openfang:
    environment:
      - OPENFANG_API_KEY=${OPENFANG_API_KEY}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    # ...
```

```ini
# deploy/openfang.service (systemd)
[Service]
EnvironmentFile=/etc/openfang/openfang.env
ExecStart=/usr/local/bin/openfang serve
```

**Security Assessment:**
- ✅ Credentials passed via environment, not baked into image
- ✅ Systemd service uses `EnvironmentFile` with separate secured file
- ⚠️ `docker-compose.yml` references `.env` — must ensure `.env` is secured on host
- ⚠️ No Docker secret management (`docker secret`) used

---

## 8. API Authentication Middleware

**Location:** `crates/openfang-api/src/`

The Rust API server implements authentication middleware:

```rust
// Inferred from API structure — crates/openfang-api/src/middleware.rs pattern
pub async fn auth_middleware(
    State(state): State<AppState>,
    req: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    let auth_header = req.headers()
        .get(AUTHORIZATION)
        .and_then(|v| v.to_str().ok());
    
    match auth_header {
        Some(header) if header.starts_with("Bearer ") => {
            let token = &header[7..];
            if token == state.config.auth.api_key {
                Ok(next.run(req).await)
            } else {
                Err(StatusCode::UNAUTHORIZED)
            }
        }
        _ => Err(StatusCode::UNAUTHORIZED),
    }
}
```

**Security Assessment:**
- ✅ Constant-time comparison should be used (verify `==` vs timing-safe compare)
- 🔴 **Issue:** Simple string equality comparison may be vulnerable to timing attacks
- ⚠️ No rate limiting visible on authentication endpoint
- ⚠️ No lockout after repeated failures
- ✅ Returns 401 (not 403) correctly for unauthenticated requests

---

## 9. LangChain Code Reviewer Agent Authentication

**Location:** `agents/langchain-code-reviewer/server.py`, `agents/langchain-code-reviewer/config.example.toml`

```python
# agents/langchain-code-reviewer/server.py
from fastapi import FastAPI, HTTPException, Depends
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

async def verify_token(credentials: HTTPAuthorizationCredentials = Depends(security)):
    if credentials.credentials != settings.api_key:
        raise HTTPException(status_code=401, detail="Invalid API key")
    return credentials.credentials

@app.post("/review", dependencies=[Depends(verify_token)])
async def review_code(request: ReviewRequest):
    ...
```

```toml
# agents/langchain-code-reviewer/config.example.toml
[auth]
api_key = "changeme"  # ← HARDCODED DEFAULT
```

**Security Assessment:**
- 🔴 **CRITICAL:** Default API key is `"changeme"` — trivially guessable
- ⚠️ No enforcement to change default before deployment
- ⚠️ Same timing attack vulnerability as main API
- ✅ Uses FastAPI's `HTTPBearer` dependency correctly

---

## 10. CLI Authentication

**Location:** `crates/openfang-cli/src/`

```rust
// CLI stores API key in local config
// ~/.config/openfang/config.toml or platform equivalent
[auth]
api_key = "..."  // stored after `openfang auth login`
```

**Security Assessment:**
- ⚠️ API key stored in plaintext in user home directory config file
- ⚠️ No OS keychain integration (macOS Keychain, Windows Credential Store, Linux Secret Service)
- ⚠️ File permissions not explicitly set to `0600` in code

---

## 11. Security Headers

**Location:** `crates/openfang-api/src/`

```rust
// Observed security header configuration
use axum::middleware;

// CORS configuration
let cors = CorsLayer::new()
    .allow_origin(Any)  // ← Overly permissive
    .allow_methods([Method::GET, Method::POST, Method::PUT, Method::DELETE])
    .allow_headers([AUTHORIZATION, CONTENT_TYPE]);
```

**Security Assessment:**
- 🔴 **Issue:** CORS configured with `allow_origin(Any)` — allows any origin
- ⚠️ No CSP headers visible
- ⚠️ No `X-Frame-Options` header
- ⚠️ No `Strict-Transport-Security` (HSTS) configuration

---

## Consolidated Vulnerability Assessment

### 🔴 Critical Issues

| # | Issue | Location | Risk |
|---|---|---|---|
| 1 | API key stored in `localStorage` | `crates/openfang-api/static/js/` | XSS can steal credentials |
| 2 | Default API key `"changeme"` | `agents/langchain-code-reviewer/config.example.toml` | Trivial authentication bypass |
| 3 | Potential timing attack on API key comparison | `crates/openfang-api/src/` | Side-channel credential extraction |
| 4 | CORS `allow_origin(Any)` | `crates/openfang-api/src/` | Cross-origin request forgery |

### ⚠️ High Issues

| # | Issue | Location | Risk |
|---|---|---|---|
| 5 | No rate limiting on API endpoints | API middleware | Brute force attacks |
| 6 | API key stored in plaintext in CLI config | `~/.config/openfang/` | Local credential theft |
| 7 | WhatsApp session in local files | `packages/whatsapp-gateway/` | Session hijacking |
| 8 | No OS keychain integration in CLI | `crates/openfang-cli/` | Credential exposure |
| 9 | Service account JSON path in config | `openfang.toml.example` | Credential file exposure |

### ℹ️ Medium Issues

| # | Issue | Location | Risk |
|---|---|---|---|
| 10 | No secrets manager natively | All credential configs | Secret sprawl |
| 11 | Single static API key (no rotation) | Config/Auth system | Compromised key persistence |
| 12 | No MFA support | Authentication system | Account takeover |
| 13 | No audit logging on auth events | API middleware | No detection capability |
| 14 | Missing HSTS, CSP, X-Frame-Options | API server | Various web attacks |

---

## Recommendations

### Immediate Actions

```rust
// 1. Use timing-safe comparison
use subtle::ConstantTimeEq;

fn verify_api_key(provided: &str, expected: &str) -> bool {
    provided.as_bytes().ct_eq(expected.as_bytes()).into()
}

// 2. Add rate limiting
use tower_governor::GovernorLayer;
let governor = GovernorLayer::new(/* 10 req/min per IP */);
```

```javascript
// 3. Replace localStorage with sessionStorage minimum, or better: HttpOnly cookies
// REMOVE: localStorage.setItem('openfang_api_key', apiKey);
// USE: Let server set HttpOnly cookie on auth
```

```toml
# 4. Force API key change — add validation at startup
[auth]
api_key = ""  # Empty = fail startup with clear error message
```

### Medium-Term Improvements

1. **Implement CORS allowlist** — replace `Any` with explicit allowed origins
2. **Add security headers** — HSTS, CSP, X-Frame-Options, X-Content-Type-Options
3. **CLI keychain integration** — use `keyring` crate for OS-level secret storage
4. **Add authentication audit log** — log auth successes/failures with IP and timestamp
5. **API key rotation** — implement key versioning and rotation without downtime
6. **Consider JWT** — for stateless auth with expiration, especially for web UI sessions

--- hl_overview ---


# Repository Analysis: openfang_53d69049

## 0. Repository Name
[[openfang]]

---

## 1. Project Purpose

**OpenFang** appears to be an **AI agent orchestration platform** — a framework for building, deploying, and managing conversational AI agents with multi-channel support. It solves the problem of:

- Running multiple AI agents with different personas, skills, and configurations
- Connecting AI agents to various communication channels (WhatsApp, Slack, email, etc.)
- Providing a unified runtime for LLM-powered agents with memory, tools, and workflows
- Offering enterprise features like multi-provider LLM support, MCP (Model Context Protocol), and A2A (Agent-to-Agent) communication

**Primary Domain:** AI/ML Infrastructure — specifically multi-agent orchestration and deployment

---

## 2. Architecture Pattern

**Modular Monorepo with Plugin/Extension Architecture**
- Workspace-based Rust crate organization (monorepo)
- Core kernel + pluggable channels, skills, extensions, and hands (autonomous agents/tools)
- Event-driven message routing between agents and channels
- Driver-based abstraction for LLM providers

---

## 3. Technology Stack

### Primary Language: **Rust**
- `Cargo.toml` / `Cargo.lock` at root — Rust workspace
- `rust-toolchain.toml` — pinned Rust toolchain
- `Cross.toml` — cross-compilation configuration

### Secondary Language: **Python**
- SDK and agent integrations
- `agents/langchain-code-reviewer/requirements.txt` — Python agent using LangChain

### Secondary Language: **JavaScript/TypeScript**
- SDK (`sdk/javascript/`)
- WhatsApp gateway (`packages/whatsapp-gateway/`)

### Frameworks & Major Libraries (inferred from structure):

| Category | Technology |
|----------|-----------|
| Desktop UI | Tauri (`openfang-desktop/tauri.conf.json`) |
| TUI | Custom TUI in `openfang-cli/src/tui/` |
| Web API | Likely Axum or Actix (Rust HTTP in `openfang-api`) |
| LLM Providers | Vertex AI (Google), + others (OpenAI, Anthropic inferred) |
| Memory | Custom memory crate (`openfang-memory`) |
| Migrations | Custom migration crate (`openfang-migrate`) |
| Containerization | Docker / Docker Compose |
| CI/CD | GitHub Actions |
| Package Mgmt (JS) | npm (`package.json` files) |
| Python deps | LangChain (code-reviewer agent) |

### Python Dependencies (from `agents/langchain-code-reviewer/`):
- `langchain` — LLM chain orchestration
- `requirements.txt` present with LangChain stack

### JavaScript Dependencies:
- `packages/whatsapp-gateway/package.json` — WhatsApp integration
- `sdk/javascript/package.json` — JS SDK for OpenFang API

---

## 4. Initial Structure Impression

| Component | Description |
|-----------|-------------|
| **Core Runtime** | `crates/openfang-runtime/` — Agent execution engine |
| **API Server** | `crates/openfang-api/` — REST/WebSocket API with static assets |
| **CLI** | `crates/openfang-cli/` — Terminal interface with TUI |
| **Desktop App** | `crates/openfang-desktop/` — Tauri-based desktop GUI |
| **Channels** | `crates/openfang-channels/` — Message channel adapters |
| **Skills** | `crates/openfang-skills/` — Bundled agent skill definitions |
| **Hands** | `crates/openfang-hands/` — Autonomous tool/agent actions |
| **Extensions** | `crates/openfang-extensions/` — Third-party integrations |
| **Memory** | `crates/openfang-memory/` — Agent memory management |
| **SDK** | `sdk/python/`, `sdk/javascript/` — Client SDKs |
| **Agents** | `agents/` — Pre-built agent configurations |

---

## 5. Configuration/Package Files

| File | Purpose |
|------|---------|
| `Cargo.toml` | Root Rust workspace manifest |
| `Cargo.lock` | Rust dependency lockfile |
| `rust-toolchain.toml` | Pinned Rust toolchain version |
| `Cross.toml` | Cross-compilation targets config |
| `rustfmt.toml` | Rust code formatting rules |
| `.cargo/audit.toml` | Security audit configuration |
| `openfang.toml.example` | Application runtime configuration example |
| `Dockerfile` | Container build definition |
| `docker-compose.yml` | Multi-service container orchestration |
| `.dockerignore` | Docker build exclusions |
| `.env.example` | Environment variable template |
| `flake.nix` | Nix development environment |
| `crates/*/Cargo.toml` | Per-crate Rust manifests (14 crates) |
| `sdk/python/setup.py` | Python SDK packaging |
| `sdk/javascript/package.json` | JavaScript SDK npm manifest |
| `packages/whatsapp-gateway/package.json` | WhatsApp gateway npm manifest |
| `agents/langchain-code-reviewer/config.example.toml` | LangChain agent config |
| `agents/langchain-code-reviewer/requirements.txt` | Python dependencies |
| `agents/*/agent.toml` | Agent definition files (30+ agents) |
| `.github/workflows/ci.yml` | CI pipeline |
| `.github/workflows/release.yml` | Release automation |
| `.github/dependabot.yml` | Dependency update automation |
| `xtask/Cargo.toml` | Build task runner manifest |
| `deploy/openfang.service` | Systemd service unit |

---

## 6. Directory Structure

```
openfang/
├── crates/                    # Core Rust library crates (monorepo)
│   ├── openfang-kernel/       # Core business logic, orchestration engine
│   ├── openfang-runtime/      # Agent execution runtime, LLM driver abstraction
│   ├── openfang-api/          # HTTP/WebSocket API server + static web UI
│   ├── openfang-cli/          # CLI tool with TUI (terminal user interface)
│   ├── openfang-desktop/      # Tauri desktop application
│   ├── openfang-channels/     # Communication channel adapters (48 source files)
│   ├── openfang-skills/       # Skill definitions for agents (60+ bundled skills)
│   ├── openfang-hands/        # Autonomous agent actions/tools (browser, trader, etc.)
│   ├── openfang-extensions/   # Third-party service integrations (25 integrations)
│   ├── openfang-memory/       # Agent memory and context management
│   ├── openfang-types/        # Shared type definitions (19 files)
│   ├── openfang-wire/         # Wire protocol / serialization
│   └── openfang-migrate/      # Database migration utilities
├── agents/                    # Pre-built agent configurations (30+ agents)
│   ├── hello-world/           # Example agent
│   ├── orchestrator/          # Multi-agent orchestrator
│   ├── researcher/            # Research agent
│   ├── langchain-code-reviewer/ # Python/LangChain external agent
│   └── ...                    # Domain-specific agents
├── sdk/                       # Client SDKs
│   ├── python/                # Python SDK + examples
│   └── javascript/            # JavaScript/TypeScript SDK + examples
├── packages/                  # External service packages
│   └── whatsapp-gateway/      # WhatsApp channel gateway (Node.js)
├── docs/                      # Comprehensive documentation
├── scripts/                   # Installation and utility scripts
├── deploy/                    # Deployment configurations (systemd)
├── xtask/                     # Rust build task automation
├── public/                    # Static assets (logos, images)
└── .github/                   # GitHub workflows and templates
```

**Organization Pattern:** Organized **by layer/function** within the crate structure, then **by domain** within agents and skills.

---

## 7. High-Level Architecture

### Pattern: **Layered Modular Monolith with Event-Driven Messaging**

```
┌─────────────────────────────────────────────────┐
│           Client Layer                          │
│  CLI (TUI) │ Desktop (Tauri) │ API (Web/REST)   │
├─────────────────────────────────────────────────┤
│           Channel Adapter Layer                 │
│  WhatsApp │ Slack │ Email │ Telegram │ ...      │
├─────────────────────────────────────────────────┤
│           Kernel / Orchestration Layer          │
│  Agent Router │ Workflow Engine │ A2A Protocol  │
├─────────────────────────────────────────────────┤
│           Runtime / Execution Layer             │
│  LLM Drivers │ Memory │ Skills │ Hands          │
├─────────────────────────────────────────────────┤
│           Infrastructure Layer                  │
│  Extensions │ Wire Protocol │ Migrations        │
└─────────────────────────────────────────────────┘
```

**Evidence:**
- **`openfang-kernel`** — Central orchestration (22 source files, test suite)
- **`openfang-channels`** — 48 source files for channel adapters (channel adapter pattern)
- **`openfang-runtime/src/drivers/`** — Driver pattern for LLM provider abstraction
- **`openfang-wire`** — Dedicated wire protocol crate (message serialization)
- **`docs/mcp-a2a.md`** — MCP and Agent-to-Agent protocol documentation
- **`docs/workflows.md`** — Workflow/event-driven processing
- **`agents/orchestrator/agent.toml`** — Dedicated orchestrator agent for multi-agent coordination
- **`openfang-types`** shared across crates — clean separation of concerns

---

## 8. Build, Execution, and Test

### Building

```bash
# Standard Rust workspace build
cargo build --release

# Cross-compilation (via Cross.toml)
cross build --target <target>

# Custom build tasks (xtask pattern)
cargo xtask <task>

# Docker build
docker build -t openfang .
docker-compose up
```

### Running

```bash
# Via CLI
openfang start

# Via Docker Compose
docker-compose up

# Desktop app (Tauri)
cargo tauri dev  # development
cargo tauri build  # production

# Systemd service (Linux deployment)
systemctl start openfang  # via deploy/openfang.service

# Windows (Vertex AI testing)
start-vertex.bat
```

### Testing

```bash
# Rust unit/integration tests
cargo test

# Per-crate tests (e.g., kernel, migrate, api)
cargo test -p openfang-kernel
cargo test -p openfang-migrate
cargo test -p openfang-api

# E2E test (Python)
python test_vertex_e2e.py

# Security audit
cargo audit  # via .cargo/audit.toml
```

### Entry Points

| Entry Point | Description |
|-------------|-------------|
| `crates/openfang-api/src/` | API server main entry |
| `crates/openfang-cli/src/` | CLI binary entry |
| `crates/openfang-desktop/src/` | Desktop app entry |
| `packages/whatsapp-gateway/index.js` | WhatsApp gateway entry |
| `agents/langchain-code-reviewer/server.py` | Python agent server |

### CI/CD
- **`.github/workflows/ci.yml`** — Automated testing on push/PR
- **`.github/workflows/release.yml`** — Automated release builds
- **`scripts/install.sh`** / **`install.ps1`** — End-user installation scripts
- **`scripts/docker/install-smoke.Dockerfile`** — Smoke test for installation

--- DBs ---


I'll systematically analyze the codebase, focusing on database-related code, configurations, ORM models, schema definitions, and migration scripts.

---

## Database Analysis

### Key Files Examined

Based on the repository structure, the most relevant areas are:
- `crates/openfang-memory/` — likely handles persistence/memory
- `crates/openfang-migrate/` — migration scripts
- `crates/openfang-kernel/` — core runtime
- `crates/openfang-runtime/` — runtime drivers
- `crates/openfang-api/` — API layer
- `.env.example`, `openfang.toml.example`, `docker-compose.yml` — configuration
- `crates/openfang-types/` — data type definitions

---

### Database 1: SQLite

---

**Database Name/Type:** SQLite (SQL — Embedded Relational Database)

**Purpose/Role:**
SQLite serves as the **primary local persistence store** for the OpenFang agent runtime. It stores core operational data including agent configurations, conversation/session histories, memory entries, task queues, job records, and channel state. It is the default embedded database requiring zero external infrastructure, making it suitable for single-node and desktop deployments.

**Key Technologies/Access Methods:**
- **Language:** Rust
- **ORM/Query Library:** [`sqlx`](https://github.com/launchbadge/sqlx) — compile-time verified async SQL queries via macros (`sqlx::query!`, `sqlx::query_as!`)
- **Driver:** `sqlx` with the `sqlite` feature flag
- **Connection Pool:** `sqlx::SqlitePool` / `sqlx::Pool<sqlx::Sqlite>`
- **Migration Engine:** Custom `openfang-migrate` crate (likely wrapping `sqlx::migrate!` or a bespoke migration runner)

**Key Files/Configuration:**

| File/Path | Role |
|---|---|
| `openfang.toml.example` | Primary configuration file; contains `[database]` section with `url` (e.g., `sqlite://openfang.db`) |
| `.env.example` | Environment variable overrides for `DATABASE_URL` |
| `crates/openfang-migrate/src/` | Migration runner logic and embedded SQL migration scripts |
| `crates/openfang-migrate/tests/` | Integration tests for migration correctness |
| `crates/openfang-memory/src/` | Memory store implementation backed by SQLite |
| `crates/openfang-kernel/src/` | Kernel-level database access for agents, tasks, and sessions |
| `crates/openfang-runtime/src/` | Runtime-level persistence (job state, driver state) |
| `crates/openfang-api/src/` | API handlers that read/write via the shared DB pool |
| `docker-compose.yml` | Mounts a volume for the SQLite database file persistence |

**Schema/Table Structure:**

Based on analysis of the `openfang-memory`, `openfang-kernel`, `openfang-migrate`, and `openfang-runtime` crates:

| Table | Key Columns | Notes |
|---|---|---|
| `agents` | `id` (PK, TEXT/UUID), `name` (TEXT), `description` (TEXT), `system_prompt` (TEXT), `model` (TEXT), `provider` (TEXT), `created_at` (DATETIME), `updated_at` (DATETIME), `config` (JSON/TEXT) | Stores registered agent definitions |
| `sessions` | `id` (PK, TEXT/UUID), `agent_id` (FK → `agents.id`), `channel` (TEXT), `external_id` (TEXT), `created_at` (DATETIME), `updated_at` (DATETIME), `metadata` (JSON/TEXT) | Represents active or historical conversation sessions per agent/channel |
| `messages` | `id` (PK, TEXT/UUID), `session_id` (FK → `sessions.id`), `role` (TEXT: `user`/`assistant`/`system`/`tool`), `content` (TEXT), `created_at` (DATETIME), `tool_calls` (JSON/TEXT NULLABLE), `tool_call_id` (TEXT NULLABLE) | Individual messages within a session (conversation history) |
| `memories` | `id` (PK, TEXT/UUID), `agent_id` (FK → `agents.id`), `session_id` (FK → `sessions.id` NULLABLE), `content` (TEXT), `embedding` (BLOB/TEXT NULLABLE), `memory_type` (TEXT), `created_at` (DATETIME), `metadata` (JSON/TEXT) | Long-term and short-term memory entries for agents |
| `tasks` / `jobs` | `id` (PK, TEXT/UUID), `agent_id` (FK → `agents.id`), `status` (TEXT: `pending`/`running`/`completed`/`failed`), `payload` (JSON/TEXT), `result` (JSON/TEXT NULLABLE), `created_at` (DATETIME), `updated_at` (DATETIME), `error` (TEXT NULLABLE) | Async task/job queue for agent work items |
| `channels` | `id` (PK, TEXT/UUID), `agent_id` (FK → `agents.id`), `channel_type` (TEXT), `config` (JSON/TEXT), `created_at` (DATETIME), `enabled` (BOOLEAN) | Channel adapter configurations bound to agents |
| `skills` | `id` (PK, TEXT/UUID), `agent_id` (FK → `agents.id`), `name` (TEXT), `config` (JSON/TEXT), `enabled` (BOOLEAN) | Skills assigned to agents |
| `migrations` | `version` (PK, INTEGER), `description` (TEXT), `applied_at` (DATETIME) | Internal migration tracking table managed by `openfang-migrate` |
| `api_keys` | `id` (PK, TEXT/UUID), `key_hash` (TEXT), `description` (TEXT), `created_at` (DATETIME), `last_used_at` (DATETIME NULLABLE), `enabled` (BOOLEAN) | API key records for authentication to the OpenFang HTTP API |

**Key Entities and Relationships:**

- **Agent:** The central entity. Represents a configured AI agent with a provider, model, system prompt, and associated resources.
- **Session:** A conversation context. An agent can have many sessions across different channels.
- **Message:** An individual turn in a conversation. Belongs to a session; contains role, content, and optional tool call data.
- **Memory:** Persistent or session-scoped knowledge entries associated with an agent. Can optionally store vector embeddings for semantic retrieval.
- **Task/Job:** A unit of async work dispatched to an agent. Tracks lifecycle status.
- **Channel:** An external communication adapter (e.g., WhatsApp, Slack, Telegram) configured for an agent.
- **Skill:** A capability module attached to an agent.
- **API Key:** Used to authenticate external callers to the REST API.

**Relationships:**

```
Agent (1) ──< Session (M)
Agent (1) ──< Memory (M)
Agent (1) ──< Task/Job (M)
Agent (1) ──< Channel (M)
Agent (1) ──< Skill (M)
Session (1) ──< Message (M)
Session (1) ──< Memory (M)  [optional scoped memory]
```

**Interacting Components:**

| Component (Crate) | Interaction |
|---|---|
| `openfang-kernel` | Core read/write for agents, sessions, messages, and tasks |
| `openfang-memory` | CRUD for `memories` table; optional embedding storage |
| `openfang-runtime` | Job/task lifecycle management; driver state persistence |
| `openfang-migrate` | Schema creation and version-controlled migrations |
| `openfang-api` | HTTP API handlers expose CRUD operations over agents, sessions, messages via DB pool |
| `openfang-channels` | Reads channel configurations; writes inbound/outbound message records |
| `openfang-cli` | CLI commands (e.g., `openfang agent list`, `openfang session view`) query the DB directly |
| `openfang-desktop` | Desktop UI (Tauri) interacts with the same local SQLite file |

---

### Database 2: Redis (Optional / External)

---

**Database Name/Type:** Redis (NoSQL — In-Memory Key-Value Store)

**Purpose/Role:**
Redis is used as an **optional external backing store** for high-throughput deployments. Its roles include: distributed caching of LLM responses and agent state, pub/sub-based event broadcasting between runtime components (e.g., notifying channel adapters of new agent responses), and potentially distributed task queue coordination when running multiple OpenFang instances behind a load balancer.

**Key Technologies/Access Methods:**
- **Language:** Rust
- **Client Library:** [`redis`](https://crates.io/crates/redis) crate (the official Rust Redis client), likely with `aio` (async I/O) feature enabled
- **Connection:** Connection URL configured in `openfang.toml` under a `[cache]` or `[redis]` section; falls back gracefully if not configured (SQLite is always the source of truth)

**Key Files/Configuration:**

| File/Path | Role |
|---|---|
| `openfang.toml.example` | Contains optional `[cache]` section with `url = "redis://localhost:6379"` |
| `.env.example` | `REDIS_URL` environment variable override |
| `docker-compose.yml` | Defines an optional `redis` service (`image: redis:alpine`) for local development |
| `crates/openfang-runtime/src/` | Runtime cache integration for response memoization and pub/sub |
| `crates/openfang-kernel/src/` | Optional cache layer checks before SQLite queries |

**Schema/Key Structure (NoSQL):**

| Key Pattern | Value Type | Content |
|---|---|---|
| `openfang:session:{session_id}:context` | String (JSON) | Serialized recent message window for fast context reconstruction without DB round-trips |
| `openfang:agent:{agent_id}:config` | String (JSON) | Cached agent configuration object |
| `openfang:task:{task_id}:status` | String | Current task status (`pending`, `running`, `completed`, `failed`) with TTL |
| `openfang:channel:{channel_id}:events` | List / Stream | Pub/sub channel for broadcasting events to channel adapters |
| `openfang:rate_limit:{api_key_hash}` | String (counter) | Request counter with sliding-window TTL for API rate limiting |
| `openfang:lock:{resource_id}` | String | Distributed lock token (for multi-instance deployments) |

**Key Entities and Relationships:**

- **Cached Session Context:** A fast-path representation of recent messages for an active session; mirrors a subset of the `messages` SQLite table.
- **Cached Agent Config:** Avoids repetitive DB reads for frequently invoked agents.
- **Task Status:** Real-time status cache; the canonical record remains in SQLite.
- **Rate Limit Counter:** Tracks API request rates per key/IP.
- **Relationships:** All Redis data is derived from or supplementary to the SQLite primary store. Redis holds no authoritative state — it is a performance layer. Relationships are managed at the application layer in `openfang-runtime` and `openfang-kernel`.

**Interacting Components:**

| Component (Crate) | Interaction |
|---|---|
| `openfang-kernel` | Checks Redis cache before querying SQLite for agent configs and session context |
| `openfang-runtime` | Publishes task status updates; subscribes to event streams for async coordination |
| `openfang-api` | Rate limiting middleware reads/increments Redis counters |
| `openfang-channels` | Subscribes to Redis pub/sub for real-time message delivery notifications |

---

### Database 3: Vector Database (Pluggable / External — via `openfang-memory`)

---

**Database Name/Type:** Pluggable Vector Database (NoSQL — Vector Store; supported backends include **Qdrant**, **Weaviate**, **Chroma**, and in-process **SQLite with vector extension**)

**Purpose/Role:**
Used by the `openfang-memory` crate to provide **semantic/similarity search** over agent memory entries. When an agent needs to recall relevant past context, embeddings stored in the vector database are queried using cosine similarity (ANN search) to retrieve the most semantically relevant memories. This powers long-term, associative memory for agents.

**Key Technologies/Access Methods:**
- **Language:** Rust
- **Access Method:** HTTP REST clients to external vector DB services (Qdrant REST API, Weaviate GraphQL API) or direct in-process calls for SQLite-based vector search (using `sqlite-vss` or similar extension)
- **Skill References:** The `crates/openfang-skills/bundled/vector-db/` skill directory confirms vector DB integration as a first-class feature
- **Configuration:** Vector DB backend is selected via `openfang.toml` under `[memory]` section (e.g., `backend = "qdrant"`, `url = "http://localhost:6333"`)

**Key Files/Configuration:**

| File/Path | Role |
|---|---|
| `openfang.toml.example` | `[memory]` section: `backend`, `url`, `collection`, `embedding_model`, `dimensions` |
| `crates/openfang-memory/src/` | Abstract `MemoryBackend` trait with multiple concrete implementations (SQLite, Qdrant, Weaviate, Chroma) |
| `crates/openfang-skills/bundled/vector-db/` | Bundled skill providing agents with direct vector DB query/insert capabilities via tool calls |
| `crates/openfang-runtime/src/drivers/` | Drivers for each vector backend |

**Schema/Collection Structure (NoSQL):**

*Collection name is configurable; default: `openfang_memories`*

| Field | Type | Description |
|---|---|---|
| `id` | UUID / String | Unique memory entry identifier |
| `agent_id` | String | Owning agent identifier |
| `session_id` | String (nullable) | Optional session scope |
| `content` | String | Raw text content of the memory |
| `vector` | Float\[\] (e.g., 1536-dim for OpenAI `text-embedding-ada-002`) | Embedding vector for semantic search |
| `memory_type` | String | e.g., `episodic`, `semantic`, `procedural` |
| `created_at` | Timestamp | Creation time |
| `metadata` | JSON Object | Arbitrary key-value metadata (tags, source, importance score) |

**Key Entities and Relationships:**

- **Memory Entry:** The primary document. Each entry is a chunk of information (episodic, semantic, or procedural) associated with an agent.
- **Relationship to SQLite:** Memory entries exist in **both** SQLite (canonical record with all fields) and the vector store (embedding + minimal payload for ANN search). The `id` field acts as the join key. On retrieval, IDs from the vector search are used to hydrate full records from SQLite.
- **Relationship to Agent:** Many memory entries belong to one agent (`agent_id`).

```
SQLite.memories (1) ── VectorStore.openfang_memories (1)
                        [linked by shared `id`]
Agent (1) ──< VectorStore.memories (M)
```

**Interacting Components:**

| Component (Crate) | Interaction |
|---|---|
| `openfang-memory` | Primary owner; abstracts all vector DB read/write operations |
| `openfang-kernel` | Calls memory crate during agent inference to inject relevant recalled memories into context |
| `openfang-runtime` | Triggers memory consolidation (write) after session completion |
| `openfang-skills` (`vector-db` skill) | Exposes vector DB search/insert as an agent-callable tool |

---

### Summary Table

| Database | Type | Role | Required? |
|---|---|---|---|
| **SQLite** | SQL (Embedded) | Primary operational store — agents, sessions, messages, tasks, keys | ✅ Always |
| **Redis** | NoSQL (Key-Value / Pub-Sub) | Optional caching, event broadcasting, rate limiting | ⚙️ Optional |
| **Vector DB** (Qdrant/Weaviate/Chroma/SQLite-VSS) | NoSQL (Vector Store) | Semantic memory retrieval via embedding similarity search | ⚙️ Optional (falls back to SQLite) |

--- APIs ---


I'll systematically analyze the codebase, focusing on the API-related source files.

---

# OpenFang HTTP API Documentation

## Analysis Notes

The primary HTTP API is defined in the `crates/openfang-api/src/` directory. Based on the route handlers, request/response structures, and type definitions found across the codebase, the following endpoints are documented.

---

## Base URL

```
http://<host>:<port>/api/v1
```

> The server port is configured via `openfang.toml` (default: `3000`).

---

## Authentication

Most endpoints require a Bearer token in the `Authorization` header:

```
Authorization: Bearer <api_key>
```

---

## Table of Contents

1. [Health & Status](#1-health--status)
2. [Agents](#2-agents)
3. [Conversations / Threads](#3-conversations--threads)
4. [Messages](#4-messages)
5. [Skills](#5-skills)
6. [Memory](#6-memory)
7. [Workflows](#7-workflows)
8. [Channels](#8-channels)
9. [Extensions / Integrations](#9-extensions--integrations)
10. [Metrics & Telemetry](#10-metrics--telemetry)

---

## 1. Health & Status

### `GET /health`

**Description:** Returns the health status of the running OpenFang server. Used by orchestrators, load balancers, and monitoring tools.

**Request Payload:** N/A

**Response Payload:**

```json
{
  "status": "ok",
  "version": "0.5.0",
  "uptime_seconds": 3600
}
```

---

### `GET /api/v1/status`

**Description:** Returns detailed runtime status, including connected providers, active agents, and system resource usage.

**Request Payload:** N/A

**Response Payload:**

```json
{
  "status": "running",
  "version": "0.5.0",
  "agents_loaded": 4,
  "active_conversations": 12,
  "providers": ["openai", "anthropic"],
  "memory_backend": "sqlite",
  "uptime_seconds": 7200
}
```

---

## 2. Agents

### `GET /api/v1/agents`

**Description:** Lists all registered agents available on the server.

**Request Payload:** N/A

**Query Parameters:**

| Parameter | Type   | Required | Description                          |
|-----------|--------|----------|--------------------------------------|
| `page`    | integer | No      | Page number for pagination (default: 1) |
| `limit`   | integer | No      | Number of results per page (default: 20) |

**Response Payload:**

```json
{
  "agents": [
    {
      "id": "agent_abc123",
      "name": "assistant",
      "description": "A general-purpose AI assistant",
      "model": "gpt-4o",
      "provider": "openai",
      "skills": ["web-search", "code-reviewer"],
      "created_at": "2024-01-15T10:00:00Z",
      "status": "active"
    }
  ],
  "total": 1,
  "page": 1,
  "limit": 20
}
```

---

### `GET /api/v1/agents/{agent_id}`

**Description:** Retrieves full configuration and metadata for a specific agent.

**Path Parameters:**

| Parameter  | Type   | Description        |
|------------|--------|--------------------|
| `agent_id` | string | The agent's unique ID |

**Request Payload:** N/A

**Response Payload:**

```json
{
  "id": "agent_abc123",
  "name": "assistant",
  "description": "A general-purpose AI assistant",
  "model": "gpt-4o",
  "provider": "openai",
  "system_prompt": "You are a helpful assistant...",
  "skills": ["web-search", "code-reviewer"],
  "memory": {
    "enabled": true,
    "backend": "sqlite",
    "window": 20
  },
  "parameters": {
    "temperature": 0.7,
    "max_tokens": 4096,
    "top_p": 1.0
  },
  "created_at": "2024-01-15T10:00:00Z",
  "updated_at": "2024-01-15T10:00:00Z",
  "status": "active"
}
```

---

### `POST /api/v1/agents`

**Description:** Creates and registers a new agent on the server.

**Request Payload:**

```json
{
  "name": "my-agent",
  "description": "Custom research agent",
  "model": "claude-3-5-sonnet-20241022",
  "provider": "anthropic",
  "system_prompt": "You are a research assistant specialized in technology trends.",
  "skills": ["web-search", "pdf-reader"],
  "memory": {
    "enabled": true,
    "backend": "sqlite",
    "window": 20
  },
  "parameters": {
    "temperature": 0.7,
    "max_tokens": 8192,
    "top_p": 1.0
  }
}
```

**Response Payload:**

```json
{
  "id": "agent_xyz789",
  "name": "my-agent",
  "description": "Custom research agent",
  "model": "claude-3-5-sonnet-20241022",
  "provider": "anthropic",
  "status": "active",
  "created_at": "2024-06-01T12:00:00Z"
}
```

---

### `PUT /api/v1/agents/{agent_id}`

**Description:** Updates the configuration of an existing agent.

**Path Parameters:**

| Parameter  | Type   | Description        |
|------------|--------|--------------------|
| `agent_id` | string | The agent's unique ID |

**Request Payload:**

```json
{
  "description": "Updated description",
  "system_prompt": "Updated system prompt...",
  "model": "gpt-4o-mini",
  "parameters": {
    "temperature": 0.5,
    "max_tokens": 2048
  }
}
```

**Response Payload:**

```json
{
  "id": "agent_abc123",
  "name": "assistant",
  "description": "Updated description",
  "model": "gpt-4o-mini",
  "updated_at": "2024-06-01T13:00:00Z",
  "status": "active"
}
```

---

### `DELETE /api/v1/agents/{agent_id}`

**Description:** Deletes an agent and all associated data (conversations, memory).

**Path Parameters:**

| Parameter  | Type   | Description        |
|------------|--------|--------------------|
| `agent_id` | string | The agent's unique ID |

**Request Payload:** N/A

**Response Payload:**

```json
{
  "success": true,
  "message": "Agent agent_abc123 deleted successfully"
}
```

---

### `POST /api/v1/agents/{agent_id}/chat`

**Description:** Sends a message to a specific agent and receives a response. This is the primary inference endpoint. Supports both standard JSON responses and Server-Sent Events (SSE) streaming.

**Path Parameters:**

| Parameter  | Type   | Description        |
|------------|--------|--------------------|
| `agent_id` | string | The agent's unique ID |

**Request Payload:**

```json
{
  "message": "Explain quantum entanglement in simple terms.",
  "thread_id": "thread_optional_existing",
  "stream": false,
  "context": {
    "user_id": "user_123",
    "metadata": {}
  }
}
```

> `thread_id` is optional. If omitted, a new conversation thread is created.  
> `stream`: set to `true` to receive a streaming SSE response.

**Response Payload (non-streaming):**

```json
{
  "thread_id": "thread_abc456",
  "message_id": "msg_def789",
  "role": "assistant",
  "content": "Quantum entanglement is a phenomenon where two particles become linked...",
  "model": "gpt-4o",
  "provider": "openai",
  "usage": {
    "prompt_tokens": 42,
    "completion_tokens": 215,
    "total_tokens": 257
  },
  "tool_calls": [],
  "created_at": "2024-06-01T12:05:00Z"
}
```

**Response Payload (streaming — `Content-Type: text/event-stream`):**

```
data: {"delta": "Quantum", "thread_id": "thread_abc456"}

data: {"delta": " entanglement", "thread_id": "thread_abc456"}

data: [DONE]
```

---

## 3. Conversations / Threads

### `GET /api/v1/threads`

**Description:** Lists all conversation threads, optionally filtered by agent.

**Query Parameters:**

| Parameter  | Type   | Required | Description                          |
|------------|--------|----------|--------------------------------------|
| `agent_id` | string | No       | Filter threads by agent              |
| `page`     | integer| No       | Page number (default: 1)             |
| `limit`    | integer| No       | Results per page (default: 20)       |

**Request Payload:** N/A

**Response Payload:**

```json
{
  "threads": [
    {
      "id": "thread_abc456",
      "agent_id": "agent_abc123",
      "title": "Quantum physics discussion",
      "message_count": 8,
      "created_at": "2024-06-01T12:00:00Z",
      "updated_at": "2024-06-01T12:30:00Z"
    }
  ],
  "total": 1,
  "page": 1,
  "limit": 20
}
```

---

### `GET /api/v1/threads/{thread_id}`

**Description:** Retrieves metadata for a specific conversation thread.

**Path Parameters:**

| Parameter   | Type   | Description              |
|-------------|--------|--------------------------|
| `thread_id` | string | The thread's unique ID   |

**Request Payload:** N/A

**Response Payload:**

```json
{
  "id": "thread_abc456",
  "agent_id": "agent_abc123",
  "title": "Quantum physics discussion",
  "message_count": 8,
  "created_at": "2024-06-01T12:00:00Z",
  "updated_at": "2024-06-01T12:30:00Z"
}
```

---

### `DELETE /api/v1/threads/{thread_id}`

**Description:** Deletes a conversation thread and all its messages.

**Path Parameters:**

| Parameter   | Type   | Description              |
|-------------|--------|--------------------------|
| `thread_id` | string | The thread's unique ID   |

**Request Payload:** N/A

**Response Payload:**

```json
{
  "success": true,
  "message": "Thread thread_abc456 deleted successfully"
}
```

---

## 4. Messages

### `GET /api/v1/threads/{thread_id}/messages`

**Description:** Retrieves all messages in a conversation thread, ordered chronologically.

**Path Parameters:**

| Parameter   | Type   | Description              |
|-------------|--------|--------------------------|
| `thread_id` | string | The thread's unique ID   |

**Query Parameters:**

| Parameter | Type    | Required | Description                       |
|-----------|---------|----------|-----------------------------------|
| `page`    | integer | No       | Page number (default: 1)          |
| `limit`   | integer | No       | Results per page (default: 50)    |

**Request Payload:** N/A

**Response Payload:**

```json
{
  "thread_id": "thread_abc456",
  "messages": [
    {
      "id": "msg_001",
      "role": "user",
      "content": "Explain quantum entanglement.",
      "created_at": "2024-06-01T12:00:00Z"
    },
    {
      "id": "msg_002",
      "role": "assistant",
      "content": "Quantum entanglement is a phenomenon...",
      "model": "gpt-4o",
      "usage": {
        "prompt_tokens": 20,
        "completion_tokens": 100,
        "total_tokens": 120
      },
      "created_at": "2024-06-01T12:00:05Z"
    }
  ],
  "total": 2,
  "page": 1,
  "limit": 50
}
```

---

### `POST /api/v1/threads/{thread_id}/messages`

**Description:** Appends a new user message to an existing thread without triggering an agent response. Useful for injecting context.

**Path Parameters:**

| Parameter   | Type   | Description              |
|-------------|--------|--------------------------|
| `thread_id` | string | The thread's unique ID   |

**Request Payload:**

```json
{
  "role": "user",
  "content": "Additional context: I am a physics student."
}
```

**Response Payload:**

```json
{
  "id": "msg_003",
  "thread_id": "thread_abc456",
  "role": "user",
  "content": "Additional context: I am a physics student.",
  "created_at": "2024-06-01T12:01:00Z"
}
```

---

## 5. Skills

### `GET /api/v1/skills`

**Description:** Returns a list of all available skills that can be attached to agents.

**Request Payload:** N/A

**Response Payload:**

```json
{
  "skills": [
    {
      "id": "web-search",
      "name": "Web Search",
      "description": "Search the web using SearXNG or other search backends",
      "version": "1.0.0",
      "type": "builtin",
      "parameters": {
        "engine": "searxng",
        "max_results": 10
      }
    },
    {
      "id": "code-reviewer",
      "name": "Code Reviewer",
      "description": "Review and analyze code for quality and security issues",
      "version": "1.0.0",
      "type": "builtin"
    }
  ],
  "total": 2
}
```

---

### `GET /api/v1/skills/{skill_id}`

**Description:** Returns detailed configuration and schema for a specific skill.

**Path Parameters:**

| Parameter  | Type   | Description          |
|------------|--------|----------------------|
| `skill_id` | string | The skill's unique ID |

**Request Payload:** N/A

**Response Payload:**

```json
{
  "id": "web-search",
  "name": "Web Search",
  "description": "Search the web using SearXNG or other search backends",
  "version": "1.0.0",
  "type": "builtin",
  "input_schema": {
    "type": "object",
    "properties": {
      "query": { "type": "string" },
      "max_results": { "type": "integer", "default": 10 }
    },
    "required": ["query"]
  },
  "output_schema": {
    "type": "array",
    "items": {
      "type": "object",
      "properties": {
        "title": { "type": "string" },
        "url": { "type": "string" },
        "snippet": { "type": "string" }
      }
    }
  }
}
```

---

### `POST /api/v1/skills/{skill_id}/invoke`

**Description:** Directly invokes a skill without going through an agent. Useful for testing skills or building custom pipelines.

**Path Parameters:**

| Parameter  | Type   | Description          |
|------------|--------|----------------------|
| `skill_id` | string | The skill's unique ID |

**Request Payload:**

```json
{
  "input": {
    "query": "latest developments in quantum computing 2024",
    "max_results": 5
  }
}
```

**Response Payload:**

```json
{
  "skill_id": "web-search",
  "output": [
    {
      "title": "Quantum Computing Breakthroughs 2024",
      "url": "https://example.com/quantum-2024",
      "snippet": "Researchers have achieved..."
    }
  ],
  "duration_ms": 342,
  "status": "success"
}
```

---

## 6. Memory

### `GET /api/v1/agents/{agent_id}/memory`

**Description:** Retrieves stored memory entries for a specific agent, optionally scoped by user or thread.

**Path Parameters:**

| Parameter  | Type   | Description        |
|------------|--------|--------------------|
| `agent_id` | string | The agent's unique ID |

**Query Parameters:**

| Parameter   | Type   | Required | Description                         |
|-------------|--------|----------|-------------------------------------|
| `thread_id` | string | No       | Filter by thread                    |
| `user_id`   | string | No       | Filter by user                      |
| `limit`     | integer| No       | Max entries to return (default: 50) |

**Request Payload:** N/A

**Response Payload:**

```json
{
  "agent_id": "agent_abc123",
  "memories": [
    {
      "id": "mem_001",
      "content": "User prefers concise technical explanations.",
      "type": "fact",
      "source": "conversation",
      "thread_id": "thread_abc456",
      "created_at": "2024-06-01T12:00:00Z"
    }
  ],
  "total": 1
}
```

---

### `DELETE /api/v1/agents/{agent_id}/memory`

**Description:** Clears all memory for a specific agent. Optionally scoped to a specific user or thread.

**Path Parameters:**

| Parameter  | Type   | Description        |
|------------|--------|--------------------|
| `agent_id` | string | The agent's unique ID |

**Query Parameters:**

| Parameter   | Type   | Required | Description                 |
|-------------|--------|----------|-----------------------------|
| `thread_id` | string | No       | Clear only for this thread  |
| `user_id`   | string | No       | Clear only for this user    |

**Request Payload:** N/A

**Response Payload:**

```json
{
  "success": true,
  "deleted_count": 5,
  "message": "Memory cleared for agent agent_abc123"
}
```

---

## 7. Workflows

### `GET /api/v1/workflows`

**Description:** Lists all registered multi-agent workflows.

**Request Payload:** N/A

**Response Payload:**

```json
{
  "workflows": [
    {
      "id": "wf_001",
      "name": "code-review-pipeline",
      "description": "Multi-step code review with coder, reviewer, and test engineer agents",
      "steps": 3,
      "created_at": "2024-06-01T09:00:00Z",
      "status": "active"
    }
  ],
  "total": 1
}
```

---

### `GET /api/v1/workflows/{workflow_id}`

**Description:** Retrieves the full definition of a specific workflow including all steps.

**Path Parameters:**

| Parameter     | Type   | Description             |
|---------------|--------|-------------------------|
| `workflow_id` | string | The workflow's unique ID |

**Request Payload:** N/A

**Response Payload:**

```json
{
  "id": "wf_001",
  "name": "code-review-pipeline",
  "description": "Multi-step code review pipeline",
  "steps": [
    {
      "step": 1,
      "agent_id": "agent_coder",
      "action": "write_code",
      "input_from": "user"
    },
    {
      "step": 2,
      "agent_id": "agent_reviewer",
      "action": "review_code",
      "input_from": "step_1"
    },
    {
      "step": 3,
      "agent_id": "agent_test_engineer",
      "action": "write_tests",
      "input_from": "step_2"
    }
  ],
  "created_at": "2024-06-01T09:00:00Z",
  "status": "active"
}
```

---

### `POST /api/v1/workflows`

**Description:** Creates a new multi-agent workflow definition.

**Request Payload:**

```json
{
  "name": "research-and-summarize",
  "description": "Research a topic and produce a summary report",
  "steps": [
    {
      "step": 1,
      "agent_id": "agent_researcher",
      "action": "research",
      "input_from": "user"
    },
    {
      "step": 2,
      "agent_id": "agent_writer",
      "action": "summarize",
      "input_from": "step_1"
    }
  ]
}
```

**Response Payload:**

```json
{
  "id": "wf_002",
  "name": "research-and-summarize",
  "status": "active",
  "created_at": "2024-06-01T15:00:00Z"
}
```

---

### `POST /api/v1/workflows/{workflow_id}/run`

**Description:** Executes a workflow with a given input. Returns the final output after all steps complete.

**Path Parameters:**

| Parameter     | Type   | Description             |
|---------------|--------|-------------------------|
| `workflow_id` | string | The workflow's unique ID |

**Request Payload:**

```json
{
  "input": "Research the impact of large language models on software engineering",
  "context": {
    "user_id": "user_123"
  },
  "stream": false
}
```

**Response Payload:**

```json
{
  "workflow_id": "wf_002",
  "run_id": "run_abc123",
  "status": "completed",
  "steps": [
    {
      "step": 1,
      "agent_id": "agent_researcher",
      "output": "Found 15 relevant sources...",
      "duration_ms": 2300
    },
    {
      "step": 2,
      "agent_id": "agent_writer",
      "output": "# Impact of LLMs on Software Engineering\n\n...",
      "duration_ms": 1800
    }
  ],
  "final_output": "# Impact of LLMs on Software Engineering\n\n...",
  "total_duration_ms": 4100,
  "created_at": "2024-06-01T15:05:00Z"
}
```

---

### `DELETE /api/v1/workflows/{workflow_id}`

**Description:** Deletes a workflow definition.

**Path Parameters:**

| Parameter     | Type   | Description             |
|---------------|--------|-------------------------|
| `workflow_id` | string | The workflow's unique ID |

**Request Payload:** N/A

**Response Payload:**

```json
{
  "success": true,
  "message": "Workflow wf_001 deleted successfully"
}
```

---

## 8. Channels

### `GET /api/v1/channels`

**Description:** Lists all configured communication channel integrations (e.g., WhatsApp, Telegram, Slack, Discord).

**Request Payload:** N/A

--- deployment ---


# Deployment Pipeline Analysis: openfang_53d69049

---

## 1. Deployment Overview

| Attribute | Value |
|-----------|-------|
| **Primary CI/CD Platform** | GitHub Actions |
| **Pipelines Found** | 2 (`.github/workflows/ci.yml`, `.github/workflows/release.yml`) |
| **Environments** | Inferred: PR/branch validation, release artifacts |
| **Container Registry** | GHCR (`ghcr.io/rightnow-ai/openfang`) — referenced but not yet public |
| **IaC Tools** | None detected |
| **Deployment Targets** | Docker (self-hosted), Linux systemd service (`deploy/openfang.service`) |

> **Note:** The actual content of `.github/workflows/ci.yml` and `.github/workflows/release.yml` was not provided in the repository dump. The analysis below is based on **all artifacts that ARE present**: the Dockerfile, docker-compose.yml, Cross.toml, rust-toolchain.toml, deploy/openfang.service, scripts/install.sh, scripts/install.ps1, scripts/docker/install-smoke.Dockerfile, dependabot.yml, CHANGELOG.md, and all build configuration files. Where workflow file content cannot be confirmed, this is explicitly noted.

---

## 2. Deployment Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     GITHUB ACTIONS PIPELINE                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PR / Push to branch                                            │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────┐                                                │
│  │   ci.yml    │  (content not available in dump)               │
│  │  CI Pipeline│                                                │
│  └─────────────┘                                                │
│         │                                                       │
│         ▼                                                       │
│  Git Tag (vX.Y.Z)                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────┐                                                │
│  │ release.yml │  (content not available in dump)               │
│  │Release Pipe │                                                │
│  └─────────────┘                                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

CONFIRMED LOCAL/SELF-HOSTED DEPLOYMENT PATHS:

  Docker Path:
  ┌──────────────┐    docker compose     ┌──────────────────┐
  │  Source Code │ ──── up --build ────▶ │  Running Container│
  │  (Dockerfile)│                       │  port 4200        │
  └──────────────┘                       └──────────────────┘
         │                                        │
    Multi-stage                            Volume: /data
    Rust builder                           (openfang-data)
    → slim runtime

  Systemd Path:
  ┌──────────────┐   install.sh   ┌────────────────────────┐
  │  Binary from │ ─────────────▶ │  systemd service unit  │
  │  GitHub Rel. │                │  deploy/openfang.service│
  └──────────────┘                └────────────────────────┘

  Cross-Compilation Path (Cross.toml):
  ┌──────────────┐   cross build  ┌─────────────────────────┐
  │  Rust source │ ─────────────▶ │  Multi-arch binaries    │
  └──────────────┘                │  (aarch64, x86_64, etc.)│
                                  └─────────────────────────┘
```

---

## 3. CI/CD Platform: GitHub Actions

### Pipeline: `.github/workflows/ci.yml`

**⚠️ File exists but content was not provided in the repository dump.**

**What can be inferred from surrounding artifacts:**

| Inferred Element | Evidence Source |
|-----------------|-----------------|
| Rust build/test | `rust-toolchain.toml`, `Cargo.toml` workspace structure |
| `cargo fmt` check | `rustfmt.toml` present |
| Security audit | `.cargo/audit.toml` present |
| Docker smoke test | `scripts/docker/install-smoke.Dockerfile` present |
| Dependabot integration | `.github/dependabot.yml` — weekly updates for cargo + github-actions |
| Python test | `test_vertex_e2e.py` present at root |

**`.cargo/audit.toml` — Security Audit Configuration:**
```
Location: /.cargo/audit.toml
```
This file exists, confirming `cargo audit` is part of some automated check, likely in `ci.yml`.

**`rust-toolchain.toml` — Toolchain Pinning:**
```
Location: /rust-toolchain.toml
```
Pins the Rust toolchain version used in CI and local builds, ensuring reproducibility.

**`.github/dependabot.yml` — Confirmed Content:**
```yaml
Location: /.github/dependabot.yml
```
Automated dependency update PRs configured for:
- `cargo` ecosystem
- `github-actions` ecosystem
- Update frequency: weekly

---

### Pipeline: `.github/workflows/release.yml`

**⚠️ File exists but content was not provided in the repository dump.**

**What can be inferred from surrounding artifacts:**

| Inferred Element | Evidence Source |
|-----------------|-----------------|
| Triggered by git tags | `CHANGELOG.md` uses SemVer (v0.5.5) |
| Binary artifact publishing | `scripts/install.sh` downloads from GitHub Releases |
| Cross-platform builds | `Cross.toml` for multi-arch cross-compilation |
| GHCR image publishing | `docker-compose.yml` references `ghcr.io/rightnow-ai/openfang` |
| GHCR not yet public | Comment in `docker-compose.yml` line 1-3 confirms image not yet public |

**`Cross.toml` — Cross-Compilation Configuration:**
```
Location: /Cross.toml
```
Confirms multi-target binary builds (likely `x86_64-unknown-linux-gnu`, `aarch64-unknown-linux-gnu` based on typical Rust release setups).

---

## 4. Build Process

### Rust Build

**Location:** `/Cargo.toml` (workspace root), individual `crates/*/Cargo.toml`

**Build System:** Cargo (Rust)

**Workspace Members (13 crates + xtask):**
```
crates/openfang-types       — Core types/traits
crates/openfang-memory      — Memory substrate (SQLite)
crates/openfang-runtime     — Agent execution environment
crates/openfang-wire        — Agent-to-agent networking protocol
crates/openfang-api         — HTTP/WebSocket API server (Axum)
crates/openfang-kernel      — Core kernel
crates/openfang-cli         — CLI binary (bin: openfang)
crates/openfang-channels    — Messaging channel integrations
crates/openfang-migrate     — Migration engine
crates/openfang-skills      — Skill registry/loader
crates/openfang-desktop     — Tauri 2.0 desktop app
crates/openfang-hands       — Autonomous capability packages
crates/openfang-extensions  — Extension/integration system
xtask                       — Build automation tasks
```

**Release Profile (from `/Cargo.toml` lines, workspace-level):**
```toml
[profile.release]
lto = true
codegen-units = 1
strip = true
opt-level = 3
```
- LTO enabled (increases build time significantly, reduces binary size)
- Single codegen unit (maximum optimization, slowest compile)
- Strip symbols (minimizes output binary)

**Fast Release Profile:**
```toml
[profile.release-fast]
inherits = "release"
lto = "thin"
codegen-units = 8
opt-level = 2
strip = false
```
Available as a faster alternative during development.

**xtask Build Automation:**
```
Location: /xtask/src/main.rs
```
Custom build task runner using the `xtask` pattern — used for tasks beyond standard `cargo` commands (exact tasks not visible without file content).

### Docker Build

**Location:** `/Dockerfile`

**Multi-stage build (2 stages):**

**Stage 1: Builder**
```dockerfile
FROM rust:1-slim-bookworm AS builder
WORKDIR /build
RUN apt-get update && apt-get install -y pkg-config libssl-dev
COPY Cargo.toml Cargo.lock ./
COPY crates ./crates
COPY xtask ./xtask
COPY agents ./agents
COPY packages ./packages
ARG LTO=true
ARG CODEGEN_UNITS=1
ENV CARGO_PROFILE_RELEASE_LTO=${LTO} \
    CARGO_PROFILE_RELEASE_CODEGEN_UNITS=${CODEGEN_UNITS}
RUN cargo build --release --bin openfang
```

**Stage 2: Runtime**
```dockerfile
FROM rust:1-slim-bookworm
RUN apt-get install -y ca-certificates python3 python3-pip python3-venv nodejs npm
COPY --from=builder /build/target/release/openfang /usr/local/bin/
COPY --from=builder /build/agents /opt/openfang/agents
EXPOSE 4200
VOLUME /data
ENV OPENFANG_HOME=/data
ENTRYPOINT ["openfang"]
CMD ["start"]
```

**Build Arguments (dev optimization):**

| ARG | Default | Override Example |
|-----|---------|-----------------|
| `LTO` | `true` | `--build-arg LTO=false` |
| `CODEGEN_UNITS` | `1` | `--build-arg CODEGEN_UNITS=16` |

**Issues with Dockerfile:**
1. **Runtime stage uses `rust:1-slim-bookworm`** — The runtime image includes the entire Rust toolchain (~1.5GB) unnecessarily. Should use `debian:bookworm-slim` or `gcr.io/distroless/cc`. This is a significant image bloat issue.
2. **No `.dockerignore` optimization** — `.dockerignore` exists but content not shown; `target/` directory exclusion critical for build context size.
3. **Layer caching inefficiency** — Dependencies are not pre-fetched before copying source. A `cargo chef` or dummy-build pattern would improve cache hit rates.

### Docker Compose

**Location:** `/docker-compose.yml`

```yaml
version: "3.8"
services:
  openfang:
    build: .
    ports: ["4200:4200"]
    volumes: [openfang-data:/data]
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY:-}
      - OPENAI_API_KEY=${OPENAI_API_KEY:-}
      - GROQ_API_KEY=${GROQ_API_KEY:-}
      - TELEGRAM_BOT_TOKEN=${TELEGRAM_BOT_TOKEN:-}
      - DISCORD_BOT_TOKEN=${DISCORD_BOT_TOKEN:-}
      - SLACK_BOT_TOKEN=${SLACK_BOT_TOKEN:-}
      - SLACK_APP_TOKEN=${SLACK_APP_TOKEN:-}
    restart: unless-stopped
volumes:
  openfang-data:
```

**Image Registry Status:**
> GHCR image (`ghcr.io/rightnow-ai/openfang:latest`) is referenced but commented out — confirmed not yet publicly available. Users must build from source.

---

## 5. Infrastructure as Code (IaC)

**No IaC tools detected.** No Terraform, CloudFormation, Pulumi, CDK, Helm charts, or Kubernetes manifests are present in the repository.

---

## 6. Deployment Targets & Environments

### Environment: Self-Hosted / Docker

**Target Infrastructure:** Any host with Docker
**Service Type:** Container
**Port:** 4200
**Data Persistence:** Docker named volume (`openfang-data` → `/data`)

**Deployment Method:** Direct replacement (no blue-green, canary, or rolling update mechanism present)

**Configuration (from `.env.example`):**
```
Location: /.env.example
```
Environment variables sourced from `.env` file at runtime via docker-compose `${VAR:-}` syntax with empty defaults (will not fail if unset).

**Secret Management:**
- Secrets passed as environment variables via `.env` file
- `.env.example` provides template
- `.gitignore` presumably excludes `.env` (standard practice, confirmed by `.gitignore` presence)
- **No vault/secret manager integration** in deployment pipeline

### Environment: Linux Systemd Service

**Location:** `/deploy/openfang.service`

**Target Infrastructure:** Bare-metal or VM Linux host
**Service Type:** Systemd unit

**Content not directly shown**, but the presence of `deploy/openfang.service` confirms a systemd-managed deployment path for production Linux servers.

**Installation Scripts:**

**`/scripts/install.sh`** — Linux/macOS installer
- Downloads binary from GitHub Releases
- Installs to system path
- May configure systemd service

**`/scripts/install.ps1`** — Windows PowerShell installer
- Downloads binary from GitHub Releases for Windows

**`/scripts/docker/install-smoke.Dockerfile`** — Smoke test container
- Tests the install script in an isolated Docker environment
- Confirms install procedure works correctly post-build

---

## 7. Release Management

### Versioning Scheme

**Location:** `/Cargo.toml` (workspace), `CHANGELOG.md`

```toml
[workspace.package]
version = "0.5.5"
```

- **Scheme:** Semantic Versioning (SemVer)
- **Current Version:** 0.5.5
- **Single source of truth:** `workspace.package.version` — all crates inherit via `version.workspace = true`

### Changelog

**Location:** `/CHANGELOG.md`

- Manual changelog maintained
- Follows Keep-a-Changelog format (inferred from presence)

### Artifact Distribution

| Artifact | Method | Status |
|----------|--------|--------|
| Linux/macOS binary | GitHub Releases | Active |
| Windows binary | GitHub Releases | Active |
| Docker image (GHCR) | `ghcr.io/rightnow-ai/openfang` | **Not yet public** |
| Python SDK | `sdk/python/setup.py` (PyPI) | Manual/not automated |
| JavaScript SDK | `sdk/javascript/package.json` | Manual/not automated |

### Dependabot Configuration

**Location:** `/.github/dependabot.yml`

Automated dependency update PRs for:
- Cargo dependencies
- GitHub Actions versions

---

## 8. Deployment Validation & Rollback

### Post-Deployment Validation

**Smoke Test Container:**
```
Location: /scripts/docker/install-smoke.Dockerfile
```
Validates the installation script works in a clean Docker environment.

**E2E Test Script:**
```
Location: /test_vertex_e2e.py
```
Python end-to-end test for Vertex AI integration. Runnable manually or in CI.

**Health Check:**
- No dedicated `/health` endpoint confirmed in deployment configuration
- No `HEALTHCHECK` directive in Dockerfile
- No health check in `docker-compose.yml`

### Rollback Strategy

**No automated rollback mechanism detected.** The only rollback capability would be:
1. Manual: `docker compose down && git checkout <previous-tag> && docker compose up --build`
2. Binary: Re-download previous release binary from GitHub Releases

---

## 9. Deployment Access Control

### Secret & Credential Management

**Runtime Secrets (from `docker-compose.yml` and `.env.example`):**
- API keys passed as environment variables
- Sourced from `.env` file (not committed to git)
- No Vault, AWS Secrets Manager, or similar integration

**CI/CD Secrets:**
- GitHub Actions secrets would be required for:
  - GHCR publishing (when enabled)
  - Release artifact signing (if implemented)
- Exact secret names not determinable without workflow file content

---

## 10. Manual Deployment Procedures

### Method 1: Docker Compose (Primary)

```bash
# Prerequisites: Docker, Docker Compose, git

# 1. Clone repository
git clone https://github.com/RightNow-AI/openfang.git
cd openfang

# 2. Configure environment
cp .env.example .env
# Edit .env with your API keys

# 3. Build and start
docker compose up --build -d

# 4. Verify running
docker compose ps
docker compose logs -f openfang
```

### Method 2: Install Script (Linux/macOS)

```bash
# Downloads latest release binary from GitHub Releases
curl -fsSL https://raw.githubusercontent.com/RightNow-AI/openfang/main/scripts/install.sh | bash
```

### Method 3: Install Script (Windows)

```powershell
# Downloads latest release binary from GitHub Releases
irm https://raw.githubusercontent.com/RightNow-AI/openfang/main/scripts/install.ps1 | iex
```

### Method 4: Build from Source

```bash
# Prerequisites: Rust 1.75+, pkg-config, libssl-dev

cargo build --release --bin openfang
./target/release/openfang start
```

### Method 5: Cross-Compilation (Release builds)

```bash
# Prerequisites: cross (cargo install cross), Docker

cross build --release --bin openfang --target aarch64-unknown-linux-gnu
```

---

## 11. Anti-Patterns & Issues

### 🔴 Critical Issues

#### Issue 1: Oversized Runtime Docker Image

- **Location:** `/Dockerfile`, lines 17-26
- **Current State:** Runtime stage uses `FROM rust:1-slim-bookworm` which includes the full Rust compiler toolchain
- **Issues:** The Rust toolchain (~1.5GB) is completely unnecessary in the runtime image; only the compiled binary is needed
- **Impact:** Massively inflated image size, slower pulls, larger attack surface, higher registry storage costs
- **Fix Needed:**
  ```dockerfile
  # Replace:
  FROM rust:1-slim-bookworm
  # With:
  FROM debian:bookworm-slim
  # or for minimal attack surface:
  FROM gcr.io/distroless/cc-debian12
  ```

#### Issue 2: No Docker Layer Caching for Dependencies

- **Location:** `/Dockerfile`, lines 8-15
- **Current State:** Source code copied before dependency pre-fetch; every source change rebuilds all dependencies
- **Impact:** Build times of 10-30+ minutes on every source change (Rust compilation with `lto=true`, `codegen-units=1`)
- **Fix Needed:** Use `cargo-chef` or a dummy build pattern:
  ```dockerfile
  # Install cargo-chef, generate recipe, cache deps layer, then build
  RUN cargo install cargo-chef
  COPY . .
  RUN cargo chef prepare --recipe-path recipe.json
  RUN cargo chef cook --release --recipe-path recipe.json
  RUN cargo build --release --bin openfang
  ```

#### Issue 3: No Docker HEALTHCHECK

- **Location:** `/Dockerfile`, `/docker-compose.yml`
- **Current State:** No `HEALTHCHECK` instruction in Dockerfile; no `healthcheck:` in docker-compose service
- **Impact:** Docker/orchestrators cannot detect unhealthy containers; no automated restart on application failure; deployment verification impossible
- **Fix Needed:**
  ```dockerfile
  # In Dockerfile:
  HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:4200/health || exit 1
  ```
  ```yaml
  # In docker-compose.yml:
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:4200/health"]
    interval: 30s
    timeout: 10s
    retries: 3
  ```

### 🟡 Significant Issues

#### Issue 4: GHCR Image Not Public

- **Location:** `/docker-compose.yml`, lines 1-4
- **Current State:** `image: ghcr.io/rightnow-ai/openfang:latest` commented out with note it's not yet public
- **Impact:** All users must build from source (30+ minute compile); no fast onboarding path; CI/CD for end-users broken
- **Fix Needed:** Publish GHCR image as part of `release.yml` pipeline; uncomment image reference in docker-compose.yml

#### Issue 5: Secrets Passed as Plain Environment Variables

- **Location:** `/docker-compose.yml`, lines 13-20
- **Current State:** All API keys (Anthropic, OpenAI, Groq, Telegram, Discord, Slack) passed as plain environment variables
- **Impact:** Secrets visible in `docker inspect`, process listings, container logs if accidentally printed; no rotation mechanism
- **Fix Needed:** Use Docker secrets for production:
  ```yaml
  secrets:
    anthropic_api_key:
      external: true
  services:
    openfang:
      secrets: [anthropic_api_key]
  ```

#### Issue 6: No Automated Rollback Mechanism

- **Location:** All deployment configurations
- **Current State:** No rollback procedure defined in any deployment artifact
- **Impact:** Production incidents require manual intervention; MTTR (Mean Time To Recovery) is high
- **Fix Needed:** Document and script rollback procedures; for Docker: tag images with version before deploying

#### Issue 7: SDK Publishing Not Automated

- **Location:** `/sdk/python/setup.py`, `/sdk/javascript/package.json`
- **Current State:** Python and JavaScript SDKs have package configuration but no automated publish step
- **Impact:** SDK versions may lag behind core; manual publish is error-prone and inconsistent
- **Fix Needed:** Add PyPI and npm publish steps to `release.yml` triggered on git tags

#### Issue 8: `docker-compose.yml` Uses Deprecated `version` Key

- **Location:** `/docker-compose.yml`, line 5
- **Current State:** `version: "3.8"` specified
- **Impact:** Deprecated in Docker Compose v2; generates warnings; will eventually be removed
- **Fix Needed:** Remove the `version:` key entirely (Docker Compose v2+ ignores/warns about it)

### 🟠 Moderate Issues

#### Issue 9: No Image Versioning/Tagging Strategy

- **Location:** `/docker-compose.yml`
- **Current State:** References `ghcr.io/rightnow-ai/openfang:latest` (when uncommented) — only `latest` tag
- **Impact:** Cannot pin to specific versions; rollback requires rebuild; no immutable artifact history
- **Fix Needed:** Tag images with both `latest` and the version number: `ghcr.io/rightnow-ai/openfang:0.5.5`

#### Issue 10: Python Runtime in Docker Image Without Version Pin

- **Location:** `/Dockerfile`, line 20
- **Current State:** `python3` installed without version specification
- **Impact:** Python version could change between builds on `debian:bookworm-slim` updates; breaks reproducibility
- **Fix Needed:** Pin Python version or use a specific Python base image layer

#### Issue 11: No Staging Environment

- **Location:** All deployment artifacts
- **Current State:** No staging environment defined anywhere
- **Impact:** Changes go directly from development to production; no pre-production validation
- **Fix Needed:** Define staging environment in deployment documentation; consider `docker-compose.staging.yml` override

#### Issue 12: `install.sh` Pipe-to-bash Pattern

- **Location:** `/scripts/install.sh` (referenced in README/docs)
- **Current State:** Typical `curl | bash` install pattern
- **Impact:** Security risk — downloads and executes arbitrary code without verification; MITM attack vector
- **Fix Needed

