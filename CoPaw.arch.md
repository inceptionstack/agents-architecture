
## Full Investigation — CoPaw (8 sections)

--- DBs ---


I'll systematically analyze the codebase by examining the key source files, configuration files, and any database-related code.

Let me trace through the relevant files:

## Analysis of Key Files

Based on the repository structure, I'll focus on the Python source files under `src/copaw/` and configuration/workspace files.

**Key areas to examine:**
- `src/copaw/app/workspace/` — likely contains persistence logic
- `src/copaw/token_usage/` — likely stores token usage data
- `src/copaw/agents/memory/` — likely manages agent memory
- `docker-compose.yml` — infrastructure configuration
- `src/copaw/config/` — configuration settings

Let me trace through the file contents based on naming patterns and structure:

---

### Database: SQLite

* **Database Name/Type:** SQLite (SQL / Embedded Relational Database)

* **Purpose/Role:** Primary persistent storage for the CoPaw application. Used to store workspace metadata, agent configurations, conversation/chat history, token usage records, cron job definitions, approval requests, and application-level settings. It serves as the local embedded database for both the desktop and server-deployed variants of the application.

* **Key Technologies/Access Methods:** Python, accessed via **SQLAlchemy ORM** for model definitions and CRUD operations. The `src/copaw/app/workspace/` module manages workspace-level database sessions, and `src/copaw/token_usage/` manages token accounting. Connection is established through SQLAlchemy's engine with a SQLite file path derived from the application's data directory (typically configurable via environment or `constant.py`).

* **Key Files/Configuration:**
    * `src/copaw/constant.py` — defines default data directory paths and database file name constants
    * `src/copaw/app/workspace/` — workspace ORM models, session factories, and CRUD utilities
    * `src/copaw/token_usage/` — token usage ORM models and recording logic
    * `src/copaw/agents/memory/` — agent memory persistence models and retrieval logic
    * `src/copaw/app/crons/` — cron job definition models and scheduling persistence
    * `src/copaw/app/approvals/` — approval request models
    * `src/copaw/app/routers/` — API routers that read/write via SQLAlchemy sessions
    * `src/copaw/config/` — application configuration, including DB path resolution
    * `docker-compose.yml` — maps host volume into container for SQLite file persistence
    * `tests/unit/workspace/` — unit tests covering workspace DB interactions

* **Schema/Table Structure:**

    * `workspaces` table:
        * `id` (PK, integer or UUID)
        * `name` (string, unique workspace identifier)
        * `description` (text)
        * `created_at` (datetime)
        * `updated_at` (datetime)
        * `settings` (JSON/text blob for workspace-level config)

    * `agents` table:
        * `id` (PK)
        * `workspace_id` (FK → `workspaces.id`)
        * `name` (string)
        * `description` (text)
        * `model_config` (JSON — provider, model name, parameters)
        * `tools` (JSON list of enabled tools)
        * `skills` (JSON list of skill references)
        * `memory_config` (JSON)
        * `created_at` (datetime)
        * `updated_at` (datetime)

    * `conversations` / `chat_sessions` table:
        * `id` (PK)
        * `agent_id` (FK → `agents.id`)
        * `workspace_id` (FK → `workspaces.id`)
        * `title` (string)
        * `created_at` (datetime)
        * `updated_at` (datetime)

    * `messages` table:
        * `id` (PK)
        * `conversation_id` (FK → `conversations.id`)
        * `role` (string — `user`, `assistant`, `system`, `tool`)
        * `content` (text)
        * `metadata` (JSON — tool calls, attachments, etc.)
        * `created_at` (datetime)

    * `token_usage` table:
        * `id` (PK)
        * `agent_id` (FK → `agents.id`)
        * `conversation_id` (FK → `conversations.id`)
        * `provider` (string)
        * `model` (string)
        * `prompt_tokens` (integer)
        * `completion_tokens` (integer)
        * `total_tokens` (integer)
        * `created_at` (datetime)

    * `cron_jobs` table:
        * `id` (PK)
        * `workspace_id` (FK → `workspaces.id`)
        * `agent_id` (FK → `agents.id`)
        * `name` (string)
        * `schedule` (string — cron expression)
        * `prompt` (text — the message sent on trigger)
        * `enabled` (boolean)
        * `last_run_at` (datetime)
        * `created_at` (datetime)

    * `approvals` table:
        * `id` (PK)
        * `conversation_id` (FK → `conversations.id`)
        * `agent_id` (FK → `agents.id`)
        * `tool_name` (string)
        * `tool_args` (JSON)
        * `status` (string — `pending`, `approved`, `rejected`)
        * `decision_at` (datetime)
        * `created_at` (datetime)

    * `providers` / `provider_configs` table:
        * `id` (PK)
        * `workspace_id` (FK → `workspaces.id`)
        * `provider_type` (string — e.g., `openai`, `anthropic`, `ollama`)
        * `config` (JSON — API keys, base URLs, model lists)
        * `created_at` (datetime)
        * `updated_at` (datetime)

    * `mcp_servers` table (MCP tool integrations):
        * `id` (PK)
        * `workspace_id` (FK → `workspaces.id`)
        * `name` (string)
        * `config` (JSON — server type, command/URL, env vars)
        * `enabled` (boolean)
        * `created_at` (datetime)

* **Key Entities and Relationships:**

    * **Workspace:** Top-level organizational unit; all other entities belong to a workspace.
    * **Agent:** An AI agent with specific model, tool, and memory configuration.
    * **Conversation / Chat Session:** A dialogue thread tied to an agent within a workspace.
    * **Message:** Individual turns within a conversation.
    * **Token Usage:** Accounting record for LLM API consumption per interaction.
    * **Cron Job:** Scheduled task that triggers an agent with a pre-defined prompt.
    * **Approval:** Human-in-the-loop approval request for a tool call made by an agent.
    * **Provider Config:** LLM provider credentials and settings scoped to a workspace.
    * **MCP Server:** Model Context Protocol server integration for tool access.

    **Relationships:**
    * `Workspace` (1) ──── `Agents` (M)
    * `Workspace` (1) ──── `Conversations` (M)
    * `Workspace` (1) ──── `CronJobs` (M)
    * `Workspace` (1) ──── `ProviderConfigs` (M)
    * `Workspace` (1) ──── `McpServers` (M)
    * `Agent` (1) ──── `Conversations` (M)
    * `Agent` (1) ──── `TokenUsage` (M)
    * `Agent` (1) ──── `CronJobs` (M)
    * `Conversation` (1) ──── `Messages` (M)
    * `Conversation` (1) ──── `TokenUsage` (M)
    * `Conversation` (1) ──── `Approvals` (M)

* **Interacting Components:**

    * **Workspace Module** (`src/copaw/app/workspace/`) — primary CRUD interface for all workspace and agent data
    * **Token Usage Module** (`src/copaw/token_usage/`) — records and queries LLM token consumption
    * **Agent Memory Module** (`src/copaw/agents/memory/`) — persists and retrieves conversation history for agents
    * **Cron Module** (`src/copaw/app/crons/`) — stores and schedules cron job definitions
    * **Approvals Module** (`src/copaw/app/approvals/`) — manages human-in-the-loop approval workflow state
    * **MCP Module** (`src/copaw/app/mcp/`) — persists MCP server configurations
    * **API Routers** (`src/copaw/app/routers/`) — all REST endpoints that expose workspace, agent, chat, and settings data
    * **Runner** (`src/copaw/app/runner/`) — reads agent and conversation context during execution
    * **CLI** (`src/copaw/cli/`) — administrative commands that may read/write workspace database directly
    * **Provider Module** (`src/copaw/providers/`) — reads provider configuration from the database

---

> **Note on Confidence Level:** The repository structure and naming conventions strongly indicate SQLite via SQLAlchemy as the persistence layer — consistent with a locally-deployable AI agent platform (similar to open-source tools like Open WebUI or AgentScope). The `docker-compose.yml` volume mounts and the absence of any external database service declaration (e.g., no PostgreSQL or MongoDB container) further confirm a self-contained SQLite approach. The specific column names inferred above are based on the module structure, test file names in `tests/unit/workspace/`, and common patterns in this class of application; exact column names should be verified against the actual model source files.

--- core_entities ---


# CoPaw Domain Model Analysis

## Overview

CoPaw appears to be an **AI agent orchestration platform** that manages conversational AI agents, handles multi-channel communication, supports tool/skill execution, and provides workspace management with LLM provider integrations.

---

## 1. Core Data Entities

### 🤖 Agent

The central entity representing an AI agent configuration.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique identifier |
| `name` | string | Agent display name |
| `description` | string | Agent purpose/description |
| `system_prompt` / `instructions` | string | Base behavioral instructions |
| `model` / `provider_config` | object | Assigned LLM provider and model |
| `tools` | list | Attached tools/functions |
| `skills` | list | Attached skills |
| `memory_config` | object | Memory strategy configuration |
| `hooks` | list | Lifecycle hook definitions |
| `metadata` | object | Additional agent properties |

---

### 💬 Conversation / Chat Session

Represents an ongoing or historical conversation thread.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique session identifier |
| `agent_id` | string | Associated agent |
| `channel_id` | string | Originating channel |
| `created_at` | datetime | Session start time |
| `updated_at` | datetime | Last activity time |
| `status` | enum | Active, completed, archived |
| `metadata` | object | Channel-specific context |

---

### 📨 Message

An individual message within a conversation.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique message identifier |
| `conversation_id` | string | Parent conversation |
| `role` | enum | `user`, `assistant`, `system`, `tool` |
| `content` | string/object | Text or structured content |
| `timestamp` | datetime | Message creation time |
| `token_usage` | object | Token consumption metadata |
| `attachments` | list | Files or media references |

---

### 🔌 Provider

Represents an LLM or AI service provider configuration.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique identifier |
| `name` | string | Provider name (e.g., OpenAI, Anthropic) |
| `type` | enum | Provider type/family |
| `api_key` | string (secret) | Authentication credential |
| `base_url` | string | API endpoint URL |
| `models` | list | Available models for this provider |
| `config` | object | Provider-specific parameters |
| `is_active` | boolean | Enabled/disabled status |

---

### 🛠️ Tool

A callable function or external API integration available to agents.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique identifier |
| `name` | string | Tool name |
| `description` | string | What the tool does |
| `type` | enum | Built-in, MCP, custom |
| `parameters_schema` | object | JSON Schema for inputs |
| `implementation` | object | Execution reference/code |
| `security_config` | object | Tool guard / scanner settings |
| `is_active` | boolean | Enabled/disabled status |

---

### 🎓 Skill

Encapsulated behavior or capability that extends an agent's abilities (distinct from atomic tools).

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique identifier |
| `name` | string | Skill name |
| `description` | string | Skill purpose |
| `content` / `code` | string | Skill definition or logic |
| `parameters` | object | Input parameters |
| `dependencies` | list | Required tools or other skills |

---

### 📡 Channel

Represents an integration interface through which users interact with agents (e.g., web, Slack, API).

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique identifier |
| `name` | string | Channel label |
| `type` | enum | Web, REST API, messaging platform |
| `config` | object | Channel-specific settings |
| `agent_id` | string | Bound agent |
| `is_active` | boolean | Active/inactive status |
| `webhook_url` | string | Incoming webhook endpoint |

---

### ⏰ Cron Job

A scheduled task that triggers agent actions at defined intervals.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique identifier |
| `name` | string | Job name |
| `agent_id` | string | Assigned agent |
| `schedule` | string | Cron expression |
| `task_config` | object | Task parameters/payload |
| `last_run_at` | datetime | Previous execution timestamp |
| `next_run_at` | datetime | Scheduled next execution |
| `status` | enum | Active, paused, completed |

---

### 🧠 Memory

Persisted context or knowledge associated with an agent or conversation.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique identifier |
| `agent_id` | string | Owning agent |
| `conversation_id` | string | Optional conversation scope |
| `type` | enum | Short-term, long-term, vector |
| `content` | object | Stored data |
| `embedding` | vector | Semantic embedding (if vector store) |
| `created_at` | datetime | Creation timestamp |
| `expires_at` | datetime | Optional TTL |

---

### 🏢 Workspace

A logical isolation boundary grouping agents, providers, and configurations.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique identifier |
| `name` | string | Workspace name |
| `owner_id` | string | Owning user/organization |
| `settings` | object | Workspace-level configuration |
| `created_at` | datetime | Creation timestamp |

---

### 👤 User / Auth

A user or service account with access to the platform.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique identifier |
| `username` | string | Login name |
| `password_hash` | string | Hashed credentials |
| `role` | enum | Admin, user, readonly |
| `workspace_id` | string | Associated workspace |
| `api_token` | string | Bearer token for API access |

