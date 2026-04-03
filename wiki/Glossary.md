# Glossary

Key terms and acronyms used across the agent architecture analyses.

→ Back to [Home](Home.md)

---

## Protocols & Standards

| Term | Definition |
|------|-----------|
| **MCP** | Model Context Protocol — open standard for connecting LLMs to external tools/data sources via stdio or HTTP. Created by Anthropic. Most agents support it. |
| **A2A** | Agent-to-Agent protocol — Google's draft spec for inter-agent HTTP communication. Used by MicroClaw and OpenFang. |
| **ACP** | Agent Communication Protocol — alternative inter-agent protocol via stdio. Used alongside A2A in some frameworks. |
| **WIT** | WebAssembly Interface Types — defines typed interfaces for WASM components. Used by IronClaw and Moltis for plugin contracts. |
| **GraphQL** | Query language for APIs. Used by Moltis for client-gateway communication with subscriptions. |
| **APNS** | Apple Push Notification Service — used by Moltis's courier relay agent for iOS push notifications. |

## Agent Patterns

| Term | Definition |
|------|-----------|
| **ReAct** | Reasoning + Acting — agent loop pattern where the LLM alternates between thinking and tool execution. Most common agent loop pattern. |
| **MemGPT** | Memory-augmented GPT — Letta's approach to giving agents persistent, structured, multi-tier memory with automatic context management. |
| **Sleeptime** | Background memory consolidation that runs between turns — a dedicated agent distills conversation into durable knowledge while the main agent is idle. From Letta. |
| **Hook Pipeline** | AOP-style (Aspect-Oriented Programming) interception of tool calls. Hooks run before/after tool execution for logging, security, transformation. From MicroClaw. |
| **Agent-as-MCP-Server** | Pattern where agents expose themselves as MCP-callable tools, enabling recursive agent composition. From Nanobot. |
| **Digital Twin** | An always-available AI version of an employee that responds using their SOUL + memory. Anyone with the link can chat. From enterprise OpenClaw platforms. |

## Security Concepts

| Term | Definition |
|------|-----------|
| **WASM Sandbox** | WebAssembly-based isolation — tool/plugin code runs in a memory-safe VM with capability-based access control. Used by IronClaw, Moltis, ZeptoClaw. |
| **Landlock** | Linux Security Module (LSM) that restricts filesystem and network access per-process without requiring root. Used by ZeptoClaw. |
| **Bubblewrap (bwrap)** | Lightweight OCI-compatible sandboxing tool using Linux namespaces. Used by ZeptoClaw. |
| **Firejail** | Profile-based Linux sandboxing using namespaces and seccomp. Used by ZeptoClaw. |
| **Apple Sandbox** | macOS `sandboxd`-based process isolation. Used by ZeptoClaw and NanoClaw (Apple Containers). |
| **Taint Tracking** | Tracing data provenance through an agent pipeline to detect potential leakage or policy violations. From ZeptoClaw's `src/safety/` module. |
| **Network Policy** | YAML-defined per-domain allow/deny rules controlling what network endpoints an agent can reach. Strongest implementation in NemoClaw. |
| **Plan A / Plan E** | Soft enforcement (system prompt injection of allowed tools) + post-execution audit. Used in enterprise OpenClaw multi-tenant platforms. |
| **Agent Modes** | Safe / Standard / Unrestricted execution levels with explicit escalation required. From ZeptoClaw. |
| **Credential Sanitization** | Stripping API keys from environment before exposing to sandbox; credentials injected only at container start, never readable inside. From NemoClaw. |
| **Coverage Ratchet** | CI rule that prevents test coverage percentage from decreasing between commits. From NemoClaw. |

## Memory & State

| Term | Definition |
|------|-----------|
| **Blocks** | Named, versioned, in-context memory slots editable by the agent as tools. From Letta's MemGPT architecture. |
| **Passages** | Archival memory entries embedded as vectors (pgvector) for semantic search. From Letta. |
| **Compaction** | Summarizing older conversation turns to stay within context window limits. Various approaches across frameworks. |
| **BM25** | Best Matching 25 — probabilistic text retrieval algorithm used for keyword-based memory search. Used by PicoClaw and ZeptoClaw. |
| **HNSW** | Hierarchical Navigable Small World — approximate nearest neighbor algorithm for vector search. Used by ZeptoClaw alongside BM25. |
| **pgvector** | PostgreSQL extension for vector similarity search. Used by Letta and IronClaw for archival memory. |

## Infrastructure

| Term | Definition |
|------|-----------|
| **Firecracker** | AWS's lightweight microVM technology (powers Lambda and Fargate). Used in enterprise OpenClaw multi-tenant platforms via AgentCore. |
| **NIM** | NVIDIA Inference Microservice — containerized GPU inference. Integrated by NemoClaw for local model serving. |
| **Tauri** | Rust-based framework for building native desktop apps with web frontends. Used by OpenFang. |
| **Temporal** | Workflow orchestration engine. Used by RepoSwarm for investigation workflows. |
| **LiteLLM** | Python proxy that provides a unified OpenAI-compatible API across 100+ LLM providers. Used by Letta. |

## Language & Build

| Term | Definition |
|------|-----------|
| **Cargo workspace** | Rust's monorepo package management — multiple crates in one repo sharing dependencies. Used by 5/12 Rust-based agents. |
| **cargo-deny** | Supply chain security tool that checks Rust dependencies for vulnerabilities, license compliance, and banned crates. |
| **cargo-audit** | Rust dependency vulnerability scanner using the RustSec advisory database. |
| **Nix** | Declarative package manager and build system enabling reproducible builds. Used by Hermes and MicroClaw. |

---

→ Back to [Home](Home.md) · [Overview](Overview.md)
