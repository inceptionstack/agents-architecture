# Architecture Patterns

This page catalogues the dominant architectural patterns found across the 13 agent frameworks. Most agents combine multiple patterns; the primary pattern drives the overall code organization while secondary patterns handle specific concerns.

See also: [Tool Systems](Tool-Systems.md), [Multi-Channel](Multi-Channel.md), [Language Ecosystem](Language-Ecosystem.md).

---

## Pattern Taxonomy

### 1. Plugin / Skill Architecture
**Agents:** IronClaw, MicroClaw, Moltis, NanoClaw, Nanobot, OpenFang, PicoClaw, ZeptoClaw, CoPaw, Hermes Agent

The most universal pattern — every framework has an extensible capability system. The implementation diverges significantly:

| Agent | Plugin Unit | Mechanism | Isolation |
|-------|-------------|-----------|-----------|
| IronClaw | WASM module | WIT interface types | Memory-isolated |
| Moltis | WASM module | WIT + Wasmtime | Memory-isolated |
| MicroClaw | Skill directory | Script hooks | Process-level |
| NanoClaw | Skill directory (`.claude/skills/`) | Claude AI reads SKILL.md | Container-isolated |
| Nanobot | MCP server | MCP protocol | Process-level |
| OpenFang | Skill + Hand + Extension | Driver dispatch | Process-level |
| PicoClaw | Skill registry | Go interface | In-process |
| ZeptoClaw | Skill package | Dynamic loader | Configurable |
| CoPaw | Tool function | Python function-calling | In-process |
| Hermes Agent | Skill directory + tool registry | Python function-calling | In-process |

**WASM is the most secure plugin isolation.** IronClaw and Moltis compile plugins to WebAssembly with formally-defined WIT interfaces — each plugin gets its own linear memory space and can only access host functions explicitly declared in the interface. No container overhead; microsecond startup. See [Security Models](Security-Models.md) for ranking.

**MCP-as-plugin** (Nanobot) is architecturally elegant: the same protocol used for external tools is used for all plugins, including agents themselves. See [Tool Systems](Tool-Systems.md).

---

### 2. Event-Driven / Message Bus Architecture
**Agents:** MicroClaw, Moltis, OpenFang, PicoClaw, ZeptoClaw, NanoClaw, Nanobot

Most agents use an internal event bus for decoupled async communication between components.

| Agent | Bus Implementation | Notable Use |
|-------|-------------------|-------------|
| PicoClaw | `pkg/bus/` + per-agent `pkg/agent/eventbus.go` | App-wide pub/sub, per-agent sub-bus |
| ZeptoClaw | `src/bus/` (Criterion-benchmarked) | Channels → Agent → Tools pipeline |
| Moltis | Typed broadcast event enum | Compile-time checked event types |
| MicroClaw | Hooks system (`src/hooks.rs`) | AOP-style interception events |
| OpenFang | Event-driven routing in kernel | Agent routing + workflow events |

Moltis has the most type-safe approach — events are a Rust enum (`typed-broadcast-event-enum`), so the compiler catches missing match arms. Other frameworks use string-keyed maps or channels without type checking.

---

### 3. ReAct Agent Loop
**Agents:** Hermes Agent, IronClaw, Letta, MicroClaw, PicoClaw, ZeptoClaw (and implicitly most others)

The core reasoning pattern: **Reason → Act → Observe → Repeat**. The LLM produces a thought, calls a tool, receives the result, and continues until done.

```
User Input
    ↓
System Prompt + Context
    ↓
LLM Response
    ↓ (if tool call)
Tool Execution ──→ Safety/Approval Check
    ↓
Tool Result back to context
    ↓
Repeat until done
```

Key variation: how tool calls are parsed. Hermes Agent maintains **12+ per-model parsers** because each LLM has subtly different tool-call formatting. Others use a generic OpenAI-compatible function-calling format and only support models that conform to it. See [LLM Integration](LLM-Integration.md).

---