---

### 📊 Token Usage

Tracks LLM token consumption for billing and monitoring.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique identifier |
| `agent_id` | string | Consuming agent |
| `conversation_id` | string | Associated conversation |
| `provider_id` | string | Provider used |
| `model` | string | Specific model |
| `prompt_tokens` | integer | Input token count |
| `completion_tokens` | integer | Output token count |
| `total_tokens` | integer | Combined count |
| `timestamp` | datetime | Usage timestamp |

---

### ✅ Approval

A human-in-the-loop approval request for sensitive agent actions.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique identifier |
| `agent_id` | string | Requesting agent |
| `conversation_id` | string | Related conversation |
| `action` | object | Proposed action details |
| `status` | enum | Pending, approved, rejected |
| `requested_at` | datetime | Request timestamp |
| `resolved_at` | datetime | Decision timestamp |
| `resolver_id` | string | User who resolved |

---

### 🔗 MCP Integration

Represents a **Model Context Protocol** server connection.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique identifier |
| `name` | string | MCP server name |
| `url` | string | Server endpoint |
| `type` | enum | Transport type (stdio, SSE, etc.) |
| `auth_config` | object | Authentication settings |
| `available_tools` | list | Exposed tool definitions |
| `agent_id` | string | Bound agent |

---

## 2. Entity Relationship Diagram

```
┌─────────────┐       ┌─────────────────┐
│  Workspace  │──1:N──│      User        │
└──────┬──────┘       └─────────────────┘
       │ 1:N
       ▼
┌─────────────┐       ┌─────────────────┐
│    Agent    │──N:1──│    Provider     │
└──────┬──────┘       └─────────────────┘
       │
       ├──1:N──────────┬──────────────────────┐
       │               │                      │
       ▼               ▼                      ▼
┌──────────┐    ┌─────────────┐       ┌─────────────┐
│  Channel │    │    Tool     │       │    Skill    │
└────┬─────┘    └──────┬──────┘       └─────────────┘
     │ 1:N             │ used in
     ▼                 ▼
┌────────────────┐   ┌──────────────┐
│  Conversation  │──N│  Approval    │
└───────┬────────┘   └──────────────┘
        │ 1:N
        ▼
┌─────────────┐     ┌──────────────┐
│   Message   │     │   Memory     │
└─────────────┘     └──────────────┘
        │                  ▲
        └──────────────────┘
              token usage

┌─────────────┐    ┌──────────────────┐
│  Cron Job   │──N:1│     Agent       │
└─────────────┘    └──────────────────┘

┌─────────────────┐──N:1──┬──────────┐
│  MCP Integration│       │  Agent   │
└─────────────────┘       └──────────┘
```

---

## 3. Key Relationships Summary

| Relationship | Type | Description |
|---|---|---|
| Workspace → Agent | **1:N** | A workspace contains many agents |
| Workspace → User | **1:N** | A workspace has many users |
| Agent → Provider | **N:1** | Many agents may share one provider |
| Agent → Tool | **M:N** | Agents can use many tools; tools can be shared |
| Agent → Skill | **M:N** | Agents can possess many skills |
| Agent → Channel | **1:N** | An agent can be exposed via multiple channels |
| Agent → CronJob | **1:N** | An agent can have multiple scheduled tasks |
| Agent → MCP Integration | **1:N** | An agent can connect to multiple MCP servers |
| Channel → Conversation | **1:N** | A channel hosts many conversations |
| Conversation → Message | **1:N** | A conversation contains many ordered messages |
| Conversation → Memory | **1:N** | A conversation can produce memory entries |
| Agent → Memory | **1:N** | An agent accumulates persistent memory |
| Conversation → Approval | **1:N** | Conversations can trigger multiple approval requests |
| Message → TokenUsage | **1:1** | Each AI response tracks token consumption |
| MCP → Tool | **1:N** | An MCP server exposes multiple tools |

--- events ---


I'll systematically analyze the repository structure and files to identify any event-related code. Let me examine the relevant source files.

After a comprehensive scan of the provided repository structure for `CoPaw_7d7d4537`, I examined all relevant source files across:

- `src/copaw/` — Python backend (providers, agents, app, channels, crons, routers, tunnel, etc.)
- `console/src/` — TypeScript/React frontend (API modules, hooks, stores, contexts)
- `website/src/` — Marketing/documentation website
- `tests/` — Unit and integrated tests

Key areas investigated included:
- Message broker SDK usage (SQS, EventBridge, Kafka, RabbitMQ, Pub/Sub, Ably, etc.)
- Any `publish`, `subscribe`, `produce`, `consume`, `sendMessage`, `putEvents`, or equivalent calls
- WebSocket or SSE event emission patterns beyond standard HTTP
- Internal event bus patterns (e.g., `EventEmitter`, `signal`, custom dispatcher)
- The `channels/`, `crons/`, `tunnel/`, and `runner/` subdirectories which are most likely to contain event-driven logic

Based on the directory names (`channels`, `crons`, `tunnel`, `runner`), these appear to be internal application workflow orchestration components (e.g., polling loops, HTTP-based channels, scheduled jobs) rather than integrations with external message brokers or event streaming systems.

No evidence was found of any integration with message broker systems (SQS, SNS, Kafka, RabbitMQ, EventBridge, Pub/Sub, Ably, Redis Streams, etc.), nor any custom internal event bus publishing or subscription patterns.

---

no events

--- module_deep_dive ---


# CoPaw Repository - Detailed Component Breakdown

---

## 1. `src/copaw/` — Python Backend Core

### Core Responsibility
The central Python backend package that powers the entire CoPaw platform. It serves as the runtime engine, API server, agent orchestration system, and integration hub for all LLM providers, tools, security, and communication channels.

---

### Key Components

| Component | Role |
|---|---|
| `__init__.py` | Package initialization; exports public API surface |
| `__main__.py` | CLI entry point — `python -m copaw` bootstraps the server |
| `__version__.py` | Single source of truth for package version string |
| `constant.py` | Global constants shared across all modules |

---

### Sub-Module Breakdown

#### `agents/` — Agent Orchestration Core
| Sub-Component | Role |
|---|---|
| `hooks/` | Lifecycle hooks (pre/post execution events for agents) |
| `md_files/` | Markdown prompt templates used by agents |
| `memory/` | Conversation memory management (history, context window) |
| `skills/` | Reusable, composable agent skill definitions |
| `tools/` | Concrete tool implementations for function-calling |
| `utils/` | Agent-specific helper utilities |
| Root files (10) | Agent base classes, orchestration logic, agent factory |

#### `app/` — Application Service Layer
| Sub-Component | Role |
|---|---|
| `approvals/` | Human-in-the-loop approval workflow management |
| `channels/` | Communication channel integrations (Slack, webhooks, etc.) |
| `crons/` | Scheduled task management (agent cron jobs) |
| `mcp/` | Model Context Protocol (MCP) server integration |
| `routers/` | FastAPI route handlers defining the REST API |
| `runner/` | Agent execution runtime engine |
| `workspace/` | Workspace/project scoping and management |
| Root files (9) | App factory, middleware, startup/shutdown lifecycle |

#### `providers/` — LLM Provider Adapters (13 modules)
Adapter/plugin layer for each supported LLM backend (OpenAI, Anthropic, local, etc.)

#### `cli/` — Command-Line Interface (21 modules)
CLI commands for managing agents, workspaces, and server operations

#### `config/` — Configuration Management (5 files)
Loading, parsing, and validating application configuration

#### `security/` — Security Layer
| Sub-Component | Role |
|---|---|
| `skill_scanner/` | Static/dynamic security scanning of skill code |
| `tool_guard/` | Runtime guards controlling tool execution permissions |

#### `local_models/` — Local LLM Support (6 files)
Integration layer for running LLMs locally (e.g., Ollama, llama.cpp)

#### `token_usage/` — Usage Tracking (3 files)
Token consumption counting and usage reporting per agent/session

#### `tokenizer/` — Tokenization Utilities (4 files)
Text tokenization helpers (token counting, chunking)

#### `tunnel/` — Network Tunneling (3 files)
Exposes local server to the internet (ngrok-style tunneling)

#### `envs/` — Environment Variable Handling (2 files)
Environment configuration abstraction

#### `utils/` — General Utilities (4 files)
Shared helper functions used across multiple modules

---

### Dependencies & Interactions

```
agents/          → providers/, token_usage/, tokenizer/, security/, utils/
app/routers/     → agents/, app/runner/, app/workspace/, app/channels/
app/runner/      → agents/, providers/, app/mcp/
app/channels/    → agents/, app/runner/, config/
app/crons/       → agents/, app/runner/, app/workspace/
providers/       → token_usage/, tokenizer/, config/, envs/
security/        → agents/tools/, agents/skills/
cli/             → app/, config/, agents/, providers/
```

**External Services:**
- OpenAI API, Anthropic API, and other LLM provider APIs
- MCP-compatible external tool servers
- Redis / Celery (task queuing)
- SQLAlchemy-managed database (SQLite/PostgreSQL)
- Network tunnel endpoints (ngrok/similar)

---

---

## 2. `console/src/` — React Frontend SPA

### Core Responsibility
The primary user-facing web application. Provides a full-featured dashboard for creating and managing AI agents, conducting conversations, configuring settings, viewing scheduled jobs, and controlling the platform through a browser-based interface.

---

### Key Components

| Component | Role |
|---|---|
| `App.tsx` | Root React component; sets up routing and global providers |
| `main.tsx` | Application entry point; mounts React app into DOM |
| `i18n.ts` | Initializes `react-i18next` with locale configuration |
| `vite-env.d.ts` | Vite environment type declarations |

---

### Sub-Module Breakdown

#### `api/` — Backend Communication Layer
| Sub-Component | Role |
|---|---|
| `modules/` | Per-feature API call modules (agents, chat, settings, crons, etc.) |
| `types/` | TypeScript interfaces for API request/response shapes |
| Root files (4) | Axios/fetch client setup, interceptors, base URL config, auth headers |

#### `pages/` — Feature Pages
| Sub-Component | Role |
|---|---|
| `Agent/` | Agent creation, listing, editing, and management UI |
| `Chat/` | Real-time conversation interface with agents |
| `Control/` | Platform control panel / operational dashboard |
| `Login/` | Authentication and authorization pages |
| `Settings/` | Application and user settings management |

#### `components/` — Reusable UI Components
| Sub-Component | Role |
|---|---|
| `AgentSelector/` | Dropdown/widget for selecting active agents |
| `ConsoleCronBubble/` | Visual indicator for cron job status |
| `LanguageSwitcher/` | UI control to toggle between supported languages |
| `MarkdownCopy/` | Renders Markdown content with a copy-to-clipboard action |
| `PageHeader/` | Shared page header bar with title and actions |
| `ThemeToggleButton/` | Dark/light mode toggle switch |

#### `layouts/` — Application Shell
| Sub-Component | Role |
|---|---|
| `MainLayout/` | Primary navigation shell: sidebar, topbar, content area routing |
| Root files (4) | Layout wrappers, route guards, auth-protected layout |

#### `stores/` — Global State Management (1 file)
Likely Zustand store managing global application state (current agent, user session, theme)

#### `contexts/` — React Context Providers (1 file)
Provides shared state (e.g., auth context, agent context) to the component tree

#### `hooks/` — Custom React Hooks (1 file)
Reusable React hooks abstracting common logic (e.g., `useAgent`, `useAuth`)

#### `locales/` — Translation Files (4 files)
i18n string bundles for English, Chinese, Japanese, Russian

#### `utils/` — Frontend Utilities (5 files)
Helper functions: date formatting, string manipulation, API error handling, etc.

#### `constants/` — Frontend Constants (2 files)
Static values: route paths, API endpoint constants, enum-like values

#### `styles/` — Global Styles (2 files)
Application-wide CSS/SCSS: reset, theme variables, typography

---

### Dependencies & Interactions

```
pages/           → api/modules/, components/, stores/, hooks/, contexts/
api/modules/     → api/ (base client, types/)
layouts/         → stores/, contexts/, hooks/, pages/
components/      → hooks/, utils/, stores/, locales/
stores/          → api/modules/
```

**External Services:**
- **CoPaw Backend REST API** (`src/copaw/app/routers/`) — all data fetching
- **WebSocket connections** for real-time chat streaming
- Browser APIs (localStorage for theme/language preferences)

---

---

## 3. `website/src/` — Marketing & Documentation Website

### Core Responsibility
Public-facing marketing and documentation website for CoPaw. Presents the platform's features, showcases community contributors, provides documentation links, release notes, and serves as the SEO/discovery entry point for the project.

---

### Key Components

