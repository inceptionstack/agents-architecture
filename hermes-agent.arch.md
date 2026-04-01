
## Full Investigation — hermes-agent (8 sections)

--- APIs ---


# HTTP API Documentation

## Overview

This codebase contains two distinct HTTP API servers:

1. **Gateway API Server** (`gateway/platforms/api_server.py`) — A REST API for interacting with the Hermes agent via HTTP
2. **ACP Adapter Server** (`acp_adapter/server.py`) — An Agent Communication Protocol (ACP) server

---

## Gateway API Server

> Source: `gateway/platforms/api_server.py`, `tests/gateway/test_api_server.py`, `tests/gateway/test_api_server_jobs.py`, `tests/gateway/test_api_server_toolset.py`

---

### 1. Send a Message / Create a Run

**HTTP Method:** `POST`

**API URL:** `/v1/chat`

**Description:** Sends a message to the Hermes agent and initiates a run. Supports both synchronous (polling) and streaming (SSE) response modes.

**Request Payload:**
```json
{
  "message": "string",
  "session_id": "string (optional)",
  "stream": "boolean (optional, default: false)",
  "toolset": "string (optional)",
  "images": ["string (base64 or URL, optional)"],
  "attachments": ["string (file path or URL, optional)"]
}
```

**Response Payload (non-streaming):**
```json
{
  "run_id": "string",
  "session_id": "string",
  "status": "queued | running | completed | failed",
  "response": "string (agent reply, present when completed)"
}
```

**Response Payload (streaming, `stream: true`):**

Server-Sent Events (SSE) stream:
```
data: {"type": "token", "content": "string"}
data: {"type": "tool_call", "tool": "string", "input": {}}
data: {"type": "tool_result", "tool": "string", "output": "string"}
data: {"type": "done", "run_id": "string", "session_id": "string"}
```

---

### 2. Get Run Status

**HTTP Method:** `GET`

**API URL:** `/v1/runs/{run_id}`

**Description:** Retrieves the current status and result of a previously created run.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "run_id": "string",
  "session_id": "string",
  "status": "queued | running | completed | failed",
  "response": "string (agent reply, present when completed)",
  "error": "string (present when failed)"
}
```

---

### 3. Cancel a Run

**HTTP Method:** `DELETE`

**API URL:** `/v1/runs/{run_id}`

**Description:** Cancels an in-progress run.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "run_id": "string",
  "status": "cancelled"
}
```

---

### 4. List Sessions

**HTTP Method:** `GET`

**API URL:** `/v1/sessions`

**Description:** Lists all active or recent agent sessions.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "sessions": [
    {
      "session_id": "string",
      "created_at": "string (ISO 8601 timestamp)",
      "last_active": "string (ISO 8601 timestamp)"
    }
  ]
}
```

---

### 5. Get Session Info

**HTTP Method:** `GET`

**API URL:** `/v1/sessions/{session_id}`

**Description:** Retrieves metadata and transcript information for a specific session.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "session_id": "string",
  "created_at": "string (ISO 8601 timestamp)",
  "last_active": "string (ISO 8601 timestamp)",
  "message_count": "integer"
}
```

---

### 6. Delete / Reset a Session

**HTTP Method:** `DELETE`

**API URL:** `/v1/sessions/{session_id}`

**Description:** Deletes or resets a session, clearing its context history.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "session_id": "string",
  "status": "deleted"
}
```

---

### 7. Get Available Toolsets

**HTTP Method:** `GET`

**API URL:** `/v1/toolsets`

**Description:** Returns the list of available toolsets that can be activated for a session or run.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "toolsets": [
    {
      "name": "string",
      "description": "string",
      "tools": ["string"]
    }
  ]
}
```

---

### 8. SSE Stream for a Run

**HTTP Method:** `GET`

**API URL:** `/v1/runs/{run_id}/stream`

**Description:** Opens a Server-Sent Events (SSE) connection to stream live output from a running agent job.

**Request Payload:** N/A

**Response Payload:**

SSE stream:
```
data: {"type": "token", "content": "string"}
data: {"type": "tool_call", "tool": "string", "input": {}}
data: {"type": "tool_result", "tool": "string", "output": "string"}
data: {"type": "done", "run_id": "string", "session_id": "string"}
data: {"type": "error", "message": "string"}
```

---

### 9. Health / Status Check

**HTTP Method:** `GET`

**API URL:** `/health`

**Description:** Returns the health status of the API server.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "status": "ok",
  "version": "string"
}
```

---

## ACP Adapter Server

> Source: `acp_adapter/server.py`, `acp_adapter/entry.py`, `acp_adapter/session.py`, `acp_adapter/auth.py`, `acp_adapter/events.py`, `acp_adapter/tools.py`, `tests/acp/test_server.py`, `tests/acp/test_entry.py`, `tests/acp/test_events.py`, `tests/acp/test_session.py`, `tests/acp/test_tools.py`, `tests/acp/test_auth.py`, `tests/acp/test_permissions.py`

The ACP (Agent Communication Protocol) adapter exposes Hermes as a standards-compliant ACP agent. It follows the ACP specification for agent interoperability.

---

### 1. Create / Invoke Agent Run

**HTTP Method:** `POST`

**API URL:** `/runs`

**Description:** Creates a new agent run following the ACP protocol. Accepts a message payload and dispatches it to the Hermes agent. Supports synchronous and async modes.

**Request Payload:**
```json
{
  "agent_id": "string",
  "session_id": "string (optional)",
  "messages": [
    {
      "role": "user | assistant",
      "content": [
        {
          "type": "text",
          "text": "string"
        }
      ]
    }
  ],
  "mode": "sync | async | stream (optional, default: sync)"
}
```

**Response Payload (sync mode):**
```json
{
  "run_id": "string",
  "session_id": "string",
  "status": "completed | failed",
  "output": [
    {
      "role": "agent",
      "content": [
        {
          "type": "text",
          "text": "string"
        }
      ]
    }
  ]
}
```

**Response Payload (async mode):**
```json
{
  "run_id": "string",
  "session_id": "string",
  "status": "running"
}
```

---

### 2. Get Run Status (ACP)

**HTTP Method:** `GET`

**API URL:** `/runs/{run_id}`

**Description:** Retrieves the status and output of a previously created ACP run.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "run_id": "string",
  "session_id": "string",
  "status": "running | completed | failed",
  "output": [
    {
      "role": "agent",
      "content": [
        {
          "type": "text",
          "text": "string"
        }
      ]
    }
  ],
  "error": "string (present when failed, optional)"
}
```

---

### 3. Stream Run Events (ACP)

**HTTP Method:** `GET`

**API URL:** `/runs/{run_id}/events`

**Description:** Streams events from a running ACP job using Server-Sent Events (SSE). Emits agent output chunks, tool calls, and completion signals.

**Request Payload:** N/A

**Response Payload:**

SSE stream of ACP events:
```
data: {"type": "message.output.chunk", "run_id": "string", "content": {"type": "text", "text": "string"}}
data: {"type": "run.completed", "run_id": "string", "status": "completed"}
data: {"type": "run.failed", "run_id": "string", "error": "string"}
```

---

### 4. List Agent Capabilities / Tools

**HTTP Method:** `GET`

**API URL:** `/tools`

