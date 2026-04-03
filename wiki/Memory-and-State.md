# Memory and State

Memory is one of the hardest problems in agent design: LLMs have finite context windows, but agents need to remember things across many interactions. This page compares how all 13 frameworks approach memory, persistence, and state management.

Related: [Tool Systems](Tool-Systems.md), [LLM Integration](LLM-Integration.md), [Unique Innovations](Unique-Innovations.md).

---

## Memory Taxonomy

Most frameworks distinguish between:
- **Working memory** — the current context window (ephemeral)
- **Session memory** — the current conversation history (persisted, loaded on reconnect)
- **Long-term / archival memory** — information that survives across sessions, searchable

The key challenge is **context compaction**: what do you do when session + working memory exceeds the LLM's context window?

---

## Storage Backends

| Agent | Primary Storage | Format | Vector Search |
|-------|----------------|--------|---------------|
| CoPaw | PostgreSQL or SQLite | SQLAlchemy ORM | Not confirmed |
| Hermes Agent | SQLite | aiosqlite sessions | Honcho service (external) |
| IronClaw | PostgreSQL + libSQL | SQLAlchemy ORM | pgvector embeddings |
| Letta | PostgreSQL + SQLite | SQLAlchemy ORM + Alembic | **pgvector** (archival memory) |
| MicroClaw | SQLite | `microclaw-storage` crate | Vector embeddings crate |
| Moltis | SQLite | sqlx | Not confirmed (50+ crates) |
| NanoClaw | SQLite | better-sqlite3 | No |
| Nanobot | SQLite | GORM | No |
| NemoClaw | None (session-based) | Container filesystem | No |
| OpenFang | Custom memory crate | Wire protocol serialization | Not confirmed |
| PicoClaw | JSONL files | Flat files | **BM25** text search |
| ZeptoClaw | Embedded + long-term | BM25 + **HNSW** vector | BM25 + HNSW |

**PostgreSQL + pgvector** (Letta, IronClaw) provides production-grade semantic similarity search over long-term memories. **HNSW** (ZeptoClaw) is an in-process approximate nearest-neighbor index — no external database required, suitable for edge deployments.

**JSONL** (PicoClaw) is the simplest approach — zero database dependencies, human-readable, but no access control and limited query capability.

---

## Letta's Three-Tier Memory (MemGPT Pattern)

Letta has the most sophisticated memory architecture, implementing the [MemGPT](Glossary.md#memgpt) pattern:

```
┌────────────────────────────────────────┐
│  Tier 1: In-Context Memory Blocks      │  ← Always in prompt
│  (persona, human, working notes)       │
├────────────────────────────────────────┤
│  Tier 2: Conversational Memory         │  ← Recent messages, loaded on demand
│  (recent N messages in session)        │
├────────────────────────────────────────┤
│  Tier 3: Archival Memory               │  ← pgvector, semantic search
│  (all past knowledge, searchable)      │
└────────────────────────────────────────┘
```

**Memory Blocks** (`letta/orm/block.py`) are named segments (e.g., `<persona>`, `<human>`, `<working_memory>`) that are always injected into the system prompt. The agent can use tools to read/write blocks, enabling explicit memory management.

**Block History** (`block_history.py`) tracks every write to every block, enabling temporal queries: "what did the agent know at time T?"

**Archival Memory** uses pgvector for semantic similarity search. When the agent wants to remember something long-term, it calls a tool to write to archival memory. To recall, it searches semantically.

**Summarizer Pipeline** (`letta/services/summarizer/`) automatically compresses older conversational memory while preserving key facts, triggered when the context window fills.

---

## Context Compaction Strategies

| Agent | Strategy | Mechanism |
|-------|----------|-----------|
| Letta | Summarize + archive | Summarizer pipeline; archival search |
| Nanobot | Compact + truncate | `pkg/agents/compact.go`, `truncate.go` |
| Hermes Agent | Compression | `agent/compression.py` |
| PicoClaw | Context management | `pkg/agent/` context window management |
| ZeptoClaw | Compaction | `src/agent/` compaction logic |
| MicroClaw | Memory service | `memory_service.rs` + `memory_backend.rs` |
| IronClaw | Document chunking | `document chunking` + vector embeddings |
| NanoClaw | None explicit | Fresh container per task resets context |
| NemoClaw | None | Session-only; no persistence |

**NanoClaw's approach is the most radical**: each task runs in a fresh container, so there's no accumulated context to manage. The tradeoff is zero cross-task memory — each task starts cold.

---

## Session Management

| Agent | Session Scope | Persistence |
|-------|--------------|-------------|
| CoPaw | Per-agent, per-channel | DB-backed (SQLAlchemy) |
| Hermes Agent | Per-platform user | SQLite, Honcho service |
| IronClaw | Per thread (thread_id) | PostgreSQL |
| Letta | Per agent instance | PostgreSQL, agent versioning |
| MicroClaw | Per channel connection | SQLite + memory service |
| Moltis | `moltis-sessions` crate | SQLite |
| NanoClaw | Per task (ephemeral) | SQLite (task history), container destroyed after |
| Nanobot | Session store with migrations | SQLite GORM |
| NemoClaw | Container lifespan | Container filesystem only |
| OpenFang | Per agent | Custom memory crate |
| PicoClaw | JSONL backend | Flat JSONL files |
| ZeptoClaw | `src/session/` | Embedded storage + repair |

ZeptoClaw's `session/` includes **session repair** — detecting and fixing corrupted session state. This signals production operational maturity.

---

## Vector Search Approaches

Three distinct approaches to semantic memory search:

### pgvector (Letta, IronClaw)
- PostgreSQL extension
- SQL-native vector similarity queries
- Production-grade, requires Postgres infrastructure
- Supports HNSW and IVFFlat indexes

### HNSW (ZeptoClaw)
- Hierarchical Navigable Small World in-process index
- No external database
- ~90%+ recall at high speed
- Suitable for edge/embedded deployments

### BM25 (PicoClaw, ZeptoClaw)
- Classic TF-IDF-based text search
- Exact keyword matching, no embedding model needed
- PicoClaw implements BM25 in pure Go (`pkg/utils/`)
- ZeptoClaw uses BM25 alongside HNSW for hybrid search

**ZeptoClaw's hybrid BM25 + HNSW** is the most flexible: keyword search finds exact terms; vector search finds semantic matches; results can be merged/ranked.

---

## Honcho Integration (Hermes Agent)

Hermes Agent integrates with **Honcho** — an external memory-as-a-service platform. Honcho provides:
- User/session hierarchies
- Long-term fact storage
- SDK-driven API

This is the only framework delegating memory management to an external service. Advantages: memory outlives the agent process; shared across multiple agent instances. Disadvantage: external service dependency and data egress.

---

## Memory Security

- **Encrypted columns** — Letta migrated from plaintext to encrypted LLM API keys in DB columns (migration `8149a781ac1b_backfill_encrypted_columns_for_`). Historical backups pre-migration contain plaintext keys.
- **Memory hygiene** — ZeptoClaw has `memory hygiene` and `snapshots` — automated cleanup and point-in-time recovery.
- **MicroClaw's `filter-global-structured-memory` hook** — can restrict what goes into memory, useful for preventing PII accumulation.
- **Letta's BYOK columns** — bring-your-own-key encrypted storage for provider credentials.

---

*Back to [Home](Home.md) | See also: [Unique Innovations](Unique-Innovations.md) | [Security Models](Security-Models.md)*
