# Tool Systems

Every agent framework must solve the same problem: how do you expose capabilities to an LLM, execute them safely, and feed results back? This page compares the tool system architectures across all 13 frameworks.

Related: [Architecture Patterns](Architecture-Patterns.md), [Security Models](Security-Models.md), [LLM Integration](LLM-Integration.md).

---

## What Is a Tool System?

A tool system has four components:
1. **Registry** — how tools are discovered and indexed
2. **Schema** — how tool signatures are communicated to the LLM
3. **Execution** — how tool code is run (and by whom)
4. **Result handling** — how output feeds back into the agent loop

---

## Tool Registry Patterns

| Agent | Registry Mechanism | Discovery |
|-------|-------------------|-----------|
| Hermes Agent | Central `registry.py` maps tool names → handler functions | Static at startup; skills add tools dynamically |
| IronClaw | WASM plugin loader; built-in tools compiled in | WASM modules scanned at startup |
| Letta | `ToolManager` service; `tool_sandbox/` for execution | DB-backed tool catalog; MCP client adds remote tools |
| MicroClaw | Tool trait in `microclaw-tools` crate; `src/tools/mod.rs` registry | Trait impls auto-discovered via Rust feature flags |
| Moltis | WASM plugin tools via Wasmtime; built-in `moltis-tools` crate | WASM manifests; Cargo feature flags |
| NanoClaw | Skills in `.claude/skills/` directories | Claude reads SKILL.md at startup |
| Nanobot | MCP servers as plugins; `pkg/servers/` for built-ins | MCP tool listing protocol (`tools/list`) |
| NemoClaw | OpenClaw plugin system; tools via Claude inside sandbox | Blueprint config; `.agents/skills/` guidance |
| OpenFang | Skills (60+), Hands, Extensions; driver dispatch | Static crate + manifest scanning |
| PicoClaw | `pkg/tools/registry.go`; 20+ built-ins; MCP tools | Registry pattern at startup; hot-reload partial |
| ZeptoClaw | Tool registry + `src/tools/` modules; MCP tools | Feature-gated at compile time + dynamic skills |
| CoPaw | `agents/tools/` directory; skill scanner pre-validates | Directory scan; AgentScope runtime wires them |

---

## Schema Communication to LLM

Most frameworks convert tool definitions to JSON Schema and pass them in the system prompt or as a separate `tools` parameter (OpenAI-compatible function-calling format).

Notable exceptions:
- **Hermes Agent** — Per-model schema serializers. DeepSeek, Qwen, Mistral, and others each have a custom parser/serializer because their tool-call formats differ from OpenAI's. See [LLM Integration](LLM-Integration.md).
- **Nanobot** — MCP's `tools/list` response IS the schema. No translation layer needed; LLM receives the MCP schema directly.
- **Letta** — Auto-generates JSON Schema from Python function signatures + docstrings via `schema_generator`.

---

## Execution Models

### In-Process
Tools run as functions/methods in the same process as the agent. Fastest, least isolated.
- CoPaw, Hermes Agent, PicoClaw (most tools), Nanobot

### Subprocess
Tools spawn child processes. Limited isolation via OS process boundaries.
- MicroClaw (bash hook can be blocked), Letta (local subprocess option), ZeptoClaw (native mode)

### WASM Sandbox
Tools compiled to WebAssembly with WIT interfaces. Memory-isolated, capability-restricted.
- **IronClaw** (`ironclaw-tools` WASM modules via Wasmtime)
- **Moltis** (WASM plugin tools via Wasmtime + WIT)

**Why WASM matters:** Each plugin has its own linear memory space and can only call host functions explicitly declared in the WIT interface. A rogue plugin cannot read the agent's secrets or call arbitrary system calls. Startup latency is microseconds vs. seconds for Docker. See [Security Models](Security-Models.md).

### Container Isolation
Tools run in Docker or platform-native containers.
- **NanoClaw** — entire task in a fresh container (container-per-task)
- **Letta** — Docker/Modal sandbox for tool execution
- **NemoClaw** — Docker container with network policy enforcement
- **ZeptoClaw** — Docker is one of 6 selectable sandbox runtimes

### MCP Protocol Bridge
Tools accessed via MCP servers (separate processes or remote HTTP).
- **Nanobot** — all tools are MCP servers
- **MicroClaw**, **IronClaw**, **Letta**, **PicoClaw**, **ZeptoClaw** — MCP as an additional tool source

---

## MCP Integration