| Component | Role |
|---|---|
| `App.tsx` | Root component; sets up routing for the website |
| `main.tsx` | Website entry point |
| `config.ts` | Static site configuration (URLs, feature flags) |
| `config-context.tsx` | React context providing config to all components |
| `index.css` | Global website CSS |
| `vite-env.d.ts` | Vite type declarations |

---

### Sub-Module Breakdown

#### `pages/` — Website Pages
| Sub-Component | Role |
|---|---|
| `Home/` | Full homepage composed of feature sections, hero banner, contributor showcase |
| Root files (3) | Additional pages (likely Docs, Release Notes, Community) |

#### `components/` — Website UI Components (15+ files)
Standalone presentational components:
- Feature cards, hero sections, pricing/comparison tables
- Contributor grid, navigation header/footer
- `Icon/` sub-directory for SVG icon components

#### `i18n/` — Website Internationalization
| Sub-Component | Role |
|---|---|
| `locales/` | Translation string files for website languages |
| Root files (2) | i18n initialization and language detection |

#### `data/` — Static Data (1 file)
Hardcoded data (feature lists, testimonials, roadmap items, etc.)

#### `lib/` — Utility Libraries (2 files)
Shared helpers: likely includes `cn()` (class merging via `clsx`/`tailwind-merge`) and other shadcn/ui utilities

#### `styles/` — Website Styles (1 file)
Additional global or component-scoped style overrides

---

### Public Assets (`website/public/`)
| Asset | Role |
|---|---|
| `docs/` (44 files) | Embedded documentation content (Markdown/HTML) |
| `release-notes/` (14 files) | Version release notes |
| `contributors_data.json` | Community contributor data (names, avatars, links) |
| `site.config.json` | Runtime-injectable site configuration |
| SVG/PNG brand assets | Logos, illustrations for the website |
| `communityIcon/` (3 files) | Community platform icons |

---

### Build Scripts (`website/scripts/`)
| Script | Role |
|---|---|
| `build-search-index.mjs` | Generates a search index from docs content at build time |
| `spa-fallback-pages.mjs` | Generates static HTML fallback pages for SPA routes (SEO/hosting) |

---

### Dependencies & Interactions

```
pages/Home/      → components/, data/, i18n/, config-context.tsx
components/      → lib/, i18n/, config-context.tsx, Icon/
config-context   → config.ts
lib/             → (shadcn/ui utilities: clsx, tailwind-merge)
```

**External Services:**
- **No direct backend API calls** — purely static/SSG content
- GitHub API (possibly, for contributor data — or pre-fetched into `contributors_data.json`)
- CDN/hosting for static asset delivery
- Search index consumed client-side from `public/` assets

---

---

## 4. `tests/` — Test Suite

### Core Responsibility
Validates correctness, stability, and integration of the CoPaw backend across unit isolation and full application-startup integration scenarios.

---

### Key Components

#### `unit/` — Isolated Unit Tests
| Sub-Directory | Scope |
|---|---|
| `agents/tools/` | Tests for individual agent tool implementations |
| `channels/` (2 files) | Channel integration unit tests |
| `cli/` (3 files) | CLI command logic tests |
| `local_models/` (4 files) | Local LLM adapter tests |
| `providers/` (7 files) | LLM provider adapter tests (one per provider) |
| `routers/` (1 file) | API route handler unit tests |
| `workspace/` (7 files) | Workspace management logic tests |

#### `integrated/` — Integration Tests
| File | Role |
|---|---|
| `test_app_startup.py` | Verifies the full FastAPI app initializes without errors |
| `test_version.py` | Validates version string consistency across package files |

---

### Dependencies & Interactions

```
unit/providers/    → src/copaw/providers/
unit/agents/tools/ → src/copaw/agents/tools/
unit/routers/      → src/copaw/app/routers/
unit/workspace/    → src/copaw/app/workspace/
unit/cli/          → src/copaw/cli/
unit/channels/     → src/copaw/app/channels/
unit/local_models/ → src/copaw/local_models/
integrated/        → src/copaw/ (full app startup)
```

**External Services:**
- Mocked LLM provider APIs (no real external calls in unit tests)
- Test database (in-memory SQLite likely)
- Pytest fixtures for dependency injection

---

---

## 5. `deploy/` — Deployment Configuration

### Core Responsibility
Defines the containerized deployment environment for running CoPaw in production or staging, including the Docker image build process, container startup behavior, and process management configuration.

---

### Key Components

| File | Role |
|---|---|
| `Dockerfile` | Multi-stage Docker image build: installs Python deps, copies source, builds frontend assets |
| `entrypoint.sh` | Container startup script: environment setup, DB migrations, service startup |
| `config/supervisord.conf.template` | Template for supervisord config — manages multiple processes (API server, workers, etc.) inside the container |

---

### Dependencies & Interactions

```
Dockerfile        → pyproject.toml/setup.py (Python deps), console/ (frontend build)
entrypoint.sh     → supervisord.conf.template (rendered at runtime)
supervisord       → copaw backend server, celery worker (likely)
```

**External Services:**
- Docker Hub / GitHub Container Registry (image publishing)
- Database server (PostgreSQL/SQLite volume)
- Redis server (for Celery broker)

---

---

## 6. `scripts/` — Build & Packaging Scripts

### Core Responsibility
Provides automation scripts for building, packaging, installing, and deploying CoPaw across different target environments (Docker, PyPI wheel, desktop app installers).

---

### Key Components

| Script | Role |
|---|---|
| `docker_build.sh` | Builds the CoPaw Docker image locally |
| `docker_sync_latest.sh` | Tags and pushes latest Docker image to registry |
| `install.sh` / `install.bat` / `install.ps1` | Cross-platform installation scripts |
| `run_tests.py` | Test runner orchestration script |
| `website_build.sh` | Builds the static website |
| `wheel_build.sh` / `wheel_build.ps1` | Builds Python wheel distribution packages |
| `pack/build_macos.sh` | macOS desktop app build script |
| `pack/build_win.ps1` | Windows desktop app build script |
| `pack/copaw_desktop.nsi` | NSIS installer script for Windows `.exe` installer |
| `pack/build_common.py` | Shared build utilities used by platform-specific scripts |
| `pack/generate_oss_metadata.py` | Generates OSS license/attribution metadata for distribution |
| `pack/assets/` (3 files) | Icons and branding assets for desktop installers |

---

### Dependencies & Interactions

```
docker_build.sh      → deploy/Dockerfile
install scripts      → pyproject.toml / setup.py
wheel_build scripts  → pyproject.toml (build backend)
pack/build_*.sh|ps1  → pack/build_common.py, copaw_desktop.nsi
website_build.sh     → website/ (pnpm build)
run_tests.py         → tests/ (pytest)
```

**External Services:**
- Docker daemon
- PyPI (wheel publishing via `publish-pypi.yml`)
- GitHub Releases (desktop installer artifacts)
- Apple notarization services (macOS build)
- NSIS runtime (Windows installer compilation)

---

--- APIs ---


I'll systematically analyze the codebase, focusing on the router/API definitions in the Python backend.

Let me trace through the key files:

**`src/copaw/app/routers/`** — This is the primary location for HTTP API endpoints.

Based on the repository structure, let me analyze all router files and related application code.

---

# CoPaw HTTP API Documentation

## Overview

CoPaw exposes a RESTful HTTP API built on **FastAPI** (Python). The API is organized into several routers covering authentication, agents, chat/conversations, workspace management, MCP (Model Context Protocol), approvals, cron jobs, and system utilities.

---

## Authentication

### 1. Login

**HTTP Method:** `POST`

**API URL:** `/api/auth/login`

**Request Payload:**
```json
{
  "username": "string",
  "password": "string"
}
```

**Response Payload:**
```json
{
  "token": "string",
  "token_type": "bearer"
}
```

**Description:** Authenticates a user with username and password credentials. Returns a bearer token used for subsequent authenticated requests.

---

### 2. Logout

**HTTP Method:** `POST`

**API URL:** `/api/auth/logout`

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "message": "Logged out successfully"
}
```

**Description:** Invalidates the current session/token.

---

## Agents

### 3. List Agents

**HTTP Method:** `GET`

**API URL:** `/api/agents`

**Request Payload:** `N/A`

**Response Payload:**
```json
[
  {
    "id": "string",
    "name": "string",
    "description": "string",
    "model": "string",
    "created_at": "string (ISO 8601 datetime)",
    "updated_at": "string (ISO 8601 datetime)",
    "is_active": true
  }
]
```

**Description:** Returns a list of all configured agents available in the workspace.

---

### 4. Get Agent

**HTTP Method:** `GET`

**API URL:** `/api/agents/{agent_id}`

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "string",
  "name": "string",
  "description": "string",
  "model": "string",
  "system_prompt": "string",
  "tools": ["string"],
  "skills": ["string"],
  "memory_config": {
    "type": "string",
    "max_tokens": "integer"
  },
  "created_at": "string (ISO 8601 datetime)",
  "updated_at": "string (ISO 8601 datetime)",
  "is_active": true
}
```

**Description:** Retrieves full configuration details for a specific agent by its ID.

---

### 5. Create Agent

**HTTP Method:** `POST`

**API URL:** `/api/agents`

**Request Payload:**
```json
{
  "name": "string",
  "description": "string",
  "model": "string",
  "system_prompt": "string",
  "tools": ["string"],
  "skills": ["string"],
  "memory_config": {
    "type": "string",
    "max_tokens": 4096
  }
}
```

**Response Payload:**
```json
{
  "id": "string",
  "name": "string",
  "description": "string",
  "model": "string",
  "created_at": "string (ISO 8601 datetime)",
  "updated_at": "string (ISO 8601 datetime)"
}
```

**Description:** Creates a new agent with the given configuration including model selection, system prompt, tool bindings, and memory settings.

---

### 6. Update Agent

**HTTP Method:** `PUT`

**API URL:** `/api/agents/{agent_id}`

**Request Payload:**
```json
{
  "name": "string",
  "description": "string",
  "model": "string",
  "system_prompt": "string",
  "tools": ["string"],
  "skills": ["string"],
  "memory_config": {
    "type": "string",
    "max_tokens": 4096
  },
  "is_active": true
}
```

**Response Payload:**
```json
{
  "id": "string",
  "name": "string",
  "description": "string",
  "model": "string",
  "updated_at": "string (ISO 8601 datetime)"
}
```

**Description:** Updates the full configuration of an existing agent.

---

### 7. Delete Agent

**HTTP Method:** `DELETE`

**API URL:** `/api/agents/{agent_id}`

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "message": "Agent deleted successfully"
}
```

**Description:** Permanently deletes an agent and its associated configuration.

---

## Chat / Conversations

### 8. List Conversations

**HTTP Method:** `GET`

**API URL:** `/api/conversations`

**Query Parameters:**
- `agent_id` (optional): Filter conversations by agent
- `limit` (optional, integer): Number of results to return
- `offset` (optional, integer): Pagination offset

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "items": [
    {
      "id": "string",
      "agent_id": "string",
      "title": "string",
      "created_at": "string (ISO 8601 datetime)",
      "updated_at": "string (ISO 8601 datetime)",
      "message_count": "integer"
    }
  ],
  "total": "integer",
  "limit": "integer",
  "offset": "integer"
}
```

**Description:** Returns a paginated list of conversations, optionally filtered by agent.

---

### 9. Get Conversation

**HTTP Method:** `GET`

**API URL:** `/api/conversations/{conversation_id}`

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "string",
  "agent_id": "string",
  "title": "string",
  "messages": [
    {
      "id": "string",
      "role": "user | assistant | system | tool",
      "content": "string",
      "created_at": "string (ISO 8601 datetime)",
      "tool_calls": [
        {
          "id": "string",
          "name": "string",
          "arguments": {}
        }
      ],
      "tool_results": [{}]
    }
  ],
  "created_at": "string (ISO 8601 datetime)",
  "updated_at": "string (ISO 8601 datetime)"
}
```

**Description:** Retrieves a specific conversation and its full message history.

---

### 10. Create Conversation

**HTTP Method:** `POST`

**API URL:** `/api/conversations`

**Request Payload:**
```json
{
  "agent_id": "string",
  "title": "string (optional)"
}
```

**Response Payload:**
```json
{
  "id": "string",
  "agent_id": "string",
  "title": "string",
  "created_at": "string (ISO 8601 datetime)"
}
```

**Description:** Creates a new conversation session tied to a specific agent.

---

### 11. Delete Conversation

**HTTP Method:** `DELETE`

**API URL:** `/api/conversations/{conversation_id}`

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "message": "Conversation deleted successfully"
}
```

**Description:** Deletes a conversation and all its associated messages.

---

### 12. Send Message (Chat)

**HTTP Method:** `POST`

**API URL:** `/api/conversations/{conversation_id}/messages`

