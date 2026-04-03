# Best-of-Breed Agent Design

What would the ideal AI agent look like if we combined the best architectural ideas from all 12 frameworks? This page synthesizes the strongest patterns into a hypothetical "best of breed" blueprint.

→ See also: [Architecture Patterns](Architecture-Patterns.md) · [Security Models](Security-Models.md) · [Unique Innovations](Unique-Innovations.md)

---

## Core Runtime

| Component | Source | Why |
|-----------|--------|-----|
| **Language: Rust workspace** | IronClaw, MicroClaw, ZeptoClaw | Memory safety, strong typing, fearless concurrency — 42% of agents chose Rust for good reason |
| **Agent loop: ReAct pattern** | Hermes | Production-proven with per-model tool call parsers supporting 12+ model formats |
| **Memory architecture** | Letta | Three-tier memory (in-context Blocks, conversational, archival with pgvector) + automatic summarization |

## Extension System

| Component | Source | Why |
|-----------|--------|-----|
| **WASM plugin sandbox** | IronClaw, Moltis | Memory-isolated, capability-based, language-agnostic plugins via WIT interfaces |
| **Hook pipeline** | MicroClaw | AOP-style interception — security policies as composable filesystem scripts |
| **MCP-native tool protocol** | Nanobot | MCP as the standard protocol for all tool extensions |
| **Skills marketplace** | MicroClaw (ClawHub) | Centralized skill discovery, review, integrity verification, and installation |

## Security Stack

This is where the most interesting cross-pollination happens. See [Security Models](Security-Models.md) for the full deep-dive.

| Layer | Source | Mechanism |
|-------|--------|-----------|
| **Sandbox runtimes (6 options)** | ZeptoClaw | Docker, Bubblewrap, Firejail, Landlock (Linux LSM), Apple Sandbox, native — operator chooses the tradeoff |
| **Network policy engine** | NemoClaw | Dual-layer DNS proxy + gateway enforcement with preset library |
| **Taint tracking** | ZeptoClaw | Data provenance traced through the entire pipeline |
| **Agent mode controls** | ZeptoClaw | Safe / Standard / Unrestricted with explicit escalation |
| **Encrypted vault** | Moltis | ChaCha20-Poly1305 encrypted secrets + OS keychain integration |
| **Security test suite** | Hermes | 18+ categories — SQL injection, SSRF, prompt injection, symlink, path traversal |
| **WASM isolation** | IronClaw | Plugin isolation without container overhead |

## LLM Integration

| Component | Source | Why |
|-----------|--------|-----|
| **Multi-provider routing** | Letta (via LiteLLM) | Broadest provider coverage (20+) |
| **Smart model selection** | Hermes | Automatic model routing based on task type |
| **Credential pool + rotation** | Hermes | Multi-key load balancing across API keys |
| **Provider resilience** | ZeptoClaw | Rotation, fallback, retry, cooldown, quota management |
| **Context compaction** | Letta | Automatic summarization when context window fills |
| **Token budget enforcement** | ZeptoClaw | Per-session and per-agent cost limits |

See [LLM Integration](LLM-Integration.md) for detailed comparison.

## Multi-Agent

| Component | Source | Why |
|-----------|--------|-----|
| **Orchestration patterns** | Letta | Round-robin, supervisor, sleeptime consolidation |
| **A2A + ACP protocols** | MicroClaw | HTTP and stdio inter-agent communication |
| **Agent-as-MCP-server** | Nanobot | Recursive agent composition — agents callable as tools |
| **Delegation tool** | Hermes | Spawn sub-agents with full context passing |

## Communication Channels

| Component | Source | Why |
|-----------|--------|-----|
| **Channel abstraction** | MicroClaw, PicoClaw | Registry pattern supporting 15+ platforms |
| **Event bus** | PicoClaw | Decoupled async communication between components |
| **GraphQL API** | Moltis | Flexible query language for client apps |
| **Native mobile** | Moltis | Rust→Swift FFI bridge for iOS/macOS |

See [Multi-Channel](Multi-Channel.md) for platform coverage comparison.

## Operations

