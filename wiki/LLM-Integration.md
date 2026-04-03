# LLM Integration

All 12 frameworks must solve the same problem: connect to one or more LLM providers, send prompts, parse tool calls from responses, and feed results back. This page compares how they do it.

Related: [Tool Systems](Tool-Systems.md), [Memory and State](Memory-and-State.md), [Architecture Patterns](Architecture-Patterns.md).

---

## Provider Support Matrix

| Agent | Anthropic | OpenAI | Google/Gemini | AWS Bedrock | Local/GGUF | Other |
|-------|-----------|--------|---------------|-------------|------------|-------|
| CoPaw | ✓ | ✓ | ✓ | ✓ | ✓ (Ollama) | 13 providers total; ModelScope |
| Hermes Agent | ✓ | ✓ | ✓ | — | ✓ (vLLM, llama.cpp) | DeepSeek, Qwen, Kimi, GLM, Mistral, Llama, Hermes |
| IronClaw | ✓ | ✓ | ✓ | ✓ | — | GitHub Copilot, NearAI, Codex |
| Letta | ✓ | ✓ | ✓ | ✓ | ✓ (Ollama, vLLM) | Together, Groq, Fireworks, Mistral, DeepSeek, MiniMax, xAI |
| MicroClaw | ✓ | ✓ | — | — | — | MCP-routed |
| Moltis | ✓ | ✓ | — | — | ✓ (GGUF) | Local inference |
| NanoClaw | ✓ (Claude CLI) | — | — | — | — | Claude-only |
| Nanobot | ✓ | ✓ | — | — | — | OpenAI Completions + Responses APIs |
| NemoClaw | ✓ (via sandbox) | — | — | — | — | NVIDIA NIM (GPU) |
| OpenFang | ✓ | ✓ | ✓ (Vertex AI) | — | — | Driver pattern |
| PicoClaw | ✓ | ✓ | — | ✓ | — | Azure OpenAI, GitHub Copilot, Codex |
| ZeptoClaw | ✓ | ✓ | ✓ | — | — | Vertex AI; rotation/fallback |

**Letta** (via LiteLLM) has the broadest provider coverage. **NanoClaw** is the most opinionated — Claude-only by design.

---

## Provider Abstraction Patterns

### Strategy Pattern
A `Provider` trait/interface with per-provider implementations. The agent configures which provider to use; swapping providers requires only config change.
- **IronClaw** — `src/providers/` with Rust trait
- **PicoClaw** — `pkg/providers/` with factory pattern (`ProviderFactory`)
- **ZeptoClaw** — `src/providers/` with retry/fallback/rotation wrapper

### LiteLLM Routing (Letta)
Letta delegates all provider routing to **LiteLLM**, a Python library that normalizes OpenAI-compatible APIs across 100+ providers. Advantages: broadest coverage, single upgrade path. Disadvantages: external dependency, LiteLLM quirks propagate.

### Driver Pattern (OpenFang)
OpenFang's `openfang-runtime/src/drivers/` implements a driver abstraction for LLM providers (Vertex AI, OpenAI, Anthropic). Drivers handle the wire protocol translation, streaming, and tool-call parsing.

### Unified Completions Orchestrator (Nanobot)
Nanobot's `pkg/complete/complete.go` is a single orchestration file that handles both OpenAI Completions API and OpenAI Responses API. Simple and auditable.

### AgentScope Runtime (CoPaw)
CoPaw delegates LLM routing to the **AgentScope** external runtime library (`agentscope==1.0.18`). This provides 13 provider adapters but creates vendor lock-in on AgentScope's release cadence.

---

## Tool Call Parsing

This is where the most interesting divergence occurs. LLMs produce tool calls in different formats, and parsing them correctly is critical for agent reliability.

### OpenAI Function-Calling Format (Most Common)
The majority of frameworks use the OpenAI function-calling standard:
```json
{
  "tool_calls": [
    {
      "id": "call_abc123",
      "type": "function",
      "function": {
        "name": "read_file",
        "arguments": "{\"path\": \"/tmp/foo.txt\"}"
      }
    }
  ]
}
```
Works for: OpenAI, Anthropic (tool_use), Google Gemini, most fine-tuned models.

### Hermes Agent's Per-Model Parsers
Hermes Agent maintains **12+ dedicated parsers** in `environments/tool_call_parsers/`:

| Model Family | Parser |
|-------------|--------|
| DeepSeek V3 | Custom XML-style parser |
| DeepSeek V3.1 | Variant parser |
| Qwen / Qwen3-Coder | Qwen-specific format |
| Kimi K2 | Kimi-specific format |
| Mistral | Mistral function call format |
| Llama 3.x | Llama tool call format |
| GLM-4.5 / GLM-4.7 | GLM-specific format |
| Hermes/NousResearch | Hermes XML format |
| Longcat | Custom format |