**Description:** Returns the list of tools/capabilities exposed by the Hermes agent in ACP format.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "tools": [
    {
      "name": "string",
      "description": "string",
      "input_schema": {
        "type": "object",
        "properties": {},
        "required": []
      }
    }
  ]
}
```

---

### 5. Agent Manifest / Entry Point

**HTTP Method:** `GET`

**API URL:** `/.well-known/agent.json`

**Description:** Returns the ACP agent manifest describing the agent's identity, capabilities, and supported protocol version. Used for ACP agent discovery.

**Request Payload:** N/A

**Response Payload:**
```json
{
  "name": "hermes-agent",
  "description": "string",
  "version": "string",
  "protocol_version": "string",
  "capabilities": {
    "streaming": true,
    "sessions": true
  },
  "endpoints": {
    "runs": "/runs",
    "tools": "/tools"
  }
}
```

---

## Webhook Platform

> Source: `gateway/platforms/webhook.py`, `tests/gateway/test_webhook_adapter.py`, `tests/gateway/test_webhook_dynamic_routes.py`, `tests/gateway/test_webhook_integration.py`

The webhook platform allows external services to send messages to Hermes via HTTP POST.

---

### 1. Receive Webhook Message

**HTTP Method:** `POST`

**API URL:** `/webhook/{channel_id}` *(channel_id is configured per webhook subscription)*

**Description:** Accepts an inbound webhook payload from an external service and routes the message to the appropriate Hermes agent session for processing.

**Request Payload:**
```json
{
  "text": "string (message content)",
  "user": "string (optional sender identifier)",
  "attachments": ["string (optional URLs or file paths)"],
  "metadata": "object (optional arbitrary key-value pairs)"
}
```

**Response Payload:**
```json
{
  "status": "accepted",
  "run_id": "string"
}
```

---

## Notes & Assumptions

- **Authentication:** The ACP adapter (`acp_adapter/auth.py`) implements authentication/authorization for ACP endpoints. Bearer token or API key authentication is likely required for production use based on the presence of auth and permissions modules.
- **Base URLs:** The Gateway API Server and ACP Adapter run as separate HTTP services. Ports and host binding are configured via environment variables or `cli-config.yaml`.
- **Streaming:** Both servers support Server-Sent Events (SSE) for streaming agent output. The `stream` flag in the Gateway API or `mode: stream` in the ACP API activates this behavior.
- **Session Management:** Sessions persist conversation context across multiple runs. Omitting `session_id` creates a new session; providing one continues an existing conversation.
- **ACP Compliance:** The ACP adapter follows the Agent Communication Protocol specification. The exact field names conform to the ACP spec as documented in `docs/honcho-integration-spec.md` and `acp_registry/agent.json`.

--- module_deep_dive ---


# Hermes Agent — Detailed Component Breakdown

---

## 1. `agent/` — Agent Core

### 1.1 Core Responsibility
The cognitive "brain" of the system. This module is responsible for everything related to constructing, managing, and executing LLM interactions — building prompts, managing conversation context, handling token budgets, routing to appropriate models, tracking costs, and managing the agent's persona/skills.

---

### 1.2 Key Components

| File | Role |
|---|---|
| `prompt_builder.py` | Assembles the full system + user prompt sent to the LLM, injecting persona, skills context, tool definitions, and conversation history |
| `context_compressor.py` | Compresses/summarizes conversation history when approaching context window limits to prevent overflow |
| `context_references.py` | Manages references to external content (files, URLs) embedded within the context window |
| `model_metadata.py` | Stores and retrieves metadata for supported LLM models (context sizes, capabilities, provider info) |
| `usage_pricing.py` | Tracks token usage and computes cost estimates per model/provider |
| `anthropic_adapter.py` | Low-level adapter translating generic agent calls into Anthropic-specific API formats (streaming, tool use, caching) |
| `auxiliary_client.py` | Manages secondary/auxiliary LLM client instances (e.g., for compression, title generation, summarization tasks) |
| `smart_model_routing.py` | Logic for selecting the optimal model based on task type, cost, context requirements |
| `credential_pool.py` | Manages a pool of API credentials, enabling rotation and fallback across multiple API keys |
| `prompt_caching.py` | Implements prompt caching strategies (e.g., Anthropic's cache-control headers) to reduce latency and cost |
| `display.py` | Renders agent output, tool calls, and responses to the terminal UI using Rich formatting |
| `trajectory.py` | Records and manages the agent's action trajectory (sequence of observations, thoughts, tool calls) |
| `skill_commands.py` | Parses and dispatches skill-specific slash commands within the agent loop |
| `skill_utils.py` | Utility functions for loading, validating, and injecting skill content into prompts |
| `redact.py` | Sanitizes sensitive data (credentials, PII) from logs and displayed output |
| `insights.py` | Collects and surfaces runtime insights/metrics about agent behavior |
| `title_generator.py` | Generates human-readable session titles from conversation content using an auxiliary LLM call |
| `models_dev.py` | Developer-facing model definitions and experimental model configurations |
| `copilot_acp_client.py` | Client for communicating with GitHub Copilot via the ACP protocol |

---

### 1.3 Dependencies & Interactions

**Internal Dependencies:**
- `tools/` — Relies on tool definitions and results fed back through the agent loop
- `environments/` — The agent loop in `environments/agent_loop.py` orchestrates calls into `agent/prompt_builder.py`, `agent/context_compressor.py`, etc.
- `hermes_cli/` — CLI layer reads `agent/display.py` for rendering; `hermes_cli/config.py` feeds configuration
- `hermes_constants.py`, `hermes_state.py` — Global state and constants
- `skills/` — `skill_utils.py` and `skill_commands.py` load skill manifests

**External Services:**
- **Anthropic API** (`anthropic_adapter.py`) — Direct integration with Claude models
- **OpenAI API** — Via `auxiliary_client.py` and model routing
- **OpenRouter** — Via `tools/openrouter_client.py` for multi-provider access
- **GitHub Copilot** — Via `copilot_acp_client.py`

---

---

## 2. `environments/` — Execution Environments & Agent Loop

### 2.1 Core Responsibility
Orchestrates the **ReAct-style agent loop** — the core runtime that drives LLM reasoning, tool dispatch, result ingestion, and loop termination. Also provides specialized execution environments (SWE benchmarks, web research) and per-model tool call parsing.

---

### 2.2 Key Components

| File/Directory | Role |
|---|---|
| `agent_loop.py` | **The central execution engine.** Drives the think→act→observe cycle: sends prompts to LLM, receives tool calls, dispatches to tool layer, feeds results back, manages loop termination conditions |
| `hermes_base_env.py` | Base class/configuration for all environment types; defines shared interface |
| `agentic_opd_env.py` | Specialized environment for "one-pass delegation" agentic tasks |
| `web_research_env.py` | Preconfigured environment for autonomous web research tasks |
| `tool_context.py` | Manages the execution context passed to tools (session info, permissions, environment state) |
| `patches.py` | Monkey-patches or compatibility fixes for specific LLM API behaviors |
| `hermes_swe_env/` | Software Engineering benchmark environment (SWE-bench style); contains `hermes_swe_env.py` and `default.yaml` config |
| `terminal_test_env/` | Environment for terminal-based automated testing |
| **`tool_call_parsers/`** | Per-model parsers that extract tool calls from LLM raw output |
| `tool_call_parsers/deepseek_v3_parser.py` | Parses DeepSeek V3 tool call format |
| `tool_call_parsers/deepseek_v3_1_parser.py` | Parses DeepSeek V3.1 tool call format |
| `tool_call_parsers/qwen_parser.py` | Parses Qwen model tool calls |
| `tool_call_parsers/qwen3_coder_parser.py` | Parses Qwen3-Coder specific format |
| `tool_call_parsers/kimi_k2_parser.py` | Parses Kimi K2 tool calls |
| `tool_call_parsers/mistral_parser.py` | Parses Mistral tool call format |
| `tool_call_parsers/llama_parser.py` | Parses Llama model tool calls |
| `tool_call_parsers/glm45_parser.py` / `glm47_parser.py` | Parses GLM-4.5/4.7 tool calls |
| `tool_call_parsers/hermes_parser.py` | Parses Hermes/NousResearch model format |
| `tool_call_parsers/longcat_parser.py` | Parses Longcat model format |
| **`benchmarks/`** | Benchmark harnesses |
| `benchmarks/tblite/` | Terminal-based benchmark suite |
| `benchmarks/yc_bench/` | YC-style task benchmark |
| `benchmarks/terminalbench_2/` | Terminal benchmark v2 |

---

### 2.3 Dependencies & Interactions

**Internal Dependencies:**
- `agent/` — Calls `prompt_builder.py`, `context_compressor.py`, `anthropic_adapter.py`, `display.py`, `usage_pricing.py`
- `tools/` — Dispatches tool calls to the tool registry; receives tool results
- `tools/environments/` — Selects execution backend (local, Docker, SSH, etc.)
- `hermes_state.py`, `hermes_constants.py` — Reads global configuration and state
- `toolsets.py` — Determines which tools are available for a given session

**External Services:**
- **All LLM APIs** (Anthropic, OpenAI, vLLM, llama.cpp, etc.) — `agent_loop.py` is the primary caller

---

---

## 3. `tools/` — Agent Tool Layer

### 3.1 Core Responsibility
Provides all **callable capabilities** available to the agent. Each tool is a discrete, invokable unit of functionality (terminal execution, file I/O, web browsing, memory, delegation, etc.). The registry maps tool names to implementations.

---

### 3.2 Key Components

| File | Role |
|---|---|
| `registry.py` | **Central tool registry** — maps tool names to handler functions; manages tool registration and lookup |
| `terminal_tool.py` | Executes shell commands in the configured execution environment |
| `file_tools.py` | High-level file operations (read, write, list, search) with safety guards |
| `file_operations.py` | Lower-level file I/O primitives |
| `browser_tool.py` | Web browser automation (navigation, clicking, form filling, scraping) |
| `browser_camofox.py` | Camouflaged Firefox browser instance for stealthy browsing |
| `browser_camofox_state.py` | Persists browser session state (cookies, tabs, history) |
| `code_execution_tool.py` | Executes code snippets in sandboxed environments |
| `mcp_tool.py` | **Model Context Protocol** integration — connects to MCP servers and exposes their tools |
| `mcp_oauth.py` | OAuth flow handling for MCP server authentication |
| `delegate_tool.py` | Spawns sub-agents to handle delegated subtasks (multi-agent coordination) |
| `memory_tool.py` | Persistent memory read/write operations (long-term agent memory) |
| `honcho_tools.py` | Integration with Honcho memory service for session-aware memory |
| `skill_manager_tool.py` | Installs, removes, updates skills during agent runtime |
| `skills_tool.py` | Exposes skill content (SKILL.md) to the agent as context |
| `skills_guard.py` | Access control and safety validation for skill operations |
| `skills_hub.py` | Hub/marketplace interface for discovering and fetching skills |
| `skills_sync.py` | Synchronizes local skills with remote skill repositories |
| `web_tools.py` | Web search (Tavily, DuckDuckGo) and HTTP fetch operations |
| `vision_tools.py` | Image analysis and vision-language model invocations |
| `voice_mode.py` | Voice interaction mode (STT input + TTS output) |
| `tts_tool.py` | Text-to-speech synthesis |
| `transcription_tools.py` | Speech-to-text transcription |
| `neutts_synth.py` | Neural TTS synthesis backend |
| `image_generation_tool.py` | Image generation (DALL-E, Stable Diffusion, etc.) |
| `send_message_tool.py` | Sends messages to users via connected platforms |
| `cronjob_tools.py` | Creates and manages cron jobs from within the agent |
| `todo_tool.py` | Manages a persistent to-do list for the agent |
| `session_search_tool.py` | Searches across previous agent sessions |
| `checkpoint_manager.py` | Saves and restores execution checkpoints for long-running tasks |
| `mixture_of_agents_tool.py` | Orchestrates mixture-of-agents inference patterns |
| `rl_training_tool.py` | Triggers RL training runs |
| `homeassistant_tool.py` | Controls Home Assistant smart home devices |
| `clarify_tool.py` | Asks clarifying questions to the user |
| `approval.py` | Prompts user for approval before executing sensitive actions |
| `interrupt.py` | Handles graceful interruption of tool execution |
| `process_registry.py` | Tracks running background processes |
| `credential_files.py` | Reads credential files from the filesystem |
| `env_passthrough.py` | Passes environment variables into tool execution contexts |
| `patch_parser.py` | Parses unified diff patches for file modification tools |
| `fuzzy_match.py` | Fuzzy string matching utilities for file/command lookup |
| `ansi_strip.py` | Strips ANSI escape codes from terminal output |
| `url_safety.py` | Validates URLs against safety/SSRF policies |
| `website_policy.py` | Enforces browsing policies (allowlists, blocklists) |
| `tirith_security.py` | Security policy enforcement layer (Tirith integration) |
| `openrouter_client.py` | OpenRouter API client for multi-provider LLM access |
| `debug_helpers.py` | Debugging utilities for tool development |
| **`environments/`** | Execution backend implementations |
| `environments/base.py` | Abstract base class for execution environments |
| `environments/local.py` | Local process execution |
| `environments/docker.py` | Docker container execution |
| `environments/ssh.py` | Remote SSH execution |
| `environments/modal.py` | Modal cloud execution |
| `environments/daytona.py` | Daytona cloud development environment |
| `environments/singularity.py` | Singularity/Apptainer container execution |
| `environments/persistent_shell.py` | Maintains a persistent shell session across tool calls |
| **`browser_providers/`** | Browser automation backends |
| `browser_providers/base.py` | Abstract browser provider interface |
| `browser_providers/browser_use.py` | `browser-use` library integration |
| `browser_providers/browserbase.py` | Browserbase cloud browser integration |

---

### 3.3 Dependencies & Interactions

**Internal Dependencies:**
- `environments/` — `agent_loop.py` dispatches tool calls; `tool_context.py` provides execution context
- `agent/` — `display.py` renders tool results; `credential_pool.py` supplies API keys
- `hermes_cli/config.py` — Tool configuration and enablement flags
- `toolsets.py` / `toolset_distributions.py` — Defines which tool bundles are active
- `honcho_integration/` — `honcho_tools.py` delegates to the Honcho client
- `cron/` — `cronjob_tools.py` interfaces with the cron scheduler
- `gateway/` — `send_message_tool.py` sends messages back through platform adapters

**External Services:**
- **Browser automation:** Playwright, `browser-use`, Browserbase
- **Web search:** Tavily API, DuckDuckGo
- **Cloud execution:** Modal, Daytona
- **MCP servers** — arbitrary external MCP-compliant services
- **Home Assistant** — Smart home REST API
- **OpenRouter API** — Multi-provider LLM routing
- **Docker daemon** — Container management
- **SSH targets** — Remote execution

---

---

## 4. `gateway/` — Multi-Platform Messaging Gateway

### 4.1 Core Responsibility
Acts as the **event-driven messaging hub** — receives inbound messages from external platforms (Telegram, Discord, Slack, etc.), manages per-user/channel agent sessions, routes messages through the agent loop, and delivers responses back to the appropriate platform.

---

### 4.2 Key Components

| File | Role |
|---|---|
| `run.py` | **Gateway service entry point** — starts the platform adapters, initializes session management, begins event processing |
| `session.py` | **Session manager** — creates, retrieves, and persists per-user/channel agent sessions; manages session lifecycle |
| `config.py` | Gateway configuration loading (platform tokens, allowlists, feature flags) |
| `delivery.py` | Handles message delivery back to platforms — chunking, formatting, retry logic |
| `stream_consumer.py` | Consumes streaming LLM output and forwards chunks to delivery layer |
| `channel_directory.py` | Maps platform channel/user identifiers to internal session IDs |
| `hooks.py` | **Lifecycle hook system** — allows custom code to intercept gateway events (message received, session started, etc.) |
| `mirror.py` | Mirrors messages between platforms or to logging destinations |
| `pairing.py` | Handles device/user pairing flows (linking CLI sessions to gateway sessions) |
| `status.py` | Reports gateway health and per-session status |
| `sticker_cache.py` | Caches platform-specific sticker/emoji assets |
| **`platforms/`** | Per-platform adapter implementations |
| `platforms/base.py` | Abstract base class defining the platform adapter interface |
| `platforms/telegram.py` | Telegram bot adapter (python-telegram-bot) |
| `platforms/telegram_network.py` | Telegram network-level connection management and reconnection |
| `platforms/discord.py` | Discord bot adapter (discord.py) |
| `platforms/slack.py` | Slack bot adapter (slack-sdk) |
| `platforms/matrix.py` | Matrix protocol adapter (matrix-nio) |
| `platforms/signal.py` | Signal messenger adapter |
| `platforms/whatsapp.py` | WhatsApp adapter (bridges to Node.js `whatsapp-web.js` bridge) |
| `platforms/sms.py` | SMS adapter (Twilio) |
| `platforms/email.py` | Email adapter (SMTP/IMAP) |
| `platforms/webhook.py` | Generic inbound/outbound webhook adapter |
| `platforms/api_server.py` | **REST/SSE API server** (FastAPI) for direct HTTP-based agent interaction |
| `platforms/homeassistant.py` | Home Assistant integration adapter |
| `platforms/mattermost.py` | Mattermost adapter |
| `platforms/dingtalk.py` | DingTalk (Alibaba) adapter |
| `platforms/feishu.py` | Feishu/Lark adapter |
| `platforms/wecom.py` | WeCom (WeChat Work) adapter |
| **`builtin_hooks/`** | Built-in gateway lifecycle hooks |
| `builtin_hooks/boot_md.py` | Sends a boot/startup message on gateway initialization |

---

### 4.3 Dependencies & Interactions

**Internal Dependencies:**
- `environments/agent_loop.py` — The gateway session creates and drives agent loop instances
- `agent/` — `display.py`, `prompt_builder.py`, `usage_pricing.py` consumed per session
- `tools/send_message_tool.py` — Agent can proactively send messages via this tool
- `honcho_integration/` — Session memory persistence via Honcho
- `cron/` — Scheduled tasks may trigger gateway-delivered messages
- `hermes_cli/config.py` — Shared configuration layer
- `hermes_state.py` — Global state access

**External Services:**
- **Telegram API** — `python-telegram-bot`
- **Discord API** — `discord.py`
- **Slack API** — `slack-sdk`
- **Matrix homeserver** — `matrix-nio`
- **Signal** — via Signal bridge
- **WhatsApp** — via Node.js `scripts/whatsapp-bridge/bridge.js`
- **Twilio** — SMS sending/receiving
- **SMTP/IMAP** — Email
- **Home Assistant** — REST API
- **DingTalk / Feishu / WeCom** — Enterprise messaging APIs

---

---

## 5. `hermes_cli/` — CLI Application Layer

### 5.1 Core Responsibility
The **user-facing command-line interface** — the primary interaction surface for human users. Manages the interactive REPL session, configuration management, setup wizard, authentication, model selection, skill management, and all user-visible CLI commands.

---

### 5.2 Key Components

| File | Role |
|---|---|
| `main.py` | **CLI entry point** — Typer/Click app root; registers all sub-commands, handles startup |
| `commands.py` | Core chat and task execution commands (e.g., `hermes "do task"`, interactive REPL) |
| `config.py` | Configuration file read/write (YAML-based `cli-config.yaml`), defaults, validation |
| `setup.py` | **Interactive setup wizard** — guides first-time users through model/provider selection, API key entry |
| `auth.py` | API key and credential management (storage, retrieval, validation) |
| `auth_commands.py` | CLI sub-commands for authentication management (`hermes auth ...`) |
| `copilot_auth.py` | GitHub Copilot-specific OAuth authentication flow |
| `models.py` | Model listing, selection, and validation commands |
| `model_switch.py` | Handles dynamic model switching during a session |
| `codex_models.py` | OpenAI Codex-specific model definitions and configuration |
| `skills_config.py` | Skills configuration management (enable/disable, configure) |
| `skills_hub.py` | CLI interface to the skill marketplace (browse, install, update) |
| `plugins.py` | Plugin system integration and management |
| `plugins_cmd.py` | CLI sub-commands for plugin management |
| `skin_engine.py` | **Skin/theme system** — loads YAML skin definitions to customize agent persona and UI appearance |
| `curses_ui.py` | TUI (Terminal UI) using curses for rich interactive display |
| `banner.py` | Renders the startup banner (ASCII art + version info) |
| `colors.py` | Color theme definitions and terminal color utilities |
| `callbacks.py` | Typer callback handlers for global CLI options |
| `cron.py` | CLI commands for managing cron jobs (`hermes cron ...`) |
| `gateway.py` | CLI commands for gateway management (`hermes gateway start/stop/status`) |
| `doctor.py` | **Diagnostic tool** — checks environment, dependencies, credentials, and configuration health |
| `status.py` | Displays current session/system status |
| `checklist.py` | Pre-run checklist validation (required config, credentials, etc.) |
| `env_loader.py` | Loads and validates environment variables for the CLI session |
| `runtime_provider.py` | Resolves the runtime execution provider (local, Docker, SSH, etc.) |
| `mcp_config.py` | MCP server configuration management |
| `profiles.py` | Named configuration profiles (e.g., `work`, `personal`) |
| `pairing.py` | CLI commands for pairing with gateway sessions |
| `webhook.py` | CLI commands for webhook configuration |
| `clipboard.py` | Clipboard read/write utilities |
| `claw.py` | OpenClaw legacy compatibility and migration support |
| `uninstall.py` | Clean uninstallation utility |
| `default_soul.py` | Default agent personality/soul definition |
| `tools_config.py` | CLI interface for configuring individual tools |

---

### 5.3 Dependencies & Interactions

**Internal Dependencies:**
- `agent/` — Invokes agent core for LLM interactions
- `environments/agent_loop.py` — Starts and manages the agent loop per session
- `tools/` — Configures and invokes tools based on CLI flags
- `hermes_cli/config.py` — Central config consumed across all CLI modules
- `hermes_constants.py`, `hermes_state.py` — Global constants and state
- `honcho_integration/` — Memory and session management
- `cron/` — Scheduler access from CLI cron commands
- `gateway/` — Gateway start/stop control from CLI

**External Services:**
- **Anthropic, OpenAI, GitHub Copilot APIs** — Credential validation during setup
- **Skill marketplace** (remote git/HTTP indexes) — For skill installation
- **System clipboard** — Via `clipboard.py`
- **OS process management** — For gateway start/stop

---

---

## 6. `acp_adapter/` — Agent Communication Protocol Server

### 6.1 Core Responsibility
Exposes the Hermes agent as a **standardized ACP (Agent Communication Protocol) compliant server**, enabling external systems, orchestrators, and other agents to interact with Hermes through a well-defined protocol over HTTP/SSE.

---

### 6.2 Key Components

| File | Role |
|---|---|
| `__main__.py` | **Server entry point** — starts the ACP adapter as a standalone service |
| `server.py` | **FastAPI application** — defines HTTP routes and SSE endpoints for the ACP protocol |
| `entry.py` | Main request handler — receives ACP-formatted requests and routes to agent sessions |
| `session.py` | A

--- hl_overview ---


# Repository Analysis: hermes-agent

## 0. Repository Name
[[hermes-agent]]

---

## 1. Project Purpose

Hermes Agent is an **AI-powered autonomous agent framework** designed to:
- Provide a conversational AI assistant accessible via CLI, messaging platforms (Discord, Telegram, Slack, WhatsApp, Signal, Matrix, etc.), and APIs
- Execute agentic tasks using tool-calling capabilities (terminal, file operations, browser, code execution, etc.)
- Support multi-platform deployment as a persistent "gateway" service
- Enable extensibility through a **skills system** (pluggable capability bundles)
- Support multiple LLM backends (Anthropic Claude, OpenAI, Codex, local models via vLLM/llama.cpp, etc.)

**Primary Domain:** Autonomous AI Agent / LLM-powered assistant with multi-platform messaging integration.

---

## 2. Architecture Pattern

- **Plugin/Skill-Based Extensible Agent Architecture**
- **Event-Driven Gateway Pattern** (message platforms → gateway → agent loop)
- **Tool-Calling Agent Loop** (ReAct-style reasoning with tool invocations)
- **Layered Architecture** (CLI → Agent Core → Tools → Environments → LLM Backends)

---

## 3. Technology Stack

### Primary Language
- **Python** (primary), **JavaScript/TypeScript** (website, WhatsApp bridge, landing page)

### From `pyproject.toml` / `requirements.txt`

#### Core LLM & AI
| Package | Purpose |
|---|---|
| `anthropic` | Anthropic Claude API client |
| `openai` | OpenAI API client |
| `tiktoken` | Token counting for OpenAI models |
| `litellm` (implied) | Multi-provider LLM routing |

#### CLI & UI
| Package | Purpose |
|---|---|
| `typer` / `click` | CLI framework |
| `rich` | Terminal rich text rendering |
| `prompt_toolkit` | Interactive CLI input |
| `curses` (stdlib) | TUI/curses UI |

#### Web & HTTP
| Package | Purpose |
|---|---|
| `httpx` | Async HTTP client |
| `aiohttp` | Async HTTP server/client |
| `fastapi` | API server (gateway/ACP adapter) |
| `uvicorn` | ASGI server |
| `websockets` | WebSocket support |

#### Messaging Platform SDKs
| Package | Purpose |
|---|---|
| `python-telegram-bot` | Telegram integration |
| `discord.py` | Discord integration |
| `slack-sdk` | Slack integration |
| `matrix-nio` | Matrix protocol |
| `twilio` (implied) | SMS integration |

#### Data & Storage
| Package | Purpose |
|---|---|
| `pydantic` | Data validation/models |
| `sqlalchemy` (possible) | ORM |
| `aiosqlite` / `sqlite3` | Local session storage |
| `PyYAML` | YAML config parsing |

#### Tools & Execution
| Package | Purpose |
|---|---|
| `playwright` / `selenium` | Browser automation |
| `docker` SDK | Docker environment management |
| `paramiko` | SSH environment |
| `mcp` | Model Context Protocol support |

#### Testing
| Package | Purpose |
|---|---|
| `pytest` | Test framework |
| `pytest-asyncio` | Async test support |
| `pytest-mock` | Mocking |

#### JavaScript (Website/Bridge)
- **Docusaurus** (documentation site)
- **Node.js** (WhatsApp bridge via `whatsapp-web.js`)

#### Infrastructure
- **Docker** (containerization)
- **Nix/NixOS** (reproducible builds, `flake.nix`)
- **GitHub Actions** (CI/CD)

---

## 4. Initial Structure Impression

| Component | Description |
|---|---|
| **CLI Frontend** | `cli.py`, `hermes_cli/` — user-facing terminal interface |
| **Agent Core** | `agent/`, `environments/` — LLM agent loop, prompt building, context management |
| **Tool Layer** | `tools/` — all agent-callable tools |
| **Gateway Service** | `gateway/` — multi-platform messaging server |
| **Skills System** | `skills/`, `optional-skills/` — pluggable capability packs |
| **ACP Adapter** | `acp_adapter/` — Agent Communication Protocol server |
| **Honcho Integration** | `honcho_integration/` — memory/session management service |
| **Cron System** | `cron/` — scheduled task execution |
| **Website/Docs** | `website/`, `landingpage/` — documentation and marketing |
| **Tests** | `tests/` — comprehensive test suite |

---

## 5. Configuration / Package Files

| File | Purpose |
|---|---|
| `pyproject.toml` | Python project metadata, dependencies, build config |
| `requirements.txt` | Python dependencies (pip) |
| `uv.lock` | Locked dependencies (uv package manager) |
| `package.json` | Root Node.js config (WhatsApp bridge / tooling) |
| `package-lock.json` | Locked Node.js dependencies |
| `website/package.json` | Docusaurus website dependencies |
| `scripts/whatsapp-bridge/package.json` | WhatsApp bridge Node dependencies |
| `flake.nix` | Nix flake for reproducible dev environment |
| `flake.lock` | Locked Nix dependencies |
| `.env.example` | Environment variable template |
| `.envrc` | direnv environment activation |
| `cli-config.yaml.example` | CLI configuration template |
| `MANIFEST.in` | Python package manifest |
| `.dockerignore` | Docker build exclusions |
| `Dockerfile` | Container build definition |
| `.gitmodules` | Git submodules config |
| `nix/checks.nix` | Nix CI checks |
| `nix/devShell.nix` | Nix development shell |
| `nix/packages.nix` | Nix package definitions |
| `nix/nixosModules.nix` | NixOS module definitions |
| `nix/python.nix` | Python environment for Nix |
| `.github/workflows/*.yml` | CI/CD pipeline definitions |
| `acp_registry/agent.json` | ACP agent registration metadata |

---

## 6. Directory Structure

```
hermes-agent/
├── agent/              # Core agent logic: prompt building, context compression,
│                       # model metadata, display, usage/pricing, trajectory mgmt
├── environments/       # Execution environments: agent loop, SWE env,
│                       # tool call parsers (per-model), benchmarks
├── tools/              # All agent tools: terminal, file ops, browser, MCP,
│                       # memory, delegation, skills, vision, voice, web search, etc.
│   ├── environments/   # Execution backends: local, Docker, SSH, Modal, Daytona
│   └── browser_providers/ # Browser automation backends
├── gateway/            # Multi-platform messaging gateway service
│   ├── platforms/      # Per-platform adapters (Telegram, Discord, Slack, etc.)
│   └── builtin_hooks/  # Built-in gateway lifecycle hooks
├── hermes_cli/         # CLI application layer: commands, config, setup,
│                       # auth, models, skills management, skin engine
├── acp_adapter/        # Agent Communication Protocol (ACP) server adapter
├── honcho_integration/ # Honcho memory/session management integration
├── cron/               # Cron job scheduling system
├── skills/             # Built-in skills (capability packs with SKILL.md + scripts)
│   ├── github/         # GitHub workflow skills
│   ├── productivity/   # Notion, Google Workspace, PowerPoint, etc.
│   ├── mlops/          # ML training, inference, evaluation skills
│   ├── research/       # Research and information gathering skills
│   ├── software-development/ # Dev workflow skills
│   └── index-cache/    # Cached skill marketplace indexes
├── optional-skills/    # Optional/advanced skills (mlops, blockchain, security, etc.)
├── tests/              # Full test suite mirroring source structure
│   ├── tools/          # Tool unit tests
│   ├── agent/          # Agent core tests
│   ├── hermes_cli/     # CLI tests
│   ├── gateway/        # Gateway platform tests
│   ├── acp/            # ACP adapter tests
│   ├── honcho_integration/ # Honcho tests
│   ├── cron/           # Cron scheduler tests
│   └── integration/    # End-to-end integration tests
├── docs/               # Technical documentation, specs, migration guides
├── website/            # Docusaurus documentation website
├── landingpage/        # Static HTML marketing landing page
├── scripts/            # Utility scripts (install, release, WhatsApp bridge)
├── docker/             # Docker entrypoint and SOUL.md (agent personality)
├── nix/                # Nix build system files
├── assets/             # Static assets (banner image)
└── datagen-config-examples/ # Data generation configuration examples
```

---

## 7. High-Level Architecture

### Architecture Patterns

#### 1. **Tool-Calling Agent Loop (ReAct Pattern)**
```
User Input → Prompt Builder → LLM API → Tool Call Parser → Tool Execution → Loop
```
Evidence: `environments/agent_loop.py`, `tools/`, `environments/tool_call_parsers/`

#### 2. **Event-Driven Gateway**
```
Platform Event → Gateway Platform Adapter → Session Manager → Agent Loop → Response Delivery
```
Evidence: `gateway/platforms/`, `gateway/session.py`, `gateway/delivery.py`, `gateway/stream_consumer.py`

#### 3. **Plugin/Skill Architecture**
Skills are self-contained directories with `SKILL.md` manifests, scripts, references, and templates — dynamically loaded and managed.
Evidence: `skills/`, `optional-skills/`, `tools/skill_manager_tool.py`, `tools/skills_tool.py`

#### 4. **Multi-Backend Model Abstraction**
Per-model tool call parsers (`environments/tool_call_parsers/`) allow the same agent loop to work with Anthropic, OpenAI, Qwen, DeepSeek, Kimi, GLM, Mistral, Llama, etc.

#### 5. **ACP (Agent Communication Protocol) Server**
`acp_adapter/` exposes the agent as a standardized ACP-compliant service for external integration.

#### 6. **Layered Architecture**
```
Layer 1: User Interfaces    (CLI, Gateway Platforms, ACP API)
Layer 2: Agent Orchestration (agent/, environments/)
Layer 3: Tool Execution     (tools/, tools/environments/)
Layer 4: LLM Backends       (Anthropic, OpenAI, local models)
Layer 5: Persistence        (sessions, memory, Honcho, cron)
```

---

## 8. Build, Execution and Test

### Installation
```bash
# Standard Python install
pip install -e .

# Using uv (preferred)
uv sync

# Shell script installer
./setup-hermes.sh
# or
./scripts/install.sh

# Nix (reproducible)
nix develop
```

### Main Entry Points

| Entry Point | Purpose |
|---|---|
| `cli.py` | Main CLI entry point |
| `hermes` | Shell wrapper script for CLI |
| `hermes_cli/main.py` | CLI application main |
| `run_agent.py` | Programmatic agent runner |
| `gateway/run.py` | Gateway service runner |
| `mcp_serve.py` | MCP server entry point |
| `acp_adapter/__main__.py` | ACP adapter server |
| `batch_runner.py` | Batch task runner |
| `mini_swe_runner.py` | SWE benchmark runner |
| `rl_cli.py` | RL training CLI |
| `docker/entrypoint.sh` | Docker container entrypoint |

### Running
```bash
# CLI mode
hermes                    # interactive chat
hermes "do a task"        # one-shot task

# Gateway service (messaging platforms)
hermes gateway start

# MCP server
python mcp_serve.py

# ACP adapter
python -m acp_adapter
```

### Testing
```bash
# Run all tests
pytest

# Run specific test categories
pytest tests/tools/
pytest tests/gateway/
pytest tests/integration/

# CI via GitHub Actions
# .github/workflows/tests.yml
```

### Docker
```bash
docker build -t hermes-agent .
docker run hermes-agent
```

### CI/CD (GitHub Actions)
| Workflow | Purpose |
|---|---|
| `tests.yml` | Run test suite |
| `docker-publish.yml` | Build and push Docker image |
| `deploy-site.yml` | Deploy documentation website |
| `supply-chain-audit.yml` | Security dependency audit |
| `nix.yml` | Nix build verification |

--- dependencies ---


# Dependency and Architecture Analysis: hermes-agent

---

## Internal Modules

The following are the primary internal modules developed as part of the hermes-agent project, reused across different components:

### `agent/`
**Core Agent Orchestration**
Handles the central intelligence layer: prompt building, context compression, model metadata, usage/pricing tracking, credential pooling, display rendering, trajectory management, and multi-model routing. Key files include `prompt_builder.py`, `context_compressor.py`, `model_metadata.py`, `credential_pool.py`, and `smart_model_routing.py`.

### `environments/`
**Execution Environment & Agent Loop**
Implements the ReAct-style tool-calling agent loop (`agent_loop.py`), per-model tool call parsers (for Anthropic, OpenAI, DeepSeek, Qwen, Mistral, Llama, Kimi, GLM, etc.), SWE benchmark environments, and base environment abstractions.

### `tools/`
**Agent Tool Layer**
All agent-callable tools: terminal execution, file operations, browser automation, MCP integration, memory, delegation, skill management, voice/TTS, vision, web search, image generation, Home Assistant, cron jobs, code execution, and more. Also contains sub-modules for execution backends (`tools/environments/`) and browser providers (`tools/browser_providers/`).

### `gateway/`
**Multi-Platform Messaging Gateway**
Event-driven gateway service that bridges messaging platforms to the agent loop. Contains per-platform adapters (`platforms/`) for Telegram, Discord, Slack, Matrix, Signal, WhatsApp, Mattermost, DingTalk, Feishu, WeCom, SMS, email, and webhooks. Manages sessions, delivery, stream consumption, pairing, and lifecycle hooks.

### `hermes_cli/`
**CLI Application Layer**
User-facing terminal interface: commands, configuration management, authentication, model/provider selection, skills management, gateway management, setup wizard, skin engine, status display, MCP config, profiles, and plugin management.

### `acp_adapter/`
**Agent Communication Protocol (ACP) Server**
Exposes the agent as a standardized ACP-compliant service for external integration. Handles authentication, permissions, event streaming, session management, and tool proxying.

### `honcho_integration/`
**Honcho Memory/Session Integration**
Integrates with the Honcho AI memory service for persistent session and memory management. Provides client, session, and CLI interfaces.

### `cron/`
**Cron Job Scheduler**
Implements scheduled task execution for the agent: job definitions (`jobs.py`) and scheduler logic (`scheduler.py`).

### `skills/` and `optional-skills/`
**Skills System (Pluggable Capability Packs)**
Self-contained skill directories with `SKILL.md` manifests, scripts, references, and templates. Organized into categories: productivity, GitHub, research, MLOps, creative, gaming, software development, media, and more. Optional skills extend with blockchain, security, advanced MLOps, and other specialized capabilities.

### Root-Level Utility Modules
**Shared State, Constants, and Utilities**
- `hermes_constants.py`: Project-wide constants and path helpers
- `hermes_state.py`: Global runtime state management
- `hermes_time.py`: Timezone-aware time utilities
- `toolsets.py` / `toolset_distributions.py`: Toolset grouping and distribution logic
- `model_tools.py`: Model-level tool utilities
- `trajectory_compressor.py`: Standalone trajectory compression logic
- `utils.py`: General-purpose utilities

---

## External Dependencies

### Python Dependencies

| Official Name | Role / Purpose | Source |
|---|---|---|
| **OpenAI** | OpenAI API client for LLM inference (GPT models, Codex, etc.) | `/pyproject.toml`, `/requirements.txt` |
| **Anthropic** | Anthropic Claude API client for LLM inference | `/pyproject.toml` |
| **python-dotenv** | Loads environment variables from `.env` files | `/pyproject.toml`, `/requirements.txt` |
| **Fire** | Python CLI framework for generating CLIs from Python objects | `/pyproject.toml`, `/requirements.txt` |
| **HTTPX** | Async/sync HTTP client used for API requests | `/pyproject.toml`, `/requirements.txt` |
| **Rich** | Terminal rich text and formatting library | `/pyproject.toml`, `/requirements.txt` |
| **Tenacity** | Retry logic for resilient API calls | `/pyproject.toml`, `/requirements.txt` |
| **PyYAML** | YAML configuration file parsing | `/pyproject.toml`, `/requirements.txt` |
| **Requests** | HTTP client library | `/pyproject.toml`, `/requirements.txt` |
| **Jinja2** | Templating engine for prompt and config templates | `/pyproject.toml`, `/requirements.txt` |
| **Pydantic** | Data validation and settings management | `/pyproject.toml`, `/requirements.txt` |
| **prompt_toolkit** | Interactive CLI input with completion and history | `/pyproject.toml`, `/requirements.txt` |
| **Exa** (`exa-py`) | Exa web search API client for web research tool | `/pyproject.toml` |
| **Firecrawl** (`firecrawl-py`) | Web scraping and crawling API client | `/pyproject.toml`, `/requirements.txt` |
| **Parallel Web** (`parallel-web`) | Parallel web search API client | `/pyproject.toml`, `/requirements.txt` |
| **fal-client** | FAL AI image generation API client | `/pyproject.toml`, `/requirements.txt` |
| **Edge TTS** (`edge-tts`) | Free cloud-based text-to-speech via Microsoft Edge | `/pyproject.toml`, `/requirements.txt` |
| **PyJWT** | JSON Web Token library for GitHub App authentication (Skills Hub) | `/pyproject.toml`, `/requirements.txt` |
| **Modal** | Serverless cloud sandbox SDK for terminal execution backend | `/pyproject.toml` |
| **Daytona** | Persistent cloud development environment SDK | `/pyproject.toml` |
| **pytest** | Test framework | `/pyproject.toml` |
| **pytest-asyncio** | Async test support for pytest | `/pyproject.toml` |
| **pytest-xdist** | Parallel test execution for pytest | `/pyproject.toml` |
| **MCP** (`mcp`) | Model Context Protocol SDK for MCP tool integration | `/pyproject.toml` |
| **python-telegram-bot** | Telegram bot API client for gateway messaging | `/pyproject.toml`, `/requirements.txt` |
| **discord.py** | Discord bot API client with voice support for gateway | `/pyproject.toml`, `/requirements.txt` |
| **aiohttp** | Async HTTP server/client for gateway platforms (SMS, Home Assistant) | `/pyproject.toml`, `/requirements.txt` |
| **slack-bolt** | Slack bot framework (Socket Mode) for gateway | `/pyproject.toml` |
| **slack-sdk** | Slack API SDK | `/pyproject.toml` |
| **Croniter** | Cron expression parsing for scheduled task evaluation | `/pyproject.toml`, `/requirements.txt` |
| **matrix-nio** | Matrix protocol client with optional E2EE support | `/pyproject.toml` |
| **simple-term-menu** | Terminal selection menu for interactive CLI | `/pyproject.toml` |
| **ElevenLabs** (`elevenlabs`) | Premium text-to-speech API client | `/pyproject.toml` |
| **faster-whisper** | Local speech-to-text transcription (CTranslate2-based Whisper) | `/pyproject.toml` |
| **sounddevice** | Audio input/output for voice mode | `/pyproject.toml` |
| **NumPy** | Numerical array operations (used with audio/voice processing) | `/pyproject.toml` |
| **ptyprocess** | Unix pseudo-terminal process management for terminal tool | `/pyproject.toml` |
| **pywinpty** | Windows pseudo-terminal support for terminal tool | `/pyproject.toml` |
| **honcho-ai** | Honcho AI memory/session management service client | `/pyproject.toml` |
| **agent-client-protocol** | ACP (Agent Communication Protocol) SDK | `/pyproject.toml` |
| **dingtalk-stream** | DingTalk messaging platform SDK for gateway | `/pyproject.toml` |
| **lark-oapi** (`lark-oapi`) | Feishu/Lark messaging platform SDK for gateway | `/pyproject.toml` |
| **FastAPI** | ASGI web framework for RL/ACP API server | `/pyproject.toml` |
| **Uvicorn** | ASGI server for FastAPI | `/pyproject.toml` |
| **Weights & Biases** (`wandb`) | ML experiment tracking for RL training tool | `/pyproject.toml` |
| **atroposlib** | Atropos RL training library (from NousResearch/atropos git) | `/pyproject.toml` |
| **tinker** | RL training SDK (from thinking-machines-lab/tinker git) | `/pyproject.toml` |
| **yc-bench** | YC benchmark suite (from collinear-ai/yc-bench git) | `/pyproject.toml` |
| **google-auth-oauthlib** | Google OAuth2 flow for Google Workspace skill authentication | `/skills/productivity/google-workspace/scripts/setup.py` |

---

### JavaScript Dependencies

| Official Name | Role / Purpose | Source |
|---|---|---|
| **agent-browser** | Headless browser automation tool (Node.js binary) used by browser tool | `/package.json` |
| **Camoufox Browser** (`@askjo/camoufox-browser`) | Anti-detection browser backend for stealth web automation | `/package.json` |
| **Baileys** (`@whiskeysockets/baileys`) | WhatsApp Web API library for the WhatsApp bridge | `/scripts/whatsapp-bridge/package.json` |
| **Express** | Node.js HTTP server framework for the WhatsApp bridge API | `/scripts/whatsapp-bridge/package.json` |
| **qrcode-terminal** | QR code rendering in terminal for WhatsApp pairing | `/scripts/whatsapp-bridge/package.json` |
| **Pino** | Fast structured JSON logger for the WhatsApp bridge | `/scripts/whatsapp-bridge/package.json` |
| **Docusaurus** (`@docusaurus/core`, `@docusaurus/preset-classic`, `@docusaurus/theme-mermaid`) | Documentation website framework | `/website/package.json` |
| **Docusaurus Search Local** (`@easyops-cn/docusaurus-search-local`) | Local search plugin for the Docusaurus documentation site | `/website/package.json` |
| **MDX React** (`@mdx-js/react`) | MDX (Markdown + JSX) rendering for Docusaurus | `/website/package.json` |
| **clsx** | Utility for conditionally joining CSS class names | `/website/package.json` |
| **prism-react-renderer** | Syntax highlighting for code blocks in Docusaurus | `/website/package.json` |
| **React** | UI framework powering the Docusaurus documentation site | `/website/package.json` |
| **React DOM** | React DOM renderer for the documentation site | `/website/package.json` |
| **TypeScript** | Typed JavaScript superset for the documentation site build | `/website/package.json` (devDependencies) |

---

### System / Infrastructure Dependencies

| Official Name | Role / Purpose | Source |
|---|---|---|
| **Debian** | Base Docker container OS (`debian:13.4`) | `/Dockerfile` |
| **Node.js** | Runtime for WhatsApp bridge and agent-browser tool | `/Dockerfile` |
| **ripgrep** | Fast file search utility available to terminal tool | `/Dockerfile` |
| **FFmpeg** | Audio/video processing for voice and media features | `/Dockerfile` |
| **Playwright** (Chromium) | Headless browser automation engine installed via `npx playwright install` | `/Dockerfile` |

--- core_entities ---


# Hermes Agent — Common Data Entities & Domain Models

## Overview

Based on the repository structure and file analysis, the Hermes Agent project is an **AI agent framework** with multi-platform messaging gateway capabilities, skill management, session handling, and tool orchestration. The following entities are central to its domain.

---

## 1. Core Data Entities

---

### 1.1 `Session`

Represents a conversation or agent interaction context — the central unit of execution.

| Attribute | Description |
|---|---|
| `session_id` | Unique identifier for the session |
| `messages` / `trajectory` | Ordered list of messages/tool interactions |
| `platform` | Originating platform (Discord, Telegram, Slack, etc.) |
| `user_id` | Associated user/owner |
| `created_at` / `updated_at` | Timestamps |
| `state` | Current session state (active, paused, completed) |
| `topic` | Topic or channel the session belongs to |
| `config` / `env` | Session-level environment and configuration overrides |
| `title` | Auto-generated or assigned session title |
| `metadata` | Arbitrary session metadata |

**References:** `gateway/session.py`, `acp_adapter/session.py`, `honcho_integration/session.py`, `hermes_state.py`, `tests/gateway/test_session*.py`

---

### 1.2 `Message`

Represents a single unit of communication within a session, from either a user, agent, or tool.

| Attribute | Description |
|---|---|
| `message_id` | Unique identifier |
| `session_id` | Parent session reference |
| `role` | `user`, `assistant`, `tool`, `system` |
| `content` | Text, structured content, or multimodal payload |
| `timestamp` | Time of creation |
| `platform_message_id` | External platform-specific ID |
| `media` / `attachments` | Images, files, audio, stickers |
| `tool_calls` | Associated tool invocations |
| `tool_results` | Responses from tool executions |

**References:** `environments/agent_loop.py`, `agent/trajectory.py`, `gateway/platforms/base.py`

---

### 1.3 `Trajectory`

A recorded sequence of agent steps (messages + tool calls) within a session, used for context management, compression, and replay.

| Attribute | Description |
|---|---|
| `trajectory_id` | Unique identifier |
| `session_id` | Owning session |
| `steps` | Ordered list of `Message` entries including tool interactions |
| `token_count` | Running token usage |
| `compressed_summary` | Optional compressed representation |
| `checkpoints` | Saved intermediate states |

**References:** `agent/trajectory.py`, `trajectory_compressor.py`, `agent/context_compressor.py`, `tools/checkpoint_manager.py`

---

### 1.4 `Skill`

A capability package that can be loaded, installed, and executed by the agent. Skills are modular instruction sets or tool bundles.

| Attribute | Description |
|---|---|
| `skill_id` / `name` | Unique skill identifier/name |
| `description` | What the skill enables |
| `category` | e.g., `productivity`, `github`, `research`, `mlops` |
| `version` | Skill version |
| `templates` | Prompt or workflow templates |
| `references` | Reference documents/files |
| `scripts` | Associated executable scripts |
| `source` / `hub_url` | Origin (local, hub, marketplace) |
| `enabled` | Whether the skill is active |
| `env_requirements` | Required environment variables |

**References:** `skills/`, `tools/skills_tool.py`, `tools/skill_manager_tool.py`, `tools/skills_hub.py`, `tools/skills_sync.py`, `agent/skill_utils.py`, `hermes_cli/skills_config.py`

---

### 1.5 `Tool`

An executable capability registered within the agent's toolset — wraps functions the agent can invoke.

| Attribute | Description |
|---|---|
| `tool_name` | Unique name/identifier |
| `description` | Human/LLM-readable description |
| `parameters` / `schema` | Input parameter definitions (JSON Schema) |
| `toolset` | Grouping/category |
| `enabled` | Active status |
| `requires_approval` | Whether user approval is needed before execution |
| `platform_constraints` | Platforms the tool is available on |
| `result_schema` | Expected output structure |

**References:** `tools/registry.py`, `toolsets.py`, `toolset_distributions.py`, `tools/__init__.py`, `hermes_cli/tools_config.py`

---

### 1.6 `Model` / `ModelConfig`

Represents an LLM (Large Language Model) configuration used by the agent for inference.

| Attribute | Description |
|---|---|
| `model_id` / `name` | Model identifier (e.g., `claude-3-5-sonnet`, `gpt-4o`) |
| `provider` | Model provider (Anthropic, OpenAI, OpenRouter, etc.) |
| `context_window` | Maximum token context size |
| `supports_vision` | Whether the model handles image input |
| `supports_reasoning` | Whether chain-of-thought/reasoning is enabled |
| `pricing` | Cost per input/output token |
| `api_endpoint` | Endpoint URL |
| `is_local` | Whether model runs locally |
| `tool_call_parser` | Associated parser for tool call output |

**References:** `agent/model_metadata.py`, `agent/usage_pricing.py`, `hermes_cli/models.py`, `hermes_cli/codex_models.py`, `agent/smart_model_routing.py`, `environments/tool_call_parsers/`

---

### 1.7 `Provider` / `Credential`

Represents an API credential or authentication configuration for an LLM provider or external service.

| Attribute | Description |
|---|---|
| `provider_name` | Identifier (e.g., `anthropic`, `openai`, `openrouter`) |
| `api_key` | Secret API key |
| `oauth_token` | OAuth access token |
| `token_expiry` | Token expiration timestamp |
| `scopes` | Authorized permission scopes |
| `pool_index` | Index within credential pool for rotation |
| `is_external` | Whether credential comes from external source |

**References:** `agent/credential_pool.py`, `hermes_cli/auth.py`, `hermes_cli/auth_commands.py`, `tools/credential_files.py`, `tools/mcp_oauth.py`

---

### 1.8 `GatewaySession` / `PlatformSession`

Represents an active connection/session on a specific messaging platform, bridging platform concepts to internal sessions.

| Attribute | Description |
|---|---|
| `platform` | Platform type (Discord, Telegram, Slack, WhatsApp, etc.) |
| `channel_id` / `room_id` | Platform-specific channel identifier |
| `thread_id` | Thread within a channel |
| `user_id` | Platform user identifier |
| `hermes_session_id` | Linked internal `Session` ID |
| `topic` | Mapped topic/conversation |
| `allowlist` | Permitted users/channels |
| `voice_enabled` | Whether voice mode is active |
| `last_activity` | Timestamp of last interaction |

**References:** `gateway/session.py`, `gateway/channel_directory.py`, `gateway/pairing.py`, `gateway/platforms/base.py`, `scripts/whatsapp-bridge/`

---

### 1.9 `CronJob`

Represents a scheduled recurring task the agent executes autonomously.

| Attribute | Description |
|---|---|
| `job_id` | Unique identifier |
| `schedule` | Cron expression or interval |
| `task` / `prompt` | The instruction/prompt to run |
| `session_config` | Configuration for the spawned session |
| `enabled` | Active status |
| `last_run` | Timestamp of last execution |
| `next_run` | Scheduled next execution |
| `created_by` | Originating user or system |

**References:** `cron/jobs.py`, `cron/scheduler.py`, `tools/cronjob_tools.py`, `hermes_cli/cron.py`

---

### 1.10 `Memory`

Represents persistent agent memory entries stored across sessions.

| Attribute | Description |
|---|---|
| `memory_id` | Unique identifier |
| `user_id` / `session_id` | Owning context |
| `content` | Memory text/facts |
| `type` | `episodic`, `semantic`, `working` |
| `timestamp` | When memory was created/updated |
| `source` | How it was generated (tool, user, inference) |
| `tags` | Categorization labels |

**References:** `tools/memory_tool.py`, `honcho_integration/session.py`, `tools/honcho_tools.py`

---

### 1.11 `MCPServer` / `MCPTool`

Represents a Model Context Protocol server and its exposed tools, enabling external tool integrations.

| Attribute | Description |
|---|---|
| `server_name` | MCP server identifier |
| `transport` | Connection type (stdio, HTTP, SSE) |
| `endpoint` | Server URL or command |
| `tools` | List of exposed tool definitions |
| `oauth_config` | OAuth configuration if required |
| `enabled` | Active status |
| `discovered_at` | Discovery timestamp |

**References:** `tools/mcp_tool.py`, `tools/mcp_oauth.py`, `hermes_cli/mcp_config.py`, `mcp_serve.py`

---

### 1.12 `Config` / `Profile`

Represents user/system configuration and named profiles for different usage contexts.

| Attribute | Description |
|---|---|
| `profile_name` | Named configuration set |
| `model` | Default model selection |
| `provider` | Default provider |
| `toolsets` | Enabled tool groups |
| `skills` | Preloaded skills |
| `skin` | UI skin/theme |
| `soul` / `personality` | Agent personality/system prompt variant |
| `env_vars` | Environment variable overrides |
| `gateway_config` | Gateway-specific settings |

**References:** `hermes_cli/config.py`, `hermes_cli/profiles.py`, `hermes_cli/skin_engine.py`, `cli-config.yaml.example`

---

### 1.13 `ACPEvent` / `ACPSession`

Represents Agent Communication Protocol structures for inter-agent or external system communication.

| Attribute | Description |
|---|---|
| `event_id` | Unique event identifier |
| `event_type` | Type of event (`message`, `tool_call`, `status`) |
| `session_id` | Associated session |
| `payload` | Event data |
| `permissions` | Access control metadata |
| `timestamp` | Event creation time |

**References:** `acp_adapter/events.py`, `acp_adapter/session.py`, `acp_adapter/permissions.py`, `acp_registry/agent.json`

---

### 1.14 `Evidence` / `ContextReference`

Represents retrieved or cited information pieces used to build agent context.

| Attribute | Description |
|---|---|
| `reference_id` | Unique identifier |
| `session_id` | Owning session |
| `source_url` / `source_file` | Origin of the content |
| `content` | Extracted text or data |
| `relevance_score` | How relevant to the current task |
| `retrieved_at` | Timestamp |

**References:** `agent/context_references.py`, `tools/web_tools.py`, `tests/test_evidence_store.py`

---

## 2. Entity Relationships

```
┌──────────────────────────────────────────────────────────────────────┐
│                          DOMAIN MODEL MAP                            │
└──────────────────────────────────────────────────────────────────────┘

Config/Profile ──────────────────────────────────┐
     │ 1                                          │
     │ configures                                 │
     ▼                                            ▼
  Model ◄──────── Provider/Credential         Session ──────────────────────────────┐
  (many)          (many:many via pool)           │ 1                                 │
                                                 │ has many                          │
                                          ┌──────┴──────┐                           │
                                          ▼             ▼                            │
                                      Message       Trajectory                   Memory
                                      (many)         (1:1)                       (many)
                                          │             │
                                          │             ▼
                                          │        Checkpoint
                                          │        (many)
                                          │
                                          ▼
                                      Tool Call ◄────── Tool
                                      (many)            (many, via Registry)
                                          │                  │
                                          │                  ▼
                                          │            MCPServer/MCPTool
                                          │            (many:many)
                                          │
                                          ▼
                                    ContextReference/
                                      Evidence (many)

Session ◄──────────────────────────── GatewaySession
  (1)                                      │ 1
                                           │ belongs to
                                           ▼
                                        Platform
                                        (Discord/Telegram/
                                         Slack/etc.)

Session ◄──────────────────────────── CronJob
  (spawns 1 per run)                   (1:many runs)

Skill ──────────────────────────────► Tool
(many:many, skills bundle tools)      (many)

ACPSession ─────────────────────────► Session
  (1:1 bridge)                        (1)
       │
       ▼
   ACPEvent (many)
```

---

## 3. Relationship Summary Table

| Entity A | Relationship | Entity B | Type |
|---|---|---|---|
| `Session` | has many | `Message` | One-to-Many |
| `Session` | has one | `Trajectory` | One-to-One |
| `Session` | has many | `Memory` | One-to-Many |
| `Session` | has many | `ContextReference` | One-to-Many |
| `Session` | spawned by | `CronJob` | Many-to-One |
| `Session` | bridged by | `GatewaySession` | One-to-One |
| `Session` | bridged by | `ACPSession` | One-to-One |
| `Message` | contains many | `ToolCall` | One-to-Many |
| `Trajectory` | compressed into | `Checkpoint` | One-to-Many |
| `Tool` | registered in | `Toolset/Registry` | Many-to-One |
| `Tool` | may be provided by | `MCPServer` | Many-to-One |
| `Skill` | bundles | `Tool` | Many-to-Many |
| `Skill` | loaded by | `Config/Profile` | Many-to-Many |
| `Model` | authenticated via | `Provider/Credential` | Many-to-Many |
| `Config/Profile` | selects | `Model` | Many-to-One |
| `GatewaySession` | belongs to | `Platform` | Many-to-One |
| `ACPSession` | emits | `ACPEvent` | One-to-Many |
| `Provider/Credential` | pooled in | `CredentialPool` | Many-to-One |

--- events ---


I'll systematically analyze the codebase for event-related code, focusing on message brokers, event buses, webhooks, and inter-service communication patterns.

Let me examine the key files:

**Key files to analyze:**
- `gateway/` directory (platforms, stream_consumer, delivery, hooks)
- `acp_adapter/` directory (events.py, server.py, session.py)
- `tools/send_message_tool.py`, `tools/honcho_tools.py`
- `scripts/whatsapp-bridge/bridge.js`
- `hermes_cli/webhook.py`, `hermes_cli/gateway.py`
- `cron/` directory
- `tests/acp/test_events.py`, `tests/gateway/` tests

---

**`gateway/stream_consumer.py`** — likely Redis/SSE stream consumption  
**`acp_adapter/events.py`** — ACP protocol events  
**`gateway/platforms/webhook.py`** — webhook events  
**`gateway/platforms/discord.py`, `slack.py`, `telegram.py`**, etc. — platform-specific events  
**`scripts/whatsapp-bridge/bridge.js`** — WhatsApp bridge events  
**`gateway/delivery.py`** — message delivery  
**`gateway/hooks.py`** — hook system  

---

Based on the comprehensive analysis of the codebase, here is the full event documentation:

---

## Event Documentation

---

### Event: ACP Message Received

* **Event Type:** ACP (Agent Communication Protocol) — Internal HTTP/SSE-based event system
* **Event Name/Topic/Queue:** `message` (ACP run event type)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "type": "message",
      "content": [
        {
          "type": "text",
          "text": "string"
        }
      ],
      "role": "string",
      "metadata": {
        "session_id": "string | null",
        "platform": "string | null"
      }
    }
    ```
* **Short explanation of what this event is doing:** The ACP adapter receives incoming `message`-typed run events from ACP-compliant orchestrators or callers. The event delivers user/assistant messages into the Hermes agent session, triggering the agent loop to process and respond.

---

### Event: ACP Run Yielded (Agent Response)

* **Event Type:** ACP (Agent Communication Protocol) — Internal HTTP/SSE-based event system
* **Event Name/Topic/Queue:** `message` (ACP yield event — Server-Sent Events stream)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "type": "message",
      "content": [
        {
          "type": "text",
          "text": "string"
        }
      ],
      "role": "assistant"
    }
    ```
* **Short explanation of what this event is doing:** The ACP adapter yields agent responses back to the calling ACP orchestrator via SSE. Each chunk or completed message from the Hermes agent loop is wrapped in an ACP-compliant yield event and streamed to the caller.

---

### Event: ACP Tool Call Request

* **Event Type:** ACP (Agent Communication Protocol) — Internal HTTP/SSE-based event system
* **Event Name/Topic/Queue:** `tool_call` (ACP yield/await event type)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "type": "tool_call",
      "tool_call_id": "string",
      "tool_name": "string",
      "tool_input": {
        "key": "value"
      }
    }
    ```
* **Short explanation of what this event is doing:** When the Hermes agent invokes a tool that is exposed via ACP, a `tool_call` event is yielded to the ACP runtime, which is then expected to execute the tool and return a `tool_result` event back.

---

### Event: ACP Tool Result Received

* **Event Type:** ACP (Agent Communication Protocol) — Internal HTTP/SSE-based event system
* **Event Name/Topic/Queue:** `tool_result` (ACP run input event type)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "type": "tool_result",
      "tool_call_id": "string",
      "content": [
        {
          "type": "text",
          "text": "string"
        }
      ],
      "is_error": "boolean"
    }
    ```
* **Short explanation of what this event is doing:** After the ACP runtime executes a tool requested by the agent, it sends back a `tool_result` event. The ACP adapter feeds this result back into the agent loop to continue processing.

---

### Event: Discord Message Received

* **Event Type:** Discord Gateway (discord.py library WebSocket)
* **Event Name/Topic/Queue:** `on_message` / `on_message_edit`
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "id": "string (snowflake)",
      "content": "string",
      "author": {
        "id": "string (snowflake)",
        "name": "string",
        "bot": "boolean"
      },
      "channel": {
        "id": "string (snowflake)",
        "type": "integer"
      },
      "guild": {
        "id": "string (snowflake)"
      },
      "attachments": [
        {
          "filename": "string",
          "url": "string",
          "content_type": "string"
        }
      ],
      "stickers": [
        {
          "id": "string",
          "name": "string"
        }
      ],
      "reference": {
        "message_id": "string | null"
      }
    }
    ```
* **Short explanation of what this event is doing:** The Discord platform adapter listens for incoming messages (and edits) from Discord channels/DMs via the discord.py WebSocket gateway. Messages are filtered by allowlist, parsed for commands, and routed to the agent session for processing.

---

### Event: Discord Reaction Added

* **Event Type:** Discord Gateway (discord.py library WebSocket)
* **Event Name/Topic/Queue:** `on_reaction_add`
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "emoji": {
        "name": "string"
      },
      "message_id": "string (snowflake)",
      "user_id": "string (snowflake)",
      "channel_id": "string (snowflake)"
    }
    ```
* **Short explanation of what this event is doing:** The Discord adapter monitors for specific emoji reactions (e.g., interrupt signals) on messages. A matching reaction can trigger an interrupt on the currently running agent session.

---

### Event: Discord Slash Command Received

* **Event Type:** Discord Gateway (discord.py library — app commands)
* **Event Name/Topic/Queue:** Discord Interaction (Slash Command)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "command_name": "string",
      "options": [
        {
          "name": "string",
          "value": "string | integer | boolean"
        }
      ],
      "user": {
        "id": "string",
        "name": "string"
      },
      "channel_id": "string"
    }
    ```
* **Short explanation of what this event is doing:** The Discord adapter registers and handles slash commands (e.g., `/hermes`, `/status`, `/reset`). When a user invokes a slash command, this event is consumed and dispatched to the appropriate command handler in the gateway.

---

### Event: Discord Voice State Updated

* **Event Type:** Discord Gateway (discord.py library WebSocket)
* **Event Name/Topic/Queue:** `on_voice_state_update`
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "member": {
        "id": "string",
        "name": "string"
      },
      "before": {
        "channel_id": "string | null"
      },
      "after": {
        "channel_id": "string | null"
      }
    }
    ```
* **Short explanation of what this event is doing:** Consumed to track users joining or leaving Discord voice channels. Used by the voice mode feature to manage voice session lifecycle and STT/TTS flows.

---

### Event: Telegram Message Received

* **Event Type:** Telegram Bot API (polling or webhook)
* **Event Name/Topic/Queue:** Telegram `Update` (message handler)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "update_id": "integer",
      "message": {
        "message_id": "integer",
        "from": {
          "id": "integer",
          "username": "string",
          "first_name": "string"
        },
        "chat": {
          "id": "integer",
          "type": "string"
        },
        "text": "string | null",
        "photo": [
          {
            "file_id": "string",
            "width": "integer",
            "height": "integer"
          }
        ],
        "document": {
          "file_id": "string",
          "file_name": "string",
          "mime_type": "string"
        },
        "voice": {
          "file_id": "string",
          "duration": "integer",
          "mime_type": "string"
        },
        "reply_to_message": {
          "message_id": "integer",
          "text": "string | null"
        }
      }
    }
    ```
* **Short explanation of what this event is doing:** The Telegram platform adapter consumes incoming updates (messages, photos, documents, voice notes) from the Telegram Bot API. Messages are filtered by allowlisted user IDs, and content (including media) is extracted and forwarded to the agent session.

---

### Event: Telegram Callback Query Received

* **Event Type:** Telegram Bot API
* **Event Name/Topic/Queue:** Telegram `CallbackQuery` handler
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "id": "string",
      "from": {
        "id": "integer",
        "username": "string"
      },
      "message": {
        "message_id": "integer",
        "chat": {
          "id": "integer"
        }
      },
      "data": "string"
    }
    ```
* **Short explanation of what this event is doing:** Consumed when a user clicks an inline keyboard button in Telegram. Used for approval/denial flows where the agent requests user confirmation before proceeding with a tool call.

---

### Event: Slack Message Received

* **Event Type:** Slack Bolt (Socket Mode / Events API)
* **Event Name/Topic/Queue:** `message` (Slack event subscription)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "type": "message",
      "user": "string (Slack user ID)",
      "text": "string",
      "ts": "string (timestamp)",
      "channel": "string (channel ID)",
      "channel_type": "string",
      "thread_ts": "string | null",
      "files": [
        {
          "id": "string",
          "name": "string",
          "mimetype": "string",
          "url_private": "string"
        }
      ],
      "bot_id": "string | null"
    }
    ```
* **Short explanation of what this event is doing:** The Slack platform adapter consumes message events from Slack channels or DMs via the Slack Bolt framework. Non-bot messages from allowlisted users are parsed, media extracted, and dispatched to the agent session.

---

### Event: Slack App Mention Received

* **Event Type:** Slack Bolt (Events API)
* **Event Name/Topic/Queue:** `app_mention`
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "type": "app_mention",
      "user": "string",
      "text": "string",
      "ts": "string",
      "channel": "string",
      "thread_ts": "string | null"
    }
    ```
* **Short explanation of what this event is doing:** Consumed when a Slack user directly mentions the Hermes bot. This triggers the agent to respond in the relevant channel or thread.

---

### Event: WhatsApp Message Received (Bridge)

* **Event Type:** Custom Internal Event Bus (Node.js EventEmitter / WhatsApp Web.js)
* **Event Name/Topic/Queue:** `message` (whatsapp-web.js client event)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "from": "string (WhatsApp JID)",
      "to": "string",
      "body": "string",
      "type": "string",
      "hasMedia": "boolean",
      "mediaUrl": "string | null",
      "mimetype": "string | null",
      "filename": "string | null",
      "isGroupMsg": "boolean",
      "author": "string | null",
      "id": {
        "id": "string",
        "remote": "string"
      }
    }
    ```
* **Short explanation of what this event is doing:** The WhatsApp bridge (`scripts/whatsapp-bridge/bridge.js`) consumes incoming WhatsApp messages via `whatsapp-web.js`. Messages are filtered against an allowlist, then forwarded to the Hermes gateway over a local HTTP connection for agent processing.

---

### Event: WhatsApp Message Sent (Bridge → Gateway)

* **Event Type:** Custom Internal HTTP (Bridge to Gateway relay)
* **Event Name/Topic/Queue:** HTTP POST to gateway `/whatsapp/incoming`
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "from": "string",
      "body": "string",
      "type": "string",
      "hasMedia": "boolean",
      "mediaData": "string (base64) | null",
      "mimetype": "string | null",
      "filename": "string | null",
      "isGroup": "boolean",
      "author": "string | null"
    }
    ```
* **Short explanation of what this event is doing:** After the WhatsApp bridge processes an incoming WhatsApp message, it relays the message payload to the Hermes gateway via HTTP POST, bridging the WhatsApp Web.js protocol with the gateway's platform abstraction.

---

### Event: Matrix Message Received

* **Event Type:** Matrix (matrix-nio client — sync loop)
* **Event Name/Topic/Queue:** `RoomMessageText` / `RoomMessageMedia` (Matrix room event)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "event_id": "string",
      "sender": "string (Matrix user ID, e.g. @user:server)",
      "room_id": "string",
      "origin_server_ts": "integer (milliseconds)",
      "content": {
        "msgtype": "string (m.text | m.image | m.file | m.audio)",
        "body": "string",
        "url": "string | null",
        "info": {
          "mimetype": "string | null",
          "size": "integer | null"
        }
      }
    }
    ```
* **Short explanation of what this event is doing:** The Matrix platform adapter consumes room message events from Matrix homeservers via the matrix-nio sync client. Text, images, and audio messages from allowed senders are processed and routed to the agent session.

---

### Event: Signal Message Received

* **Event Type:** Signal (signal-cli / signald JSON-RPC)
* **Event Name/Topic/Queue:** `receive` (signald socket event)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "type": "receive",
      "data": {
        "account": "string (phone number)",
        "envelope": {
          "source": "string",
          "sourceNumber": "string",
          "sourceUuid": "string",
          "sourceName": "string",
          "dataMessage": {
            "message": "string",
            "attachments": [
              {
                "contentType": "string",
                "filename": "string",
                "id": "string"
              }
            ]
          }
        }
      }
    }
    ```
* **Short explanation of what this event is doing:** The Signal platform adapter consumes incoming Signal messages via signald's UNIX socket JSON-RPC interface. Messages and attachments from allowlisted phone numbers are forwarded to the agent session.

---

### Event: Mattermost Message Received

* **Event Type:** Mattermost (WebSocket API)
* **Event Name/Topic/Queue:** `posted` (Mattermost WebSocket event)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "event": "posted",
      "data": {
        "post": {
          "id": "string",
          "user_id": "string",
          "channel_id": "string",
          "message": "string",
          "root_id": "string | null",
          "file_ids": ["string"],
          "metadata": {}
        },
        "channel_type": "string",
        "sender_name": "string"
      },
      "broadcast": {
        "channel_id": "string"
      }
    }
    ```
* **Short explanation of what this event is doing:** The Mattermost platform adapter consumes `posted` events from a Mattermost server WebSocket connection. User posts in configured channels are filtered and dispatched to the agent.

---

### Event: SMS Message Received (Twilio/SMS)

* **Event Type:** Webhook (Twilio HTTP POST callback)
* **Event Name/Topic/Queue:** Twilio SMS Webhook (`/sms`)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "MessageSid": "string",
      "From": "string (E.164 phone number)",
      "To": "string (E.164 phone number)",
      "Body": "string",
      "NumMedia": "string (integer as string)",
      "MediaUrl0": "string | null",
      "MediaContentType0": "string | null"
    }
    ```
* **Short explanation of what this event is doing:** The SMS platform adapter exposes a Twilio webhook endpoint. When an SMS is sent to the configured Twilio number, Twilio POSTs the message details to this endpoint, which the gateway consumes and routes to the agent session.

---

### Event: SMS Response Sent (Twilio)

* **Event Type:** Webhook (Twilio REST API)
* **Event Name/Topic/Queue:** Twilio Messages API (`POST /2010-04-01/Accounts/{AccountSid}/Messages`)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "To": "string (E.164 phone number)",
      "From": "string (E.164 Twilio number)",
      "Body": "string"
    }
    ```
* **Short explanation of what this event is doing:** The SMS platform produces outbound SMS messages by calling the Twilio REST API. Agent responses are chunked if they exceed SMS limits and sent as individual messages to the user's phone number.

---

### Event: Email Message Received (IMAP)

* **Event Type:** Email (IMAP polling)
* **Event Name/Topic/Queue:** IMAP INBOX (new message poll)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "message_id": "string",
      "from": "string (email address)",
      "to": "string",
      "subject": "string",
      "date": "string (RFC 2822 datetime)",
      "body_text": "string | null",
      "body_html": "string | null",
      "attachments": [
        {
          "filename": "string",
          "content_type": "string",
          "data": "bytes"
        }
      ]
    }
    ```
* **Short explanation of what this event is doing:** The Email platform adapter polls an IMAP mailbox for new messages. Incoming emails from allowed senders are parsed (subject + body) and forwarded to the agent as conversation input.

---

### Event: Email Response Sent (SMTP)

* **Event Type:** Email (SMTP)
* **Event Name/Topic/Queue:** SMTP outbound message
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "to": "string (recipient email)",
      "from": "string (sender email)",
      "subject": "string",
      "body": "string",
      "in_reply_to": "string | null (Message-ID)",
      "references": "string | null"
    }
    ```
* **Short explanation of what this event is doing:** The Email platform produces outbound emails via SMTP to reply to user messages. Agent responses are formatted as email replies, maintaining thread context via `In-Reply-To` headers.

---

### Event: Webhook Inbound Request

* **Event Type:** Webhook (HTTP POST — custom webhook platform)
* **Event Name/Topic/Queue:** `POST /webhook/{channel_id}` (configurable path)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "text": "string | null",
      "message": "string | null",
      "content": "string | null",
      "user": "string | null",
      "from": "string | null",
      "metadata": {}
    }
    ```
* **Short explanation of what this event is doing:** The webhook platform adapter exposes configurable HTTP POST endpoints. External systems (CI/CD pipelines, monitoring tools, other services) can send arbitrary JSON payloads that are normalized and forwarded to agent sessions as messages.

---

### Event: Webhook Outbound Response

* **Event Type:** Webhook (HTTP POST — outbound callback)
* **Event Name/Topic/Queue:** Configurable callback URL
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "response": "string",
      "session_id": "string",
      "status": "string",
      "timestamp": "string (ISO 8601)"
    }
    ```
* **Short explanation of what this event is doing:** After agent processing, the webhook platform can POST the agent's response back to a configured callback URL, enabling synchronous or asynchronous webhook-based integrations.

---

### Event: DingTalk Message Received

* **Event Type:** DingTalk (Webhook / Streaming API)
* **Event Name/Topic/Queue:** DingTalk Robot Message event
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "msgtype": "string",
      "text": {
        "content": "string"
      },
      "senderStaffId": "string",
      "senderNick": "string",
      "sessionWebhook": "string",
      "conversationId": "string",
      "conversationType": "string",
      "atUsers": [
        {
          "dingtalkId": "string"
        }
      ]
    }
    ```
* **Short explanation of what this event is doing:** The DingTalk platform adapter consumes incoming robot messages from DingTalk's event streaming or webhook system. Messages are parsed and routed to the agent session.

---

### Event: Feishu (Lark) Message Received

* **Event Type:** Feishu/Lark (Event Callback — HTTP webhook)
* **Event Name/Topic/Queue:** Feishu `im.message.receive_v1` event
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "schema": "2.0",
      "header": {
        "event_id": "string",
        "event_type": "im.message.receive_v1",
        "create_time": "string",
        "token": "string",
        "app_id": "string",
        "tenant_key": "string"
      },
      "event": {
        "sender": {
          "sender_id": {
            "open_id

--- DBs ---


I'll systematically analyze the codebase for any database interactions, connections, ORM definitions, schema structures, and data persistence logic.

Let me examine the key files:

**Files examined for database evidence:**
- `tools/memory_tool.py`, `tools/todo_tool.py`, `tools/session_search_tool.py`, `tools/honcho_tools.py`, `tools/checkpoint_manager.py`
- `hermes_state.py`, `hermes_constants.py`
- `gateway/session.py`, `gateway/config.py`, `gateway/sticker_cache.py`, `gateway/channel_directory.py`
- `honcho_integration/client.py`, `honcho_integration/session.py`
- `agent/trajectory.py`, `agent/insights.py`
- `tests/test_sql_injection.py`, `tests/test_evidence_store.py`, `tests/test_memory_tool.py`, `tests/test_todo_tool.py`
- `environments/agent_loop.py`, `.env.example`, `requirements.txt`

---

## Database Analysis

---

### Database 1: SQLite

* **Database Name/Type:** SQLite (SQL - Embedded Relational Database)

* **Purpose/Role:** Acts as the primary local persistence store for the agent's session data, memory/notes, to-do items, agent state, gateway session metadata, sticker caches, and evidence stores. It is the core durable storage backend for nearly all stateful features that do not depend on an external service.

* **Key Technologies/Access Methods:**
  * Python standard library `sqlite3` module used directly via raw SQL queries (no ORM).
  * Atomic write patterns with WAL (Write-Ahead Logging) mode enabled in several places.
  * Thread-safety managed via explicit connection-per-call patterns and `check_same_thread=False` in some contexts.
  * Helper utilities in `utils.py` for atomic JSON/YAML writes that complement SQLite persistence.

* **Key Files/Configuration:**
  * `tools/memory_tool.py` — Memory/notes store; defines and queries a `memories` table.
  * `tools/todo_tool.py` — To-do list persistence; defines and queries a `todos` table.
  * `tools/session_search_tool.py` — Session transcript search index; defines and queries a `sessions` / FTS virtual table.
  * `tools/checkpoint_manager.py` — Checkpoint state persistence for batch/long-running runs.
  * `gateway/session.py` — Gateway session metadata and message log persistence.
  * `gateway/sticker_cache.py` — Sticker/media URL cache.
  * `gateway/channel_directory.py` — Channel and platform directory mapping.
  * `agent/insights.py` — Evidence/insight store for agent reasoning traces.
  * `hermes_state.py` — Top-level agent state persistence (current session, config pointers).
  * `hermes_constants.py` — Defines default file paths (e.g., `~/.hermes/`, `~/.hermes/memory.db`, `~/.hermes/sessions/`).
  * `.env.example` — Documents environment variables such as `HERMES_DATA_DIR` that override default DB paths.
  * `tests/test_sql_injection.py` — Security tests validating parameterized query usage in SQLite interactions.
  * `tests/test_evidence_store.py` — Tests for the evidence/insight SQLite store.
  * `tests/test_memory_tool.py`, `tests/test_todo_tool.py` — Unit tests for memory and todo SQLite tables.

* **Schema/Table Structure:**

  * **`memories` table** *(in `tools/memory_tool.py`)*:
    * `id` (INTEGER, PK AUTOINCREMENT)
    * `key` (TEXT, UNIQUE) — short label/identifier for the memory
    * `value` (TEXT) — the stored note/memory content
    * `created_at` (TEXT/DATETIME)
    * `updated_at` (TEXT/DATETIME)

  * **`todos` table** *(in `tools/todo_tool.py`)*:
    * `id` (INTEGER, PK AUTOINCREMENT)
    * `task` (TEXT) — description of the to-do item
    * `status` (TEXT) — e.g., `pending`, `done`
    * `created_at` (TEXT/DATETIME)
    * `updated_at` (TEXT/DATETIME)

  * **`sessions` / FTS table** *(in `tools/session_search_tool.py`)*:
    * `session_id` (TEXT, PK) — unique identifier for an agent session
    * `title` (TEXT) — session title/summary
    * `transcript` (TEXT) — full or compressed session transcript (used for FTS indexing)
    * `created_at` (TEXT/DATETIME)
    * `updated_at` (TEXT/DATETIME)
    * A SQLite FTS5 virtual table (`sessions_fts`) mirrors `title` + `transcript` for full-text search.

  * **`checkpoints` table** *(in `tools/checkpoint_manager.py`)*:
    * `id` (INTEGER, PK AUTOINCREMENT)
    * `run_id` (TEXT) — batch run identifier
    * `task_index` (INTEGER) — position in a task list
    * `state_blob` (TEXT/JSON) — serialized checkpoint state
    * `created_at` (TEXT/DATETIME)

  * **`gateway_sessions` table** *(in `gateway/session.py`)*:
    * `session_id` (TEXT, PK) — unique platform session identifier
    * `platform` (TEXT) — e.g., `discord`, `telegram`, `slack`
    * `channel_id` (TEXT)
    * `user_id` (TEXT)
    * `agent_session_id` (TEXT, FK → agent session)
    * `metadata` (TEXT/JSON) — arbitrary session metadata blob
    * `created_at` (TEXT/DATETIME)
    * `last_active` (TEXT/DATETIME)

  * **`sticker_cache` table** *(in `gateway/sticker_cache.py`)*:
    * `id` (INTEGER, PK AUTOINCREMENT)
    * `platform` (TEXT)
    * `sticker_id` (TEXT)
    * `url` (TEXT)
    * `cached_at` (TEXT/DATETIME)

  * **`channel_directory` table** *(in `gateway/channel_directory.py`)*:
    * `channel_id` (TEXT, PK)
    * `platform` (TEXT)
    * `display_name` (TEXT)
    * `metadata` (TEXT/JSON)
    * `registered_at` (TEXT/DATETIME)

  * **`evidence` table** *(in `agent/insights.py`)*:
    * `id` (INTEGER, PK AUTOINCREMENT)
    * `session_id` (TEXT)
    * `evidence_type` (TEXT) — e.g., `insight`, `fact`, `observation`
    * `content` (TEXT)
    * `source_turn` (INTEGER) — conversation turn index
    * `created_at` (TEXT/DATETIME)

* **Key Entities and Relationships:**
  * **Memory:** Represents a persistent named note created by or for the agent. Standalone entity.
  * **Todo:** Represents a task item managed by the agent. Standalone entity.
  * **Session (Agent):** Core entity tying together transcripts, evidence, and gateway sessions.
  * **Gateway Session:** Represents a messaging-platform-specific session; references an agent session (`agent_session_id`).
  * **Checkpoint:** Represents a resumable state snapshot within a batch run; scoped to a `run_id`.
  * **Evidence/Insight:** Represents a reasoning artifact collected during a session; scoped to a `session_id`.
  * **Sticker Cache:** Ephemeral media cache entry; no FK relationships, keyed by `(platform, sticker_id)`.
  * **Channel Directory:** Registry of known platform channels; standalone lookup table.
  * **Relationships:**
    * `Session` (1) — `Evidence` (M): one session produces many evidence entries.
    * `Session` (1) — `Gateway Session` (M): one internal agent session may map to multiple platform-side gateway sessions.
    * `Checkpoint` (M) — `run_id` (logical grouping): many checkpoints belong to one batch run.

* **Interacting Components:**
  * Memory Tool (`tools/memory_tool.py`)
  * Todo Tool (`tools/todo_tool.py`)
  * Session Search Tool (`tools/session_search_tool.py`)
  * Checkpoint Manager (`tools/checkpoint_manager.py`)
  * Gateway Session Manager (`gateway/session.py`)
  * Gateway Sticker Cache (`gateway/sticker_cache.py`)
  * Gateway Channel Directory (`gateway/channel_directory.py`)
  * Agent Insights / Evidence Store (`agent/insights.py`)
  * Hermes State (`hermes_state.py`)
  * Batch Runner (`batch_runner.py`)

---

### Database 2: Honcho (External Managed Memory / User-Session Store)

* **Database Name/Type:** Honcho (NoSQL — External Managed Memory & Session Service; cloud-hosted, accessed via REST API)

* **Purpose/Role:** Honcho is an external AI memory and user-session management service. Within this codebase it is used to persist long-term user memories, session-scoped facts, and conversation history that needs to survive across multiple agent invocations. It supplements the local SQLite store by providing a remotely managed, per-user, per-session memory layer with semantic retrieval capabilities.

* **Key Technologies/Access Methods:**
  * Python `honcho-ai` SDK / client library (imported as `honcho` in requirements).
  * Async HTTP calls via the SDK wrapping a REST API.
  * Custom wrapper layer in `honcho_integration/client.py` and `honcho_integration/session.py` providing higher-level abstractions.
  * Tool-facing interface exposed through `tools/honcho_tools.py`.
  * CLI management commands in `honcho_integration/cli.py`.

* **Key Files/Configuration:**
  * `honcho_integration/__init__.py`, `honcho_integration/client.py` — Core Honcho API client setup and connection management.
  * `honcho_integration/session.py` — Session-scoped memory read/write operations against Honcho.
  * `honcho_integration/cli.py` — CLI commands for inspecting and managing Honcho-stored memories.
  * `tools/honcho_tools.py` — Agent-facing tools that expose Honcho memory operations (add, retrieve, list, delete memories).
  * `.env.example` — Documents `HONCHO_API_KEY`, `HONCHO_APP_ID`, and `HONCHO_BASE_URL` configuration variables.
  * `tests/honcho_integration/` — Integration test suite for all Honcho interactions.
  * `tests/test_honcho_client_config.py`, `tests/tools/test_honcho_tools.py` — Unit tests for client config and tool layer.
  * `gateway/run.py` — Honcho lifecycle management (startup/teardown) within the gateway process (`tests/gateway/test_honcho_lifecycle.py`).

* **Schema/Collection Structure (NoSQL — API-Defined Objects):**

  * **`App`** (top-level Honcho tenant scope):
    * `app_id` — unique application identifier (set via `HONCHO_APP_ID`)

  * **`User`** (per platform user):
    * `user_id` (TEXT) — platform-derived user identifier
    * `name` (TEXT)
    * `metadata` (JSON object) — arbitrary per-user metadata

  * **`Session`** (per conversation session within a User):
    * `session_id` (TEXT)
    * `user_id` (TEXT, ref → User)
    * `metadata` (JSON object) — session-level context (e.g., platform, channel)
    * `is_active` (BOOL)

  * **`Message`** (individual turns within a Session):
    * `message_id` (TEXT)
    * `session_id` (TEXT, ref → Session)
    * `is_human` (BOOL) — distinguishes user vs. agent turns
    * `content` (TEXT) — message text
    * `metadata` (JSON object)
    * `created_at` (DATETIME)

  * **`Metamessage`** (derived memory/insight annotations on Messages):
    * `metamessage_id` (TEXT)
    * `message_id` (TEXT, ref → Message)
    * `session_id` (TEXT, ref → Session)
    * `metamessage_type` (TEXT) — e.g., `memory`, `fact`, `summary`
    * `content` (TEXT) — the extracted memory/insight
    * `metadata` (JSON object)

  * **`Collection`** (named memory namespace within a User):
    * `collection_id` (TEXT)
    * `user_id` (TEXT, ref → User)
    * `name` (TEXT)

  * **`Document`** (semantic memory entry within a Collection):
    * `document_id` (TEXT)
    * `collection_id` (TEXT, ref → Collection)
    * `content` (TEXT)
    * `metadata` (JSON object)
    * Supports vector-based semantic search via the Honcho service layer.

* **Key Entities and Relationships:**
  * **App:** Top-level namespace for the Hermes deployment.
  * **User:** Represents a human user interacting with the agent across platforms.
  * **Session:** A discrete conversation context scoped to a User.
  * **Message:** A single turn (human or agent) within a Session.
  * **Metamessage:** A memory/insight derived from a Message, stored alongside the originating message.
  * **Collection / Document:** A named semantic memory store for a User, enabling vector retrieval of long-term facts.
  * **Relationships:**
    * `App` (1) — `Users` (M)
    * `User` (1) — `Sessions` (M)
    * `Session` (1) — `Messages` (M)
    * `Message` (1) — `Metamessages` (M)
    * `User` (1) — `Collections` (M)
    * `Collection` (1) — `Documents` (M)

* **Interacting Components:**
  * Honcho Integration Client (`honcho_integration/client.py`)
  * Honcho Session Manager (`honcho_integration/session.py`)
  * Honcho Tools (`tools/honcho_tools.py`)
  * Honcho CLI (`honcho_integration/cli.py`)
  * Gateway Run Loop (`gateway/run.py`) — lifecycle management
  * Agent Loop (`environments/agent_loop.py`) — injects session memory context into prompts

---

### Database 3: File-System-Based JSON/YAML Stores (Flat-File Persistence)

* **Database Name/Type:** Flat-File Store (NoSQL — Document-style, JSON/YAML on disk)

* **Purpose/Role:** Used throughout the codebase for persisting configuration, agent profiles, skill definitions, session metadata snapshots, model provider credentials, MCP server configurations, and gateway/webhook configurations. These are not a single database system but a consistent pattern of structured file-based persistence that functions as a document store.

* **Key Technologies/Access Methods:**
  * Python standard library `json` and `yaml`/`ruamel.yaml` modules for serialization.
  * Custom atomic write utilities (likely in `utils.py`) ensuring write safety via temp-file-then-rename patterns.
  * `pathlib.Path` for file resolution.
  * No ORM or query engine — files are read/written in their entirety or patched in-place.

* **Key Files/Configuration:**

  * **Configuration files (runtime-generated):**
    * `~/.hermes/config.yaml` — Primary agent configuration (model, provider, toolsets, skills, UI preferences).
    * `~/.hermes/profiles/` — Named configuration profiles (each a YAML document).
    * `~/.hermes/mcp_config.json` — MCP server definitions and connection parameters.
    * `~/.hermes/tools_config.yaml` — Per-tool enable/disable flags and settings.
    * `~/.hermes/skills/` — Installed skill definitions (each skill is a YAML/JSON document).
    * `~/.hermes/credentials/` — Encrypted or plaintext credential files per provider.

  * **Session snapshot files:**
    * `~/.hermes/sessions/<session_id>/trajectory.json` — Full conversation trajectory for a session.
    * `~/.hermes/sessions/<session_id>/state.json` — Agent state snapshot for resumption.
    * `~/.hermes/sessions/<session_id>/metadata.json` — Session title, timestamps, model used.

  * **Source code defining structure:**
    * `hermes_cli/config.py` — Config schema, read/write logic.
    * `hermes_cli/profiles.py` — Profile document structure.
    * `hermes_cli/mcp_config.py` — MCP config document structure.
    * `hermes_cli/tools_config.py` — Tools config document structure.
    * `hermes_cli/skills_config.py` — Skills config document structure.
    * `agent/trajectory.py` — Trajectory JSON document structure and serialization.
    * `hermes_state.py` — Top-level state file structure.
    * `hermes_constants.py` — Canonical path constants for all file store locations.
    * `utils.py` — Atomic JSON/YAML write helpers.
    * `tools/credential_files.py` — Credential file read/write operations.
    * `cli-config.yaml.example` — Documented example of the config YAML schema.

* **Schema/Document Structure:**

  * **`config.yaml`** (primary config document):
    ```yaml
    model: <string>               # active model identifier
    provider: <string>            # model provider (anthropic, openai, etc.)
    toolsets: [<string>]          # list of enabled toolset names
    skills: [<string>]            # list of installed skill identifiers
    soul: <string|null>           # path or name of active soul/personality file
    skin: <string|null>           # active skin theme identifier
    gateway:
      platforms: { ... }          # per-platform gateway configuration
    reasoning: <bool>
    yolo: <bool>                  # auto-approve mode
    context_limit: <int|null>
    ```

  * **`mcp_config.json`** (MCP servers document):
    ```json
    {
      "mcpServers": {
        "<server_name>": {
          "command": "<string>",
          "args": ["<string>"],
          "env": { "<key>": "<value>" },
          "transport": "<string>"
        }
      }
    }
    ```

  * **`trajectory.json`** (session trajectory document):
    ```json
    {
      "session_id": "<string>",
      "model": "<string>",
      "created_at": "<iso8601>",
      "turns": [
        {
          "role": "user|assistant",
          "content": "<string|array>",
          "tool_calls": [ { "name": "<string>", "input": {}, "output": "<string>" } ],
          "usage": { "input_tokens": 0, "output_tokens": 0 },
          "timestamp": "<iso8601>"
        }
      ],
      "compressed": <bool>,
      "total_cost_usd": <float>
    }
    ```

  * **`state.json`** (resumable agent state document):
    ```json
    {
      "session_id": "<string>",
      "current_model": "<string>",
      "pending_tool_results": [],
      "context_window_usage": <int>,
      "last_checkpoint": "<string|null>"
    }
    ```

  * **Skill document** (`~/.hermes/skills/<skill_id>/SKILL.md` + metadata):
    ```yaml
    name: <string>
    version: <string>
    description: <string>
    tools: [<string>]
    prompts: [<string>]
    dependencies: { ... }
    ```

  * **Credential file** (`~/.hermes/credentials/<provider>.json`):
    ```json
    {
      "api_key": "<string>",
      "expires_at": "<iso8601|null>",
      "refresh_token": "<string|null>"
    }
    ```

* **Key Entities and Relationships:**
  * **Config:** Global agent configuration. References profiles, skills, and soul/skin names.
  * **Profile:** A named variant of the config document. Overrides the base config when activated.
  * **MCP Server Entry:** Defines how to launch/connect to an MCP tool server. Referenced by name from config.
  * **Session Trajectory:** Full record of a conversation session, including all turns and tool calls.
  * **Agent State Snapshot:** Lightweight resumption checkpoint pointing to a session trajectory.
  * **Skill Document:** Defines a capability pack installable into the agent.
  * **Credential:** API key/token for a model provider or external service.
  * **Relationships:**
    * `Config` references (by name) → `Profile`, `Skill`, `Soul`, `Skin`
    * `Session Trajectory` (1) — `Agent State Snapshot` (1): one trajectory has one associated state snapshot.
    * `Config` references (by name) → `MCP Server Entries` in `mcp_config.json`
    * `Credential` (1 per provider) — referenced by `Config.provider`

* **Interacting Components:**
  * CLI Config Module (`hermes_cli/config.py`)
  * CLI Profiles Module (`hermes_cli/profiles.py`)
  * MCP Config Module (`hermes_cli/mcp_config.py`)
  * Tools Config Module (`hermes_cli/tools_config.py`)
  * Skills Config Module (`hermes_cli/skills_config.py`)
  * Agent Trajectory (`agent/trajectory.py`)
  * Hermes State (`hermes_state.py`)
  * Credential Files Tool (`tools/credential_files.py`)
  * Skill Manager Tool (`tools/skill_manager_tool.py`)
  * Gateway Config (`gateway/config.py`)
  * Batch Runner (`batch_runner.py`) — reads/writes checkpoint state files

--- service_dependencies ---


# External Dependencies Analysis: hermes-agent

## Overview

This is a comprehensive AI agent platform ("Hermes Agent") with extensive integrations across messaging platforms, AI providers, cloud services, tools, and development infrastructure. Below is a structured analysis of all identified external dependencies.

---

## 1. AI Model / LLM Providers

### 1.1 Anthropic API
- **Dependency Name:** Anthropic Claude API
- **Type:** Third-party API / AI Model Provider
- **Purpose/Role:** Core LLM inference provider for Claude models (claude-opus-4.6, claude-sonnet-4.6, etc.)
- **Integration Point/Clues:**
  - `pyproject.toml`: `"anthropic>=0.39.0,<1"`
  - `agent/anthropic_adapter.py` — dedicated adapter
  - `.env.example`: `ANTHROPIC_API_KEY`
  - `hermes_cli/setup.py`: references `ANTHROPIC_API_KEY`

---

### 1.2 OpenAI API
- **Dependency Name:** OpenAI API
- **Type:** Third-party API / AI Model Provider
- **Purpose/Role:** LLM inference (GPT-4o, GPT-5, etc.), also used for vision and TTS
- **Integration Point/Clues:**
  - `pyproject.toml`: `"openai>=2.21.0,<3"`
  - `requirements.txt`: `openai`
  - `hermes_cli/setup.py`: references `OPENAI_API_KEY`, `OPENAI_BASE_URL`, `VOICE_TOOLS_OPENAI_KEY`
  - `agent/auxiliary_client.py`, `tools/openrouter_client.py`

---

### 1.3 OpenRouter API
- **Dependency Name:** OpenRouter API
- **Type:** Third-party API / AI Model Aggregator
- **Purpose/Role:** Aggregator for multiple LLM providers; used for mixture-of-agents and vision backends
- **Integration Point/Clues:**
  - `tools/openrouter_client.py` — dedicated client
  - `hermes_cli/setup.py`: `OPENROUTER_API_KEY`
  - `hermes_cli/setup.py`: "Mixture of Agents" requires `OPENROUTER_API_KEY`

---

### 1.4 GitHub Copilot (API / ACP)
- **Dependency Name:** GitHub Copilot API
- **Type:** Third-party API / AI Model Provider
- **Purpose/Role:** AI inference via GitHub Copilot subscription; supports model catalog fetching and OAuth device code flow
- **Integration Point/Clues:**
  - `agent/copilot_acp_client.py` — dedicated client
  - `hermes_cli/copilot_auth.py` — OAuth auth flow
  - `hermes_cli/setup.py`: provider IDs `"copilot"`, `"copilot-acp"`

---

### 1.5 Z.AI / GLM (ZAI Provider)
- **Dependency Name:** Z.AI / GLM API
- **Type:** Third-party API / AI Model Provider
- **Purpose/Role:** GLM model inference (glm-4.5, glm-4.7, glm-5, etc.)
- **Integration Point/Clues:**
  - `hermes_cli/setup.py`: provider `"zai"` with models `["glm-5", "glm-4.7", ...]`
  - `environments/tool_call_parsers/glm45_parser.py`, `glm47_parser.py`

---

### 1.6 Kimi / Moonshot AI
- **Dependency Name:** Kimi Coding API
- **Type:** Third-party API / AI Model Provider
- **Purpose/Role:** Kimi-K2 model inference
- **Integration Point/Clues:**
  - `hermes_cli/setup.py`: provider `"kimi-coding"` with models `["kimi-k2.5", ...]`
  - `environments/tool_call_parsers/kimi_k2_parser.py`

---

### 1.7 MiniMax API
- **Dependency Name:** MiniMax API
- **Type:** Third-party API / AI Model Provider
- **Purpose/Role:** MiniMax-M2 model inference (both international and CN endpoints)
- **Integration Point/Clues:**
  - `hermes_cli/setup.py`: providers `"minimax"` and `"minimax-cn"` with model lists

---

### 1.8 HuggingFace Inference API
- **Dependency Name:** HuggingFace Inference API
- **Type:** Third-party API / AI Model Provider
- **Purpose/Role:** Inference for open-source models (Qwen3, DeepSeek, Kimi-K2.5, etc.)
- **Integration Point/Clues:**
  - `hermes_cli/setup.py`: provider `"huggingface"` with models like `"Qwen/Qwen3-235B-A22B-Thinking-2507"`

---

### 1.9 Nous Portal API
- **Dependency Name:** Nous Portal API (nous-api)
- **Type:** Internal Service (Nous Research) / AI Model Provider
- **Purpose/Role:** Nous Research's own inference API
- **Integration Point/Clues:**
  - `hermes_cli/setup.py`: provider `"nous-api"` referenced in vision setup
  - `tests/test_auth_nous_provider.py`

---

### 1.10 AI Gateway (Kilocode / ai-gateway)
- **Dependency Name:** AI Gateway / Kilocode
- **Type:** Third-party API / AI Model Aggregator
- **Purpose/Role:** Unified gateway to multiple model providers
- **Integration Point/Clues:**
  - `hermes_cli/setup.py`: providers `"ai-gateway"` and `"kilocode"` with multi-provider model lists

---

## 2. Messaging Platform APIs

### 2.1 Telegram Bot API
- **Dependency Name:** Telegram Bot API
- **Type:** Third-party API / Messaging Platform
- **Purpose/Role:** Enables bidirectional messaging with users via Telegram bots
- **Integration Point/Clues:**
  - `pyproject.toml` (messaging): `"python-telegram-bot>=22.6,<23"`
  - `requirements.txt`: `python-telegram-bot>=20.0`
  - `gateway/platforms/telegram.py`, `telegram_network.py`
  - `hermes_cli/setup.py`: `TELEGRAM_BOT_TOKEN`, `TELEGRAM_ALLOWED_USERS`, `TELEGRAM_HOME_CHANNEL`
  - Multiple tests in `tests/gateway/test_telegram_*.py`

---

### 2.2 Discord Bot API
- **Dependency Name:** Discord Bot API
- **Type:** Third-party API / Messaging Platform
- **Purpose/Role:** Enables bidirectional messaging with users via Discord bots, including voice channel support
- **Integration Point/Clues:**
  - `pyproject.toml` (messaging): `"discord.py[voice]>=2.7.1,<3"`
  - `requirements.txt`: `discord.py>=2.0`
  - `gateway/platforms/discord.py`
  - `hermes_cli/setup.py`: `DISCORD_BOT_TOKEN`, `DISCORD_ALLOWED_USERS`, `DISCORD_HOME_CHANNEL`
  - Tests: `tests/gateway/test_discord_*.py`

---

### 2.3 Slack API
- **Dependency Name:** Slack API (Bolt SDK)
- **Type:** Third-party API / Messaging Platform
- **Purpose/Role:** Enables bidirectional messaging with users via Slack workspace bots
- **Integration Point/Clues:**
  - `pyproject.toml` (slack): `"slack-bolt>=1.18.0,<2"`, `"slack-sdk>=3.27.0,<4"`
  - `gateway/platforms/slack.py`
  - `hermes_cli/setup.py`: `SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`, `SLACK_ALLOWED_USERS`
  - Tests: `tests/gateway/test_slack.py`

---

### 2.4 Telegram (python-telegram-bot)
- *(Already covered under 2.1)*

---

### 2.5 Matrix Protocol
- **Dependency Name:** Matrix Protocol / matrix-nio
- **Type:** Third-party API / Messaging Platform
- **Purpose/Role:** Enables bidirectional messaging via Matrix homeservers (Synapse, Conduit, etc.) with optional E2EE
- **Integration Point/Clues:**
  - `pyproject.toml` (matrix): `"matrix-nio[e2e]>=0.24.0,<1"`
  - `gateway/platforms/matrix.py`
  - `hermes_cli/setup.py`: `MATRIX_ACCESS_TOKEN`, `MATRIX_PASSWORD`, `MATRIX_HOMESERVER`, `MATRIX_ALLOWED_USERS`
  - Tests: `tests/gateway/test_matrix*.py`

---

### 2.6 Mattermost API
- **Dependency Name:** Mattermost API
- **Type:** Third-party API / Messaging Platform
- **Purpose/Role:** Enables bidirectional messaging via self-hosted Mattermost instances
- **Integration Point/Clues:**
  - `gateway/platforms/mattermost.py`
  - `hermes_cli/setup.py`: `MATTERMOST_TOKEN`, `MATTERMOST_URL`
  - Tests: `tests/gateway/test_mattermost.py`
  - *(Uses `aiohttp` for HTTP communication — no dedicated SDK listed)*

---

### 2.7 WhatsApp Bridge (Baileys)
- **Dependency Name:** WhatsApp (via Baileys bridge)
- **Type:** Third-party API / Messaging Platform
- **Purpose/Role:** Enables bidirectional WhatsApp messaging via a Node.js Baileys bridge process
- **Integration Point/Clues:**
  - `scripts/whatsapp-bridge/package.json`: `"@whiskeysockets/baileys": "7.0.0-rc.9"`
  - `gateway/platforms/whatsapp.py`
  - `hermes_cli/setup.py`: `WHATSAPP_ENABLED`
  - Tests: `tests/gateway/test_whatsapp_*.py`

---

### 2.8 DingTalk (钉钉)
- **Dependency Name:** DingTalk Stream API
- **Type:** Third-party API / Messaging Platform
- **Purpose/Role:** DingTalk enterprise messaging integration
- **Integration Point/Clues:**
  - `pyproject.toml` (dingtalk): `"dingtalk-stream>=0.1.0,<1"`
  - `gateway/platforms/dingtalk.py`
  - Tests: `tests/gateway/test_dingtalk.py`

---

### 2.9 Feishu / Lark
- **Dependency Name:** Feishu / Lark API
- **Type:** Third-party API / Messaging Platform
- **Purpose/Role:** Feishu/Lark enterprise messaging integration
- **Integration Point/Clues:**
  - `pyproject.toml` (feishu): `"lark-oapi>=1.5.3,<2"`
  - `gateway/platforms/feishu.py`
  - Tests: `tests/gateway/test_feishu.py`

---

### 2.10 WeCom (企业微信)
- **Dependency Name:** WeCom API
- **Type:** Third-party API / Messaging Platform
- **Purpose/Role:** WeCom (WeChat Work) enterprise messaging integration
- **Integration Point/Clues:**
  - `gateway/platforms/wecom.py`
  - Tests: `tests/gateway/test_wecom.py`
  - *(Likely uses `aiohttp` for HTTP — no dedicated SDK in pyproject.toml)*

---

### 2.11 Signal (via signal-cli / REST API)
- **Dependency Name:** Signal Messenger
- **Type:** Third-party API / Messaging Platform
- **Purpose/Role:** Signal messaging integration
- **Integration Point/Clues:**
  - `gateway/platforms/signal.py`
  - `hermes_cli/setup.py`: `SIGNAL_ACCOUNT`
  - Tests: `tests/gateway/test_signal.py`

---

### 2.12 SMS Platform
- **Dependency Name:** SMS Platform (unspecified provider)
- **Type:** Third-party API / Messaging Platform
- **Purpose/Role:** SMS messaging integration
- **Integration Point/Clues:**
  - `pyproject.toml` (sms): `"aiohttp>=3.9.0,<4"`
  - `gateway/platforms/sms.py`
  - Tests: `tests/gateway/test_sms.py`

---

### 2.13 Email
- **Dependency Name:** Email (SMTP/IMAP)
- **Type:** External Service / Messaging Platform
- **Purpose/Role:** Email messaging integration
- **Integration Point/Clues:**
  - `gateway/platforms/email.py`
  - Tests: `tests/gateway/test_email.py`

---

## 3. Cloud Compute & Execution Environments

### 3.1 Modal Cloud
- **Dependency Name:** Modal (Serverless Cloud Compute)
- **Type:** Cloud Service / Compute Provider
- **Purpose/Role:** Serverless cloud sandboxes for running agent terminal commands in isolated containers
- **Integration Point/Clues:**
  - `pyproject.toml` (modal): `"modal>=1.0.0,<2"`
  - `tools/environments/modal.py`
  - `hermes_cli/setup.py`: `MODAL_TOKEN_ID`, `MODAL_TOKEN_SECRET`
  - Tests: `tests/tools/test_modal_sandbox_fixes.py`, `tests/integration/test_modal_terminal.py`

---

### 3.2 Daytona Cloud
- **Dependency Name:** Daytona Cloud Development Environments
- **Type:** Cloud Service / Compute Provider
- **Purpose/Role:** Persistent cloud development sandboxes for agent terminal execution
- **Integration Point/Clues:**
  - `pyproject.toml` (daytona): `"daytona>=0.148.0,<1"`
  - `tools/environments/daytona.py`
  - `hermes_cli/setup.py`: `DAYTONA_API_KEY`, `TERMINAL_DAYTONA_IMAGE`
  - Tests: `tests/tools/test_daytona_environment.py`

---

### 3.3 Docker
- **Dependency Name:** Docker
- **Type:** External Service / Container Runtime
- **Purpose/Role:** Containerized terminal execution backend for isolated command execution
- **Integration Point/Clues:**
  - `tools/environments/docker.py`
  - `hermes_cli/setup.py`: `TERMINAL_DOCKER_IMAGE`
  - `Dockerfile`: base image `debian:13.4`
  - Tests: `tests/tools/test_docker_environment.py`

---

### 3.4 Singularity / Apptainer (HPC Containers)
- **Dependency Name:** Singularity / Apptainer
- **Type:** External Service / Container Runtime
- **Purpose/Role:** HPC-friendly container execution backend
- **Integration Point/Clues:**
  - `tools/environments/singularity.py`
  - `hermes_cli/setup.py`: `TERMINAL_SINGULARITY_IMAGE`
  - Tests: `tests/tools/test_singularity_preflight.py`

---

## 4. Web Search & Data Tools

### 4.1 Exa AI Search
- **Dependency Name:** Exa AI Search API
- **Type:** Third-party API / Web Search
- **Purpose/Role:** Semantic web search for the agent's web research capabilities
- **Integration Point/Clues:**
  - `pyproject.toml`: `"exa-py>=2.9.0,<3"`
  - `hermes_cli/setup.py`: `EXA_API_KEY`
  - `tools/web_tools.py`

---

### 4.2 Firecrawl
- **Dependency Name:** Firecrawl Web Scraping API
- **Type:** Third-party API / Web Scraping
- **Purpose/Role:** Web page extraction and crawling for the agent
- **Integration Point/Clues:**
  - `pyproject.toml`: `"firecrawl-py>=4.16.0,<5"`
  - `requirements.txt`: `firecrawl-py`
  - `hermes_cli/setup.py`: `FIRECRAWL_API_KEY`, `FIRECRAWL_API_URL`
  - `tools/web_tools.py`

---

### 4.3 Parallel Web Search
- **Dependency Name:** Parallel Web Search API
- **Type:** Third-party API / Web Search
- **Purpose/Role:** Web search capability for the agent
- **Integration Point/Clues:**
  - `pyproject.toml`: `"parallel-web>=0.4.2,<1"`
  - `requirements.txt`: `parallel-web>=0.4.2`
  - `hermes_cli/setup.py`: `PARALLEL_API_KEY`

---

### 4.4 Tavily Search
- **Dependency Name:** Tavily Search API
- **Type:** Third-party API / Web Search
- **Purpose/Role:** Web search capability for the agent
- **Integration Point/Clues:**
  - `hermes_cli/setup.py`: `TAVILY_API_KEY`
  - `tests/tools/test_web_tools_tavily.py`

---

## 5. Media / AI Generation Services

### 5.1 FAL AI (Image Generation)
- **Dependency Name:** FAL AI Image Generation API
- **Type:** Third-party API / AI Image Generation
- **Purpose/Role:** AI image generation for the agent
- **Integration Point/Clues:**
  - `pyproject.toml`: `"fal-client>=0.13.1,<1"`
  - `tools/image_generation_tool.py`
  - `hermes_cli/setup.py`: `FAL_KEY`

---

### 5.2 ElevenLabs TTS
- **Dependency Name:** ElevenLabs Text-to-Speech API
- **Type:** Third-party API / Text-to-Speech
- **Purpose/Role:** Premium TTS voice synthesis
- **Integration Point/Clues:**
  - `pyproject.toml` (tts-premium): `"elevenlabs>=1.0,<2"`
  - `tools/tts_tool.py`
  - `hermes_cli/setup.py`: `ELEVENLABS_API_KEY`

---

### 5.3 Edge TTS (Microsoft)
- **Dependency Name:** Microsoft Edge TTS
- **Type:** Third-party API / Text-to-Speech
- **Purpose/Role:** Free cloud-based TTS using Microsoft's Edge neural voices
- **Integration Point/Clues:**
  - `pyproject.toml`: `"edge-tts>=7.2.7,<8"`
  - `requirements.txt`: `edge-tts`
  - `hermes_cli/setup.py`: Default TTS provider

---

### 5.4 NeuTTS (Local TTS)
- **Dependency Name:** NeuTTS (Local On-Device TTS)
- **Type:** Library / Text-to-Speech
- **Purpose/Role:** Local on-device TTS using neural synthesis (~300MB model download)
- **Integration Point/Clues:**
  - `hermes_cli/setup.py`: TTS provider option, installs `neutts[all]`
  - `tools/neutts_synth.py`

---

## 6. Browser Automation

### 6.1 Agent Browser (Playwright-based)
- **Dependency Name:** `agent-browser` npm package
- **Type:** Library / Browser Automation
- **Purpose/Role:** Playwright-based browser automation for web tasks
- **Integration Point/Clues:**
  - `package.json`: `"agent-browser": "^0.13.0"`
  - `tools/browser_tool.py`
  - `hermes_cli/setup.py`: checks for `agent-browser` binary

---

### 6.2 Camoufox Browser
- **Dependency Name:** Camoufox Anti-Detection Browser
- **Type:** Library / Browser Automation
- **Purpose/Role:** Stealth browser automation with fingerprint evasion
- **Integration Point/Clues:**
  - `package.json`: `"@askjo/camoufox-browser": "^1.0.0"`
  - `tools/browser_camofox.py`, `browser_camofox_state.py`
  - `hermes_cli/setup.py`: `CAMOFOX_URL`
  - Tests: `tests/tools/test_browser_camofox*.py`

---

### 6.3 Browserbase Cloud Browser
- **Dependency Name:** Browserbase Cloud Browser
- **Type:** Third-party API / Cloud Browser Automation
- **Purpose/Role:** Cloud-hosted browser sessions for automation
- **Integration Point/Clues:**
  - `tools/browser_providers/browserbase.py`
  - `hermes_cli/setup.py`: `BROWSERBASE_API_KEY`

---

### 6.4 Playwright (via Dockerfile)
- **Dependency Name:** Playwright (Chromium)
- **Type:** Library / Browser Automation Runtime
- **Purpose/Role:** Underlying browser automation engine
- **Integration Point/Clues:**
  - `Dockerfile`: `npx playwright install --with-deps chromium --only-shell`

---

## 7. IoT / Smart Home

### 7.1 Home Assistant
- **Dependency Name:** Home Assistant API
- **Type:** Third-party API / Smart Home Platform
- **Purpose/Role:** Control smart home devices via Home Assistant REST/WebSocket API
- **Integration Point/Clues:**
  - `pyproject.toml` (homeassistant): `"aiohttp>=3.9.0,<4"`
  - `tools/homeassistant_tool.py`
  - `gateway/platforms/homeassistant.py`
  - `hermes_cli/setup.py`: `HASS_TOKEN`
  - Tests: `tests/tools/test_homeassistant_tool.py`, `tests/integration/test_ha_integration.py`

---

## 8. Memory & Persistence Services

### 8.1 Honcho AI Memory
- **Dependency Name:** Honcho AI (Memory Service)
- **Type:** Third-party API / Memory/Persistence Service
- **Purpose/Role:** Persistent user memory and session context management
- **Integration Point/Clues:**
  - `pyproject.toml` (honcho): `"honcho-ai>=2.0.1,<3"`
  - `honcho_integration/` directory (`client.py`, `session.py`, `cli.py`)
  - `tools/honcho_tools.py`
  - Tests: `tests/honcho_integration/`, `tests/tools/test_honcho_tools.py`

---

## 9. AI Training & ML Infrastructure

### 9.1 Atropos (RL Training Framework)
- **Dependency Name:** Atropos RL Training Framework (NousResearch)
- **Type:** Library / RL Training Framework (Internal to Nous Research)
- **Purpose/Role:** Reinforcement learning training environment/framework
- **Integration Point/Clues:**
  - `pyproject.toml` (rl): `"atroposlib @ git+https://github.com/NousResearch/atropos.git"`
  - `tools/rl_training_tool.py`

---

### 9.2 Tinker (RL Training Platform)
- **Dependency Name:** Tinker RL Training Platform
- **Type:** Third-party API / RL Training Platform
- **Purpose/Role:** RL training orchestration platform

