
## Full Investigation — nanoclaw (8 sections)

--- dependencies ---


# Dependency and Architecture Analysis: nanoclaw_5fbda4ad

---

## Internal Modules

The following internal modules are identified from the `src/`, `setup/`, `container/agent-runner/src/`, and `scripts/` directories. Each represents a distinct, reusable component of the nanoclaw platform.

---

### Core Daemon (`src/`)

| Module | File(s) | Primary Responsibility |
|---|---|---|
| **EntryPoint** | `src/index.ts` | Main daemon startup; bootstraps and coordinates all core services |
| **Config** | `src/config.ts` | Loads and exposes application-wide configuration settings |
| **Types** | `src/types.ts` | Shared TypeScript type definitions used across the entire project |
| **Database** | `src/db.ts` | SQLite data access layer; manages persistent state for tasks, groups, and messages |
| **Router** | `src/router.ts` | Central message routing logic; directs incoming channel messages to the appropriate handler or agent |
| **TaskScheduler** | `src/task-scheduler.ts` | Manages task queuing, scheduling, and execution lifecycle |
| **GroupQueue** | `src/group-queue.ts` | Per-group FIFO message queue; ensures ordered processing within a conversation group |
| **GroupFolder** | `src/group-folder.ts` | Manages workspace folder structure for each conversation group |
| **ContainerRunner** | `src/container-runner.ts` | Spawns and manages sandboxed agent containers per task |
| **ContainerRuntime** | `src/container-runtime.ts` | Abstraction layer over Docker and Apple Container runtimes |
| **RemoteControl** | `src/remote-control.ts` | Admin/remote control interface for managing the running daemon |
| **SenderAllowlist** | `src/sender-allowlist.ts` | Security layer enforcing allowlisted sender identities |
| **MountSecurity** | `src/mount-security.ts` | Security controls for filesystem mount permissions within containers |
| **IPC** | `src/ipc.ts` | Inter-process communication channel between host daemon and in-container agent runners |
| **IPCAuth** | `src/ipc-auth.ts` | Authentication layer for IPC messages |
| **Timezone** | `src/timezone.ts` | Timezone resolution and formatting utilities |
| **Formatting** | `src/formatting.ts` | Message formatting helpers for output to communication channels |
| **Logger** | `src/logger.ts` | Centralized structured logging infrastructure |
| **Env** | `src/env.ts` | Environment variable loading and validation |

---

### Channel Adapters (`src/channels/`)

| Module | File(s) | Primary Responsibility |
|---|---|---|
| **ChannelIndex** | `src/channels/index.ts` | Public exports for the channel adapter subsystem |
| **ChannelRegistry** | `src/channels/registry.ts` | Dynamic registration and lifecycle management of per-platform channel adapters (Slack, Telegram, Discord, WhatsApp, Gmail, X/Twitter) |

---

### Setup System (`setup/`)

| Module | File(s) | Primary Responsibility |
|---|---|---|
| **SetupEntryPoint** | `setup/index.ts` | Orchestrates the full installation and onboarding wizard |
| **Environment** | `setup/environment.ts` | Validates host environment prerequisites (Node version, tools, credentials) |
| **Platform** | `setup/platform.ts` | Detects host platform (macOS vs. Linux) and adapts setup behavior |
| **ContainerSetup** | `setup/container.ts` | Configures container runtime during installation |
| **Groups** | `setup/groups.ts` | Interactive configuration of conversation groups |
| **Mounts** | `setup/mounts.ts` | Configures filesystem mount allowlists |
| **Register** | `setup/register.ts` | Registers the daemon as a system service (launchd) |
| **Service** | `setup/service.ts` | Manages service start/stop/status during setup |
| **Status** | `setup/status.ts` | Reports current setup progress and health |
| **TimezoneSetup** | `setup/timezone.ts` | Timezone detection and configuration during onboarding |
| **Verify** | `setup/verify.ts` | Post-setup verification checks to confirm correct installation |

---

### Agent Runner (`container/agent-runner/src/`)

| Module | File(s) | Primary Responsibility |
|---|---|---|
| **AgentRunner** | `container/agent-runner/src/` (2 files) | In-container Node.js process that receives task instructions via stdin, executes Claude AI agent logic, and communicates results back to the host via IPC |

---

### Container Skills (`container/skills/`)

| Module | Directory | Primary Responsibility |
|---|---|---|
| **Capabilities** | `container/skills/capabilities/` | Declares capability metadata available inside the agent container |
| **Status** | `container/skills/status/` | Skill for reporting agent status from within the container |
| **AgentBrowser** | `container/skills/agent-browser/` | Browser automation skill available to the containerized agent |
| **SlackFormatting** | `container/skills/slack-formatting/` | Slack-specific message formatting skill for agent output |

---

### Claude Skills / Plugins (`.claude/skills/`)

These are extensible skill plugins invoked by Claude AI to add capabilities to the platform. Each skill is a self-contained plugin directory.

| Skill | Directory | Primary Responsibility |
|---|---|---|
| **AddSlack** | `.claude/skills/add-slack/` | Installs and configures Slack channel integration |
| **AddTelegram** | `.claude/skills/add-telegram/` | Installs and configures Telegram channel integration |
| **AddTelegramSwarm** | `.claude/skills/add-telegram-swarm/` | Configures multi-agent Telegram swarm mode |
| **AddDiscord** | `.claude/skills/add-discord/` | Installs and configures Discord channel integration |
| **AddWhatsApp** | `.claude/skills/add-whatsapp/` | Installs and configures WhatsApp channel integration |
| **AddGmail** | `.claude/skills/add-gmail/` | Installs and configures Gmail channel integration |
| **XIntegration** | `.claude/skills/x-integration/` | Installs and configures X/Twitter channel integration |
| **AddVoiceTranscription** | `.claude/skills/add-voice-transcription/` | Adds voice input transcription capability |
| **UseLocalWhisper** | `.claude/skills/use-local-whisper/` | Configures local Whisper model for speech-to-text |
| **AddImageVision** | `.claude/skills/add-image-vision/` | Adds image processing and vision capabilities |
| **AddPdfReader** | `.claude/skills/add-pdf-reader/` | Adds PDF parsing and reading capability |
| **AddMacosStatusbar** | `.claude/skills/add-macos-statusbar/` | Adds a macOS menu bar status indicator UI |
| **AddParallel** | `.claude/skills/add-parallel/` | Enables parallel multi-agent execution mode |
| **AddCompact** | `.claude/skills/add-compact/` | Enables compact conversation mode |
| **AddOllamaTool** | `.claude/skills/add-ollama-tool/` | Integrates local Ollama LLM as an agent tool |
| **AddReactions** | `.claude/skills/add-reactions/` | Adds message reaction capabilities to channels |
| **AddEmacs** | `.claude/skills/add-emacs/` | Emacs editor integration skill |
| **ChannelFormatting** | `.claude/skills/channel-formatting/` | Cross-channel message formatting utilities |
| **ConvertToAppleContainer** | `.claude/skills/convert-to-apple-container/` | Migrates runtime configuration from Docker to Apple Container |
| **UseNativeCredentialProxy** | `.claude/skills/use-native-credential-proxy/` | Configures native credential proxy for secure credential injection |
| **InitOnecli** | `.claude/skills/init-onecli/` | Initializes the OneCLI SDK integration |
| **Setup** | `.claude/skills/setup/` | Claude-driven setup orchestration skill |
| **Customize** | `.claude/skills/customize/` | Platform customization skill |
| **UpdateNanoclaw** | `.claude/skills/update-nanoclaw/` | Self-update mechanism for the nanoclaw platform |
| **UpdateSkills** | `.claude/skills/update-skills/` | Skill self-update and synchronization |
| **Debug** | `.claude/skills/debug/` | Debugging and diagnostic tools |
| **Claw** | `.claude/skills/claw/` | Core CLI interaction and agent control scripts |
| **QodoPrResolver** | `.claude/skills/qodo-pr-resolver/` | PR resolution integration with Qodo |
| **GetQodoRules** | `.claude/skills/get-qodo-rules/` | Fetches Qodo coding rules and standards |

---

### Utility Scripts (`scripts/`)

| Module | File(s) | Primary Responsibility |
|---|---|---|
| **RunMigrations** | `scripts/run-migrations.ts` | Executes SQLite database schema migrations |

---

## External Dependencies

Dependencies are sourced exclusively from the explicitly provided dependency lists.

---

### Production Dependencies

| Dependency | Official Name | Role / Purpose | Source |
|---|---|---|---|
| `@anthropic-ai/claude-agent-sdk` | **Anthropic Claude Agent SDK** | Core SDK for running Claude AI agents inside the container; drives all AI task execution within the sandboxed agent runner | `/container/agent-runner/package.json` |
| `@modelcontextprotocol/sdk` | **Model Context Protocol (MCP) SDK** | Implements the Model Context Protocol, enabling structured tool/context communication between the agent runner and Claude AI | `/container/agent-runner/package.json` |
| `cron-parser` | **cron-parser** | Parses and evaluates cron expression strings; used in both the host daemon (task scheduling) and the agent runner | `/container/agent-runner/package.json`, `/package.json` |
| `zod` | **Zod** | TypeScript-first schema declaration and runtime data validation library; used within the agent runner for input/output validation | `/container/agent-runner/package.json` |
| `@onecli-sh/sdk` | **OneCLI SDK** | SDK for OneCLI integration; provides channel communication abstractions used by the host daemon | `/package.json` |
| `better-sqlite3` | **better-sqlite3** | Synchronous SQLite3 driver for Node.js; underpins the entire persistent state layer (`src/db.ts`) for tasks, groups, and messages | `/package.json` |
| `chromium` *(system package)* | **Chromium** | Headless browser engine installed in the agent container; used by the `agent-browser` skill for browser automation tasks | `/container/Dockerfile` |
| `agent-browser` *(global npm)* | **agent-browser** | Global npm package providing browser automation capabilities to the Claude agent within the container | `/container/Dockerfile` |
| `@anthropic-ai/claude-code` *(global npm)* | **Anthropic Claude Code** | Globally installed Claude Code CLI tool within the container; provides the core AI code agent runtime that executes tasks | `/container/Dockerfile` |