### 4. Worker/Orchestrator Pattern
**Primary:** IronClaw, Letta, Moltis

These agents separate the **orchestrator** (accepts work, distributes, monitors) from **workers** (execute individual tasks). IronClaw's `src/worker/api.rs` and `src/orchestrator/api.rs` are distinct subsystems connected by a job queue. Letta's `AgentManager` service drives workers, and Moltis uses typed broadcast events to coordinate across 50+ crates.

This pattern enables horizontal scaling — multiple worker processes can consume from the same queue.

---

### 5. Gateway Pattern
**Agents:** All 13 (universally)

Every agent implements a gateway that:
- Accepts messages from multiple channels (Telegram, Slack, etc.)
- Routes to the appropriate agent instance
- Handles authentication and rate limiting at the boundary
- Converts channel-specific message formats to an internal canonical form

See [Multi-Channel](Multi-Channel.md) for channel adapter architecture details.

---

### 6. Monolith vs. Microkernel

| Approach | Agents | Characteristic |
|----------|--------|---------------|
| **Modular Monolith** | MicroClaw, ZeptoClaw, OpenFang | Single process, Rust workspace, feature-gated modules |
| **Microkernel** | Nanobot, Moltis | Small core + pluggable extension servers |
| **SOA (Service-Oriented)** | Letta | Distinct service classes per domain (AgentManager, MessageManager, etc.) |
| **Layered** | CoPaw, Hermes Agent, PicoClaw | Clear tier boundaries (API → Service → Data) |
| **CLI + Container** | NemoClaw, NanoClaw | Thin host process, workloads in containers |

**Nanobot is the purest microkernel**: the core provides MCP hosting, and all capabilities (even built-in ones) are MCP servers. An agent-as-MCP-server is recursively composable.

**Moltis at 50+ Rust crates** is the most modular — each feature is an independent crate with its own `Cargo.toml`, enabling precise dependency management and compile-time feature gating. The downside is build complexity and increased attack surface (more dependency surface area).

---

### 7. Layered Security Architecture
**Agents:** ZeptoClaw (strongest), IronClaw, Hermes Agent, MicroClaw, NemoClaw

Security-aware frameworks insert explicit layers *within* the agent loop rather than treating security as a cross-cutting concern:

```
Input → Taint Tagging (ZeptoClaw) → Agent Loop → Safety Check → Tool Execution → Sandbox Runtime
                                                         ↑
                                                  Hook Interception (MicroClaw)
                                                  Policy Enforcement (NemoClaw)
                                                  WASM Boundary (IronClaw/Moltis)
```

See [Security Models](Security-Models.md) for the full ranking and analysis.

---

## Pattern Combinations by Agent

| Agent | Primary Pattern | Secondary Patterns |
|-------|----------------|-------------------|
| CoPaw | Multi-Tier + Plugin | Event-driven (channels), AgentScope runtime |
| Hermes Agent | ReAct + Event-Driven | Plugin/Skills, Gateway |
| IronClaw | WASM Plugin + Worker/Orchestrator | Layered security, SSE streaming |
| Letta | Layered SOA | Service manager, plugin, event-driven |
| MicroClaw | Modular Monolith + Plugin | Hook/AOP, event bus, A2A/ACP |
| Moltis | Microkernel + Event-Driven | WASM plugin, Rust↔Swift FFI, GraphQL |
| NanoClaw | Event-Driven Gateway | Container-per-task, Plugin/Skills |
| Nanobot | MCP Microkernel | Plugin, event-driven, hot-reload |
| NemoClaw | CLI + Policy Engine | Container runtime, DNS proxy |
| OpenFang | Layered Modular Monolith | Driver pattern, event-driven |
| PicoClaw | Plugin/Registry + Event-Driven | Gateway, bus, hardware device |
| ZeptoClaw | Modular Monolith + Event-Driven | Plugin, layered security, hardware |

---

*Back to [Home](Home.md) | Next: [Tool Systems](Tool-Systems.md)*