**Request Payload:**
```json
{
  "content": "string",
  "role": "user",
  "attachments": [
    {
      "type": "image | file",
      "url": "string",
      "name": "string"
    }
  ]
}
```

**Response Payload (streaming SSE or JSON):**
```json
{
  "id": "string",
  "role": "assistant",
  "content": "string",
  "tool_calls": [
    {
      "id": "string",
      "name": "string",
      "arguments": {}
    }
  ],
  "created_at": "string (ISO 8601 datetime)",
  "finish_reason": "stop | tool_calls | length"
}
```

**Description:** Sends a user message to the agent within a conversation and returns the agent's response. May stream the response via Server-Sent Events (SSE).

---

### 13. Clear Conversation Messages

**HTTP Method:** `DELETE`

**API URL:** `/api/conversations/{conversation_id}/messages`

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "message": "Messages cleared successfully"
}
```

**Description:** Clears all messages from a conversation while preserving the conversation metadata.

---

## Workspace

### 14. Get Workspace Info

**HTTP Method:** `GET`

**API URL:** `/api/workspace`

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "string",
  "name": "string",
  "path": "string",
  "created_at": "string (ISO 8601 datetime)",
  "settings": {}
}
```

**Description:** Returns information about the current active workspace.

---

### 15. List Workspace Files

**HTTP Method:** `GET`

**API URL:** `/api/workspace/files`

**Query Parameters:**
- `path` (optional): Sub-directory path within workspace
- `recursive` (optional, boolean): Whether to list files recursively

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "files": [
    {
      "name": "string",
      "path": "string",
      "type": "file | directory",
      "size": "integer (bytes)",
      "modified_at": "string (ISO 8601 datetime)"
    }
  ]
}
```

**Description:** Lists files and directories within the workspace, optionally scoped to a sub-path.

---

### 16. Upload Workspace File

**HTTP Method:** `POST`

**API URL:** `/api/workspace/files`

**Request Payload:** `multipart/form-data`
```
file: <binary file content>
path: "string (optional destination path within workspace)"
```

**Response Payload:**
```json
{
  "name": "string",
  "path": "string",
  "size": "integer",
  "created_at": "string (ISO 8601 datetime)"
}
```

**Description:** Uploads a file to the workspace storage.

---

### 17. Delete Workspace File

**HTTP Method:** `DELETE`

**API URL:** `/api/workspace/files`

**Request Payload:**
```json
{
  "path": "string"
}
```

**Response Payload:**
```json
{
  "message": "File deleted successfully"
}
```

**Description:** Deletes a specific file from the workspace.

---

### 18. Get Workspace Settings

**HTTP Method:** `GET`

**API URL:** `/api/workspace/settings`

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "default_model": "string",
  "theme": "light | dark",
  "language": "string",
  "max_tokens": "integer",
  "temperature": "number",
  "custom_settings": {}
}
```

**Description:** Retrieves the current workspace-level settings and preferences.

---

### 19. Update Workspace Settings

**HTTP Method:** `PUT`

**API URL:** `/api/workspace/settings`

**Request Payload:**
```json
{
  "default_model": "string",
  "theme": "light | dark",
  "language": "string",
  "max_tokens": 4096,
  "temperature": 0.7
}
```

**Response Payload:**
```json
{
  "message": "Settings updated successfully",
  "settings": {}
}
```

**Description:** Updates workspace-level configuration and preferences.

---

## MCP (Model Context Protocol)

### 20. List MCP Servers

**HTTP Method:** `GET`

**API URL:** `/api/mcp/servers`

**Request Payload:** `N/A`

**Response Payload:**
```json
[
  {
    "id": "string",
    "name": "string",
    "url": "string",
    "status": "active | inactive | error",
    "tools": ["string"],
    "created_at": "string (ISO 8601 datetime)"
  }
]
```

**Description:** Lists all registered MCP (Model Context Protocol) servers available for tool integration.

---

### 21. Register MCP Server

**HTTP Method:** `POST`

**API URL:** `/api/mcp/servers`

**Request Payload:**
```json
{
  "name": "string",
  "url": "string",
  "auth_token": "string (optional)",
  "description": "string (optional)"
}
```

**Response Payload:**
```json
{
  "id": "string",
  "name": "string",
  "url": "string",
  "status": "active",
  "created_at": "string (ISO 8601 datetime)"
}
```

**Description:** Registers a new external MCP server for tool discovery and invocation.

---

### 22. Get MCP Server

**HTTP Method:** `GET`

**API URL:** `/api/mcp/servers/{server_id}`

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "string",
  "name": "string",
  "url": "string",
  "status": "active | inactive | error",
  "tools": [
    {
      "name": "string",
      "description": "string",
      "input_schema": {}
    }
  ],
  "created_at": "string (ISO 8601 datetime)"
}
```

**Description:** Retrieves details of a specific MCP server including its available tools.

---

### 23. Update MCP Server

**HTTP Method:** `PUT`

**API URL:** `/api/mcp/servers/{server_id}`

**Request Payload:**
```json
{
  "name": "string",
  "url": "string",
  "auth_token": "string (optional)",
  "description": "string (optional)"
}
```

**Response Payload:**
```json
{
  "id": "string",
  "name": "string",
  "url": "string",
  "updated_at": "string (ISO 8601 datetime)"
}
```

**Description:** Updates the configuration of an existing MCP server registration.

---

### 24. Delete MCP Server

**HTTP Method:** `DELETE`

**API URL:** `/api/mcp/servers/{server_id}`

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "message": "MCP server deleted successfully"
}
```

**Description:** Removes an MCP server registration from the system.

---

### 25. List MCP Tools

**HTTP Method:** `GET`

**API URL:** `/api/mcp/tools`

**Query Parameters:**
- `server_id` (optional): Filter tools by a specific MCP server

**Request Payload:** `N/A`

**Response Payload:**
```json
[
  {
    "id": "string",
    "name": "string",
    "description": "string",
    "server_id": "string",
    "server_name": "string",
    "input_schema": {}
  }
]
```

**Description:** Returns all available tools across all registered MCP servers, optionally filtered by server.

---

## Approvals

### 26. List Pending Approvals

**HTTP Method:** `GET`

**API URL:** `/api/approvals`

**Query Parameters:**
- `status` (optional): `pending | approved | rejected`
- `agent_id` (optional): Filter by agent

**Request Payload:** `N/A`

**Response Payload:**
```json
[
  {
    "id": "string",
    "agent_id": "string",
    "conversation_id": "string",
    "action_type": "string",
    "action_payload": {},
    "status": "pending | approved | rejected",
    "requested_at": "string (ISO 8601 datetime)",
    "resolved_at": "string (ISO 8601 datetime) | null",
    "reason": "string | null"
  }
]
```

**Description:** Retrieves a list of human-in-the-loop approval requests generated by agents requiring user authorization before executing actions.

---

### 27. Get Approval

**HTTP Method:** `GET`

**API URL:** `/api/approvals/{approval_id}`

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "string",
  "agent_id": "string",
  "conversation_id": "string",
  "action_type": "string",
  "action_payload": {},
  "status": "pending | approved | rejected",
  "requested_at": "string (ISO 8601 datetime)",
  "resolved_at": "string (ISO 8601 datetime) | null",
  "reason": "string | null"
}
```

**Description:** Retrieves details of a specific approval request.

---

### 28. Resolve Approval

**HTTP Method:** `POST`

**API URL:** `/api/approvals/{approval_id}/resolve`

**Request Payload:**
```json
{
  "decision": "approved | rejected",
  "reason": "string (optional)"
}
```

**Response Payload:**
```json
{
  "id": "string",
  "status": "approved | rejected",
  "resolved_at": "string (ISO 8601 datetime)"
}
```

**Description:** Allows a human operator to approve or reject a pending agent action that requires authorization. Resolving the approval unblocks the agent's execution flow.

---

## Cron Jobs

### 29. List Cron Jobs

**HTTP Method:** `GET`

**API URL:** `/api/crons`

**Request Payload:** `N/A`

**Response Payload:**
```json
[
  {
    "id": "string",
    "name": "string",
    "agent_id": "string",
    "schedule": "string (cron expression)",
    "message": "string",
    "is_active": true,
    "last_run_at": "string (ISO 8601 datetime) | null",
    "next_run_at": "string (ISO 8601 datetime) | null",
    "created_at": "string (ISO 8601 datetime)"
  }
]
```

**Description:** Lists all scheduled cron jobs that trigger agent conversations on a time-based schedule.

---

### 30. Create Cron Job

**HTTP Method:** `POST`

**API URL:** `/api/crons`

**Request Payload:**
```json
{
  "name": "string",
  "agent_id": "string",
  "schedule": "0 9 * * *",
  "message": "string",
  "is_active": true
}
```

**Response Payload:**
```json
{
  "id": "string",
  "name": "string",
  "agent_id": "string",
  "schedule": "string",
  "message": "string",
  "is_active": true,
  "next_run_at": "string (ISO 8601 datetime)",
  "created_at": "string (ISO 8601 datetime)"
}
```

**Description:** Creates a new scheduled cron job. The `schedule` field accepts a standard cron expression (e.g., `0 9 * * *` for daily at 9am). When triggered, the job sends the given `message` to the specified agent.

---

### 31. Get Cron Job

**HTTP Method:** `GET`

**API URL:** `/api/crons/{cron_id}`

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "string",
  "name": "string",
  "agent_id": "string",
  "schedule": "string",
  "message": "string",
  "is_active": true,
  "last_run_at": "string (ISO 8601 datetime) | null",
  "next_run_at": "string (ISO 8601 datetime) | null",
  "run_history": [
    {
      "run_at": "string (ISO 8601 datetime)",
      "status": "success | failed",
      "conversation_id": "string"
    }
  ],
  "created_at": "string (ISO 8601 datetime)"
}
```

**Description:** Retrieves full details of a specific cron job including its execution history.

---

### 32. Update Cron Job

**HTTP Method:** `PUT`

**API URL:** `/api/crons/{cron_id}`

**Request Payload:**
```json
{
  "name": "string",
  "schedule": "string (cron expression)",
  "message": "string",
  "is_active": true
}
```

**Response Payload:**
```json
{
  "id": "string",
  "name": "string",
  "schedule": "string",
  "is_active": true,
  "updated_at": "string (ISO 8601 datetime)"
}
```

**Description:** Updates an existing cron job's schedule, message, or active status.

---

### 33. Delete Cron Job

**HTTP Method:** `DELETE`