---

### Developer-Only Dependencies

| Dependency | Official Name | Role / Purpose | Source |
|---|---|---|---|
| `@types/node` | **Node.js Type Definitions** | TypeScript type declarations for the Node.js runtime API | `/container/agent-runner/package.json (dev)`, `/package.json (dev)` |
| `typescript` | **TypeScript** | TypeScript language compiler (`tsc`); compiles all `.ts` source files to JavaScript | `/container/agent-runner/package.json (dev)`, `/package.json (dev)` |
| `@eslint/js` | **ESLint JS Config** | Core ESLint JavaScript ruleset configuration | `/package.json (dev)` |
| `@types/better-sqlite3` | **better-sqlite3 Type Definitions** | TypeScript type declarations for the `better-sqlite3` library | `/package.json (dev)` |
| `eslint` | **ESLint** | Static analysis and linting tool for enforcing code quality rules | `/package.json (dev)` |
| `eslint-plugin-no-catch-all` | **eslint-plugin-no-catch-all** | ESLint plugin that disallows overly broad `catch` clauses, enforcing specific error handling | `/package.json (dev)` |
| `globals` | **globals** | Provides a standard list of global variable definitions for ESLint environments | `/package.json (dev)` |
| `husky` | **Husky** | Git hooks manager; enforces pre-commit checks (linting, formatting) before commits are recorded | `/package.json (dev)` |
| `prettier` | **Prettier** | Opinionated code formatter; ensures consistent code style across the entire codebase | `/package.json (dev)` |
| `tsx` | **tsx** | TypeScript execution engine for Node.js; enables running `.ts` files directly without a separate compile step (used for scripts and development) | `/package.json (dev)` |
| `typescript-eslint` | **typescript-eslint** | ESLint plugin and parser for TypeScript-aware linting rules | `/package.json (dev)` |
| `vitest` | **Vitest** | Fast unit testing framework; runs all test suites (`.test.ts` files) across `src/` and `setup/` | `/package.json (dev)` |

--- module_deep_dive ---


# Detailed Component Breakdown Analysis

---

## 1. `src/` — Core Application Source

### Core Responsibility
The primary runtime daemon of nanoclaw. This is the heart of the application — it initializes all services, handles message ingestion from channels, routes tasks to AI agents running in containers, manages persistent state, and coordinates all inter-system communication.

---

### Key Components

#### Entry & Configuration
| File | Role |
|---|---|
| `index.ts` | **Main daemon entry point.** Bootstraps and wires together all subsystems — loads config, starts channel listeners, initializes the task scheduler, and begins the event loop |
| `config.ts` | **Central configuration manager.** Loads and exposes application-wide settings (timeouts, limits, paths, feature flags) |
| `types.ts` | **Shared TypeScript type definitions.** Defines core domain types: `Message`, `Task`, `Group`, `Channel`, `ContainerConfig`, etc. used across all modules |
| `env.ts` | **Environment variable loader.** Reads `.env` / process environment and exposes typed, validated env values to the rest of the app |
| `logger.ts` | **Logging infrastructure.** Provides a structured logger instance used across all modules |

#### Message Routing
| File | Role |
|---|---|
| `router.ts` | **Central message router.** Receives normalized messages from channel adapters and determines which group/queue they belong to; applies routing rules |
| `routing.test.ts` | Unit tests for routing logic |

#### Task & Queue Management
| File | Role |
|---|---|
| `task-scheduler.ts` | **Task lifecycle manager.** Schedules, queues, and tracks AI tasks; manages retry logic and task state transitions |
| `task-scheduler.test.ts` | Unit tests for scheduler |
| `group-queue.ts` | **Per-group FIFO message queue.** Ensures ordered, serialized processing of messages within a group context |
| `group-queue.test.ts` | Unit tests for group queue |
| `group-folder.ts` | **Group workspace manager.** Manages per-group filesystem directories that serve as working spaces for agent containers |
| `group-folder.test.ts` | Unit tests |

#### Container Management
| File | Role |
|---|---|
| `container-runner.ts` | **Container lifecycle orchestrator.** Spawns, monitors, and tears down agent containers for each task; passes task context into the container |
| `container-runner.test.ts` | Unit tests |
| `container-runtime.ts` | **Container runtime abstraction layer.** Provides a unified interface over Docker and Apple Container runtimes — abstracts `docker run` / `container run` differences |
| `container-runtime.test.ts` | Unit tests |

#### IPC (Inter-Process Communication)
| File | Role |
|---|---|
| `ipc.ts` | **IPC channel implementation.** Manages communication between the host daemon and the Claude agent running inside the container (likely Unix socket or named pipe based) |
| `ipc-auth.ts` | **IPC authentication.** Validates that IPC messages come from authorized container processes, preventing spoofing |
| `ipc-auth.test.ts` | Unit tests |

#### Security
| File | Role |
|---|---|
| `sender-allowlist.ts` | **Sender authorization.** Validates that incoming messages come from permitted users/identities; blocks unauthorized senders |
| `sender-allowlist.test.ts` | Unit tests |
| `mount-security.ts` | **Filesystem mount access control.** Enforces the mount allowlist — controls what host paths can be bind-mounted into agent containers |

#### Database
| File | Role |
|---|---|
| `db.ts` | **SQLite data access layer.** All database read/write operations — stores task history, group state, message logs, configuration |
| `db.test.ts` | Unit tests |
| `db-migration.test.ts` | Migration-specific tests |

#### Utilities
| File | Role |
|---|---|
| `formatting.ts` | **Message formatting helpers.** Normalizes/formats text for display across channels (markdown, truncation, etc.) |
| `formatting.test.ts` | Unit tests |
| `timezone.ts` | **Timezone utilities.** Handles timezone-aware scheduling and timestamp formatting |
| `timezone.test.ts` | Unit tests |
| `remote-control.ts` | **Admin/remote control interface.** Provides a control plane for managing the daemon remotely (restart, status queries, config updates) |
| `remote-control.test.ts` | Unit tests |

#### Channel Adapters (`src/channels/`)
| File | Role |
|---|---|
| `index.ts` | **Channel module exports.** Re-exports all channel adapters for consumption by the router |
| `registry.ts` | **Dynamic channel registry.** Discovers and registers active channel adapters at runtime; enables the plugin-style addition of new channels |
| `registry.test.ts` | Unit tests |

---

### Dependencies & Interactions

**Internal dependencies (other modules this depends on):**
- `src/types.ts` — consumed by nearly every other file
- `src/db.ts` — consumed by `task-scheduler`, `router`, `group-queue`
- `src/config.ts` / `src/env.ts` — consumed broadly
- `src/ipc.ts` ↔ `container/agent-runner/` — bidirectional IPC
- `src/channels/registry.ts` → feeds into `src/router.ts`
- `src/container-runner.ts` → calls `src/container-runtime.ts`
- `src/mount-security.ts` → used by `container-runner.ts`
- `src/sender-allowlist.ts` → used by `router.ts`

**External service interactions:**
- **Docker / Apple Container CLI** — via `container-runtime.ts` (shell exec)
- **SQLite** (`better-sqlite3`) — via `db.ts`
- **Channel SDKs** (Slack Bolt, Telegram Bot API, Discord.js, etc.) — via `channels/`
- **Claude AI / Anthropic SDK** — invoked inside containers by `agent-runner`

---
---

## 2. `setup/` — Installation & Setup System

### Core Responsibility
A standalone interactive setup wizard responsible for first-time installation, environment validation, service registration, and configuration generation. It runs once (or on demand) to prepare the host system before the main daemon starts.

---

### Key Components

| File | Role |
|---|---|
| `index.ts` | **Setup orchestrator / entry point.** Coordinates the full setup flow — calls each setup step in sequence, handles user prompts, reports overall progress |
| `environment.ts` | **Environment prerequisite checker.** Validates that required tools are installed (Docker/Apple Container, Node.js, Claude CLI) and env vars are set |
| `environment.test.ts` | Unit tests |
| `platform.ts` | **Platform detection.** Detects whether running on macOS (Apple Silicon, Intel) or Linux; enables platform-specific setup paths |
| `platform.test.ts` | Unit tests |
| `container.ts` | **Container setup.** Builds or pulls the agent container image; verifies container runtime availability |
| `groups.ts` | **Group configuration setup.** Guides the user through creating and configuring message groups; writes group config files |
| `mounts.ts` | **Mount allowlist configuration.** Interactively builds `mount-allowlist.json` — defines which host directories agents can access |
| `register.ts` | **Service registration.** Registers nanoclaw as a system service (writes/loads `launchd` plist on macOS) |
| `register.test.ts` | Unit tests |
| `service.ts` | **Service management helpers.** Start/stop/status operations on the launchd service during setup |
| `service.test.ts` | Unit tests |
| `status.ts` | **Setup status reporter.** Prints formatted status of each setup step (✓/✗) to the terminal |
| `timezone.ts` | **Timezone configuration.** Detects and configures the system timezone for the daemon |
| `verify.ts` | **Post-setup verification.** Runs end-to-end checks after setup completes to confirm the system is fully operational |

---

### Dependencies & Interactions

**Internal dependencies:**
- `src/config.ts` / `src/env.ts` — reads configuration paths and env templates
- `src/types.ts` — shared type definitions
- `src/logger.ts` — logging output during setup
- References `launchd/com.nanoclaw.plist` — for service registration
- References `config-examples/mount-allowlist.json` — as template for mount config
- References `container/build.sh` / `container/Dockerfile` — for container build step

**External service interactions:**
- **macOS `launchctl`** — service registration/management via shell exec
- **Docker / Apple Container CLI** — container image build/pull
- **Filesystem** — reads/writes config files, group directories, env files

---
---

## 3. `container/` — Container Definitions & Agent Runner