[MCP (Model Context Protocol)](Glossary.md#mcp) has become the de facto standard for agent tool extensibility. **11 of 12 frameworks** implement MCP client or server capabilities.

| Agent | MCP Role | Notes |
|-------|----------|-------|
| Nanobot | **Core architecture** — MCP host | Agent IS an MCP host; agents expose themselves as MCP servers |
| IronClaw | Client + bridge | WASM-sandboxed MCP bridge |
| Letta | Client | OAuth-secured MCP connections |
| MicroClaw | Client | `mcp.rs` + separate MCP configs |
| Moltis | Client crate (`moltis-mcp`) | Dedicated crate |
| NanoClaw | Client (`.mcp.json`) | MCP config inside container |
| NemoClaw | Via plugin | OpenClaw plugin uses MCP |
| OpenFang | Client + A2A | `docs/mcp-a2a.md` |
| PicoClaw | Client (`pkg/mcp/`) | `pkg/tools/registry.go` integrates MCP tools |
| ZeptoClaw | Client + server (`src/mcp_server/`) | Both client and MCP server (stdio) |
| Hermes Agent | Client | MCP tool in `tools/` |
| CoPaw | Client (`app/mcp/`) | MCP integration module |

**Nanobot's agent-as-MCP-server** is the most architecturally novel: an agent can be called by another agent through the MCP protocol. This enables recursive agent composition without a custom A2A protocol.

---

## Approval Gates

10 of 12 frameworks have human-in-the-loop approval before executing sensitive operations.

| Agent | Approval Mechanism | Scope |
|-------|-------------------|-------|
| CoPaw | `app/approvals/` module | Configurable per-tool |
| Hermes Agent | `approval.py`; `test_force_dangerous_override.py` | Per-action; "yolo mode" bypasses |
| IronClaw | Safety crate rule engine | Risk-scored operations |
| Letta | Sandbox confirmation; MCP OAuth | Tool sandbox and remote tool auth |
| MicroClaw | `block-bash` hook can require approval | Via hook scripts |
| ZeptoClaw | Approval tool (`src/tools/`); `r8r_bridge/` (approval channel) | Explicit approval tool + relay |
| PicoClaw | Confirmation prompts | Per-sensitive-tool |
| NanoClaw | Sender allowlist (prevents unexpected principals) | Channel-level |
| NemoClaw | Network policy requires approval for new domains | Policy presets |
| Nanobot | `pkg/confirm/` confirmation prompts | Per-action |

**ZeptoClaw's R8r bridge** is unique: it routes approval requests through a dedicated relay that deduplicates requests, tracks events, and monitors health — making approval a first-class infrastructure concern rather than a simple Y/N prompt.

---

## Built-In Tool Counts

| Agent | Approx. Tools | Notable Unique Tools |
|-------|--------------|---------------------|
| Hermes Agent | 40+ | CamoFox browser, RL training, mixture-of-agents, delegation |
| OpenFang | 60+ skills + 25 extensions | Hands (autonomous), LangChain integration |
| ZeptoClaw | 30+ | Android tools, hardware tools, approval tool, diff tool |
| PicoClaw | 20+ | Hardware (I2C/SPI), ASR/TTS, BM25 memory search |
| MicroClaw | ~20 | Nostr channel, A2A tool, structured memory |
| Letta | ~15 + MCP | Archival memory tools, voice tools |
| IronClaw | Built-in + WASM | WASM-sandboxed; fuzzing harness for edge cases |
| Moltis | Built-in + WASM | CalDAV, browser, canvas |
| Nanobot | MCP servers (~6 built-in servers) | Artifacts, tasks, workflows, skills |
| NanoClaw | Skills-based | Browser, PDF, vision, parallel execution |
| CoPaw | ~15 | AgentScope tools, skill-scanned |
| NemoClaw | Via Claude in sandbox | Network-policy-controlled |

---

## Tool Security Summary

For sandbox approaches, isolation levels, and security rankings, see [Security Models](Security-Models.md).

Key insight: **WASM (IronClaw, Moltis) provides the best isolation-to-overhead tradeoff** — memory safe, capability-restricted, microsecond startup. Container-per-task (NanoClaw) provides the best blast-radius containment for multi-task workloads. The 6-runtime strategy (ZeptoClaw) gives operators the flexibility to choose.

---

*Back to [Home](Home.md) | See also: [Security Models](Security-Models.md) | [LLM Integration](LLM-Integration.md)*