Why this matters: models fine-tuned on Chinese datasets (Qwen, Kimi, GLM, DeepSeek) often use different tool-call delimiters or JSON structures than OpenAI's standard. A single generic parser silently fails on edge cases; per-model parsers handle each model's idiosyncrasies explicitly.

This is the most model-diverse tool call parsing system of any framework analyzed.

---

## Streaming

All frameworks that support real-time streaming use **Server-Sent Events (SSE)** for streaming LLM token output to clients. WebSocket is used for bidirectional chat interfaces.

| Agent | Streaming Protocol | Notes |
|-------|-------------------|-------|
| IronClaw | SSE | `/api/chat` returns `text/event-stream` |
| Letta | SSE + WebSocket | `letta/server/ws_api/` for WebSocket |
| Hermes Agent | SSE | Gateway streams token-by-token |
| MicroClaw | SSE | `src/web/stream.rs` |
| PicoClaw | SSE | Web backend streaming |
| ZeptoClaw | SSE | `src/api/` Axum SSE |
| Nanobot | Event stream | Custom Go HTTP event stream |
| Moltis | GraphQL subscriptions | Unique — uses async-graphql subscriptions |

**Moltis** is unique in using **GraphQL subscriptions** instead of SSE. This enables typed streaming via the GraphQL schema, but requires GraphQL clients on the frontend.

---

## Provider Resilience

Production LLM providers have rate limits, quota exhaustion, and outages. How do frameworks handle this?

| Agent | Retry | Fallback | Rotation | Cooldown | Budget |
|-------|-------|----------|----------|----------|--------|
| ZeptoClaw | ✓ | ✓ | ✓ | ✓ | ✓ |
| Hermes Agent | ✓ | ✓ | ✓ (credential pool) | — | — |
| Letta | ✓ (LiteLLM) | ✓ (LiteLLM) | — | — | — |
| IronClaw | ✓ | ✓ (PKCE per provider) | — | — | — |
| PicoClaw | ✓ | ✓ | — | — | — |
| MicroClaw | ✓ | — | — | — | — |
| Others | Basic retry | — | — | — | — |

**ZeptoClaw** has the most complete resilience system:
- **Rotation**: multiple API keys load-balanced across requests
- **Fallback**: if provider A fails, try provider B
- **Cooldown**: after N consecutive failures, back off before retrying
- **Quota management**: track remaining quota and switch providers proactively
- **Budget enforcement**: hard cap on token spend per session/agent

**Hermes Agent's credential pool** (`credential pool with rotation`) allows multiple API keys per provider to be load-balanced — useful for high-throughput scenarios where a single key hits rate limits.

---

## OAuth and Token Management

Production deployments require secure OAuth flows for LLM provider authentication. OAuth 2.0 + PKCE is the standard:

| Agent | OAuth + PKCE | Token Rotation | Encrypted Storage |
|-------|-------------|----------------|------------------|
| IronClaw | ✓ | ✓ | OS keychain |
| Hermes Agent | ✓ | ✓ (credential pool) | YAML (unencrypted) |
| Moltis | ✓ | ✓ | vault crate |
| PicoClaw | ✓ | ✓ | `pkg/credential/` |
| ZeptoClaw | ✓ | ✓ | AES-256-GCM |
| Letta | ✓ (MCP OAuth) | — | BYOK columns |
| Nanobot | ✓ (MCP OAuth) | — | — |
| CoPaw | ✓ (partial) | — | — |
| Others | Token-based | — | Env vars |

[PKCE (Proof Key for Code Exchange)](Glossary.md#pkce) prevents authorization code interception attacks, critical when the agent acts as a public OAuth client without a secure client secret.

---

## Smart Model Routing (Hermes Agent)

Hermes Agent implements **smart model routing** — automatically selecting the best model for a task type:
- Simple queries → faster/cheaper models (e.g., Qwen)
- Complex coding tasks → capable models (e.g., DeepSeek Coder, Claude)
- Image analysis → vision-capable models

Combined with the credential pool and 12+ parsers, Hermes has the most sophisticated LLM integration of any Python-based framework analyzed.

---

## Context Window Management

See [Memory and State](Memory-and-State.md) for detailed compaction strategies. Summary:

- **Letta**: Summarizer pipeline + archival memory (most sophisticated)
- **Nanobot**: `compact.go` + `truncate.go`
- **Hermes**: `agent/compression.py`
- **NanoClaw**: Fresh container per task (no compaction needed)
- **ZeptoClaw**: Compaction + budget enforcement (token limits)

---

*Back to [Home](Home.md) | See also: [Tool Systems](Tool-Systems.md) | [Memory and State](Memory-and-State.md)*