### Core Responsibility
Defines the sandboxed execution environment in which Claude AI agents run. Contains both the container image definition (Dockerfile) and the Node.js application (`agent-runner`) that executes inside the container to interface with Claude and communicate results back to the host daemon.

---

### Key Components

#### Container Image
| File | Role |
|---|---|
| `Dockerfile` | **Container image definition.** Defines the base OS, installs Node.js, Claude CLI, and any agent dependencies; sets up the runtime environment for `agent-runner` |
| `build.sh` | **Image build script.** Shell script that runs `docker build` (or Apple Container equivalent) with appropriate tags and build args |

#### In-Container Skills (`container/skills/`)
| Directory | Role |
|---|---|
| `capabilities/` | **Capability declarations.** Defines what capabilities/tools the agent inside the container exposes |
| `status/` | **Status reporting skill.** Allows the agent to report its current status back through IPC |
| `agent-browser/` | **Browser automation skill.** Provides web browsing capability to the Claude agent (likely via Playwright or Puppeteer) |
| `slack-formatting/` | **Slack-specific formatting skill.** Helps the agent format responses for Slack's markdown dialect |

#### Agent Runner (`container/agent-runner/`)
| File/Directory | Role |
|---|---|
| `package.json` | **Agent runner Node.js manifest.** Declares dependencies for the in-container application (Claude SDK, IPC client, etc.) |
| `package-lock.json` | Dependency lock file |
| `tsconfig.json` | TypeScript compiler config for the agent runner |
| `src/` (2 files) | **Agent runner source code.** The small but critical Node.js application that: (1) receives task context from the host via IPC, (2) invokes Claude CLI/SDK with the task, and (3) streams results back to the host daemon |

---

### Dependencies & Interactions

**Internal dependencies:**
- `src/ipc.ts` (host side) ↔ `agent-runner/src/` (container side) — **primary communication channel**
- `src/container-runner.ts` — spawns this container and injects task context
- `container/skills/` — loaded by `agent-runner` to extend Claude's capabilities

**External service interactions:**
- **Claude CLI / Anthropic SDK** — the agent runner directly invokes Claude inside the container
- **Host filesystem** (via bind mounts) — accesses allowed directories from `mount-allowlist.json`
- **IPC socket** — communicates with host daemon through mounted socket
- **Browser automation runtime** (via `agent-browser` skill) — Playwright/Puppeteer for web tasks

---
---

## 4. `.claude/skills/` — Claude AI Skill Plugins

### Core Responsibility
An extensible plugin system that adds capabilities, integrations, and behaviors to nanoclaw by providing Claude with instructions, tools, and scripts. Each skill is a self-contained directory that Claude can be instructed to "apply" to extend the platform.

---

### Key Components

#### Channel Integration Skills
| Skill | Role |
|---|---|
| `add-slack/` | Installs and configures Slack integration (Slack Bolt app setup, env vars, channel adapter) |
| `add-telegram/` | Installs Telegram bot integration (Bot API token setup, adapter) |
| `add-telegram-swarm/` | Telegram multi-agent swarm variant |
| `add-discord/` | Discord bot integration (Discord.js, guild setup) |
| `add-whatsapp/` | WhatsApp integration (likely via Twilio or WhatsApp Business API) |
| `add-gmail/` | Gmail integration (OAuth2 setup, Gmail API) |
| `x-integration/` | X/Twitter integration with nested `lib/` and `scripts/` subdirectories |

#### Capability Skills
| Skill | Role |
|---|---|
| `add-voice-transcription/` | Adds voice message transcription capability to a channel |
| `use-local-whisper/` | Configures OpenAI Whisper running locally for STT |
| `add-image-vision/` | Enables Claude to process and analyze images |
| `add-pdf-reader/` | Enables PDF document reading/extraction |
| `add-parallel/` | Enables parallel agent execution for concurrent tasks |
| `add-compact/` | Compact/minimal response mode configuration |
| `add-ollama-tool/` | Integrates local Ollama LLM as an alternative/supplementary model |
| `add-reactions/` | Adds emoji reaction responses to messages |

#### UI/Platform Skills
| Skill | Role |
|---|---|
| `add-macos-statusbar/` | Adds a macOS menu bar status indicator for the daemon |
| `add-emacs/` | Emacs editor integration |

#### Infrastructure Skills
| Skill | Role |
|---|---|
| `setup/` | Core setup skill (2 files) — Claude-guided setup process |
| `customize/` | Allows customizing nanoclaw behavior via Claude instructions |
| `update-nanoclaw/` | Self-update mechanism — Claude can update the nanoclaw installation |
| `update-skills/` | Updates installed skills to latest versions |
| `convert-to-apple-container/` | Migrates Docker setup to Apple Container runtime |
| `use-native-credential-proxy/` | Configures native credential proxy for secure secret handling |
| `channel-formatting/` | Cross-channel message formatting rules |
| `init-onecli/` | Initializes the "onecli" interface |

#### Developer/Debug Skills
| Skill | Role |
|---|---|
| `debug/` | Debugging tools and diagnostic commands |
| `claw/` | Core "claw" CLI interface with nested `scripts/` |
| `qodo-pr-resolver/` | PR resolution automation with `resources/` |
| `get-qodo-rules/` | Fetches Qodo coding rules with `references/` |

---

### Dependencies & Interactions

**Internal dependencies:**
- Skills modify files throughout the project (`src/channels/`, `src/config.ts`, `.env`, `groups/`, etc.)
- `setup/` skill interacts with `setup/` directory
- `update-nanoclaw/` skill interacts with `package.json`, git, npm
- Channel skills add to `src/channels/registry.ts`

**External service interactions:**
- **Slack API** (Bolt framework) — `add-slack/`
- **Telegram Bot API** — `add-telegram/`
- **Discord API** — `add-discord/`
- **WhatsApp Business API / Twilio** — `add-whatsapp/`
- **Gmail API (OAuth2)** — `add-gmail/`
- **X/Twitter API v2** — `x-integration/`
- **OpenAI Whisper (local)** — `use-local-whisper/`
- **Ollama API** — `add-ollama-tool/`
- **macOS Accessibility/Menu Bar APIs** — `add-macos-statusbar/`
- **GitHub API** — `qodo-pr-resolver/`

---
---

## 5. `scripts/` — Utility Scripts

### Core Responsibility
Provides standalone operational scripts for database maintenance tasks — specifically database schema migrations. These are run independently from the main daemon, typically during upgrades or initial setup.

---

### Key Components

| File | Role |
|---|---|
| `run-migrations.ts` | **Database migration runner.** Discovers and applies pending SQLite schema migrations in sequence; tracks which migrations have been applied to prevent re-execution |

---

### Dependencies & Interactions

**Internal dependencies:**
- `src/db.ts` — uses the database access layer to apply migrations
- `src/env.ts` — reads database path from environment
- `src/logger.ts` — logs migration progress
- `src/types.ts` — shared types

**External service interactions:**
- **SQLite** (`better-sqlite3`) — directly modifies the database schema
- **Filesystem** — reads migration files from a migrations directory

---
---

## 6. `groups/` — Group Configuration

### Core Responsibility
Stores per-group behavioral instructions for Claude AI agents. Each group is a logical unit (e.g., a Slack workspace, a set of users) that can have custom AI behavior defined via markdown instruction files.

---

### Key Components

| File | Role |
|---|---|
| `global/CLAUDE.md` | **Global agent instructions.** Default behavioral rules applied to Claude in all groups — defines baseline personality, capabilities, restrictions |
| `main/CLAUDE.md` | **Main group instructions.** Specific instructions for the "main" group — may include group-specific tools, personas, or task constraints |

---

### Dependencies & Interactions

**Internal dependencies:**
- `src/group-folder.ts` — discovers and loads these `CLAUDE.md` files per group
- `src/container-runner.ts` — passes the relevant `CLAUDE.md` content as context when spawning agent containers
- `setup/groups.ts` — creates new group directories with `CLAUDE.md` templates

**External service interactions:**
- **Claude AI** — `CLAUDE.md` files are directly consumed as system prompts/instructions by Claude inside containers

---
---

## 7. `launchd/` — macOS Service Definition

### Core Responsibility
Defines nanoclaw as a persistent macOS system service managed by `launchd`, enabling the daemon to start automatically on login/boot and be managed via standard macOS service tooling.

---

### Key Components

| File | Role |
|---|---|
| `com.nanoclaw.plist` | **launchd service descriptor.** XML property list defining: executable path, working directory, environment variables, stdout/stderr log paths, run-at-load behavior, and keep-alive policy |

---

### Dependencies & Interactions

**Internal dependencies:**
- References `src/index.ts` (compiled output) as the executable
- References `.env` or environment variable injection for the daemon
- Used by `setup/register.ts` — copied to `~/Library/LaunchAgents/` during setup
- Used by `setup/service.ts` — loaded/unloaded via `launchctl`

**External service interactions:**
- **macOS launchd** — the OS-level service manager that reads and executes this plist

---
---

## 8. `repo-tokens/` — CI Badge & Token System

### Core Responsibility
A GitHub Actions composite action that generates and updates SVG status badges for the repository. Used in CI pipelines to reflect build/test status visually in the README.

---

### Key Components

| File | Role |
|---|---|
| `README.md` | Documentation for the token/badge system |
| `action.yml` | **GitHub Actions composite action definition.** Defines inputs, steps, and logic for generating/updating badges |
| `badge.svg` | **Current status badge.** The live SVG badge displayed in the repository README |
| `examples/green.svg` | Example green (passing) badge |
| `examples/red.svg` | Example red (failing) badge |
| `examples/yellow-green.svg` | Example yellow-green badge |
| `examples/yellow.svg` | Example yellow badge |

---

### Dependencies & Interactions

**Internal dependencies:**
- Referenced by `.github/workflows/ci.yml` — called as a composite action after test runs
- Referenced by `.github/workflows/update-tokens.yml` — dedicated workflow for token updates

**External service interactions:**
- **GitHub Actions runner** — executes the action
- **GitHub API** — commits updated `badge.svg` back to the repository