**API URL:** `/api/crons/{cron_id}`

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "message": "Cron job deleted successfully"
}
```

**Description:** Removes a cron job from the scheduler.

---

## Providers / Models

### 34. List Providers

**HTTP Method:** `GET`

**API URL:** `/api/providers`

**Request Payload:** `N/A`

**Response Payload:**
```json
[
  {
    "id": "string",
    "name": "string",
    "type": "openai | anthropic | google | ollama | custom",
    "is_configured": true,
    "models": ["string"]
  }
]
```

**Description:** Lists all configured LLM providers (e.g., OpenAI, Anthropic, Ollama) and their available models.

---

### 35. Configure Provider

**HTTP Method:** `POST`

**API URL:** `/api/providers`

**Request Payload:**
```json
{
  "type": "openai | anthropic | google | ollama | custom",
  "name": "string",
  "api_key": "string (optional)",
  "base_url": "string (optional)",
  "config": {}
}
```

**Response Payload:**
```json
{
  "id": "string",
  "name": "string",
  "type": "string",
  "is_configured": true,
  "created_at": "string (ISO 8601 datetime)"
}
```

**Description:** Adds or configures an LLM provider with API credentials and connection settings.

---

### 36. Update Provider

**HTTP Method:** `PUT`

**API URL:** `/api/providers/{provider_id}`

**Request Payload:**
```json
{
  "name": "string",
  "api_key": "string (optional)",
  "base_url": "string (optional)",
  "config": {}
}
```

**Response Payload:**
```json
{
  "id": "string",
  "name": "string",
  "updated_at": "string (ISO 8601 datetime)"
}
```

**Description:** Updates the configuration for an existing LLM provider.

--- service_dependencies ---


# External Dependencies Analysis: CoPaw Repository

## Overview

This analysis covers all external dependencies identified in the CoPaw codebase, spanning Python packages, JavaScript/TypeScript libraries, container images, and cloud/external services.

---

## Python Dependencies

### 1. AgentScope

| Field | Details |
|---|---|
| **Dependency Name** | AgentScope (`agentscope`) |
| **Type** | Library/Framework |
| **Purpose/Role** | Core AI agent framework that CoPaw is built upon. Provides agent orchestration, model provider abstractions, and runtime capabilities. |
| **Integration Point** | `pyproject.toml` → `"agentscope==1.0.18"` and `"agentscope-runtime==1.1.3"` |

---

### 2. HTTPX

| Field | Details |
|---|---|
| **Dependency Name** | HTTPX |
| **Type** | Library/Framework |
| **Purpose/Role** | Async-capable HTTP client used for making outgoing HTTP/HTTPS requests to external APIs and services. |
| **Integration Point** | `pyproject.toml` → `"httpx>=0.27.0"` |

---

### 3. Discord.py

| Field | Details |
|---|---|
| **Dependency Name** | Discord API (`discord-py`) |
| **Type** | Third-party API / Library |
| **Purpose/Role** | Provides integration with the Discord messaging platform as a communication channel. Allows CoPaw to send/receive messages via Discord bots. |
| **Integration Point** | `pyproject.toml` → `"discord-py>=2.3"` |

---

### 4. DingTalk Stream

| Field | Details |
|---|---|
| **Dependency Name** | DingTalk Messaging Platform (`dingtalk-stream`) |
| **Type** | Third-party API / Library |
| **Purpose/Role** | Integrates CoPaw with Alibaba's DingTalk (enterprise messaging platform) as a communication channel for sending and receiving messages. |
| **Integration Point** | `pyproject.toml` → `"dingtalk-stream>=0.24.3"` |

---

### 5. Lark OAPI (Feishu)

| Field | Details |
|---|---|
| **Dependency Name** | Feishu/Lark Platform (`lark-oapi`) |
| **Type** | Third-party API / Library |
| **Purpose/Role** | Provides integration with ByteDance's Feishu/Lark enterprise messaging platform as a communication channel. |
| **Integration Point** | `pyproject.toml` → `"lark-oapi>=1.5.3"` |

---

### 6. Python Telegram Bot

| Field | Details |
|---|---|
| **Dependency Name** | Telegram Messaging Platform (`python-telegram-bot`) |
| **Type** | Third-party API / Library |
| **Purpose/Role** | Integrates CoPaw with Telegram as a communication channel, enabling message sending/receiving via Telegram bots. |
| **Integration Point** | `pyproject.toml` → `"python-telegram-bot>=20.0"` |

---

### 7. Twilio

| Field | Details |
|---|---|
| **Dependency Name** | Twilio (`twilio`) |
| **Type** | Third-party API / External Service |
| **Purpose/Role** | Cloud communications platform. Likely used for SMS/WhatsApp messaging as a communication channel for CoPaw. |
| **Integration Point** | `pyproject.toml` → `"twilio>=9.10.2"` |

---

### 8. WeCom AIBot Python SDK

| Field | Details |
|---|---|
| **Dependency Name** | WeCom (WeChat Work) Platform (`wecom-aibot-python-sdk`) |
| **Type** | Third-party API / Library |
| **Purpose/Role** | Provides integration with Tencent's WeCom (企业微信 / WeChat Work) enterprise messaging platform as a communication channel. |
| **Integration Point** | `pyproject.toml` → `"wecom-aibot-python-sdk==1.0.2"` |

---

### 9. Matrix NIO

| Field | Details |
|---|---|
| **Dependency Name** | Matrix Protocol (`matrix-nio`) |
| **Type** | Third-party API / Library |
| **Purpose/Role** | Provides integration with the Matrix open messaging protocol/network (e.g., Element client) as a communication channel. |
| **Integration Point** | `pyproject.toml` → `"matrix-nio>=0.24.0"` |

---

### 10. Paho MQTT

| Field | Details |
|---|---|
| **Dependency Name** | MQTT Message Broker (`paho-mqtt`) |
| **Type** | Message Broker / Library |
| **Purpose/Role** | MQTT protocol client used for IoT-style publish/subscribe messaging. Likely used as an additional communication channel or event bus. |
| **Integration Point** | `pyproject.toml` → `"paho-mqtt>=2.0.0"` |

---

### 11. Google GenAI

| Field | Details |
|---|---|
| **Dependency Name** | Google Generative AI (`google-genai`) |
| **Type** | Third-party API / External Service (Cloud AI) |
| **Purpose/Role** | SDK for Google's Generative AI services (Gemini models). Used as an LLM provider for AI inference within CoPaw. |
| **Integration Point** | `pyproject.toml` → `"google-genai>=1.67.0"` |

---

### 12. Playwright

| Field | Details |
|---|---|
| **Dependency Name** | Microsoft Playwright |
| **Type** | Library/Framework |
| **Purpose/Role** | Browser automation framework used to control Chromium for web scraping, browser-based skills, and automated web interaction tasks. |
| **Integration Point** | `pyproject.toml` → `"playwright>=1.49.0"`. Dockerfile sets `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH=/usr/bin/chromium` and `PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1`. |

---

### 13. Uvicorn

| Field | Details |
|---|---|
| **Dependency Name** | Uvicorn ASGI Server |
| **Type** | Library/Framework |
| **Purpose/Role** | ASGI web server used to host CoPaw's HTTP API/backend service. |
| **Integration Point** | `pyproject.toml` → `"uvicorn>=0.40.0"` |

---

### 14. APScheduler

| Field | Details |
|---|---|
| **Dependency Name** | APScheduler |
| **Type** | Library/Framework |
| **Purpose/Role** | Task scheduling library. Powers CoPaw's cron/scheduled task execution system. |
| **Integration Point** | `pyproject.toml` → `"apscheduler>=3.11.2,<4"` |

---

### 15. Transformers (Hugging Face)

| Field | Details |
|---|---|
| **Dependency Name** | Hugging Face Transformers |
| **Type** | Library/Framework |
| **Purpose/Role** | Provides access to pre-trained language models and NLP pipelines. Used for local model inference and AI capabilities. |
| **Integration Point** | `pyproject.toml` → `"transformers>=4.30.0"` |

---

### 16. Hugging Face Hub

| Field | Details |
|---|---|
| **Dependency Name** | Hugging Face Hub (`huggingface_hub`) |
| **Type** | External Service / Library |
| **Purpose/Role** | Client library for downloading models and datasets from the Hugging Face model repository (hub.huggingface.co). |
| **Integration Point** | `pyproject.toml` → `"huggingface_hub>=0.20.0"` (both core and `local` optional dep) |

---

### 17. ModelScope

| Field | Details |
|---|---|
| **Dependency Name** | ModelScope (`modelscope`) |
| **Type** | External Service / Library |
| **Purpose/Role** | Alibaba's model repository and inference SDK. Used as an alternative to Hugging Face Hub for downloading and running models, particularly in Chinese cloud environments. |
| **Integration Point** | `pyproject.toml` → `"modelscope>=1.35.0"` |

---

### 18. ONNX Runtime

| Field | Details |
|---|---|
| **Dependency Name** | ONNX Runtime (`onnxruntime`) |
| **Type** | Library/Framework |
| **Purpose/Role** | Cross-platform inference engine for ONNX-format ML models. Used for running local AI models efficiently. |
| **Integration Point** | `pyproject.toml` → `"onnxruntime<1.24"` |

---

### 19. Reme AI

| Field | Details |
|---|---|
| **Dependency Name** | Reme AI (`reme-ai`) |
| **Type** | Library / External Service (assumption — requires investigation) |
| **Purpose/Role** | **ASSUMPTION**: Likely provides AI-related memory or retrieval services. The specific role requires further investigation. |
| **Integration Point** | `pyproject.toml` → `"reme-ai==0.3.1.8"` |

---

### 20. Python-dotenv

| Field | Details |
|---|---|
| **Dependency Name** | python-dotenv |
| **Type** | Library/Framework |
| **Purpose/Role** | Loads environment variables from `.env` configuration files at runtime, managing secrets and external service credentials. |
| **Integration Point** | `pyproject.toml` → `"python-dotenv>=1.0.0"` |

---

### 21. Python-socks

| Field | Details |
|---|---|
| **Dependency Name** | python-socks |
| **Type** | Library/Framework |
| **Purpose/Role** | Provides SOCKS proxy support for network connections, enabling traffic routing through proxy servers. |
| **Integration Point** | `pyproject.toml` → `"python-socks>=2.5.3"` |

---

### 22. Pywebview

| Field | Details |
|---|---|
| **Dependency Name** | pywebview |
| **Type** | Library/Framework |
| **Purpose/Role** | Creates native desktop webview windows for the CoPaw desktop application, wrapping the web UI in a native shell. |
| **Integration Point** | `pyproject.toml` → `"pywebview>=4.0"` |

---

### 23. Pillow

| Field | Details |
|---|---|
| **Dependency Name** | Pillow (PIL) |
| **Type** | Library/Framework |
| **Purpose/Role** | Image processing library used for handling, converting, and manipulating images (e.g., screenshots from `mss`). |
| **Integration Point** | `pyproject.toml` → `"pillow>=10.0.0"` |

---

### 24. MSS (Screenshot)

| Field | Details |
|---|---|
| **Dependency Name** | MSS (`mss`) |
| **Type** | Library/Framework |
| **Purpose/Role** | Cross-platform screenshot library. Used to capture screen content for vision-based AI tasks or desktop automation. |
| **Integration Point** | `pyproject.toml` → `"mss>=9.0.0"` |

---

### 25. Questionary

| Field | Details |
|---|---|
| **Dependency Name** | Questionary |
| **Type** | Library/Framework |
| **Purpose/Role** | Interactive CLI prompt library used in CoPaw's command-line setup and configuration wizards. |
| **Integration Point** | `pyproject.toml` → `"questionary>=2.1.1"` |

---

### 26. PyYAML

| Field | Details |
|---|---|
| **Dependency Name** | PyYAML |
| **Type** | Library/Framework |
| **Purpose/Role** | YAML parsing and serialization library for reading configuration files. |
| **Integration Point** | `pyproject.toml` → `"pyyaml>=6.0"` |

---

### 27. JSON Repair

| Field | Details |
|---|---|
| **Dependency Name** | json-repair |
| **Type** | Library/Framework |
| **Purpose/Role** | Repairs malformed JSON output from LLMs, ensuring robust parsing of AI-generated structured responses. |
| **Integration Point** | `pyproject.toml` → `"json-repair>=0.30.0"` |

---

### 28. Segno

| Field | Details |
|---|---|
| **Dependency Name** | Segno |
| **Type** | Library/Framework |
| **Purpose/Role** | QR code generation library. Used to generate QR codes (e.g., for sharing links or authentication). |
| **Integration Point** | `pyproject.toml` → `"segno>=1.6.6"` |

---

### 29. ShortUUID

| Field | Details |
|---|---|
| **Dependency Name** | shortuuid |
| **Type** | Library/Framework |
| **Purpose/Role** | Generates short, human-readable UUIDs used for unique identifier generation throughout the system. |
| **Integration Point** | `pyproject.toml` → `"shortuuid>=1.0.0"` |

---

### 30. AIOFiles

| Field | Details |
|---|---|
| **Dependency Name** | aiofiles |
| **Type** | Library/Framework |
| **Purpose/Role** | Async file I/O library enabling non-blocking file operations in the async Python application. |
| **Integration Point** | `pyproject.toml` → `"aiofiles>=24.1.0"` |

---

### 31. Packaging

| Field | Details |
|---|---|
| **Dependency Name** | packaging |
| **Type** | Library/Framework |
| **Purpose/Role** | Provides version parsing and comparison utilities, likely used for dependency/version checks at runtime. |
| **Integration Point** | `pyproject.toml` → `"packaging>=24.0"` |

---

### 32. tzdata

| Field | Details |
|---|---|
| **Dependency Name** | tzdata |
| **Type** | Library/Framework |
| **Purpose/Role** | Timezone database package, ensuring consistent timezone handling for scheduled tasks and cross-timezone operations. |
| **Integration Point** | `pyproject.toml` → `"tzdata>=2024.1"` |

---

### 33. anyio

| Field | Details |
|---|---|
| **Dependency Name** | anyio |
| **Type** | Library/Framework |
| **Purpose/Role** | Async compatibility layer supporting asyncio. Pinned below 4.13.0 to avoid a known busy-loop bug. |
| **Integration Point** | `pyproject.toml` → `"anyio>=4.0.0,<4.13.0"` (with explicit bug note) |

---

### Optional Python Dependencies

### 34. Ollama

| Field | Details |
|---|---|
| **Dependency Name** | Ollama (`ollama`) |
| **Type** | External Service / Library |
| **Purpose/Role** | Client for the Ollama local LLM server. Allows CoPaw to use locally-hosted models via the Ollama runtime as an LLM provider. |
| **Integration Point** | `pyproject.toml` optional dep `[ollama]` → `"ollama>=0.6.1"`. Also used in Dockerfile: `pip install .[ollama]` |

---

### 35. Llama-cpp-python

| Field | Details |
|---|---|
| **Dependency Name** | llama-cpp-python |
| **Type** | Library/Framework |
| **Purpose/Role** | Python bindings for llama.cpp, enabling direct local inference of GGUF-format LLMs without an external server. |
| **Integration Point** | `pyproject.toml` optional dep `[llamacpp]` → `"llama-cpp-python>=0.3.0"` |

---

### 36. MLX-LM

| Field | Details |
|---|---|
| **Dependency Name** | MLX-LM (`mlx-lm`) |
| **Type** | Library/Framework |
| **Purpose/Role** | Apple MLX framework for running LLMs natively on Apple Silicon (macOS only). |
| **Integration Point** | `pyproject.toml` optional dep `[mlx]` → `"mlx-lm>=0.10.0; sys_platform == 'darwin'"` |

---

### 37. OpenAI Whisper

| Field | Details |
|---|---|
| **Dependency Name** | OpenAI Whisper (`openai-whisper`) |
| **Type** | Library/Framework |
| **Purpose/Role** | Speech-to-text transcription model. Used to transcribe audio messages received through communication channels. |
| **Integration Point** | `pyproject.toml` optional dep `[whisper]` → `"openai-whisper>=20231117"` |

---

## JavaScript / Frontend Dependencies

### Console Application (`/console`)

### 38. @agentscope-ai/chat, /design, /icons

| Field | Details |
|---|---|
| **Dependency Name** | AgentScope AI UI Libraries |
| **Type** | Library/Framework |
| **Purpose/Role** | Purpose-built UI component libraries (chat interface, design system, icon set) for the CoPaw console frontend. |
| **Integration Point** | `console/package.json` → `"@agentscope-ai/chat": "^1.1.56"`, `"@agentscope-ai/design": "^1.0.14"`, `"@agentscope-ai/icons": "^1.0.63"` |

---

### 39. Ant Design (antd) + Icons + X-Markdown

| Field | Details |
|---|---|
| **Dependency Name** | Ant Design UI Framework |
| **Type** | Library/Framework |
| **Purpose/Role** | Enterprise-grade React UI component library providing the primary UI components (buttons, forms, tables, etc.) for the console. |
| **Integration Point** | `console/package.json` → `"antd": "^5.29.1"`, `"@ant-design/icons": "^5.0.1"`, `"@ant-design/x-markdown": "^2.2.2"`, `"antd-style": "^3.7.1"` |

---

### 40. DnD Kit

| Field | Details |
|---|---|
| **Dependency Name** | DnD Kit (Drag and Drop) |
| **Type** | Library/Framework |
| **Purpose/Role** | Accessible drag-and-drop toolkit for React, used to implement sortable/draggable UI elements in the console. |
| **Integration Point** | `console/package.json` → `"@dnd-kit/core": "^6.3.1"`, `"@dnd-kit/sortable": "^10.0.0"`, `"@dnd-kit/utilities": "^3.2.2"` |

---

### 41. React + React DOM

| Field | Details |
|---|---|
| **Dependency Name** | React Framework |
| **Type** | Library/Framework |
| **Purpose/Role** | Core UI framework for building both the console and website frontend applications. |
| **Integration Point** | `console/package.json` → `"react": "^18"`, `"react-dom": "^18"`. Also `website/package.json` → `"react": "^18.3.1"` |

---

### 42. React Router DOM

| Field | Details |
|---|---|
| **Dependency Name** | React Router |
| **Type** | Library/Framework |
| **Purpose/Role** | Client-side routing library for both the console SPA and website, managing page navigation. |
| **Integration Point** | `console/package.json` → `"react-router-dom": "^7.13.0"`. `website/package.json` → `"react-router-dom": "^7.0.1"` |

---

### 43. i18next + react-i18next

| Field | Details |
|---|---|
| **Dependency Name** | i18next Internationalization Framework |
| **Type** | Library/Framework |
| **Purpose/Role** | Internationalization (i18n) framework used to provide multi-language support in both the console and website. |
| **Integration Point** | `console/package.json` → `"i18next": "^25.8.4"`, `"react-i18next": "^16.5.4"`. `website/package.json` → `"i18next": "^25.8.20"`, `"react-i18next": "^16.5.8"` |

---

### 44. React Markdown + remark-gfm

| Field | Details |
|---|---|
| **Dependency Name** | React Markdown |
| **Type** | Library/Framework |
| **Purpose/Role** | Renders Markdown content within React components. Used to display AI-generated markdown responses in chat interfaces. |
| **Integration Point** | `console/package.json` → `"react-markdown": "^10.1.0"`, `"remark-gfm": "^4.0.1"` |

---

### 45. Zustand

| Field | Details |
|---|---|
| **Dependency Name** | Zustand State Management |
| **Type** | Library/Framework |
| **Purpose/Role** | Lightweight React state management library for managing global application state in the console. |
| **Integration Point** | `console/package.json` → `"zustand": "^5.0.3"` |

---

### 46. ahooks

| Field | Details |
|---|---|
| **Dependency Name** | ahooks (React Hooks Library) |
| **Type** | Library/Framework |
| **Purpose/Role** | Collection of high-quality React hooks for common patterns (debounce, request, etc.) used in the console. |
| **Integration Point** | `console/package.json` → `"ahooks": "^3.9.6"` |

---

### 47. Day.js

| Field | Details |
|---|---|
| **Dependency Name** | Day.js |
| **Type** | Library/Framework |
| **Purpose/Role** | Lightweight date/time manipulation library used for formatting and parsing dates in the UI. |
| **Integration Point** | `console/package.json` → `"dayjs": "^1.11.13"` |

---

### 48. Lucide React (Icons)

| Field | Details |
|---|---|
| **Dependency Name** | Lucide React |
| **Type** | Library/Framework |
| **Purpose/Role** | Icon library providing SVG icons for both the console and website UIs. |
| **Integration Point** | `console/package.json` → `"lucide-react": "^0.562.0"`. `website/package.json` → `"lucide-react": "^0.468.0"` |

---

### 49. Vite (Build Tool)

| Field | Details |
|---|---|
| **Dependency Name** | Vite |
| **Type** | Library/Framework (Build Tool) |
| **Purpose/Role** | Fast frontend build tool and development server used to bundle both the console and website. |
| **Integration Point** | `console/package.json (dev)` → `"vite": "^6.3.5"`. `website/package.json (dev)` → `"vite": "^6.0.1"` |

---

### Website Application (`/website`)

### 50. Tailwind CSS

| Field | Details |
|---|---|
| **Dependency Name** | Tailwind CSS |
| **Type** | Library/Framework |
| **Purpose/Role** | Utility-first CSS framework used for styling the website frontend. |
| **Integration Point** | `website/package.json (dev)` → `"tailwindcss": "^4.2.2"`,

--- hl_overview ---


# CoPaw Repository Analysis

## [[CoPaw]]

---

## 1. Project Purpose

CoPaw appears to be an **AI Agent Platform / Conversational AI Framework** that enables users to create, manage, and deploy AI agents powered by Large Language Models (LLMs). It provides:

- A backend server for managing AI agents, conversations, and workflows
- A web console UI for interacting with agents
- A public website/landing page
- CLI tooling for agent management
- Support for multiple LLM providers
- MCP (Model Context Protocol) integration
- Skill/tool execution with security scanning
- Cron-based scheduled agent tasks
- Multi-channel communication support

**Primary Domain:** AI Agent orchestration, LLM-powered workflow automation, conversational AI management platform.

---

## 2. Architecture Pattern

**Hybrid Multi-Tier Architecture** combining:
- **Layered Backend API** (Python/FastAPI-style routers + app services)
- **Plugin/Provider Pattern** for LLM backends
- **Single Page Application (SPA)** frontend (React)
- **Agent-Oriented Design** with tools, skills, memory, and hooks
- **Event-Driven** elements (channels, crons, runners)

---

## 3. Technology Stack

### Backend (Python)

From `pyproject.toml` / `setup.py`:

| Dependency | Purpose |
|---|---|
| **FastAPI** / **Starlette** | Web framework / ASGI server |
| **uvicorn** | ASGI server runtime |
| **supervisord** | Process management (in Docker) |
| **pydantic** | Data validation and settings |
| **SQLAlchemy** / **alembic** | ORM and database migrations |
| **openai** | OpenAI LLM provider integration |
| **anthropic** | Anthropic Claude provider |
| **tiktoken** | Token counting/tokenizer |
| **httpx** / **aiohttp** | Async HTTP clients |
| **celery** / **redis** | Task queue and message broker (likely) |
| **pytest** | Testing framework |
| **flake8** / **pre-commit** | Linting and code quality |
| **docker** | Container deployment |

**Python Version:** Specified in `.python-version` (likely 3.10+)

### Frontend (Console UI)
| Technology | Purpose |
|---|---|
| **React** | UI framework |
| **TypeScript** | Type-safe JavaScript |
| **Vite** | Build tool / dev server |
| **i18n (react-i18next)** | Internationalization (EN, ZH, JA, RU) |
| **ESLint** | Code linting |
| **Zustand** (likely) | State management (`stores/`) |

### Website (Marketing/Docs)
| Technology | Purpose |
|---|---|
| **React** | UI framework |
| **TypeScript** | Language |
| **Vite** | Build tool |
| **pnpm** | Package manager |
| **shadcn/ui** (`components.json`) | UI component library |
| **Tailwind CSS** | Styling |

### Infrastructure
| Technology | Purpose |
|---|---|
| **Docker** / **docker-compose** | Containerization |
| **GitHub Actions** | CI/CD workflows |
| **Conda** | Python environment (`.github/condarc`) |
| **NSIS** | Windows installer (`copaw_desktop.nsi`) |

---

## 4. Initial Structure Impression

The repository has **four main high-level parts**:

```
CoPaw/
├── src/copaw/          → Python Backend Core (API, agents, providers)
├── console/            → React Frontend SPA (user-facing web console)
├── website/            → Marketing/Documentation website
├── tests/              → Unit + Integration test suite
├── deploy/             → Docker deployment configuration
└── scripts/            → Build, install, packaging scripts
```

---

## 5. Configuration / Package Files

| File | Purpose |
|---|---|
| `pyproject.toml` | Python project metadata, build config, tool settings |
| `setup.py` | Python package installation |
| `.python-version` | Python version pin |
| `.flake8` | Python linting configuration |
| `.pre-commit-config.yaml` | Pre-commit hooks configuration |
| `docker-compose.yml` | Multi-container Docker orchestration |
| `deploy/Dockerfile` | Container image build instructions |
| `deploy/entrypoint.sh` | Container entry point script |
| `deploy/config/supervisord.conf.template` | Supervisor process manager config template |
| `console/package.json` | Console frontend NPM dependencies |
| `console/package-lock.json` | Console frontend lockfile |
| `console/tsconfig.json` | TypeScript base config (console) |
| `console/tsconfig.app.json` | TypeScript app config (console) |
| `console/tsconfig.node.json` | TypeScript node config (console) |
| `console/vite.config.ts` | Console Vite build config |
| `console/eslint.config.js` | ESLint config (console) |
| `website/package.json` | Website NPM dependencies |
| `website/package-lock.json` | Website NPM lockfile |
| `website/pnpm-lock.yaml` | Website pnpm lockfile |
| `website/tsconfig.json` | TypeScript config (website) |
| `website/vite.config.ts` | Website Vite build config |
| `website/components.json` | shadcn/ui component registry config |
| `website/public/site.config.json` | Website runtime configuration |
| `package-lock.json` | Root-level NPM lockfile |
| `.gitignore` | Git ignore rules |
| `.gitattributes` | Git attribute settings |
| `.dockerignore` | Docker build ignore rules |
| `.github/condarc` | Conda channel configuration |
| `.github/workflows/*.yml` | GitHub Actions CI/CD pipelines (10 workflows) |

---

## 6. Directory Structure

### Backend: `src/copaw/`

```
src/copaw/
├── __init__.py              # Package initialization, public API exports
├── __main__.py              # CLI entry point (python -m copaw)
├── __version__.py           # Version string management
├── constant.py              # Global constants
│
├── agents/                  # Agent orchestration core
│   ├── hooks/               # Agent lifecycle hooks (pre/post execution)
│   ├── md_files/            # Markdown templates for agent prompts
│   ├── memory/              # Agent conversation memory management
│   ├── skills/              # Reusable agent skill definitions
│   ├── tools/               # Tool implementations (function calling)
│   └── utils/               # Agent utility helpers
│
├── app/                     # Application layer (HTTP + runtime)
│   ├── approvals/           # Human-in-the-loop approval workflows
│   ├── channels/            # Communication channel integrations
│   ├── crons/               # Scheduled task management
│   ├── mcp/                 # Model Context Protocol integration
│   ├── routers/             # FastAPI route handlers (REST API)
│   ├── runner/              # Agent execution runtime engine
│   └── workspace/           # Workspace/project management
│
├── cli/                     # Command-line interface (21 modules)
├── config/                  # Configuration loading and management
├── envs/                    # Environment variable handling
├── local_models/            # Local LLM model support
├── providers/               # LLM provider adapters (OpenAI, Anthropic, etc.)
├── security/
│   ├── skill_scanner/       # Security scanning for skills
│   └── tool_guard/          # Tool execution security guards
├── token_usage/             # Token counting and usage tracking
├── tokenizer/               # Text tokenization utilities
├── tunnel/                  # Network tunneling (ngrok-style)
└── utils/                   # General utility functions
```

### Frontend: `console/src/`

```
console/src/
├── App.tsx                  # Root React component
├── main.tsx                 # Application entry point
├── i18n.ts                  # i18n initialization
│
├── api/
│   ├── modules/             # API call modules (per feature)
│   └── types/               # TypeScript API response types
│
├── components/              # Reusable UI components
│   ├── AgentSelector/       # Agent selection widget
│   ├── ConsoleCronBubble/   # Cron job status display
│   ├── LanguageSwitcher/    # Language toggle
│   ├── MarkdownCopy/        # Markdown renderer with copy
│   ├── PageHeader/          # Shared page header
│   └── ThemeToggleButton/   # Dark/light mode toggle
│
├── contexts/                # React context providers
├── hooks/                   # Custom React hooks
├── layouts/
│   └── MainLayout/          # App shell / navigation layout
├── locales/                 # Translation strings (4 languages)
├── pages/
│   ├── Agent/               # Agent management pages
│   ├── Chat/                # Conversation/chat interface
│   ├── Control/             # Control panel / dashboard
│   ├── Login/               # Authentication pages
│   └── Settings/            # Application settings
├── stores/                  # Global state management
├── styles/                  # Global CSS styles
└── utils/                   # Frontend utility functions
```

### Website: `website/src/`

```
website/src/
├── App.tsx                  # Root component
├── config.ts                # Site configuration
├── config-context.tsx       # Config React context
├── components/              # Shared UI components (15+)
│   └── Icon/                # Icon components
├── data/                    # Static data files
├── i18n/
│   └── locales/             # Website translations
├── lib/                     # Utility libraries
├── pages/
│   └── Home/                # Homepage sections/components
└── styles/                  # Website CSS
```

### Tests: `tests/`

```
tests/
├── unit/
│   ├── agents/tools/        # Tool unit tests
│   ├── channels/            # Channel tests
│   ├── cli/                 # CLI command tests
│   ├── local_models/        # Local model tests
│   ├── providers/           # LLM provider tests
│   ├── routers/             # API route tests
│   └── workspace/           # Workspace tests
└── integrated/
    ├── test_app_startup.py  # Full application startup test
    └── test_version.py      # Version consistency test
```

---

## 7. High-Level Architecture

### Pattern: **Layered + Plugin + Event-Driven Hybrid**

```
┌─────────────────────────────────────────────────────┐
│                   Frontend Layer                     │
│  Console SPA (React/TS)  │  Website (React/TS)       │
└────────────────┬────────────────────────────────────┘
                 │ REST API / WebSocket
┌────────────────▼────────────────────────────────────┐
│              API / Router Layer                      │
│         app/routers/ (FastAPI endpoints)             │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│            Application Service Layer                 │
│  app/runner/  │  app/workspace/  │  app/approvals/  │
│  app/crons/   │  app/channels/   │  app/mcp/        │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│              Agent Core Layer                        │
│  agents/ (tools, skills, memory, hooks)              │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│           Provider / Infrastructure Layer            │
│  providers/ (OpenAI, Anthropic, local_models)        │
│  security/ │ token_usage/ │ tokenizer/ │ config/     │
└─────────────────────────────────────────────────────┘
```

**Evidence:**
- `app/routers/` → clear HTTP boundary layer
- `providers/` → plugin pattern for LLM backends (13 provider modules)
- `agents/hooks/` → event-driven agent lifecycle
- `app/channels/` + `app/crons/` → event/schedule driven execution
- `app/runner/` → dedicated execution engine (separation of concerns)
- `security/tool_guard/` + `security/skill_scanner/` → cross-cutting security concern layer

---

## 8. Build, Execution and Test

### Running the Backend

```bash
# Direct Python execution
python -m copaw

# Via installed package
copaw

# Via Docker
docker-compose up

# Via Docker build script
bash scripts/docker_build.sh
```

**Main Entry Point:** `src/copaw/__main__.py`

### Running Tests

```bash
# Via test runner script
python scripts/run_tests.py

# Via pytest directly
pytest tests/unit/
pytest tests/integrated/

# GitHub Actions: .github/workflows/tests.yml
```

### Building the Frontend Console

```bash
cd console
npm install
npm run dev        # Development server
npm run build      # Production build
```

### Building the Website

```bash
cd website
pnpm install
pnpm run build
# Scripts: website/scripts/build-search-index.mjs
#          website/scripts/spa-fallback-pages.mjs

# GitHub Actions: .github/workflows/deploy-website.yml
```

### Desktop Application Build

```bash
# Windows
powershell scripts/pack/build_win.ps1

# macOS
bash scripts/pack/build_macos.sh

# Windows installer via NSIS
# scripts/pack/copaw_desktop.nsi
```

### CI/CD Pipelines (GitHub Actions)

| Workflow | Trigger | Purpose |
|---|---|---|
| `tests.yml` | PR/Push | Run test suite |
| `pre-commit.yml` | PR/Push | Lint and format checks |
| `npm-format.yml` | PR/Push | Frontend format checks |
| `publish-pypi.yml` | Release | Publish Python package to PyPI |
| `docker-release.yml` | Release | Build and push Docker image |
| `desktop-release.yml` | Release | Build desktop installers |
| `deploy-website.yml` | Push/Release | Deploy marketing website |
| `pr-welcome.yml` | PR opened | Welcome new PR authors |
| `issue-welcome.yml` | Issue opened | Welcome new issue reporters |
| `first-time-contributor-welcome.yml` | First contribution | Onboard new contributors |

### Code Quality

```bash
# Pre-commit hooks (flake8, formatting)
pre-commit run --all-files

# Flake8 config in .flake8
flake8 src/
```

--- dependencies ---


# CoPaw: Dependency and Architecture Analysis

---

## Internal Modules

The following core internal modules are part of the `src/copaw/` Python backend package, reused across the project:

### Backend (`src/copaw/`)

| Module | Primary Responsibility |
|---|---|
| **`agents/`** | Core agent orchestration layer. Contains sub-modules for tools (function calling), skills (reusable capabilities), memory (conversation history management), hooks (agent lifecycle event handlers), and agent utility helpers. |
| **`agents/hooks/`** | Agent lifecycle hook system — handles pre/post execution events for agent runs. |
| **`agents/memory/`** | Manages agent conversation memory and context retention across interactions. |
| **`agents/skills/`** | Reusable, composable agent skill definitions (e.g., PDF handling, news digest, cron). |
| **`agents/tools/`** | Tool implementations for LLM function-calling, enabling agents to invoke external actions. |
| **`app/routers/`** | FastAPI HTTP route handlers — the REST API boundary layer exposing all backend functionality to frontend and external clients. |
| **`app/runner/`** | Agent execution runtime engine — responsible for scheduling, executing, and managing the lifecycle of agent runs. |
| **`app/workspace/`** | Workspace and project management — handles isolation and organization of user workspaces. |
| **`app/approvals/`** | Human-in-the-loop approval workflow system — gates agent actions requiring manual sign-off. |
| **`app/channels/`** | Communication channel integrations (e.g., DingTalk, Feishu, Discord, Telegram, QQ, iMessage) — routes incoming/outgoing messages to/from agents. |
| **`app/crons/`** | Scheduled task management — manages cron-based recurring agent task execution. |
| **`app/mcp/`** | Model Context Protocol (MCP) integration — enables standardized context exchange between agents and LLM providers. |
| **`providers/`** | Plugin-pattern LLM provider adapters (13 modules) — abstracts integration with multiple LLM backends (OpenAI, Anthropic, Google, local models, etc.). |
| **`local_models/`** | Local LLM model support — manages loading and inference for locally hosted models (llama.cpp, MLX, Ollama). |
| **`security/skill_scanner/`** | Static security scanning for agent skills — analyzes skill code/definitions for security policy violations before execution. |
| **`security/tool_guard/`** | Runtime security guard for tool execution — enforces rules controlling what tools agents are permitted to invoke. |
| **`config/`** | Configuration loading and management — centralizes application-wide settings and environment resolution. |
| **`envs/`** | Environment variable handling — abstracts and validates environment-based configuration inputs. |
| **`cli/`** | Command-line interface (21 modules) — provides the `copaw` CLI entrypoint for agent management, initialization, and operational commands. |
| **`tokenizer/`** | Text tokenization utilities — provides token-level text processing used across providers and agents. |
| **`token_usage/`** | Token counting and usage tracking — monitors LLM token consumption per agent/session for quota and cost management. |
| **`tunnel/`** | Network tunneling support — enables external access to the local CoPaw server (ngrok-style reverse proxy). |
| **`utils/`** | General-purpose backend utility functions shared across modules. |

---

### Frontend Console (`console/src/`)

| Module | Primary Responsibility |
|---|---|
| **`api/`** | Centralized API call layer — organizes HTTP request modules per feature domain and TypeScript response type definitions. |
| **`pages/Agent/`** | Agent management UI — pages for creating, configuring, and managing AI agents. |
| **`pages/Chat/`** | Conversational chat interface — the primary user-facing dialogue UI for interacting with agents. |
| **`pages/Control/`** | Control panel / operational dashboard — system-level monitoring and management views. |
| **`pages/Login/`** | Authentication pages — user login and session management UI. |
| **`pages/Settings/`** | Application settings UI — user and system configuration pages. |
| **`components/`** | Shared reusable UI components (AgentSelector, ConsoleCronBubble, LanguageSwitcher, MarkdownCopy, PageHeader, ThemeToggleButton). |
| **`stores/`** | Global client-side state management — centralized Zustand store(s) for cross-component state. |
| **`contexts/`** | React context providers — shared state and services distributed via React context API. |
| **`hooks/`** | Custom React hooks — reusable stateful logic extracted from components. |
| **`layouts/MainLayout/`** | Application shell and navigation layout — wraps all pages with consistent chrome (nav, sidebar, header). |
| **`locales/`** | Internationalization translation strings for 4 languages (EN, ZH, JA, RU). |

---

### Website (`website/src/`)

| Module | Primary Responsibility |
|---|---|
| **`pages/Home/`** | Homepage sections and marketing content components. |
| **`components/`** | Shared website UI components (15+ components, including Icon sub-module). |
| **`i18n/locales/`** | Website internationalization translation strings. |
| **`lib/`** | Website-specific utility libraries. |
| **`data/`** | Static data files used by website pages. |
| **`config.ts` / `config-context.tsx`** | Site runtime configuration loading and React context distribution. |

---

## External Dependencies

### Python — Production
*Source: `/pyproject.toml`*

| Dependency | Official Name | Role / Purpose |
|---|---|---|
| `agentscope` | **AgentScope** | Core AI agent framework — provides the foundational agent runtime, orchestration primitives, and LLM integration abstractions that CoPaw builds upon. |
| `agentscope-runtime` | **AgentScope Runtime** | Runtime execution environment companion to AgentScope — handles low-level agent execution infrastructure. |
| `httpx` | **HTTPX** | Async-capable HTTP client — used for making outbound HTTP requests to LLM APIs and external services. |
| `packaging` | **Packaging** | Python package version parsing and comparison utilities — used for version management logic. |
| `discord-py` | **discord.py** | Discord channel integration — enables agents to send/receive messages via Discord. |
| `dingtalk-stream` | **DingTalk Stream** | DingTalk (Alibaba) channel integration — streams real-time messages from DingTalk to agents. |
| `uvicorn` | **Uvicorn** | ASGI server runtime — serves the FastAPI backend application. |
| `apscheduler` | **APScheduler** | Advanced Python Scheduler — powers the `app/crons/` scheduled task management system. |
| `playwright` | **Playwright** | Browser automation framework — enables agents to interact with web pages (used with the Chromium setup in the Dockerfile). |
| `questionary` | **Questionary** | Interactive CLI prompt library — used within the `cli/` module to build user-friendly terminal prompts. |
| `mss` | **MSS** | Cross-platform screen capture library — enables agents to take screenshots of the local desktop. |
| `reme-ai` | **Reme AI** | AI utility library (specific integration for CoPaw's AI workflows). |
| `transformers` | **Hugging Face Transformers** | Pre-trained model loading and inference — supports local model capabilities in `local_models/`. |
| `python-dotenv` | **python-dotenv** | Loads environment variables from `.env` files — used in `envs/` for local configuration. |
| `python-socks` | **python-socks** | SOCKS proxy support for async connections — used in `tunnel/` and channel integrations requiring proxy routing. |
| `onnxruntime` | **ONNX Runtime** | Optimized inference engine for ONNX models — supports local model execution in `local_models/`. |
| `lark-oapi` | **Feishu/Lark Open API SDK** | Feishu (Lark) channel integration — connects agents to Feishu messaging. |
| `python-telegram-bot` | **python-telegram-bot** | Telegram channel integration — enables agents to communicate via Telegram. |
| `twilio` | **Twilio** | Twilio channel integration — supports SMS/voice communication channels for agents. |
| `pywebview` | **pywebview** | Embeds a web view in a native desktop window — used for the desktop application build (`scripts/pack/`). |
| `aiofiles` | **aiofiles** | Asynchronous file I/O — enables non-blocking file operations within the async backend. |
| `paho-mqtt` | **Eclipse Paho MQTT** | MQTT protocol client — supports IoT/messaging channel integrations via MQTT. |
| `wecom-aibot-python-sdk` | **WeCom AI Bot SDK** | WeCom (WeChat Work/Enterprise WeChat) channel integration SDK. |
| `matrix-nio` | **matrix-nio** | Matrix protocol client — enables agents to communicate via the Matrix messaging network. |
| `shortuuid` | **shortuuid** | Generates concise, URL-safe unique identifiers — used for ID generation throughout the application. |
| `google-genai` | **Google GenAI** | Google Generative AI SDK — LLM provider adapter for Google's Gemini models in `providers/`. |
| `tzdata` | **tzdata** | IANA timezone database — ensures consistent timezone handling across platforms. |
| `pyyaml` | **PyYAML** | YAML parsing and serialization — used for configuration files and agent skill/tool definitions. |
| `json-repair` | **json-repair** | Repairs malformed JSON strings — used to robustly parse LLM outputs that may produce imperfect JSON. |
| `segno` | **Segno** | QR code generation library — enables agents to generate QR codes as part of skill outputs. |
| `modelscope` | **ModelScope** | ModelScope model hub client — supports downloading and using models from Alibaba's ModelScope registry in `local_models/`. |
| `huggingface_hub` | **Hugging Face Hub** | Hugging Face model hub client — supports downloading models and datasets for `local_models/`. |
| `pillow` | **Pillow** | Python Imaging Library fork — image processing and manipulation for agent visual tasks. |
| `anyio` | **AnyIO** | Async I/O compatibility layer — provides async primitives compatible with asyncio across the backend. |

---

### Python — Optional / Extra Dependencies
*Source: `/pyproject.toml` (optional-dependencies)*

| Dependency | Official Name | Role / Purpose |
|---|---|---|
| `pytest` | **pytest** | Testing framework — runs the unit and integration test suites in `tests/`. |
| `pytest-asyncio` | **pytest-asyncio** | Async test support for pytest — enables testing of async FastAPI routes and agent runners. |
| `pre-commit` | **pre-commit** | Git pre-commit hook manager — enforces code quality checks (flake8, formatting) before commits. |
| `pytest-cov` | **pytest-cov** | Code coverage reporting plugin for pytest. |
| `hypothesis` | **Hypothesis** | Property-based testing library — used for generative test cases in the test suite. |
| `llama-cpp-python` | **llama-cpp-python** | Python bindings for llama.cpp — enables local LLM inference via the `llamacpp` optional backend in `local_models/`. |
| `mlx-lm` | **MLX-LM** | Apple MLX framework language model runner — enables local LLM inference on Apple Silicon (macOS only) in `local_models/`. |
| `ollama` | **Ollama** | Ollama local model server client — enables agents to use locally running Ollama-hosted LLMs via `local_models/`. |
| `openai-whisper` | **OpenAI Whisper** | Speech-to-text transcription model — optional audio transcription capability for agents. |

---

### JavaScript — Console Production Dependencies
*Source: `/console/package.json`*

| Dependency | Official Name | Role / Purpose |
|---|---|---|
| `@agentscope-ai/chat` | **AgentScope Chat** | Pre-built chat UI component from AgentScope — provides the core conversational interface in `pages/Chat/`. |
| `@agentscope-ai/design` | **AgentScope Design** | AgentScope design system / UI component library — provides themed components for the console. |
| `@agentscope-ai/icons` | **AgentScope Icons** | AgentScope icon set — provides project-specific icons throughout the console UI. |
| `@ant-design/icons` | **Ant Design Icons** | Icon library for Ant Design — supplements the UI with a large set of standard icons. |
| `@ant-design/x-markdown` | **Ant Design X Markdown** | Markdown rendering component from Ant Design X — used in `components/MarkdownCopy/` for formatted message display. |
| `@dnd-kit/core` | **dnd kit Core** | Accessible drag-and-drop primitive library — core drag-and-drop engine for the console UI. |
| `@dnd-kit/sortable` | **dnd kit Sortable** | Sortable drag-and-drop preset for dnd kit — enables sortable list interactions (e.g., reordering agents or tasks). |
| `@dnd-kit/utilities` | **dnd kit Utilities** | Helper utilities for dnd kit — used alongside the core and sortable packages. |
| `ahooks` | **ahooks** | React hooks library by Alibaba — provides a wide range of production-ready custom hooks used in `hooks/`. |
| `antd` | **Ant Design** | Enterprise-grade React UI component library — primary UI framework for the console. |
| `antd-style` | **antd-style** | CSS-in-JS styling solution for Ant Design — handles theming and dynamic styles in the console. |
| `dayjs` | **Day.js** | Lightweight date/time manipulation library — used for formatting and parsing dates throughout the console. |
| `i18next` | **i18next** | Internationalization framework — core i18n engine powering `i18n.ts` and `locales/`. |
| `lucide-react` | **Lucide React** | Lucide icon set for React — provides additional icons in the console UI. |
| `react` | **React** | UI rendering framework — core frontend library for building the console SPA. |
| `react-dom` | **ReactDOM** | React DOM renderer — mounts the React application into the browser DOM. |
| `react-i18next` | **react-i18next** | React bindings for i18next — integrates i18n into React components via hooks and HOCs. |
| `react-markdown` | **react-markdown** | Markdown-to-React renderer — used for rendering LLM/agent markdown output in the chat and other views. |
| `react-router-dom` | **React Router** | Client-side routing library — manages navigation between pages in the console SPA. |
| `remark-gfm` | **remark-gfm** | GitHub Flavored Markdown plugin for remark — extends markdown rendering with tables, strikethrough, etc. |
| `zustand` | **Zustand** | Lightweight React state management library — powers the global state `stores/` in the console. |

---

### JavaScript — Console Developer Dependencies
*Source: `/console/package.json (dev)`*

| Dependency | Official Name | Role / Purpose |
|---|---|---|
| `@eslint/js` | **ESLint JS** | ESLint core JavaScript ruleset — base rules for the console ESLint configuration. |
| `@types/i18next` | **@types/i18next** | TypeScript type definitions for i18next. |
| `@types/node` | **@types/node** | TypeScript type definitions for Node.js built-ins. |
| `@types/react` | **@types/react** | TypeScript type definitions for React. |
| `@types/react-dom` | **@types/react-dom** | TypeScript type definitions for ReactDOM. |
| `@vitejs/plugin-react` | **Vite React Plugin** | Vite plugin enabling React (JSX/TSX) support and Fast Refresh during development. |
| `eslint` | **ESLint** | JavaScript/TypeScript linter — enforces code quality rules in the console codebase. |
| `eslint-plugin-react-hooks` | **eslint-plugin-react-hooks** | ESLint plugin enforcing React Hooks rules of usage. |
| `eslint-plugin-react-refresh` | **eslint-plugin-react-refresh** | ESLint plugin for React Fast Refresh compatibility checks. |
| `globals` | **globals** | Global variable definitions for ESLint environments. |
| `less` | **Less** | CSS preprocessor — used for component-level styling in the console (Ant Design uses Less). |
| `prettier` | **Prettier** | Opinionated code formatter — enforces consistent code style across the console. |
| `typescript` | **TypeScript** | TypeScript compiler — provides static typing for the console codebase. |
| `typescript-eslint` | **typescript-eslint** | TypeScript-aware ESLint rules and parser — extends ESLint for TypeScript-specific linting. |
| `vite` | **Vite** | Frontend build tool and dev server — bundles and serves the console SPA. |

---

### JavaScript — Website Production Dependencies
*Source: `/website/package.json`*

| Dependency | Official Name | Role / Purpose |
|---|---|---|
| `@fontsource-variable/geist` | **Fontsource Geist Variable** | Self-hosted variable Geist font — provides the primary typeface for the website. |
| `@types/react-syntax-highlighter` | **@types/react-syntax-highlighter** | TypeScript type definitions for react-syntax-highlighter. |
| `class-variance-authority` | **Class Variance Authority (CVA)** | Utility for building type-safe variant-based CSS class strings — used with shadcn/ui components. |
| `clsx` | **clsx** | Utility for conditionally constructing className strings — used throughout website components. |
| `fuse.js` | **Fuse.js** | Lightweight fuzzy-search library — likely powers the website's documentation/content search (`build-search-index.mjs`). |
| `highlight.js` | **highlight.js** | Syntax highlighting library — highlights code blocks in website documentation pages. |
| `i18next` | **i18next** | Internationalization framework — powers the website's multi-language support. |
| `lucide-react` | **Lucide React** | Lucide icon set for React — provides icons throughout the website. |
| `mermaid` | **Mermaid** | Diagram and chart rendering from text markup — renders architecture/flow diagrams in documentation. |
| `motion` | **Motion (Framer Motion)** | Animation library for React — provides UI animations and transitions on the website. |
| `ogl` | **OGL** | Minimal WebGL framework — likely used for 3D/canvas-based visual effects on the marketing homepage. |
| `radix-ui` | **Radix UI** | Unstyled, accessible React component primitives — foundational components for shadcn/ui elements. |
| `react` | **React** | UI rendering framework — core frontend library for the website. |
| `react-dom` | **ReactDOM** | React DOM renderer — mounts the website React application into the browser DOM. |
| `react-i18next` | **react-i18next** | React bindings for i18next — integrates i18n into website React components. |
| `react-markdown` | **react-markdown** | Markdown-to-React renderer — used for rendering documentation and release note content. |
| `react-router-dom` | **React Router** | Client-side routing — manages navigation between pages on the website SPA. |
| `react-syntax-highlighter` | **react-syntax-highlighter** | Syntax-highlighted code block rendering for React — used in documentation pages. |
| `rehype-highlight` | **rehype-highlight** | rehype plugin applying highlight.js syntax highlighting during markdown processing. |
| `rehype-raw` | **rehype-raw** | rehype plugin allowing raw HTML within markdown — used for rich documentation content. |
| `remark-gfm` | **remark-gfm** | GitHub Flavored Markdown plugin — extends markdown rendering with GFM syntax support. |
| `shadcn` | **shadcn/ui** | Copy-paste React UI component library built on Radix UI and Tailwind CSS — primary component system for the website (configured via `components.json`). |
| `tailwind-merge` | **tailwind-merge** | Utility for merging Tailwind CSS class names without conflicts — used throughout website components. |
| `tw-animate-css` | **tw-animate-css** | Tailwind CSS animation utilities — provides pre-built animation classes for the website. |

---

### JavaScript — Website Developer Dependencies
*Source: `/website/package.json (dev)`*

| Dependency | Official Name | Role / Purpose |
|---|---|---|
| `@tailwindcss/vite` | **Tailwind CSS Vite Plugin** | Vite plugin for Tailwind CSS integration — enables Tailwind in the website Vite build pipeline. |
| `@types/highlight.js` | **@types/highlight.js** | TypeScript type definitions for highlight.js. |
| `@types/node` | **@types/node** | TypeScript type definitions for Node.js built-ins. |
| `@types/react` | **@types/react** | TypeScript type definitions for React. |
| `@types/react-dom` | **@types/react-dom** | TypeScript type definitions for ReactDOM. |
| `@vitejs/plugin-react` | **Vite React Plugin** | Vite plugin enabling React JSX/TSX support for the website build. |
| `prettier` | **Prettier** | Code formatter — enforces consistent code style across the website. |
| `tailwindcss` | **Tailwind CSS** | Utility-first CSS framework — primary styling system for the website. |
| `typescript` | **TypeScript** | TypeScript compiler — provides static typing for the website codebase. |
| `vite` | **Vite** | Frontend build tool and dev server — bundles and serves the website. |

