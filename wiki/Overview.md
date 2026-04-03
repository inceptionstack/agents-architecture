# Overview

This repository contains automated architecture documentation for **13 AI agent frameworks**. Each was analyzed by examining repository structure, dependency manifests, directory organization, and inline documentation. The output is a set of `.arch.md` files (one per agent) plus this cross-agent wiki.

For deeper context, see [Architecture Patterns](Architecture-Patterns.md), [Security Models](Security-Models.md), and [Unique Innovations](Unique-Innovations.md).

---

## The 13 Frameworks

| Agent | Language | Architecture Pattern | One-Line Description | Source |
|-------|----------|---------------------|----------------------|--------|
| **CoPaw** | Python (FastAPI) | Hybrid Multi-Tier + Plugin | AI agent platform built on AgentScope runtime with Alibaba Cloud ecosystem integration | [CoPaw.arch.md](../CoPaw.arch.md) |
| **Hermes Agent** | Python (FastAPI) | ReAct Loop + Event-Driven Gateway | Multi-model autonomous agent with 40+ tools, 12+ per-model tool-call parsers, and extensive security testing | [hermes-agent.arch.md](../hermes-agent.arch.md) |
| **IronClaw** | Rust (Tokio/Axum) | Plugin (WASM) + Worker/Orchestrator | Rust agent platform with WASM-sandboxed tool plugins via WIT interfaces and a dedicated safety crate | [ironclaw.arch.md](../ironclaw.arch.md) |
| **Letta** | Python (FastAPI) | Layered SOA + Service Manager | Stateful agent framework with three-tier persistent memory (in-context, conversational, archival/pgvector) | [letta.arch.md](../letta.arch.md) |
| **MicroClaw** | Rust (workspace) | Modular Monolith + Plugin + Event-Driven | Rust agent runtime with AOP-style hook system, ClawHub marketplace, A2A + ACP multi-agent protocols | [microclaw.arch.md](../microclaw.arch.md) |
| **Moltis** | Rust (50+ crates) | Modular Monolith / Microkernel + Event-Driven | 50-crate Rust agent with native iOS/macOS apps via Rust↔Swift FFI, GraphQL API, and CalDAV integration | [moltis.arch.md](../moltis.arch.md) |
| **NanoClaw** | TypeScript (Node.js) | Event-Driven Multi-Channel Gateway | Container-per-task Claude agent with Apple Container support, launchd integration, and IPC auth | [nanoclaw.arch.md](../nanoclaw.arch.md) |
| **Nanobot** | Go | Layered + Plugin via MCP | MCP-native Go agent where MCP is the core design primitive, including agent-as-MCP-server composition | [nanobot.arch.md](../nanobot.arch.md) |
| **NemoClaw** | JavaScript/Node.js + TypeScript | CLI + Plugin + Policy Engine + Container Runtime | Developer sandbox tool with dual-layer network enforcement (DNS proxy + gateway) and 9 security policy presets | [NemoClaw.arch.md](../NemoClaw.arch.md) |
| **OpenClaw** | TypeScript (Node.js) | Plugin-based Event-Driven Gateway / Microkernel | Self-hosted AI assistant platform with 15+ messaging channels, skills/plugin system, sub-agent orchestration, cron scheduling, and persistent memory via workspace files | [openclaw.arch.md](../openclaw.arch.md) |
| **OpenFang** | Rust (workspace) | Layered Modular Monolith + Event-Driven | Rust agent platform with "Hands" (autonomous actions), 30+ pre-built agents, Tauri desktop, and Python/JS SDKs | [openfang.arch.md](../openfang.arch.md) |
| **PicoClaw** | Go | Plugin/Registry + Event-Driven Gateway | Go agent framework with hardware device integration (I2C/SPI), ASR/TTS audio pipeline, BM25 memory search | [picoclaw.arch.md](../picoclaw.arch.md) |
| **ZeptoClaw** | Rust | Plugin-Based Modular Monolith + Event-Driven | Rust agent with 6 sandbox runtimes, safety taint tracking, Android tools, MQTT/serial channels, agent budget enforcement | [zeptoclaw.arch.md](../zeptoclaw.arch.md) |

---

## Language Distribution

| Language | Frameworks | % |
|----------|-----------|---|
| Rust | IronClaw, MicroClaw, Moltis, OpenFang, ZeptoClaw | 38% |
| Python | CoPaw, Hermes Agent, Letta | 23% |
| Go | Nanobot, PicoClaw | 15% |
| TypeScript/JS | NanoClaw, NemoClaw, OpenClaw | 23% |

See [Language Ecosystem](Language-Ecosystem.md) for analysis of why Rust dominates agent infrastructure.

---

## Common Capabilities (All 13)

- Multi-channel messaging (Telegram, Slack, Discord minimum)
- Docker containerization
- CLI entry point
- Tool-calling / function-calling agent loop
- Session/conversation persistence
- Multi-provider LLM abstraction

## Near-Universal Capabilities (11–12/13)

- MCP (Model Context Protocol) client or server — 12/13
- Approval/confirmation gates for sensitive operations — 11/13
- Cron/scheduled task execution — 11/13
- Sub-agent/delegation — 10/13

---

*Back to [Home](Home.md) | Next: [Architecture Patterns](Architecture-Patterns.md)*