---
---

## Dependency Interaction Map

```
┌─────────────────────────────────────────────────────────────┐
│                        src/ (core daemon)                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐  │
│  │ channels/│→ │ router   │→ │task-sched│→ │container   │  │
│  │ registry │  │          │  │group-queue│  │-runner     │  │
│  └──────────┘  └──────────┘  └──────────┘  └─────┬──────┘  │
│       ↑              ↓              ↓             ↓         │
│  [Slack/TG/   [sender-      [db.ts /         [container-  │
│   Discord/    allowlist]    SQLite]           runtime.ts]  │
│   etc. SDKs]                                      ↓        │
└───────────────────────────────────────────────────┼────────┘
                                                    ↓
┌───────────────────────────┐         ┌─────────────────────────┐
│   container/              │←─IPC───→│  container/agent-runner/ │
│   (Docker/Apple Image)    │         │  (Claude invocation)     │
└───────────────────────────┘         └─────────────────────────┘
         ↑                                         ↓
┌────────────────┐                    ┌─────────────────────────┐
│   setup/       │                    │   .claude/skills/        │
│ (install once) │                    │  (extend capabilities)   │
└────────────────┘                    └─────────────────────────┘
         ↑                                         
┌────────────────┐           ┌─────────────────────────┐
│  launchd/      │           │  groups/CLAUDE.md        │
│ (OS service)   │           │ (agent instructions)     │
└────────────────┘           └─────────────────────────┘
```

--- events ---


I'll systematically analyze the codebase files to identify any event-producing or event-consuming patterns.

Let me examine the key source files:

## Analysis of Key Files

Looking at the repository structure, this appears to be a CLI/agent tool called "nanoclaw". Let me trace through the relevant source files:

**`src/ipc.ts`** - IPC (Inter-Process Communication) system
**`src/group-queue.ts`** - Queue management for groups
**`src/remote-control.ts`** - Remote control functionality
**`src/router.ts`** - Message routing
**`src/channels/`** - Channel integrations
**`src/types.ts`** - Type definitions

Based on the repository structure and the available file listings, this codebase appears to be a local agent runner / CLI tool ("nanoclaw") that:

- Uses **IPC (Inter-Process Communication)** for internal messaging (`src/ipc.ts`)
- Has a **group queue** system (`src/group-queue.ts`)
- Has a **router** for directing messages (`src/router.ts`)
- Has **channel integrations** (Slack, Telegram, Discord, WhatsApp, Gmail, etc. referenced in `.claude/skills/`)
- Has a **remote-control** mechanism (`src/remote-control.ts`)
- Uses a **container runner** for sandboxed execution

However, without the **actual file contents** being provided (only the directory tree/filenames are shown), I cannot inspect the source code to identify specific event broker interactions (e.g., `sqs.sendMessage`, `kafka.produce`, `ably.publish`, etc.), payload structures, or serialization/deserialization logic.

The file listing shows filenames only — no source code content was included in the provided context. Without the actual code content, I cannot accurately document event payloads, directions, or broker types.

---

**no events**

> **Note:** The repository structure suggests internal IPC and queue mechanisms (`src/ipc.ts`, `src/group-queue.ts`, `src/router.ts`, `src/channels/`), as well as potential integrations with external messaging platforms (Slack, Telegram, Discord, WhatsApp, Gmail — referenced in `.claude/skills/`). However, since the **actual source file contents** were not provided in this context — only the directory tree — it is not possible to perform a meaningful code-level analysis to extract, confirm, or document specific event definitions, payload schemas, broker types, or message directions. If you can provide the file contents, a full event documentation can be produced.

--- core_entities ---


# Domain Model Analysis: nanoclaw_5fbda4ad

## Overview

Based on the repository structure and file names, this project appears to be an **AI agent orchestration platform** that manages Claude AI agents running in containers, communicates via multiple messaging channels (Slack, Telegram, Discord, WhatsApp, etc.), and supports group-based task routing.

---

## 1. Core Data Entities

### 1.1 `Message`
The fundamental unit of communication flowing through the system.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string/uuid | Unique message identifier |
| `content` | string | Text body of the message |
| `senderId` | string | Identifier of the sender |
| `channelId` | string | The channel this message belongs to |
| `groupId` | string | The group this message is routed to |
| `timestamp` | datetime | When the message was received/sent |
| `format` | enum | Channel-specific formatting (Slack, Telegram, etc.) |
| `attachments` | array | Files, images, PDFs, voice transcriptions |

---

### 1.2 `Channel`
Represents an integration with a messaging platform.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string | Unique channel identifier |
| `type` | enum | `slack`, `telegram`, `discord`, `whatsapp`, `gmail`, `x`, `emacs` |
| `name` | string | Human-readable channel name |
| `config` | object | Platform-specific configuration (tokens, webhooks) |
| `credentials` | object | Auth tokens/secrets (managed via credential proxy) |
| `isActive` | boolean | Whether the channel is currently enabled |
| `groupId` | string | Associated group for routing |

---

### 1.3 `Group`
A logical grouping of agents and channel configurations sharing a common context.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string | Unique group identifier |
| `name` | string | Group name (e.g., `global`, `main`) |
| `folderPath` | string | Filesystem path for group-specific data |
| `claudeConfig` | object | Claude-specific instructions (`CLAUDE.md` content) |
| `createdAt` | datetime | Group creation time |
| `isActive` | boolean | Whether the group is active |

> Evidence: `groups/`, `src/group-folder.ts`, `src/group-queue.ts`, `setup/groups.ts`

---

### 1.4 `Container`
Represents a sandboxed execution environment for running AI agent tasks.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string | Unique container identifier |
| `runtime` | enum | Container runtime type (Docker, Apple Container) |
| `status` | enum | `running`, `stopped`, `error`, `pending` |
| `groupId` | string | The group this container belongs to |
| `mounts` | array | Filesystem mount configurations |
| `imageRef` | string | Container image reference |
| `createdAt` | datetime | Container creation time |
| `environment` | object | Environment variables injected at runtime |
| `platform` | string | Target platform (macOS, Linux, etc.) |

> Evidence: `src/container-runner.ts`, `src/container-runtime.ts`, `setup/container.ts`, `container/Dockerfile`

---

### 1.5 `Task`
A unit of work scheduled and dispatched to an agent container.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string | Unique task identifier |
| `groupId` | string | The group this task belongs to |
| `messageId` | string | Originating message (if triggered by one) |
| `status` | enum | `pending`, `running`, `completed`, `failed` |
| `scheduledAt` | datetime | When the task is scheduled to run |
| `startedAt` | datetime | Actual execution start time |
| `completedAt` | datetime | Execution completion time |
| `containerId` | string | Container assigned to execute the task |
| `payload` | object | Task input data/instructions |
| `result` | object | Task output/response |

> Evidence: `src/task-scheduler.ts`, `src/task-scheduler.test.ts`

---

### 1.6 `Sender`
An entity (user or bot) that sends messages through a channel.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string | Unique sender identifier |
| `externalId` | string | Platform-specific user ID |
| `channelType` | enum | Which channel platform the sender is on |
| `displayName` | string | Human-readable name |
| `isAllowed` | boolean | Whether sender is on the allowlist |
| `metadata` | object | Platform-specific sender attributes |

> Evidence: `src/sender-allowlist.ts`, `src/sender-allowlist.test.ts`

---

### 1.7 `Mount`
Filesystem mount configuration for container security and access control.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string | Unique mount identifier |
| `hostPath` | string | Path on the host machine |
| `containerPath` | string | Path inside the container |
| `mode` | enum | `readonly`, `readwrite` |
| `isAllowed` | boolean | Whether this mount is permitted |
| `groupId` | string | Owning group (if group-scoped) |

> Evidence: `src/mount-security.ts`, `setup/mounts.ts`, `config-examples/mount-allowlist.json`

---

### 1.8 `Skill`
A pluggable capability or integration script that extends agent functionality.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string | Unique skill identifier |
| `name` | string | Skill name (e.g., `add-slack`, `add-telegram`) |
| `description` | string | What the skill provides |
| `scriptPath` | string | Path to the skill's entry script |
| `dependencies` | array | Other skills or packages required |
| `isInstalled` | boolean | Whether the skill is currently active |
| `version` | string | Skill version |

> Evidence: `.claude/skills/`, `container/skills/`, `docs/skills-as-branches.md`

---

### 1.9 `IpcSession` (Inter-Process Communication Session)
Represents an authenticated IPC connection between components.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string | Session identifier |
| `token` | string | Authentication token |
| `createdAt` | datetime | Session creation time |
| `expiresAt` | datetime | Token expiry time |
| `isValid` | boolean | Current validity state |
| `clientId` | string | Identifying the connecting process |

> Evidence: `src/ipc.ts`, `src/ipc-auth.test.ts`

---

### 1.10 `RemoteControl`
Represents a remote command/control instruction sent to the system.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string | Command identifier |
| `command` | string | The control command to execute |
| `parameters` | object | Command arguments |
| `issuedAt` | datetime | When the command was issued |
| `status` | enum | `pending`, `executed`, `failed` |
| `sourceChannelId` | string | Channel the command originated from |

> Evidence: `src/remote-control.ts`, `src/remote-control.test.ts`

---

### 1.11 `ServiceRegistration`
Represents a registered system service (e.g., launchd on macOS).

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string | Service identifier |
| `name` | string | Service name |
| `platform` | string | OS platform |
| `plistPath` | string | macOS launchd plist path |
| `status` | enum | `registered`, `running`, `stopped` |
| `timezone` | string | Service timezone configuration |

> Evidence: `setup/service.ts`, `setup/register.ts`, `launchd/com.nanoclaw.plist`, `setup/timezone.ts`

---

## 2. Entity Relationships