| Component | Source | Why |
|-----------|--------|-----|
| **6 deployment targets** | Moltis | Docker, Fly.io, Railway, Render, DigitalOcean, Nix |
| **Config hot-reload** | Nanobot | fswatch-based zero-downtime configuration changes |
| **OpenTelemetry** | Letta, ZeptoClaw | Distributed tracing and metrics |
| **LLM trace replay** | IronClaw | Deterministic testing without live LLM calls |
| **Coverage ratchet** | NemoClaw | CI prevents test coverage from decreasing |

## Novel Features Worth Including

| Feature | Source | Value |
|---------|--------|-------|
| **Hardware I2C/SPI/MQTT** | PicoClaw, ZeptoClaw | IoT agent integration |
| **Audio pipeline (ASR/TTS)** | PicoClaw | Voice-first agent interaction |
| **Container-per-task** | NanoClaw | Maximum isolation for untrusted workloads |
| **Tauri desktop app** | OpenFang | Native desktop experience |
| **WebAuthn passkeys** | Moltis | Passwordless authentication |

---

## Architecture Diagram

```
┌──────────────────────────────────────────────────────────────┐
│  Client Layer                                                 │
│  ┌──────────┐ ┌─────────┐ ┌──────┐ ┌───────┐ ┌──────────┐ │
│  │ Telegram │ │ Discord │ │ Slack│ │ WebUI │ │ iOS/macOS│ │
│  └────┬─────┘ └────┬────┘ └──┬───┘ └───┬───┘ └────┬─────┘ │
├───────┴────────────┴─────────┴─────────┴──────────┴─────────┤
│  Gateway Layer (Event Bus + Channel Registry)                 │
├─────────────────────────────────────────────────────────────-─┤
│  Agent Core (Rust)                                            │
│  ┌──────────┐ ┌──────────────┐ ┌─────────────────────────┐  │
│  │ ReAct    │ │ Hook Pipeline│ │ Multi-Agent Orchestrator │  │
│  │ Loop     │ │ (AOP)        │ │ (A2A + MCP)             │  │
│  └────┬─────┘ └──────────────┘ └─────────────────────────┘  │
├───────┴──────────────────────────────────────────────────────┤
│  Tool Execution Layer                                         │
│  ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌──────────────┐  │
│  │ WASM     │ │ Docker   │ │ Landlock  │ │ MCP Servers  │  │
│  │ Plugins  │ │ Sandbox  │ │ (LSM)     │ │ (stdio/HTTP) │  │
│  └──────────┘ └──────────┘ └───────────┘ └──────────────┘  │
├──────────────────────────────────────────────────────────────┤
│  Memory Layer (Letta-style)                                   │
│  ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌──────────────┐  │
│  │ Blocks   │ │ Sessions │ │ Archival  │ │ Sleeptime    │  │
│  │ (context)│ │ (SQLite) │ │ (pgvector)│ │ Consolidator │  │
│  └──────────┘ └──────────┘ └───────────┘ └──────────────┘  │
├──────────────────────────────────────────────────────────────┤
│  Security Layer                                               │
│  ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌──────────────┐  │
│  │ Taint    │ │ Network  │ │ Encrypted │ │ Agent Mode   │  │
│  │ Tracking │ │ Policy   │ │ Vault     │ │ Controls     │  │
│  └──────────┘ └──────────┘ └───────────┘ └──────────────┘  │
├──────────────────────────────────────────────────────────────┤
│  LLM Providers (LiteLLM + Provider Rotation + Budget)        │
│  Anthropic · OpenAI · Bedrock · Gemini · Local · 15+ more   │
└──────────────────────────────────────────────────────────────┘
```

---

## Key Design Principles

1. **Defense in depth** — Security at every layer (WASM + sandbox + network + taint + vault), not just one
2. **Operator choice** — Don't hardcode one sandbox; let operators choose the isolation level
3. **Protocol-first** — MCP for tools, A2A for agents, GraphQL for clients
4. **Memory as first-class** — Not an afterthought; structured, versioned, searchable, with background consolidation
5. **Composable policies** — Hook scripts, not hardcoded rules; operators can add/remove security policies

---

*This design doesn't exist as a single project today. It represents the state of the art as of April 2026, synthesized from 12 production agent frameworks.*

→ Back to [Home](Home.md)