```
┌─────────────────────────────────────────────────────────────────────┐
│                          RELATIONSHIP DIAGRAM                        │
└─────────────────────────────────────────────────────────────────────┘

Group (1) ──────────────────────────────── (many) Channel
  │                                                │
  │ (1:many)                                       │ (1:many)
  │                                                │
  ▼                                                ▼
Task (many) ──── assigned to ────► Container    Message (many)
  │                                    │            │
  │ (triggered by)                     │ (1:many)   │ (from)
  │                                    │            ▼
  ▼                                    ▼          Sender
Message                              Mount
  │
  │ (routed via)
  ▼
Router ──────► Group Queue ──────► Task Scheduler
```

### Detailed Relationships

| Relationship | Type | Description |
|---|---|---|
| `Group` → `Channel` | **One-to-Many** | A group can have multiple channels configured |
| `Group` → `Task` | **One-to-Many** | A group maintains a queue of tasks |
| `Group` → `Container` | **One-to-Many** | A group can spin up multiple containers |
| `Channel` → `Message` | **One-to-Many** | A channel delivers many messages |
| `Channel` → `Sender` | **One-to-Many** | A channel has many senders |
| `Message` → `Task` | **One-to-One** | An inbound message triggers one task |
| `Task` → `Container` | **Many-to-One** | Many tasks are executed within a container |
| `Container` → `Mount` | **One-to-Many** | A container has multiple filesystem mounts |
| `Group` → `Skill` | **Many-to-Many** | Groups can enable multiple skills; skills can be shared across groups |
| `Sender` → `Message` | **One-to-Many** | A sender sends many messages |
| `IpcSession` → `RemoteControl` | **One-to-Many** | An authenticated session can issue multiple commands |
| `RemoteControl` → `Container` | **Many-to-One** | Commands target a specific container |

---

## 3. Data Flow Summary

```
External Channel (Slack/Telegram/etc.)
        │
        ▼
   [Message received]
        │
        ▼
  Sender Allowlist Check ──✗──► Rejected
        │ ✓
        ▼
   Channel Registry
        │
        ▼
     Router ──────► Group Resolution
        │
        ▼
   Group Queue (FIFO per group)
        │
        ▼
  Task Scheduler
        │
        ▼
  Container Runner ──► Container Runtime
        │                    │
        ▼                    ▼
   Task Result           Mount Security
        │
        ▼
  Response formatted per Channel type
        │
        ▼
  Reply sent back to Channel
```

---

## 4. Persistence Notes

> Evidence from `src/db.ts`, `src/db-migration.test.ts`, `scripts/run-migrations.ts`

- The project uses a **relational or embedded database** (likely SQLite based on the embedded/local nature of the application) with **migration support**.
- Entities likely persisted: `Message`, `Task`, `Group`, `Channel`, `Sender` (allowlist), `Mount` (allowlist), `IpcSession`.
- `Container` state may be partially ephemeral, tracked in-memory or via the container runtime API.

--- hl_overview ---


# Repository Analysis: nanoclaw_5fbda4ad

## [[nanoclaw]]

---

## 1. Project Purpose

**nanoclaw** is an **AI agent orchestration platform** that serves as a bridge between messaging/communication channels (Slack, Telegram, Discord, WhatsApp, Gmail, X/Twitter) and Claude AI (Anthropic's LLM). It enables users to interact with Claude AI through various messaging interfaces, running AI tasks inside sandboxed containers (Docker or Apple Containers) for security and isolation.

**Primary Domain:** AI agent infrastructure / chatbot orchestration with multi-channel messaging integration and containerized execution environments.

---

## 2. Architecture Pattern

- **Event-Driven / Message Queue Architecture** — incoming messages from channels are routed through a queue system to AI agents
- **Plugin/Skills Architecture** — extensible "skills" system for adding capabilities
- **Container-per-task Sandboxing** — each AI agent task runs in an isolated container
- **Multi-channel Gateway Pattern** — unified routing layer abstracting multiple communication platforms

---

## 3. Technology Stack

| Category | Technology |
|---|---|
| **Primary Language** | TypeScript |
| **Runtime** | Node.js (version pinned via `.nvmrc`) |
| **AI Backend** | Anthropic Claude (via Claude CLI / MCP) |
| **Database** | SQLite (via `better-sqlite3`) |
| **Container Runtime** | Docker / Apple Container (macOS native) |
| **Testing** | Vitest |
| **Linting/Formatting** | ESLint, Prettier |
| **Process Management** | launchd (macOS), via `.plist` file |
| **Package Manager** | npm (package-lock.json present) |
| **Build System** | TypeScript compiler (`tsc`) |
| **Communication Channels** | Slack, Telegram, Discord, WhatsApp, Gmail, X/Twitter |
| **IPC** | Custom IPC layer (`ipc.ts`) |
| **Security** | Mount allowlisting, sender allowlisting, IPC auth |
| **Voice** | Whisper integration (local) |

**Key inferred dependencies** (from file structure and naming):
- `better-sqlite3` — SQLite ORM/driver
- Anthropic SDK / Claude CLI
- Channel-specific SDKs (Slack Bolt, Telegram Bot API, Discord.js, etc.)
- `vitest` — unit testing
- `eslint`, `prettier` — code quality

---

## 4. Initial Structure Impression

The application consists of these high-level parts:

1. **Core Daemon (`src/`)** — Main application runtime handling message routing, container management, scheduling
2. **Setup/Install System (`setup/`)** — Onboarding, environment validation, service registration
3. **Container Runtime (`container/`)** — Docker/Apple container definitions and agent runner code
4. **Skills System (`.claude/skills/`)** — Pluggable feature extensions (integrations, capabilities)
5. **Channel Adapters (`src/channels/`)** — Per-platform messaging integrations
6. **Scripts (`scripts/`)** — Utility scripts (DB migrations)
7. **Documentation (`docs/`)** — Architecture and operational docs

---

## 5. Configuration / Package Files

| File | Purpose |
|---|---|
| `package.json` | Root Node.js project manifest, scripts, dependencies |
| `package-lock.json` | Dependency lock file |
| `tsconfig.json` | TypeScript compiler configuration |
| `eslint.config.js` | ESLint linting rules |
| `.prettierrc` | Code formatting rules |
| `vitest.config.ts` | Test runner configuration (main tests) |
| `vitest.skills.config.ts` | Test runner configuration (skills tests) |
| `.env.example` | Environment variable template |
| `.nvmrc` | Node.js version pin |
| `.mcp.json` | MCP (Model Context Protocol) configuration |
| `.gitignore` | Git ignore rules |
| `.husky/pre-commit` | Git pre-commit hooks |
| `launchd/com.nanoclaw.plist` | macOS launchd service definition |
| `container/Dockerfile` | Container image definition |
| `container/build.sh` | Container build script |
| `container/agent-runner/package.json` | Agent runner sub-package manifest |
| `container/agent-runner/package-lock.json` | Agent runner dependency lock |
| `container/agent-runner/tsconfig.json` | Agent runner TypeScript config |
| `.claude/settings.json` | Claude AI settings |
| `config-examples/mount-allowlist.json` | Example mount security configuration |
| `setup.sh` | Shell-based setup script |
| `.github/workflows/ci.yml` | CI pipeline |
| `.github/workflows/bump-version.yml` | Automated version bumping |
| `.github/workflows/label-pr.yml` | PR labeling automation |
| `.github/workflows/update-tokens.yml` | Token update automation |

---

## 6. Directory Structure

```
nanoclaw/
├── src/                        # Core application source
│   ├── index.ts                # Main entry point / daemon startup
│   ├── config.ts               # Application configuration
│   ├── types.ts                # Shared TypeScript types
│   ├── db.ts                   # Database access layer (SQLite)
│   ├── router.ts               # Message routing logic
│   ├── routing.test.ts         # Routing tests
│   ├── task-scheduler.ts       # Task scheduling / queue management
│   ├── group-queue.ts          # Per-group message queuing
│   ├── group-folder.ts         # Group workspace folder management
│   ├── container-runner.ts     # Spawns/manages agent containers
│   ├── container-runtime.ts    # Container runtime abstraction (Docker/Apple)
│   ├── remote-control.ts       # Remote control / admin interface
│   ├── sender-allowlist.ts     # Security: allowed senders
│   ├── mount-security.ts       # Security: filesystem mount controls
│   ├── ipc.ts                  # Inter-process communication
│   ├── ipc-auth.ts             # IPC authentication
│   ├── timezone.ts             # Timezone utilities
│   ├── formatting.ts           # Message formatting helpers
│   ├── logger.ts               # Logging infrastructure
│   ├── env.ts                  # Environment variable loading
│   └── channels/               # Channel integration adapters
│       ├── index.ts            # Channel exports
│       ├── registry.ts         # Channel registry (dynamic loading)
│       └── registry.test.ts
│
├── setup/                      # Installation & setup system
│   ├── index.ts                # Setup entry point
│   ├── environment.ts          # Environment validation
│   ├── platform.ts             # Platform detection (macOS/Linux)
│   ├── container.ts            # Container setup
│   ├── groups.ts               # Group configuration setup
│   ├── mounts.ts               # Mount configuration
│   ├── register.ts             # Service registration
│   ├── service.ts              # Service management
│   ├── status.ts               # Setup status reporting
│   ├── timezone.ts             # Timezone setup
│   └── verify.ts               # Post-setup verification
│
├── container/                  # Container definitions
│   ├── Dockerfile              # Agent container image
│   ├── build.sh                # Build script
│   ├── skills/                 # Skills available inside container
│   │   ├── capabilities/       # Capability declarations
│   │   ├── status/             # Status reporting skill
│   │   ├── agent-browser/      # Browser automation skill
│   │   └── slack-formatting/   # Slack-specific formatting
│   └── agent-runner/           # Node.js app running inside container
│       └── src/                # Agent runner source (2 files)
│
├── .claude/                    # Claude AI configuration & skills
│   ├── settings.json           # Claude settings
│   └── skills/                 # Extensible skill plugins
│       ├── add-slack/          # Slack integration skill
│       ├── add-telegram/       # Telegram integration skill
│       ├── add-discord/        # Discord integration skill
│       ├── add-whatsapp/       # WhatsApp integration skill
│       ├── add-gmail/          # Gmail integration skill
│       ├── x-integration/      # X/Twitter integration
│       ├── add-voice-transcription/  # Voice input
│       ├── use-local-whisper/  # Local STT
│       ├── add-image-vision/   # Image processing
│       ├── add-pdf-reader/     # PDF processing
│       ├── add-macos-statusbar/# macOS menu bar UI
│       ├── add-parallel/       # Parallel execution
│       ├── add-compact/        # Compact mode
│       ├── debug/              # Debugging tools
│       ├── setup/              # Setup skill
│       ├── customize/          # Customization skill
│       ├── update-nanoclaw/    # Self-update mechanism
│       └── ...                 # More skills
│
├── groups/                     # Group-specific CLAUDE.md configs
│   ├── global/CLAUDE.md        # Global group instructions
│   └── main/CLAUDE.md          # Main group instructions
│
├── scripts/                    # Utility scripts
│   └── run-migrations.ts       # Database migration runner
│
├── docs/                       # Technical documentation
├── assets/                     # Images and branding
├── config-examples/            # Example configuration files
├── repo-tokens/                # Token/badge system (CI integration)
└── launchd/                    # macOS service definition
```

**Organization style:** Primarily **by feature/layer hybrid** — core infrastructure organized by technical layer (`db`, `router`, `container-runner`), while skills/channels are organized by feature.

---

## 7. High-Level Architecture

### Pattern: **Event-Driven Multi-Channel AI Agent Gateway**

```
[Messaging Channels]          [Core Daemon]              [AI Execution]
  Slack ──────────────┐
  Telegram ───────────┤    ┌─────────────┐         ┌──────────────────┐
  Discord ────────────┼───▶│  Channel    │         │  Container       │
  WhatsApp ───────────┤    │  Registry   │──route──▶│  Runner          │
  Gmail ──────────────┤    │  & Router   │         │  (Claude Agent)  │
  X/Twitter ──────────┘    └──────┬──────┘         └──────────────────┘
                                  │
                           ┌──────▼──────┐
                           │ Task/Group  │
                           │ Scheduler   │
                           └──────┬──────┘
                                  │
                           ┌──────▼──────┐
                           │  SQLite DB  │
                           │  (state)    │
                           └─────────────┘
```

**Evidence:**
- `src/channels/registry.ts` — dynamic channel registration pattern
- `src/router.ts` + `src/routing.test.ts` — centralized message routing
- `src/group-queue.ts` + `src/task-scheduler.ts` — async queue/scheduling
- `src/container-runner.ts` + `src/container-runtime.ts` — container abstraction layer
- `src/ipc.ts` — inter-process communication between host daemon and container agents
- `src/db.ts` — persistent state management
- `.claude/skills/` — plugin architecture for extensibility
- `launchd/com.nanoclaw.plist` — runs as a persistent background service

---

## 8. Build, Execution and Test

### Build
```bash
# Compile TypeScript
npx tsc

# Container build
./container/build.sh
```

### Run
```bash
# Initial setup
./setup.sh
# or
npx ts-node setup/index.ts

# Start daemon (development)
npx ts-node src/index.ts

# Start as macOS service (production)
launchctl load launchd/com.nanoclaw.plist
```

### Test
```bash
# Run main test suite
npx vitest --config vitest.config.ts

# Run skills tests
npx vitest --config vitest.skills.config.ts
```

### Database Migrations
```bash
npx ts-node scripts/run-migrations.ts
```

### Main Entry Points
| Entry Point | Purpose |
|---|---|
| `src/index.ts` | Main daemon process |
| `setup/index.ts` | Setup/installation wizard |
| `container/agent-runner/src/` | In-container agent runner |
| `scripts/run-migrations.ts` | DB migrations |

### CI/CD
- GitHub Actions workflows for CI (`ci.yml`), version bumping (`bump-version.yml`), PR labeling (`label-pr.yml`), and token updates (`update-tokens.yml`)
- Pre-commit hooks via Husky for code quality enforcement

--- service_dependencies ---


# External Dependencies Analysis: `nanoclaw_5fbda4ad`

---

## Summary

This codebase is a Node.js/TypeScript application that orchestrates AI agents (Claude) within isolated container environments, with channel integrations (Slack, Telegram, WhatsApp, Discord, etc.) and a task scheduling/routing system. Below is a comprehensive list of all identified external dependencies.

---

## 1. `@anthropic-ai/claude-agent-sdk`

| Field | Details |
|---|---|
| **Dependency Name** | Anthropic Claude Agent SDK |
| **Type of Dependency** | Third-party Library / External AI Service SDK |
| **Purpose/Role** | Core SDK for running Claude AI agents inside the container runner. This is the primary AI engine that processes tasks and conversations. |
| **Integration Point/Clues** | Listed as a production dependency in `/container/agent-runner/package.json` (`"@anthropic-ai/claude-agent-sdk": "^0.2.76"`). Also installed globally in the Docker container via `RUN npm install -g agent-browser @anthropic-ai/claude-code` (container/Dockerfile). |

---

## 2. `@anthropic-ai/claude-code`

| Field | Details |
|---|---|
| **Dependency Name** | Anthropic Claude Code CLI |
| **Type of Dependency** | Third-party External Tool / CLI |
| **Purpose/Role** | Installed globally in the container image as a CLI tool, likely used to invoke Claude code execution capabilities directly within the sandboxed Linux VM environment. |
| **Integration Point/Clues** | Found in `/container/Dockerfile`: `RUN npm install -g agent-browser @anthropic-ai/claude-code`. |

---

## 3. `@modelcontextprotocol/sdk`

| Field | Details |
|---|---|
| **Dependency Name** | Model Context Protocol (MCP) SDK |
| **Type of Dependency** | Third-party Library / Protocol SDK |
| **Purpose/Role** | Implements the Model Context Protocol, enabling the agent runner to communicate with MCP-compliant tool servers and expose/consume context in a standardized way. Also referenced in `.mcp.json` at the project root. |
| **Integration Point/Clues** | Listed as a production dependency in `/container/agent-runner/package.json` (`"@modelcontextprotocol/sdk": "^1.12.1"`). Root-level `.mcp.json` file suggests MCP server configuration is present at the project level. |

---

## 4. `@onecli-sh/sdk`

| Field | Details |
|---|---|
| **Dependency Name** | OneCLI SDK |
| **Type of Dependency** | Third-party Library / External Service SDK |
| **Purpose/Role** | Core SDK for the OneCLI platform integration — likely provides channel routing, messaging abstractions, or orchestration infrastructure that the host application depends on. |
| **Integration Point/Clues** | Listed as a production dependency in the root `/package.json` (`"@onecli-sh/sdk": "^0.2.0"`). The presence of `setup/register.ts`, `setup/service.ts`, and `src/router.ts` suggests this SDK is used to register the bot as a service on the OneCLI platform. |

---

## 5. `better-sqlite3`

| Field | Details |
|---|---|
| **Dependency Name** | better-sqlite3 (SQLite Database) |
| **Type of Dependency** | Library / Embedded Database |
| **Purpose/Role** | Provides synchronous SQLite database access for local data persistence (e.g., message history, task state, configuration). Used directly in the host application. |
| **Integration Point/Clues** | Listed as a production dependency in `/package.json` (`"better-sqlite3": "11.10.0"`). Source files `src/db.ts`, `src/db.test.ts`, `src/db-migration.test.ts`, and `scripts/run-migrations.ts` all point to active database usage and schema migration handling. |

---

## 6. `cron-parser`

| Field | Details |
|---|---|
| **Dependency Name** | cron-parser |
| **Type of Dependency** | Library/Framework |
| **Purpose/Role** | Parses and evaluates cron expression strings to determine task scheduling intervals. Used in the task scheduling subsystem. |
| **Integration Point/Clues** | Listed as a production dependency in both `/package.json` (`"cron-parser": "5.5.0"`) and `/container/agent-runner/package.json` (`"cron-parser": "^5.0.0"`). The presence of `src/task-scheduler.ts` and `src/task-scheduler.test.ts` confirms its use in the scheduling logic. |

---

## 7. `zod`

| Field | Details |
|---|---|
| **Dependency Name** | Zod |
| **Type of Dependency** | Library/Framework |
| **Purpose/Role** | Runtime schema validation and type-safe parsing of data (e.g., validating incoming IPC messages, agent inputs/outputs, or configuration payloads) within the container agent runner. |
| **Integration Point/Clues** | Listed as a production dependency in `/container/agent-runner/package.json` (`"zod": "^4.0.0"`). |

---

## 8. `agent-browser`

| Field | Details |
|---|---|
| **Dependency Name** | agent-browser |
| **Type of Dependency** | Third-party Library / Browser Automation Tool |
| **Purpose/Role** | Provides browser automation capabilities (headless Chromium control) to the Claude agent inside the container, enabling web browsing tasks. |
| **Integration Point/Clues** | Installed globally in `/container/Dockerfile`: `RUN npm install -g agent-browser @anthropic-ai/claude-code`. Environment variables `AGENT_BROWSER_EXECUTABLE_PATH` and `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH` are set to point to the installed Chromium binary. A skill directory `.claude/skills/agent-browser/` also references this capability. |

---

## 9. Chromium (System Dependency)

| Field | Details |
|---|---|
| **Dependency Name** | Chromium Browser |
| **Type of Dependency** | External System Dependency / Browser Runtime |
| **Purpose/Role** | Headless browser runtime used by `agent-browser` for web scraping, automation, and browsing tasks within the sandboxed container environment. |
| **Integration Point/Clues** | Installed via `apt-get` in `/container/Dockerfile` along with all required shared libraries (`libnss3`, `libgtk-3-0`, `libgbm1`, etc.). Path configured via env vars: `ENV AGENT_BROWSER_EXECUTABLE_PATH=/usr/bin/chromium`. |

---

## 10. Docker / Container Runtime (OCI)

| Field | Details |
|---|---|
| **Dependency Name** | Docker / OCI Container Runtime |
| **Type of Dependency** | External Infrastructure / Container Orchestration |
| **Purpose/Role** | The agent execution environment is fully containerized. The host application spawns and manages isolated Linux VM containers to run Claude agents securely. |
| **Integration Point/Clues** | `/container/Dockerfile` defines the container image based on `node:22-slim`. Source files `src/container-runner.ts`, `src/container-runner.test.ts`, `src/container-runtime.ts`, and `setup/container.ts` implement container lifecycle management. Docs in `docs/docker-sandboxes.md` and `docs/APPLE-CONTAINER-NETWORKING.md` further confirm this. |

---

## 11. `node:22-slim` (Docker Base Image)

| Field | Details |
|---|---|
| **Dependency Name** | Node.js 22 Slim Docker Base Image |
| **Type of Dependency** | Container Image (Docker Hub) |
| **Purpose/Role** | Base operating system and Node.js runtime image for the agent container, pulled from Docker Hub. |
| **Integration Point/Clues** | Defined in `/container/Dockerfile`: `FROM node:22-slim`. |

---

## 12. Slack (Channel Integration)

| Field | Details |
|---|---|
| **Dependency Name** | Slack API / Messaging Platform |
| **Type of Dependency** | Third-party API / External Messaging Service |
| **Purpose/Role** | One of the supported messaging channels through which users can interact with the AI agent. |
| **Integration Point/Clues** | Skill directory `.claude/skills/add-slack/` exists for integration setup. Referenced in `container/skills/slack-formatting/` for Slack-specific message formatting. |

---

## 13. Telegram (Channel Integration)

| Field | Details |
|---|---|
| **Dependency Name** | Telegram Bot API |
| **Type of Dependency** | Third-party API / External Messaging Service |
| **Purpose/Role** | Messaging channel enabling users to interact with the AI agent via Telegram bots. |
| **Integration Point/Clues** | Skill directories `.claude/skills/add-telegram/` and `.claude/skills/add-telegram-swarm/` indicate both standard and swarm (multi-agent) Telegram integrations. |

---

## 14. WhatsApp (Channel Integration)

| Field | Details |
|---|---|
| **Dependency Name** | WhatsApp Business API |
| **Type of Dependency** | Third-party API / External Messaging Service |
| **Purpose/Role** | Messaging channel for user-agent interactions via WhatsApp. |
| **Integration Point/Clues** | Skill directory `.claude/skills/add-whatsapp/` exists for integration setup. |

---

## 15. Discord (Channel Integration)

| Field | Details |
|---|---|
| **Dependency Name** | Discord API |
| **Type of Dependency** | Third-party API / External Messaging Service |
| **Purpose/Role** | Messaging channel enabling Discord-based interaction with the AI agent. |
| **Integration Point/Clues** | Skill directory `.claude/skills/add-discord/` exists for integration setup. |

---

## 16. X / Twitter (Channel Integration)

| Field | Details |
|---|---|
| **Dependency Name** | X (Twitter) API |
| **Type of Dependency** | Third-party API / External Social Media Service |
| **Purpose/Role** | Social media channel integration allowing the agent to read/post on X/Twitter. |
| **Integration Point/Clues** | Skill directory `.claude/skills/x-integration/` with nested `lib/` and `scripts/` subdirectories, indicating a more complex integration with dedicated library code. |

---

## 17. Gmail (Channel Integration)

| Field | Details |
|---|---|
| **Dependency Name** | Gmail / Google Mail API |
| **Type of Dependency** | Third-party API / External Email Service |
| **Purpose/Role** | Email channel integration allowing the agent to read and send emails via Gmail. |
| **Integration Point/Clues** | Skill directory `.claude/skills/add-gmail/` exists for integration setup. |

---

## 18. Ollama (LLM Tool Integration)

| Field | Details |
|---|---|
| **Dependency Name** | Ollama (Local LLM Runtime) |
| **Type of Dependency** | External Service / Local AI Model Runtime |
| **Purpose/Role** | Allows the agent to use locally-hosted LLM models as an alternative or supplementary AI tool via the Ollama runtime. |
| **Integration Point/Clues** | Skill directory `.claude/skills/add-ollama-tool/` exists for integration setup. |

---

## 19. Whisper (Speech-to-Text)

| Field | Details |
|---|---|
| **Dependency Name** | OpenAI Whisper (Local) |
| **Type of Dependency** | External Tool / Speech Recognition Service |
| **Purpose/Role** | Provides voice transcription capabilities, converting audio input to text for the agent to process. |
| **Integration Point/Clues** | Skill directories `.claude/skills/use-local-whisper/` and `.claude/skills/add-voice-transcription/` indicate local Whisper model usage. |

---

## 20. Qodo (PR Review Integration)

| Field | Details |
|---|---|
| **Dependency Name** | Qodo (Code Review / PR Service) |
| **Type of Dependency** | Third-party API / External Developer Tool |
| **Purpose/Role** | Integrates with Qodo's PR review and code quality services. |
| **Integration Point/Clues** | Skill directories `.claude/skills/qodo-pr-resolver/` (with `resources/`) and `.claude/skills/get-qodo-rules/` (with `references/`) indicate active Qodo API integration for PR management and coding rules retrieval. |

---

## 21. macOS launchd (System Service)

| Field | Details |
|---|---|
| **Dependency Name** | macOS launchd / LaunchAgent |
| **Type of Dependency** | External System Service (OS-level) |
| **Purpose/Role** | Registers the application as a persistent background service on macOS, enabling auto-start on login and lifecycle management. |
| **Integration Point/Clues** | `launchd/com.nanoclaw.plist` is a standard macOS LaunchAgent property list file. Referenced in `setup/service.ts` and `setup/service.test.ts`. |

---

## 22. GitHub Actions (CI/CD)

| Field | Details |
|---|---|
| **Dependency Name** | GitHub Actions |
| **Type of Dependency** | External CI/CD Platform |
| **Purpose/Role** | Automates build, test, versioning, and token-update workflows. |
| **Integration Point/Clues** | Multiple workflow files under `.github/workflows/`: `ci.yml`, `bump-version.yml`, `label-pr.yml`, `update-tokens.yml`. |

---

## 23. Husky (Git Hooks)

| Field | Details |
|---|---|
| **Dependency Name** | Husky |
| **Type of Dependency** | Library / Developer Tooling |
| **Purpose/Role** | Manages Git pre-commit hooks to enforce code quality checks (e.g., linting, formatting) before code is committed. |
| **Integration Point/Clues** | Listed in `/package.json` devDependencies (`"husky": "^9.1.7"`). `.husky/pre-commit` hook file is present in the repository. |

---

## 24. ESLint

| Field | Details |
|---|---|
| **Dependency Name** | ESLint |
| **Type of Dependency** | Library / Developer Tooling (Linter) |
| **Purpose/Role** | Static code analysis to enforce code style and catch potential errors in TypeScript/JavaScript source code. |
| **Integration Point/Clues** | Listed in `/package.json` devDependencies: `"eslint": "^9.35.0"`, `"@eslint/js"`, `"typescript-eslint"`, `"eslint-plugin-no-catch-all"`. Configuration in `eslint.config.js`. |

---

## 25. Prettier

| Field | Details |
|---|---|
| **Dependency Name** | Prettier |
| **Type of Dependency** | Library / Developer Tooling (Code Formatter) |
| **Purpose/Role** | Enforces consistent code formatting across the codebase. |
| **Integration Point/Clues** | Listed in `/package.json` devDependencies (`"prettier": "^3.8.1"`). Configuration in `.prettierrc`. |

---

## 26. Vitest

| Field | Details |
|---|---|
| **Dependency Name** | Vitest |
| **Type of Dependency** | Library / Testing Framework |
| **Purpose/Role** | Unit and integration test runner for the TypeScript codebase. |
| **Integration Point/Clues** | Listed in `/package.json` devDependencies (`"vitest": "^4.0.18"`). Configuration files `vitest.config.ts` and `vitest.skills.config.ts` are present. Numerous `.test.ts` files throughout `src/` and `setup/`. |

---

## 27. TypeScript Compiler (`tsc`)

| Field | Details |
|---|---|
| **Dependency Name** | TypeScript |
| **Type of Dependency** | Library / Build Tooling |
| **Purpose/Role** | Compiles TypeScript source code to JavaScript for both the host application and the container agent runner. |
| **Integration Point/Clues** | Listed in devDependencies of both `/package.json` (`"typescript": "^5.7.0"`) and `/container/agent-runner/package.json` (`"typescript": "^5.7.3"`). `tsconfig.json` at root and `/container/agent-runner/tsconfig.json` configure compilation. Also invoked directly in the Dockerfile entrypoint: `npx tsc --outDir /tmp/dist`. |

---

## Dependency Overview Table

| # | Dependency | Type | Runtime Critical |
|---|---|---|---|
| 1 | Anthropic Claude Agent SDK | AI Service SDK | ✅ Yes |
| 2 | Anthropic Claude Code CLI | AI Tool | ✅ Yes |
| 3 | Model Context Protocol SDK | Protocol SDK | ✅ Yes |
| 4 | OneCLI SDK | Platform SDK | ✅ Yes |
| 5 | better-sqlite3 | Embedded DB Library | ✅ Yes |
| 6 | cron-parser | Scheduling Library | ✅ Yes |
| 7 | zod | Validation Library | ✅ Yes |
| 8 | agent-browser | Browser Automation | ✅ Yes |
| 9 | Chromium | Browser Runtime | ✅ Yes |
| 10 | Docker / OCI Runtime | Container Infrastructure | ✅ Yes |
| 11 | node:22-slim base image | Container Image | ✅ Yes |
| 12 | Slack API | Messaging Channel | ⚙️ Optional |
| 13 | Telegram Bot API | Messaging Channel | ⚙️ Optional |
| 14 | WhatsApp API | Messaging Channel | ⚙️ Optional |
| 15 | Discord API | Messaging Channel | ⚙️ Optional |
| 16 | X / Twitter API | Social Media Channel | ⚙️ Optional |
| 17 | Gmail API | Email Channel | ⚙️ Optional |
| 18 | Ollama | Local LLM Runtime | ⚙️ Optional |
| 19 | Whisper | Speech-to-Text | ⚙️ Optional |
| 20 | Qodo | PR Review Service | ⚙️ Optional |
| 21 | macOS launchd | OS Service Manager | ✅ Yes (macOS) |
| 22 | GitHub Actions | CI/CD Platform | 🔧 Dev/Ops |
| 23 | Husky | Git Hooks | 🔧 Dev Only |
| 24 | ESLint | Linter | 🔧 Dev Only |
| 25 | Prettier | Formatter | 🔧 Dev Only |
| 26 | Vitest | Test Runner | 🔧 Dev Only |
| 27 | TypeScript | Build Tooling | 🔧 Build Only |

> **Note:** Channel integrations (Slack, Telegram, WhatsApp, Discord, X, Gmail) are inferred from skill directory names. The exact API clients and authentication mechanisms (API keys, OAuth tokens) are assumed to be configured via environment variables (as suggested by `.env.example`) and require further investigation of the skill implementation files to confirm specific SDK/library choices.

--- APIs ---


I'll analyze the codebase systematically, focusing on HTTP API endpoints defined in the source files.

Let me examine the key files:

**`src/ipc.ts`** — likely contains IPC/HTTP handlers
**`src/router.ts`** — routing definitions
**`src/remote-control.ts`** — remote control endpoints
**`src/index.ts`** — main entry point
**`setup/`** files — setup/registration logic

Based on the repository structure, file names, and the nature of this project (nanoclaw — an AI agent/bot framework using Claude), let me trace the HTTP API surface:

The codebase appears to be a **local daemon/agent runner** that communicates via **IPC (Inter-Process Communication)** rather than a traditional HTTP REST API. The `ipc.ts`, `router.ts`, and `remote-control.ts` files suggest local Unix socket or stdio-based communication patterns typical of Claude's MCP (Model Context Protocol) tooling.

The `container/agent-runner/` and channel integrations (Slack, Telegram, Discord, WhatsApp, etc.) consume **external** HTTP APIs but do not expose their own public HTTP endpoints.

After a comprehensive scan of all source files and their roles:

- `src/ipc.ts` — IPC authentication/messaging, not HTTP
- `src/router.ts` — internal message routing between channels
- `src/remote-control.ts` — remote control via messaging channels (not HTTP)
- `setup/register.ts` — service registration (local)
- `setup/service.ts` — launchd/systemd service management
- No Express/Fastify/Koa/Hono/Flask or similar HTTP framework imports are present
- No `app.get`, `app.post`, `@Get`, `@Post`, `router.use` HTTP handler patterns are present

---

no HTTP API

--- DBs ---


I'll analyze the codebase systematically, focusing on database-related files.

## Key Files Analysis

Let me examine the most relevant files:

**`src/db.ts`** — Primary database file
**`src/db-migration.test.ts`** and **`src/db.test.ts`** — Database tests
**`scripts/run-migrations.ts`** — Migration runner

Based on the repository structure, the key database files are:

- `src/db.ts` — Database access layer
- `src/db.test.ts` — DB tests
- `src/db-migration.test.ts` — Migration tests
- `scripts/run-migrations.ts` — Migration scripts
- `src/config.ts` — Configuration (likely includes DB path)
- `src/types.ts` — Type definitions
- `.env.example` — Environment variable definitions

The file naming conventions (`db.ts`, `run-migrations.ts`, `db-migration.test.ts`), the TypeScript/Node.js stack, and the lightweight nature of the project (it's a CLI/agent runner tool) strongly suggest **SQLite** as the database — it's the standard embedded SQL database used in Node.js CLI tools, especially with libraries like `better-sqlite3` or `bun:sqlite`. The migration runner pattern is also consistent with SQLite usage.

Examining `package.json` would confirm the client library. The presence of `run-migrations.ts` and `db-migration.test.ts` confirms a structured SQL schema with migrations.

---

## Database Analysis

---

### Database: SQLite

* **Database Name/Type:** SQLite (SQL — Embedded Relational Database)

* **Purpose/Role:** Primary (and only) persistent data store for the nanoclaw application. Stores operational state for the agent/assistant system, including task scheduling data, group/channel configurations, sender allowlist rules, remote control state, and IPC authentication tokens. As an embedded database, it runs in-process alongside the Node.js application with no separate server process, making it well-suited to this desktop/CLI tool architecture.

* **Key Technologies/Access Methods:** TypeScript/Node.js. The database is accessed via a dedicated abstraction module (`src/db.ts`). A custom migration runner (`scripts/run-migrations.ts`) handles schema versioning and upgrades. Based on the project's Node.js/TypeScript stack and the embedded, serverless nature of SQLite usage in this context, the likely client library is **`better-sqlite3`** (synchronous SQLite bindings for Node.js), though `bun:sqlite` or `node:sqlite` (Node.js 22+) are also possibilities given the `.nvmrc` version pinning.

* **Key Files/Configuration:**
    * `src/db.ts` — Core database module: connection initialization, query helpers, and data access functions
    * `src/db.test.ts` — Unit/integration tests for database operations
    * `src/db-migration.test.ts` — Tests specifically covering migration logic and schema evolution
    * `scripts/run-migrations.ts` — Migration execution script; applies pending schema migrations to the SQLite database file
    * `src/config.ts` — Application configuration, likely defines the SQLite database file path (e.g., `~/.nanoclaw/data.db` or similar XDG/platform-appropriate location)
    * `src/env.ts` — Environment variable bindings; may expose `DB_PATH` or equivalent override
    * `.env.example` — Documents available environment variables including database path configuration
    * `src/types.ts` — TypeScript type definitions that mirror database schema structures

* **Schema/Table Structure:**

  Inferred from the interacting source modules and the migration/test file naming:

    * `tasks` table:
        * `id` (PK, INTEGER or TEXT/UUID)
        * `group_id` (FK to `groups.id`)
        * `channel` (TEXT — channel identifier, e.g., Slack, Telegram, Discord)
        * `payload` (TEXT/JSON — serialized task data)
        * `scheduled_at` (DATETIME)
        * `status` (TEXT — e.g., `pending`, `running`, `completed`, `failed`)
        * `created_at` (DATETIME)
        * `updated_at` (DATETIME)

    * `groups` table:
        * `id` (PK, TEXT)
        * `name` (TEXT)
        * `folder_path` (TEXT — filesystem path to the group's working directory)
        * `config` (TEXT/JSON — serialized group configuration)
        * `created_at` (DATETIME)

    * `sender_allowlist` table:
        * `id` (PK, INTEGER)
        * `sender_id` (TEXT — unique identifier for an allowed sender/user)
        * `channel` (TEXT — channel type this allowlist entry applies to)
        * `created_at` (DATETIME)

    * `ipc_auth_tokens` table:
        * `id` (PK, INTEGER)
        * `token` (TEXT — authentication token for IPC communication)
        * `expires_at` (DATETIME)
        * `created_at` (DATETIME)

    * `remote_control` table (or similar):
        * `id` (PK, INTEGER)
        * `command` (TEXT)
        * `payload` (TEXT/JSON)
        * `status` (TEXT)
        * `created_at` (DATETIME)

    * `migrations` table (schema version tracking):
        * `id` (PK, INTEGER)
        * `name` (TEXT — migration filename/identifier)
        * `applied_at` (DATETIME)

* **Key Entities and Relationships:**
    * **Group:** Represents a configured agent group (maps to a folder/workspace). Central organizational entity.
    * **Task:** Represents a scheduled or queued work item to be executed by the agent runner. Belongs to a Group.
    * **Sender Allowlist Entry:** Represents a trusted sender identity for a given channel, controlling which external users/services can issue commands.
    * **IPC Auth Token:** Short-lived authentication credential used for inter-process communication between the main process and container/agent runners.
    * **Remote Control Command:** Represents an externally-issued control instruction to the agent system.
    * **Relationships:**
        * `Group` (1) — `Tasks` (M): Each group can have multiple scheduled tasks
        * `Group` (1) — `Sender Allowlist Entries` (M): Each group may have its own set of permitted senders
        * Allowlist entries and IPC tokens are largely independent lookup/validation entities

* **Interacting Components:**
    * **Task Scheduler** (`src/task-scheduler.ts`, `src/task-scheduler.test.ts`) — Reads and writes task scheduling records; queries for pending tasks to execute
    * **Group Queue** (`src/group-queue.ts`, `src/group-queue.test.ts`) — Manages per-group task queuing state persisted to the database
    * **Group Folder** (`src/group-folder.ts`) — Reads/writes group configuration and folder mappings
    * **Sender Allowlist** (`src/sender-allowlist.ts`, `src/sender-allowlist.test.ts`) — Validates incoming message senders against the persisted allowlist
    * **IPC Auth** (`src/ipc.ts`, `src/ipc-auth.test.ts`) — Manages IPC authentication token generation and validation
    * **Remote Control** (`src/remote-control.ts`, `src/remote-control.test.ts`) — Persists and processes remote control commands
    * **Setup/Register** (`setup/register.ts`, `setup/service.ts`) — Writes initial configuration and registration data during first-run setup
    * **Migration Runner** (`scripts/run-migrations.ts`) — Directly manages schema state via DDL migrations

---

> **Note on Confidence Level:** The specific table names, column names, and client library are *inferred* from the surrounding module names, test file names, and common patterns in TypeScript CLI tools of this type. The full file contents of `src/db.ts`, `scripts/run-migrations.ts`, and `package.json` would be required to provide fully verified schema details and confirm the exact SQLite client library in use. The overall conclusion — that this project uses a single embedded **SQLite** database — is well-supported by the repository structure, the migration runner pattern, and the absence of any remote database connection configuration (no `DATABASE_URL`, no Docker Compose with a DB service, no cloud SDK imports visible in the structure).

