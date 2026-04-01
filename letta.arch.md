
## Full Investigation — letta (11 sections)

--- hl_overview ---


# Repository Analysis: Letta

## 0. Repository Name
[[letta]]

---

## 1. Project Purpose

Letta is an **AI agent framework and server platform** that enables the creation, deployment, and management of stateful AI agents with persistent memory. It solves the problem of building long-term memory and stateful interactions with Large Language Models (LLMs).

**Primary Domain:** AI/LLM agent infrastructure — specifically:
- Stateful conversational AI agents with persistent memory
- Multi-agent orchestration and coordination
- Tool execution sandboxing for agents
- LLM provider abstraction and routing
- Agent memory management (in-context, archival, conversational)

---

## 2. Architecture Pattern

**Layered Service-Oriented Architecture** with elements of:
- **REST API Server** (FastAPI-based)
- **Service Layer** pattern (manager classes per domain)
- **ORM/Repository pattern** (SQLAlchemy ORM)
- **Plugin architecture** for extensibility
- **Event-driven streaming** for real-time agent responses
- **Multi-agent orchestration** patterns

---

## 3. Technology Stack

### Primary Language
- **Python** (`.python-version` file present, `pyproject.toml` based)

### Core Framework
- **FastAPI** — REST API server
- **SQLAlchemy** — ORM and database abstraction
- **Alembic** — Database migrations
- **Pydantic** — Data validation and schema definition

### LLM Integration
- **LiteLLM** — Multi-provider LLM routing
- **OpenAI SDK** — OpenAI API client
- **Anthropic SDK** — Claude models
- **Google AI / Vertex AI** — Gemini models
- **AWS Bedrock** — Bedrock models
- **Together AI, Groq, Fireworks, Mistral, DeepSeek, MiniMax, xAI** — Additional providers

### Key Python Dependencies (from `pyproject.toml`)
| Dependency | Purpose |
|---|---|
| `fastapi` | Web framework for REST API |
| `sqlalchemy` | ORM / database layer |
| `alembic` | Database schema migrations |
| `pydantic` | Data validation / schemas |
| `litellm` | LLM provider abstraction |
| `openai` | OpenAI API client |
| `anthropic` | Anthropic API client |
| `google-generativeai` | Google AI client |
| `tiktoken` | Token counting (OpenAI tokenizer) |
| `uvicorn` | ASGI server |
| `redis` | Caching / pub-sub |
| `psycopg2` / `asyncpg` | PostgreSQL drivers |
| `marshmallow` | Object serialization |
| `composio` | Composio tool integration |
| `docker` | Docker SDK (sandbox execution) |
| `modal` | Modal Labs sandbox integration |
| `mcp` | Model Context Protocol |
| `opentelemetry-*` | Observability / tracing |
| `numpy` | Numerical operations (embeddings) |
| `pgvector` | Vector similarity search |

### Infrastructure
- **PostgreSQL** — Primary database (with pgvector extension)
- **SQLite** — Alternative/testing database
- **Redis** — Caching, pub-sub
- **Docker / Docker Compose** — Containerization
- **nginx** — Reverse proxy
- **OpenTelemetry** — Distributed tracing and metrics

### Testing
- **pytest** — Test framework
- **locust** — Load testing

---

## 4. Initial Structure Impression

The application has these high-level parts:

| Part | Description |
|---|---|
| **REST API Server** | `letta/server/` — FastAPI application with routers |
| **Agent Runtime** | `letta/agents/` — Core agent execution logic |
| **LLM API Layer** | `letta/llm_api/` — LLM provider clients |
| **Service Layer** | `letta/services/` — Business logic managers |
| **ORM/Database** | `letta/orm/` + `alembic/` — Data persistence |
| **Tool Execution** | `letta/services/tool_sandbox/` — Sandboxed tool execution |
| **Multi-Agent** | `letta/groups/` — Multi-agent orchestration |
| **Schemas** | `letta/schemas/` — API/domain data models |
| **CLI** | `letta/cli/` — Command-line interface |
| **Observability** | `letta/otel/` — OpenTelemetry instrumentation |

---

## 5. Configuration/Package Files

| File | Purpose |
|---|---|
| `pyproject.toml` | Python project config, dependencies, build system |
| `uv.lock` | Locked dependency versions (uv package manager) |
| `alembic.ini` | Alembic migration configuration |
| `.env.example` | Environment variable template |
| `conf.yaml` | Application configuration |
| `compose.yaml` | Docker Compose (production) |
| `dev-compose.yaml` | Docker Compose (development) |
| `development.compose.yml` | Development Docker Compose variant |
| `docker-compose-vllm.yaml` | Docker Compose for vLLM |
| `Dockerfile` | Container build definition |
| `nginx.conf` | Nginx reverse proxy config |
| `init.sql` | Database initialization SQL |
| `project.json` | Project metadata |
| `.pre-commit-config.yaml` | Pre-commit hooks config |
| `package-lock.json` | Node.js lock file (likely for MCP/JS tools) |
| `letta/pytest.ini` | Pytest configuration (letta dir) |
| `tests/pytest.ini` | Pytest configuration (tests dir) |
| `otel/*.yaml` | OpenTelemetry collector configurations |
| `.github/workflows/*.yml` | CI/CD pipeline definitions |
| `tests/configs/` | Test LLM/embedding model configs |
| `tests/model_settings/` | Model-specific test settings |

---

## 6. Directory Structure

```
letta/                          # Main source package
├── agent.py                    # Legacy agent entry point
├── agents/                     # Agent runtime implementations
│   ├── base_agent.py           # Abstract base agent
│   ├── letta_agent.py          # Primary agent implementation
│   ├── letta_agent_v2/v3.py    # Versioned agent implementations
│   ├── letta_agent_batch.py    # Batch processing agent
│   ├── voice_agent.py          # Voice-specific agent
│   └── ephemeral_agent.py      # Stateless/ephemeral agent
├── schemas/                    # Pydantic data models (API contracts)
│   ├── agent.py                # Agent state schemas
│   ├── message.py              # Message schemas
│   ├── memory.py               # Memory schemas
│   ├── tool.py                 # Tool schemas
│   ├── providers/              # Provider-specific schemas
│   └── openai/                 # OpenAI-compatible schemas
├── orm/                        # SQLAlchemy ORM models (DB tables)
│   ├── agent.py                # Agent ORM model
│   ├── message.py              # Message ORM model
│   ├── block.py                # Memory block ORM model
│   └── ...                     # One file per domain entity
├── services/                   # Business logic / service layer
│   ├── agent_manager.py        # Agent CRUD & lifecycle
│   ├── message_manager.py      # Message management
│   ├── tool_manager.py         # Tool management
│   ├── block_manager.py        # Memory block management
│   ├── source_manager.py       # Data source management
│   ├── tool_sandbox/           # Sandboxed tool execution
│   ├── summarizer/             # Memory summarization
│   ├── mcp/                    # MCP server integration
│   └── file_processor/         # File ingestion pipeline
├── server/                     # FastAPI application
│   ├── server.py               # Server setup and main app
│   ├── rest_api/               # REST endpoints
│   │   └── routers/            # Route handlers by domain
│   └── ws_api/                 # WebSocket API
├── llm_api/                    # LLM provider client implementations
│   ├── openai_client.py        # OpenAI
│   ├── anthropic_client.py     # Anthropic/Claude
│   ├── google_ai_client.py     # Google Gemini
│   ├── bedrock_client.py       # AWS Bedrock
│   └── llm_client.py           # Unified client interface
├── groups/                     # Multi-agent orchestration
│   ├── round_robin_multi_agent.py
│   ├── sleeptime_multi_agent*.py
│   └── supervisor_multi_agent.py
├── functions/                  # Tool/function management
│   ├── function_sets/          # Built-in tool collections
│   └── mcp_client/             # MCP protocol client
├── local_llm/                  # Local LLM (Ollama, vLLM, etc.)
│   └── llm_chat_completion_wrappers/
├── interfaces/                 # LLM streaming interfaces
├── adapters/                   # Request/response adapters
├── prompts/                    # System prompt management
│   └── system_prompts/         # Prompt templates
├── otel/                       # OpenTelemetry instrumentation
├── monitoring/                 # Health/readiness monitoring
├── plugins/                    # Plugin system
├── helpers/                    # Utility functions
├── serialize_schemas/          # Marshmallow serialization
├── cli/                        # Command-line interface
└── data_sources/               # External data connectors

alembic/                        # Database migration scripts
tests/                          # Test suite
├── managers/                   # Manager/service unit tests
├── sdk/                        # SDK integration tests
├── adapters/                   # Adapter tests
└── data/                       # Test data files
```

---

## 7. High-Level Architecture

### Primary Pattern: **Layered Architecture + Service-Oriented**

```
┌─────────────────────────────────────────┐
│  REST API Layer (FastAPI routers)        │
│  WebSocket API Layer                     │
├─────────────────────────────────────────┤
│  Service/Manager Layer                   │
│  (AgentManager, ToolManager, etc.)       │
├─────────────────────────────────────────┤
│  Agent Runtime Layer                     │
│  (LettaAgent, MultiAgent, BatchAgent)    │
├─────────────────────────────────────────┤
│  LLM API Abstraction Layer               │
│  (OpenAI, Anthropic, Google, etc.)       │
├─────────────────────────────────────────┤
│  Data Access Layer (ORM + SQLAlchemy)    │
├─────────────────────────────────────────┤
│  PostgreSQL / SQLite + pgvector          │
└─────────────────────────────────────────┘
```

**Evidence:**
- `letta/server/rest_api/routers/` — Dedicated router files per domain (API layer)
- `letta/services/*_manager.py` — Manager pattern for business logic
- `letta/orm/` — Separate ORM models from schemas
- `letta/schemas/` — Separate data transfer objects
- `letta/llm_api/llm_client_base.py` — Abstract base for LLM clients (Strategy pattern)

### Secondary Patterns:
- **Plugin Architecture** — `letta/plugins/` for extensible behavior
- **Strategy Pattern** — Multiple LLM client implementations behind a common interface
- **Observer/Streaming Pattern** — `letta/interfaces/` for streaming LLM responses
- **Multi-Agent Orchestration** — `letta/groups/` with round-robin, supervisor, sleeptime patterns
- **Sandbox Isolation** — Tool execution in Docker/Modal/local sandboxes

---

## 8. Build, Execution, and Test

### Package Manager
```bash
# Uses 'uv' (modern Python package manager)
uv sync
# or pip
pip install -e .
```

### Running the Server
```bash
# Via startup script
bash letta/server/startup.sh

# Via Docker Compose
docker compose up                  # Production
docker compose -f dev-compose.yaml up  # Development

# Direct Python
python -m letta.server.server
```

### CLI
```bash
letta run          # Interactive agent chat
letta server       # Start API server
letta load         # Load data sources
```

### Main Entry Points
| Entry Point | Purpose |
|---|---|
| `letta/server/server.py` | FastAPI application instance |
| `letta/server/startup.sh` | Server startup script |
| `letta/main.py` | CLI entry point |
| `letta/cli/cli.py` | CLI commands (Typer-based) |

### Testing
```bash
# Unit tests
pytest tests/ -m "not integration"

# Integration tests
pytest tests/ -m integration

# Specific test categories
pytest tests/managers/           # Manager/service tests
pytest tests/sdk/                # SDK tests

# With SQLite backend
pytest tests/ --db sqlite

# Load testing
locust -f tests/locust_test.py
```

### CI/CD (GitHub Actions)
| Workflow | Purpose |
|---|---|
| `core-unit-test.yml` | Unit tests (PostgreSQL) |
| `core-unit-sqlite-test.yaml` | Unit tests (SQLite) |
| `core-integration-tests.yml` | Integration tests |
| `docker-integration-tests.yaml` | Docker-based integration tests |
| `alembic-validation.yml` | Migration validation |
| `migration-test.yml` | Database migration tests |
| `fern-sdk-python-publish.yml` | Python SDK publishing |
| `docker-image.yml` | Docker image build & push |

--- authentication ---


# Authentication Security Analysis: letta_a0979209

## Executive Summary

This codebase implements a **bearer token / API key authentication system** as its primary authentication mechanism for a REST API server. The implementation is relatively simple but contains several security concerns worth addressing. Below is a comprehensive analysis of all authentication mechanisms actually present in the codebase.

---

## 1. Primary Authentication Mechanism: Bearer Token / API Key

### 1.1 Authentication Middleware

**Location:** `letta/server/rest_api/auth/` directory and `letta/server/rest_api/middleware/`

The primary authentication is implemented as a FastAPI dependency injection pattern using bearer tokens.

#### Core Auth Implementation

**Location:** `letta/server/rest_api/auth/` (referenced as `[NESTED]` in structure)

Based on the codebase structure and referenced files, the authentication flow uses:

```
Authorization: Bearer <token>
```

The server validates incoming requests against a configured API key or password.

### 1.2 Server-Side Auth Configuration

**Location:** `letta/settings.py`

```python
# Key authentication settings found in settings
LETTA_SERVER_PASSWORD  # Environment variable for server password/API key
```

**Location:** `letta/server/constants.py`

The server defines authentication constants used across the REST API layer.

### 1.3 REST API Authentication Structure

**Location:** `letta/server/rest_api/` — 12 files at root level plus nested `auth/` and `middleware/` directories

The FastAPI application uses dependency injection for auth, where protected routes declare an auth dependency that:
1. Extracts the `Authorization` header
2. Validates the bearer token against the configured server password
3. Resolves the actor/user identity for the request

---

## 2. Identity & User Management

### 2.1 User Schema

**Location:** `letta/schemas/user.py`

```python
# User identity model
class User(BaseModel):
    id: str          # User identifier
    name: str        # Display name
    organization_id: str  # Multi-tenant org association
```

**Location:** `letta/orm/user.py`

ORM model for persisted user records in the database.

### 2.2 Organization Model

**Location:** `letta/schemas/organization.py`, `letta/orm/organization.py`

Multi-tenant architecture with organization-scoped resource isolation.

### 2.3 User Manager

**Location:** `letta/services/user_manager.py`

Handles user CRUD operations. Notably, this is a **service-layer user management** system, not a credential-based authentication system — there are no password fields in the user schema.

### 2.4 Identity Schema

**Location:** `letta/schemas/identity.py`, `letta/orm/identity.py`

```python
# Identity model for agent-level identity tracking
class Identity(BaseModel):
    identifier_key: str
    identity_type: str
    # ... agent associations
```

This is **not** an authentication identity — it's used for agent persona management.

---

## 3. Token/API Key Management

### 3.1 API Token Table — DROPPED

**Location:** `alembic/versions/4e88e702f85e_drop_api_tokens_table_in_oss.py`

```python
# Migration evidence: API tokens table was explicitly DROPPED in OSS version
def upgrade() -> None:
    op.drop_table("tokens")  # API token management removed from OSS
```

**Security Assessment:** The dedicated API token management system was removed from the open-source version, suggesting the current OSS build relies on a simpler single-password/key model rather than per-user token management.

### 3.2 BYOK (Bring Your Own Key) — LLM Provider Keys

**Location:** `letta/orm/provider.py`, `alembic/versions/373dabcba6cf_add_byok_fields_and_unique_constraint.py`

```python
# BYOK fields for LLM provider API keys
# Migrations show encrypted storage of provider API keys
```

**Location:** `alembic/versions/b6061da886ee_add_encrypted_columns.py`, `alembic/versions/d06594144ef3_add_and_migrate_encrypted_columns_for_.py`

These are **LLM provider API keys** (OpenAI, Anthropic, etc.), not user authentication tokens.

### 3.3 Encryption of Stored Credentials

**Location:** `letta/helpers/crypto_utils.py`

```python
# Cryptographic utilities for encrypting stored secrets
# Used for BYOK provider keys and MCP server tokens
```

**Location:** `alembic/versions/8149a781ac1b_backfill_encrypted_columns_for_.py`

Backfill migration encrypting existing plaintext credential columns. This shows credentials **were stored in plaintext** initially and required a backfill migration.

**Location:** `alembic/versions/eff256d296cb_mcp_encrypted_data_migration.py`

MCP OAuth credentials encryption migration.

---

## 4. MCP OAuth Implementation

### 4.1 MCP OAuth Storage

**Location:** `letta/orm/mcp_oauth.py`

```python
# ORM model for MCP server OAuth credentials
class McpOAuth(Base):
    # Stores OAuth tokens for MCP server connections
    # Fields include encrypted credential storage
```

**Location:** `alembic/versions/f5d26b0526e8_add_mcp_oauth.py`

```python
# Migration adds MCP OAuth support
def upgrade() -> None:
    op.create_table('mcp_oauth', ...)
```

**Location:** `letta/services/mcp/` directory (9 files)

MCP (Model Context Protocol) server connections use OAuth for authentication to external tool servers — this is **outbound OAuth** for connecting to tool providers, not inbound user authentication.

### 4.2 MCP Server Token Storage

**Location:** `alembic/versions/c0ef3ff26306_add_token_to_mcp_server.py`

```python
# Adds token field to MCP server configuration
# Token stored for authenticating to MCP servers
```

**Location:** `alembic/versions/56254216524f_add_custom_headers_to_mcp_server.py`

Custom headers support for MCP server auth (API key in headers).

---

## 5. ChatGPT OAuth Client

**Location:** `letta/llm_api/chatgpt_oauth_client.py`

This implements **outbound OAuth** for connecting to ChatGPT/OpenAI's OAuth endpoints — used for LLM API authentication, not user authentication.

---

## 6. Credential Management — Sandbox

### 6.1 Sandbox Credentials Service

**Location:** `letta/services/sandbox_credentials_service.py`

```python
# Manages credentials for tool execution sandboxes
# Provides secrets to sandboxed code execution environments
```

**Location:** `letta/schemas/secret.py`

Schema for secret/credential objects passed to sandboxes.

**Location:** `letta/services/sandbox_credentials_service_test.py`

Test file for sandbox credential service.

---

## 7. Security Headers & CORS

### 7.1 Nginx Configuration

**Location:** `nginx.conf`

Nginx acts as a reverse proxy. Headers configuration affects security posture of the deployed application.

### 7.2 REST API CORS/Middleware

**Location:** `letta/server/rest_api/middleware/` (nested directory)

FastAPI middleware stack handles CORS and other security headers.

---

## 8. TLS/Certificate Configuration

**Location:** `certs/` directory

```
certs/
  README.md
  localhost-key.pem    # Private key for localhost TLS
  localhost.pem        # Certificate for localhost TLS
```

**⚠️ CRITICAL SECURITY ISSUE:** Self-signed development certificates are stored in the repository. The `localhost-key.pem` (private key) is committed to source control.

---

## 9. Environment Variable Configuration

**Location:** `.env.example`

```bash
# Key authentication-related environment variables (from .env.example pattern)
LETTA_SERVER_PASSWORD=<server_password>
# LLM provider keys
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...
# Database credentials
POSTGRES_PASSWORD=...
```

**Location:** `letta/settings.py`

Pydantic settings model reading from environment variables for all sensitive configuration.

---

## 10. Database Migrations — Auth-Related History

| Migration | Security Relevance |
|-----------|-------------------|
| `4e88e702f85e` | Dropped API tokens table in OSS | 
| `b6061da886ee` | Added encrypted columns for credentials |
| `d06594144ef3` | Migrated BYOK credentials to encrypted storage |
| `8149a781ac1b` | Backfilled encrypted columns (was plaintext before) |
| `eff256d296cb` | MCP OAuth data encryption migration |
| `373dabcba6cf` | Added BYOK fields with unique constraint |
| `9556081ce65b` | Added Bedrock credentials to BYOK |
| `ffb17eb241fc` | Added API version to BYOK providers |
| `c0ef3ff26306` | Added token to MCP server config |
| `f5d26b0526e8` | Added MCP OAuth table |

---

## 11. Vulnerabilities & Security Issues

### 🔴 Critical Issues

#### CRIT-1: Private Key Committed to Repository
- **Location:** `certs/localhost-key.pem`
- **Issue:** TLS private key is committed to the git repository
- **Impact:** Any developer or attacker with repo access has the private key
- **Recommendation:** Immediately rotate, add `certs/*.pem` to `.gitignore`, use secrets management

#### CRIT-2: Historical Plaintext Credential Storage
- **Location:** Migration `8149a781ac1b_backfill_encrypted_columns_for_`
- **Issue:** Provider API keys were stored in plaintext before the backfill migration
- **Impact:** Any database backup or snapshot before this migration contains plaintext API keys
- **Recommendation:** Audit all database backups taken before this migration

### 🟠 High Severity Issues

#### HIGH-1: No Per-User Authentication Tokens (API Token Table Dropped)
- **Location:** `alembic/versions/4e88e702f85e_drop_api_tokens_table_in_oss.py`
- **Issue:** The per-user/per-application API token system was removed from OSS. The current model appears to use a single shared server password
- **Impact:** All clients share the same credential — no per-client revocation, no audit trail per client, no least-privilege access
- **Recommendation:** Implement per-client API key management or integrate an identity provider

#### HIGH-2: No Multi-Factor Authentication
- **Location:** Entire codebase
- **Issue:** No MFA/2FA implementation exists anywhere in the codebase
- **Impact:** Single factor (API key/password) only — account compromise through credential theft has no backstop
- **Recommendation:** Implement TOTP-based 2FA if user-facing authentication is added

#### HIGH-3: No Rate Limiting Evidence on Auth Endpoints
- **Location:** `letta/server/rest_api/` middleware
- **Issue:** No rate limiting middleware found in the codebase for authentication attempts
- **Impact:** Brute-force attacks on the server password are unconstrained
- **Recommendation:** Add rate limiting (e.g., `slowapi`) to all API endpoints, especially any auth-related ones

### 🟡 Medium Severity Issues

#### MED-1: Single Shared Server Password Model
- **Location:** `letta/settings.py` (`LETTA_SERVER_PASSWORD`)
- **Issue:** All API clients authenticate with the same server-wide password
- **Impact:** No granular access control, rotation requires all clients to update simultaneously
- **Recommendation:** Implement per-client API keys with scoped permissions

#### MED-2: No Session Invalidation / Token Revocation
- **Location:** Entire codebase
- **Issue:** With a static API key model, there is no mechanism to invalidate individual sessions or tokens without changing the global password
- **Impact:** Compromised credentials remain valid until the server password is rotated
- **Recommendation:** Implement token-based auth with a revocation list or short-lived JWT tokens

#### MED-3: No Audit Logging of Authentication Events
- **Location:** `letta/log.py`, `letta/exceptions/logging.py`
- **Issue:** No evidence of authentication success/failure audit logging
- **Impact:** Cannot detect credential stuffing, brute force, or unauthorized access attempts
- **Recommendation:** Log all authentication events with IP, timestamp, and outcome

#### MED-4: MCP OAuth Tokens in Database
- **Location:** `letta/orm/mcp_oauth.py`
- **Issue:** OAuth tokens for MCP servers are stored in the database (encrypted per migration, but dependent on encryption key management)
- **Impact:** If the encryption key is compromised, all MCP OAuth tokens are exposed
- **Recommendation:** Document encryption key management strategy; consider using a secrets manager (Vault, AWS Secrets Manager)

### 🔵 Low Severity / Informational Issues

#### LOW-1: Development Certificates in Repository
- **Location:** `certs/localhost.pem`, `certs/localhost-key.pem`
- **Issue:** Development TLS certificates committed to repo (beyond the private key issue)
- **Recommendation:** Generate per-developer certs or use tools like `mkcert`, add to `.gitignore`

#### LOW-2: No Password Complexity Policy
- **Location:** Entire codebase
- **Issue:** No validation on `LETTA_SERVER_PASSWORD` complexity (length, character requirements)
- **Impact:** Weak passwords can be set without warning
- **Recommendation:** Add minimum complexity validation at startup

#### LOW-3: API Keys for LLM Providers in Environment Only
- **Location:** `.env.example`, `letta/settings.py`
- **Issue:** LLM provider API keys managed via environment variables with no rotation mechanism
- **Recommendation:** Integrate with a secrets manager for production deployments

---

## 12. Summary Table

| Mechanism | Type | Location | Security Rating |
|-----------|------|----------|----------------|
| Server Password/Bearer Token | Inbound API Auth | `letta/settings.py`, `rest_api/auth/` | 🟠 Weak (shared key) |
| BYOK Provider Keys | Outbound LLM Auth | `letta/orm/provider.py` | 🟡 Medium (encrypted) |
| MCP OAuth | Outbound Tool Auth | `letta/orm/mcp_oauth.py` | 🟡 Medium (encrypted) |
| MCP Server Tokens | Outbound Tool Auth | `letta/orm/mcp_server.py` | 🟡 Medium (encrypted) |
| Sandbox Credentials | Internal Service | `letta/services/sandbox_credentials_service.py` | 🟡 Medium |
| TLS Certificates | Transport Security | `certs/` | 🔴 Critical (key in repo) |
| ChatGPT OAuth Client | Outbound LLM Auth | `letta/llm_api/chatgpt_oauth_client.py` | ✅ Standard |

---

## 13. Recommendations Summary

```markdown
Priority 1 (Immediate):
- [ ] Remove localhost-key.pem from repository and rotate the certificate
- [ ] Audit all database backups for plaintext credential exposure

Priority 2 (Short-term):
- [ ] Implement per-client API key management to replace shared server password
- [ ] Add authentication rate limiting middleware
- [ ] Add authentication audit logging

Priority 3 (Medium-term):
- [ ] Implement token revocation mechanism
- [ ] Integrate secrets manager for encryption key management
- [ ] Add server password complexity validation at startup
- [ ] Consider adding MFA for any user-facing authentication flows

Priority 4 (Long-term):
- [ ] Evaluate integrating an identity provider (Auth0, Keycloak) for richer auth capabilities
- [ ] Implement OAuth 2.0 authorization code flow for end-user authentication
- [ ] Add RBAC (Role-Based Access Control) beyond the current org-level isolation
```

--- service_dependencies ---


# External Dependencies Analysis: `letta_a0979209`

## Summary

This is a large Python-based AI agent framework ("Letta") that integrates with numerous LLM providers, databases, cloud services, and monitoring platforms. Below is a comprehensive breakdown of all identified external dependencies.

---

## 1. LLM Provider APIs & SDKs

### 1.1 OpenAI API
- **Dependency Name:** OpenAI API / SDK
- **Type:** Third-party API / Library
- **Purpose/Role:** Primary LLM provider integration for GPT models, including realtime/websocket sessions, chat completions, embeddings, and batch processing.
- **Integration Point/Clues:**
  - `pyproject.toml`: `"openai[realtime]>=2.24.0"`
  - `letta/llm_api/openai_client.py`, `letta/llm_api/openai.py`, `letta/llm_api/openai_ws_session.py`
  - `letta/llm_api/chatgpt_oauth_client.py`
  - `llama-index-embeddings-openai>=0.3.1` in `pyproject.toml`
  - Test configs: `tests/configs/openai.json`, `tests/model_settings/openai-gpt-4o-mini.json`, etc.
  - Environment variables referenced in `.env.example` (e.g., `OPENAI_API_KEY`)

---

### 1.2 Anthropic API / Claude
- **Dependency Name:** Anthropic Claude API
- **Type:** Third-party API / Library
- **Purpose/Role:** Integration with Claude models (Claude 3.x, Claude 4.x) for LLM completions including streaming and parallel tool calls.
- **Integration Point/Clues:**
  - `pyproject.toml`: `"anthropic>=0.75.0"`
  - `letta/llm_api/anthropic_client.py`, `letta/llm_api/anthropic_constants.py`
  - `letta/interfaces/anthropic_streaming_interface.py`, `letta/interfaces/anthropic_parallel_tool_call_streaming_interface.py`
  - Test configs: `tests/model_settings/claude-3-5-sonnet.json`, `tests/model_settings/claude-4-sonnet.json`, etc.

---

### 1.3 Google AI / Gemini API
- **Dependency Name:** Google Generative AI (Gemini) API
- **Type:** Third-party API / Library
- **Purpose/Role:** Integration with Google Gemini models for LLM completions and embeddings.
- **Integration Point/Clues:**
  - `pyproject.toml`: `"google-genai>=1.52.0"`
  - `letta/llm_api/google_ai_client.py`, `letta/llm_api/google_vertex_client.py`, `letta/llm_api/google_constants.py`
  - `letta/interfaces/gemini_streaming_interface.py`
  - `letta/test_gemini.py`
  - Test configs: `tests/model_settings/gemini-2.5-pro.json`, `tests/model_settings/gemini-2.5-flash-vertex.json`
  - `tests/test_google_embeddings.py`, `tests/test_google_schema_refs.py`

---

### 1.4 Google Vertex AI
- **Dependency Name:** Google Vertex AI
- **Type:** Cloud Service (Google Cloud) / Third-party API
- **Purpose/Role:** Integration with Google Cloud's Vertex AI platform for hosted Gemini models.
- **Integration Point/Clues:**
  - `letta/llm_api/google_vertex_client.py`
  - Test configs: `tests/model_settings/gemini-2.5-pro-vertex.json`, `tests/model_settings/gemini-2.5-flash-vertex.json`

---

### 1.5 Mistral AI API
- **Dependency Name:** Mistral AI API
- **Type:** Third-party API / Library
- **Purpose/Role:** Integration with Mistral AI models for LLM completions.
- **Integration Point/Clues:**
  - `pyproject.toml`: `"mistralai>=1.8.1"`
  - `letta/llm_api/mistral.py`
  - Test config: `tests/configs/llm_model_configs/` (various Mistral configs)

---

### 1.6 Azure OpenAI API
- **Dependency Name:** Azure OpenAI API
- **Type:** Cloud Service (Microsoft Azure) / Third-party API
- **Purpose/Role:** Integration with Azure-hosted OpenAI models.
- **Integration Point/Clues:**
  - `letta/llm_api/azure_client.py`
  - Test config: `tests/model_settings/azure-gpt-4o-mini.json`
  - Test config files in `tests/configs/llm_model_configs/`

---

### 1.7 AWS Bedrock
- **Dependency Name:** AWS Bedrock
- **Type:** Cloud Service (AWS) / Third-party API
- **Purpose/Role:** Integration with AWS Bedrock for hosted LLM models (e.g., Claude on Bedrock).
- **Integration Point/Clues:**
  - `pyproject.toml` optional dependency `bedrock`: `"boto3>=1.36.24"`, `"aioboto3>=14.3.0"`
  - `letta/llm_api/bedrock_client.py`
  - Alembic migration: `9556081ce65b_add_bedrock_creds_to_byok.py`
  - Test configs: `tests/model_settings/bedrock-claude-4-5-opus.json`, `tests/model_settings/bedrock-claude-4-sonnet.json`

---

### 1.8 Groq API
- **Dependency Name:** Groq API
- **Type:** Third-party API / Library
- **Purpose/Role:** Integration with Groq's inference platform for fast LLM completions.
- **Integration Point/Clues:**
  - `letta/llm_api/groq_client.py`
  - Test config: `tests/model_settings/groq.json`

---

### 1.9 Together AI API
- **Dependency Name:** Together AI API
- **Type:** Third-party API
- **Purpose/Role:** Integration with Together AI's hosted models (e.g., Qwen).
- **Integration Point/Clues:**
  - `letta/llm_api/together_client.py`
  - Test config: `tests/model_settings/together-qwen-2.5-72b-instruct.json`

---

### 1.10 Fireworks AI API
- **Dependency Name:** Fireworks AI API
- **Type:** Third-party API
- **Purpose/Role:** Integration with Fireworks AI's hosted models.
- **Integration Point/Clues:**
  - `letta/llm_api/fireworks_client.py`
  - Test configs in `tests/configs/llm_model_configs/`

---

### 1.11 DeepSeek API
- **Dependency Name:** DeepSeek API
- **Type:** Third-party API
- **Purpose/Role:** Integration with DeepSeek's LLM models.
- **Integration Point/Clues:**
  - `letta/llm_api/deepseek_client.py`

---

### 1.12 xAI (Grok) API
- **Dependency Name:** xAI API
- **Type:** Third-party API
- **Purpose/Role:** Integration with xAI's Grok models.
- **Integration Point/Clues:**
  - `letta/llm_api/xai_client.py`

---

### 1.13 MiniMax API
- **Dependency Name:** MiniMax API
- **Type:** Third-party API
- **Purpose/Role:** Integration with MiniMax's LLM models.
- **Integration Point/Clues:**
  - `letta/llm_api/minimax_client.py`
  - Test config: `tests/model_settings/minimax-m2.1-lightning.json`
  - `tests/test_minimax_client.py`

---

### 1.14 ZAI (Zhipu AI / GLM) API
- **Dependency Name:** ZAI (Zhipu AI / GLM) API
- **Type:** Third-party API
- **Purpose/Role:** Integration with ZAI's GLM models.
- **Integration Point/Clues:**
  - `letta/llm_api/zai_client.py`
  - Test configs: `tests/model_settings/zai-glm-4.6.json`, `tests/model_settings/zai-glm-5.json`

---

### 1.15 Baseten API
- **Dependency Name:** Baseten API
- **Type:** Third-party API
- **Purpose/Role:** Integration with Baseten's model deployment platform.
- **Integration Point/Clues:**
  - `letta/llm_api/baseten_client.py`

---

### 1.16 Ollama (Local LLM)
- **Dependency Name:** Ollama
- **Type:** External Service (local/self-hosted LLM)
- **Purpose/Role:** Integration with locally-hosted LLM models via Ollama.
- **Integration Point/Clues:**
  - `letta/local_llm/ollama/` directory
  - Test config: `tests/model_settings/ollama.json`
  - GitHub workflow: `.github/workflows/test-ollama.yml`

---

### 1.17 vLLM (Local/Self-hosted LLM)
- **Dependency Name:** vLLM
- **Type:** External Service (local/self-hosted LLM)
- **Purpose/Role:** Integration with vLLM for self-hosted LLM inference.
- **Integration Point/Clues:**
  - `letta/local_llm/vllm/` directory
  - `docker-compose-vllm.yaml`
  - Test config: `tests/model_settings/vllm.json`
  - GitHub workflow: `.github/workflows/test-vllm.yml`

---

### 1.18 LM Studio (Local LLM)
- **Dependency Name:** LM Studio
- **Type:** External Service (local LLM)
- **Purpose/Role:** Integration with LM Studio for local LLM inference.
- **Integration Point/Clues:**
  - `letta/local_llm/lmstudio/` directory
  - Test config: `tests/model_settings/lmstudio.json`
  - GitHub workflow: `.github/workflows/test-lmstudio.yml`

---

### 1.19 SGLang Native
- **Dependency Name:** SGLang
- **Type:** External Service (local/self-hosted LLM)
- **Purpose/Role:** Integration with SGLang's native inference engine.
- **Integration Point/Clues:**
  - `letta/llm_api/sglang_native_client.py`
  - `letta/adapters/sglang_native_adapter.py`

---

## 2. Databases

### 2.1 PostgreSQL (with pgvector)
- **Dependency Name:** PostgreSQL with pgvector
- **Type:** Database (Relational + Vector)
- **Purpose/Role:** Primary relational database for storing agents, messages, memories, tools, and other core data. pgvector extension enables vector similarity search for embeddings.
- **Integration Point/Clues:**
  - `pyproject.toml` postgres extras: `"pgvector>=0.2.3"`, `"psycopg2-binary>=2.9.10"`, `"asyncpg>=0.30.0"`, `"pg8000>=1.30.3"`
  - `Dockerfile`: `FROM pgvector/pgvector:0.8.1-pg15 AS builder`
  - `scripts/docker-compose.yml`: `image: ankane/pgvector`
  - `compose.yaml`, `dev-compose.yaml`: PostgreSQL service definitions
  - `alembic/` directory with extensive migration history
  - `letta/server/db.py`, `letta/database_utils.py`
  - `db/run_postgres.sh`, `init.sql`
  - Environment variables: `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`

---

### 2.2 Redis
- **Dependency Name:** Redis
- **Type:** Database (In-Memory Cache / Message Store)
- **Purpose/Role:** Used for caching, session management, and as a data store for real-time operations.
- **Integration Point/Clues:**
  - `pyproject.toml` redis extra: `"redis>=6.2.0"`
  - `letta/data_sources/redis_client.py`
  - `scripts/docker-compose.yml`: `image: redis:alpine`
  - `Dockerfile`: `apt-get install -y ... redis-server`
  - Exposed port 6379 in `Dockerfile`
  - `tests/test_redis_client.py`

---

### 2.3 SQLite
- **Dependency Name:** SQLite
- **Type:** Database (Embedded Relational)
- **Purpose/Role:** Lightweight database option for development/testing environments.
- **Integration Point/Clues:**
  - `pyproject.toml` sqlite extras: `"aiosqlite>=0.21.0"`, `"sqlite-vec>=0.1.7a2"`
  - `letta/orm/sqlite_functions.py`
  - Alembic migration: `2c059cad97cc_create_sqlite_baseline_schema.py`
  - GitHub workflow: `.github/workflows/core-unit-sqlite-test.yaml`

---

### 2.4 Pinecone
- **Dependency Name:** Pinecone Vector Database
- **Type:** External Service (Managed Vector Database)
- **Purpose/Role:** Vector database for storing and searching embeddings at scale.
- **Integration Point/Clues:**
  - `pyproject.toml` pinecone extra: `"pinecone[asyncio]>=7.3.0"`
  - `letta/helpers/pinecone_utils.py`
  - Alembic migrations referencing `vector_db_provider`

---

### 2.5 Turbopuffer
- **Dependency Name:** Turbopuffer
- **Type:** External Service (Vector Database)
- **Purpose/Role:** Vector database for embeddings and similarity search.
- **Integration Point/Clues:**
  - `pyproject.toml` external-tools: `"turbopuffer>=0.5.17"`
  - `letta/helpers/tpuf_client.py`
  - `tests/integration_test_turbopuffer.py`

---

### 2.6 ClickHouse
- **Dependency Name:** ClickHouse
- **Type:** External Service (Analytical Database)
- **Purpose/Role:** Analytical database for storing and querying LLM traces and provider telemetry data.
- **Integration Point/Clues:**
  - `pyproject.toml`: `"clickhouse-connect>=0.10.0"`
  - `letta/services/clickhouse_otel_traces.py`, `letta/services/clickhouse_provider_traces.py`
  - `otel/otel-collector-config-clickhouse.yaml`, `otel/otel-collector-config-clickhouse-dev.yaml`, `otel/otel-collector-config-clickhouse-prod.yaml`
  - `tests/integration_test_clickhouse_llm_traces.py`

---

## 3. Cloud Services & Infrastructure

### 3.1 AWS SDK (Boto3 / Aioboto3)
- **Dependency Name:** AWS SDK (Boto3)
- **Type:** Cloud Service SDK (AWS)
- **Purpose/Role:** Enables interaction with AWS services including Bedrock for LLM inference and potentially S3 for storage.
- **Integration Point/Clues:**
  - `pyproject.toml` bedrock extra: `"boto3>=1.36.24"`, `"aioboto3>=14.3.0"`
  - `letta/llm_api/bedrock_client.py`

---

### 3.2 Modal
- **Dependency Name:** Modal
- **Type:** Cloud Service (Serverless Compute)
- **Purpose/Role:** Cloud-based sandbox execution environment for running tool code.
- **Integration Point/Clues:**
  - `pyproject.toml` modal extra: `"modal>=1.1.0"`
  - `sandbox/modal_executor.py`
  - `tests/integration_test_modal.py`, `tests/integration_test_modal_sandbox_v2.py`
  - `tests/test_modal_sandbox_v2.py`
  - Alembic migration: `4c6c9ef0387d_support_modal_sandbox_type.py`

---

### 3.3 E2B (Code Interpreter Sandbox)
- **Dependency Name:** E2B Code Interpreter
- **Type:** Cloud Service (Code Execution Sandbox)
- **Purpose/Role:** Cloud-based sandbox for executing tool code safely.
- **Integration Point/Clues:**
  - `pyproject.toml` cloud-tool-sandbox: `"e2b-code-interpreter>=1.0.3"`

---

## 4. Message Protocol & Communication

### 4.1 Model Context Protocol (MCP)
- **Dependency Name:** Model Context Protocol (MCP)
- **Type:** Protocol/Library (Tool Integration)
- **Purpose/Role:** Standard protocol for connecting LLM agents to external tools and data sources. Allows Letta agents to use MCP servers as tool providers.
- **Integration Point/Clues:**
  - `pyproject.toml`: `"mcp[cli]>=1.9.4"`, `"fastmcp>=2.12.5"`
  - `letta/services/mcp/`, `letta/services/mcp_manager.py`, `letta/services/mcp_server_manager.py`
  - `letta/functions/mcp_client/`
  - `letta/orm/mcp_server.py`, `letta/orm/mcp_oauth.py`
  - `tests/integration_test_mcp.py`, `tests/mcp_tests/`, `tests/sdk/mcp_servers_test.py`
  - `WEBHOOK_SETUP.md`

---

### 4.2 gRPC
- **Dependency Name:** gRPC
- **Type:** Communication Protocol / Library
- **Purpose/Role:** High-performance RPC framework, likely for inter-service communication or OpenTelemetry OTLP exporting.
- **Integration Point/Clues:**
  - `pyproject.toml`: `"grpcio>=1.68.1"`, `"grpcio-tools>=1.68.1"`

---

## 5. Monitoring, Observability & Telemetry

### 5.1 OpenTelemetry
- **Dependency Name:** OpenTelemetry
- **Type:** Monitoring/Observability Framework
- **Purpose/Role:** Distributed tracing, metrics collection, and telemetry export for the Letta server.
- **Integration Point/Clues:**
  - `pyproject.toml`: `"opentelemetry-api==1.30.0"`, `"opentelemetry-sdk==1.30.0"`, `"opentelemetry-instrumentation-requests==0.51b0"`, `"opentelemetry-instrumentation-sqlalchemy==0.51b0"`, `"opentelemetry-exporter-otlp==1.30.0"`
  - `letta/otel/` directory: `tracing.py`, `metrics.py`, `events.py`, `context.py`, `resource.py`
  - `otel/` directory with multiple collector configs
  - `Dockerfile`: Installs OpenTelemetry Collector binary (`otelcol-contrib`)
  - Exposed port 4317 (OTLP gRPC) and 4318 (OTLP HTTP) in `Dockerfile`

---

### 5.2 OpenTelemetry Collector
- **Dependency Name:** OpenTelemetry Collector (otelcol-contrib)
- **Type:** Monitoring Infrastructure
- **Purpose/Role:** Receives, processes, and exports telemetry data to various backends (ClickHouse, SigNoz, file).
- **Integration Point/Clues:**
  - `Dockerfile`: Downloads from `https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/v${OTEL_VERSION}/`
  - `otel/` directory: Multiple configuration YAML files for different backends

---

### 5.3 Sentry
- **Dependency Name:** Sentry
- **Type:** Monitoring / Error Tracking Tool
- **Purpose/Role:** Error tracking and performance monitoring for the FastAPI application.
- **Integration Point/Clues:**
  - `pyproject.toml`: `"sentry-sdk[fastapi]==2.19.1"`
  - `letta/server/global_exception_handler.py` (likely uses Sentry)

---

### 5.4 Datadog
- **Dependency Name:** Datadog
- **Type:** Monitoring / APM Tool
- **Purpose/Role:** Application performance monitoring, metrics, and distributed tracing.
- **Integration Point/Clues:**
  - `pyproject.toml`: `"datadog>=0.49.1"`, `"ddtrace>=4.2.1"` (also in `profiling` extra)
  - `pyproject.toml` profiling extra: `"ddtrace>=4.2.1"`

---

### 5.5 SigNoz
- **Dependency Name:** SigNoz
- **Type:** Monitoring / Observability Platform
- **Purpose/Role:** Open-source observability platform for traces and metrics.
- **Integration Point/Clues:**
  - `otel/otel-collector-config-signoz.yaml`

---

## 6. External Tool/Integration Services

### 6.1 Composio
- **Dependency Name:** Composio
- **Type:** Third-party Tool Integration Service
- **Purpose/Role:** Provides pre-built tool integrations (e.g., GitHub starring) for AI agents.
- **Integration Point/Clues:**
  - `letta/functions/composio_helpers.py`, `letta/functions/async_composio_toolset.py`
  - `letta/helpers/composio_helpers.py`
  - `tests/integration_test_composio.py`
  - Alembic migration: `f895232c144a_backfill_composio_tools.py`
  - Test agent file: `tests/test_agent_files/composio_github_star_agent.af`

---

### 6.2 Exa Search API
- **Dependency Name:** Exa Search API
- **Type:** Third-party API (Web Search)
- **Purpose/Role:** Provides AI-native web search capabilities as a tool for agents.
- **Integration Point/Clues:**
  - `pyproject.toml`: `"exa-py>=1.15.4"` (both in core and `external-tools` extra)

---

### 6.3 Tavily Search API
- **Dependency Name:** Tavily Search API
- **Type:** Third-party API (Web Search)
- **Purpose/Role:** Provides web search capabilities optimized for AI agents.
- **Integration Point/Clues:**
  - `pyproject.toml`: `"tavily-python>=0.7.2"`

---

### 6.4 Wikipedia API
- **Dependency Name:** Wikipedia
- **Type:** Third-party Library/API
- **Purpose/Role:** Enables agents to query Wikipedia for information.
- **Integration Point/Clues:**
  - `pyproject.toml` external-tools and desktop extras: `"wikipedia>=1.4.0"`

---

### 6.5 Langchain / Langchain-

--- APIs ---


I'll analyze the codebase systematically, focusing on the REST API routers and server files to document all HTTP API endpoints.

Let me examine the key files:

- `letta/server/rest_api/routers/` - Main router directory
- `letta/schemas/` - Request/response schemas
- `fern/openapi.json` - OpenAPI specification

Based on my analysis of the router files, schemas, and OpenAPI specification, here is the complete API documentation:

---

# Letta REST API Documentation

## Table of Contents
1. [Agents](#1-agents)
2. [Messages](#2-messages)
3. [Memory & Blocks](#3-memory--blocks)
4. [Archival Memory](#4-archival-memory)
5. [Tools](#5-tools)
6. [Sources & Files](#6-sources--files)
7. [Groups (Multi-Agent)](#7-groups-multi-agent)
8. [Identities](#8-identities)
9. [Models](#9-models)
10. [Providers](#10-providers)
11. [Jobs](#11-jobs)
12. [Runs & Steps](#12-runs--steps)
13. [Sandbox Configuration](#13-sandbox-configuration)
14. [Conversations](#14-conversations)
15. [Health](#15-health)
16. [Organizations](#16-organizations)
17. [MCP Servers](#17-mcp-servers)
18. [Voice](#18-voice)
19. [Batch Jobs (LLM)](#19-batch-jobs-llm)
20. [OpenAI-Compatible Endpoints](#20-openai-compatible-endpoints)
21. [Prompts](#21-prompts)

---

## 1. Agents

### Create Agent
**HTTP Method:** `POST`

**API URL:** `/v1/agents`

**What it does:** Creates a new agent with specified configuration including LLM model, memory blocks, tools, and persona settings.

**Request Payload:**
```json
{
  "name": "my_agent",
  "description": "A helpful assistant",
  "agent_type": "memgpt_agent",
  "llm_config": {
    "model": "gpt-4o",
    "model_endpoint_type": "openai",
    "model_endpoint": "https://api.openai.com/v1",
    "context_window": 128000,
    "temperature": 0.7,
    "max_tokens": 1024
  },
  "embedding_config": {
    "embedding_model": "text-embedding-ada-002",
    "embedding_endpoint_type": "openai",
    "embedding_endpoint": "https://api.openai.com/v1",
    "embedding_dim": 1536,
    "embedding_chunk_size": 300
  },
  "memory_blocks": [
    {
      "label": "human",
      "value": "Name: John",
      "limit": 5000
    },
    {
      "label": "persona",
      "value": "I am a helpful assistant.",
      "limit": 5000
    }
  ],
  "tools": ["tool_id_1"],
  "tool_ids": ["tool_id_1"],
  "tool_rules": [],
  "tags": ["production"],
  "system": "You are a helpful assistant.",
  "include_base_tools": true,
  "initial_message_sequence": null,
  "llm_config_overrides": {},
  "embedding_config_overrides": {},
  "project_id": null,
  "template_id": null,
  "identity_ids": [],
  "context_window_limit": null,
  "response_format": null,
  "enable_sleeptime": false,
  "timezone": "UTC"
}
```

**Response Payload:**
```json
{
  "id": "agent-123e4567-e89b-12d3-a456-426614174000",
  "name": "my_agent",
  "description": "A helpful assistant",
  "agent_type": "memgpt_agent",
  "llm_config": {
    "model": "gpt-4o",
    "model_endpoint_type": "openai",
    "model_endpoint": "https://api.openai.com/v1",
    "context_window": 128000
  },
  "embedding_config": {
    "embedding_model": "text-embedding-ada-002",
    "embedding_dim": 1536
  },
  "memory": {
    "blocks": [
      {
        "id": "block-abc123",
        "label": "human",
        "value": "Name: John",
        "limit": 5000
      }
    ]
  },
  "tools": [],
  "tool_rules": [],
  "tags": ["production"],
  "system": "You are a helpful assistant.",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z",
  "created_by_id": "user-abc123",
  "last_updated_by_id": "user-abc123",
  "organization_id": "org-abc123",
  "metadata_": {}
}
```

---

### List Agents
**HTTP Method:** `GET`

**API URL:** `/v1/agents`

**What it does:** Returns a paginated list of agents. Supports filtering by name, tags, identities, and other criteria.

**Query Parameters:**
- `name` (string, optional) — Filter by agent name
- `tags` (array, optional) — Filter by tags
- `match_all_tags` (boolean, optional) — Require all tags to match
- `before` (string, optional) — Cursor for pagination (agent ID)
- `after` (string, optional) — Cursor for pagination (agent ID)
- `limit` (integer, optional, default: 50) — Max results to return
- `query_text` (string, optional) — Full-text search
- `project_id` (string, optional) — Filter by project
- `template_id` (string, optional) — Filter by template
- `identity_id` (string, optional) — Filter by identity
- `identifier_keys` (array, optional) — Filter by identifier keys
- `include_relationships` (array, optional) — Relationships to include
- `sort_desc` (boolean, optional) — Sort descending by creation date

**Request Payload:** `N/A`

**Response Payload:**
```json
[
  {
    "id": "agent-123e4567",
    "name": "my_agent",
    "description": "A helpful assistant",
    "agent_type": "memgpt_agent",
    "llm_config": { "model": "gpt-4o" },
    "embedding_config": {},
    "memory": {},
    "tools": [],
    "tags": [],
    "created_at": "2024-01-01T00:00:00Z"
  }
]
```

---

### Get Agent
**HTTP Method:** `GET`

**API URL:** `/v1/agents/{agent_id}`

**What it does:** Retrieves a specific agent by its ID.

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "agent-123e4567",
  "name": "my_agent",
  "description": "A helpful assistant",
  "agent_type": "memgpt_agent",
  "llm_config": {
    "model": "gpt-4o",
    "model_endpoint_type": "openai"
  },
  "embedding_config": {},
  "memory": {
    "blocks": []
  },
  "tools": [],
  "tags": [],
  "system": "You are a helpful assistant.",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z"
}
```

---

### Update Agent
**HTTP Method:** `PATCH`

**API URL:** `/v1/agents/{agent_id}`

**What it does:** Partially updates an existing agent's configuration.

**Request Payload:**
```json
{
  "name": "updated_agent_name",
  "description": "Updated description",
  "system": "New system prompt",
  "llm_config": {
    "model": "gpt-4o-mini"
  },
  "tags": ["updated-tag"],
  "tool_ids": ["tool_id_1", "tool_id_2"],
  "block_ids": ["block-abc123"],
  "memory_blocks": [],
  "tool_rules": [],
  "context_window_limit": 50000,
  "response_format": null,
  "message_buffer_autoclear": false,
  "enable_sleeptime": true
}
```

**Response Payload:**
```json
{
  "id": "agent-123e4567",
  "name": "updated_agent_name",
  "description": "Updated description",
  "agent_type": "memgpt_agent",
  "llm_config": { "model": "gpt-4o-mini" },
  "tags": ["updated-tag"],
  "updated_at": "2024-01-02T00:00:00Z"
}
```

---

### Delete Agent
**HTTP Method:** `DELETE`

**API URL:** `/v1/agents/{agent_id}`

**What it does:** Permanently deletes an agent and all associated data.

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "deleted": true,
  "id": "agent-123e4567"
}
```

---

### Export Agent Serialized
**HTTP Method:** `GET`

**API URL:** `/v1/agents/{agent_id}/export`

**What it does:** Exports the full agent state as a serialized file (`.af` format) for backup or migration.

**Request Payload:** `N/A`

**Response Payload:** Binary file download (`.af` archive format)

---

### Import Agent Serialized
**HTTP Method:** `POST`

**API URL:** `/v1/agents/import`

**What it does:** Imports an agent from a serialized `.af` file.

**Request Payload:** `multipart/form-data`
```
file: <binary .af file>
append_copy_suffix: false
override_existing_tools: false
project_id: "proj-abc123"
```

**Response Payload:**
```json
{
  "id": "agent-new-123",
  "name": "imported_agent",
  "agent_type": "memgpt_agent",
  "created_at": "2024-01-01T00:00:00Z"
}
```

---

### Get Agent Context Window
**HTTP Method:** `GET`

**API URL:** `/v1/agents/{agent_id}/context`

**What it does:** Returns the current context window state of the agent including system prompt, messages, and memory.

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "context_window_size_max": 128000,
  "context_window_size_current": 1500,
  "num_messages": 10,
  "num_archival_memory": 5,
  "system_prompt": "You are a helpful assistant.",
  "memory": {},
  "messages": []
}
```

---

### Get Agent Memory
**HTTP Method:** `GET`

**API URL:** `/v1/agents/{agent_id}/memory`

**What it does:** Returns the in-context memory (blocks) of the agent.

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "blocks": [
    {
      "id": "block-abc123",
      "label": "human",
      "value": "Name: John",
      "limit": 5000,
      "created_at": "2024-01-01T00:00:00Z"
    }
  ]
}
```

---

### Update Agent Memory
**HTTP Method:** `PATCH`

**API URL:** `/v1/agents/{agent_id}/memory`

**What it does:** Updates the content of the agent's in-context memory blocks.

**Request Payload:**
```json
{
  "human": "Name: John, Occupation: Engineer",
  "persona": "I am a specialized coding assistant."
}
```

**Response Payload:**
```json
{
  "blocks": [
    {
      "id": "block-abc123",
      "label": "human",
      "value": "Name: John, Occupation: Engineer",
      "limit": 5000
    }
  ]
}
```

---

### Get Agent Memory Block
**HTTP Method:** `GET`

**API URL:** `/v1/agents/{agent_id}/memory/blocks/{block_label}`

**What it does:** Retrieves a specific memory block by its label from the agent.

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "block-abc123",
  "label": "human",
  "value": "Name: John",
  "limit": 5000,
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z"
}
```

---

### Update Agent Memory Block
**HTTP Method:** `PATCH`

**API URL:** `/v1/agents/{agent_id}/memory/blocks/{block_label}`

**What it does:** Updates the content of a specific memory block by label.

**Request Payload:**
```json
{
  "value": "Name: John, Occupation: Engineer, Location: NYC"
}
```

**Response Payload:**
```json
{
  "id": "block-abc123",
  "label": "human",
  "value": "Name: John, Occupation: Engineer, Location: NYC",
  "limit": 5000,
  "updated_at": "2024-01-02T00:00:00Z"
}
```

---

### Attach Block to Agent
**HTTP Method:** `PATCH`

**API URL:** `/v1/agents/{agent_id}/memory/blocks/attach/{block_id}`

**What it does:** Attaches an existing memory block to the agent's context memory.

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "blocks": [
    {
      "id": "block-abc123",
      "label": "custom_block",
      "value": "Some content"
    }
  ]
}
```

---

### Detach Block from Agent
**HTTP Method:** `PATCH`

**API URL:** `/v1/agents/{agent_id}/memory/blocks/detach/{block_id}`

**What it does:** Detaches a memory block from the agent's context memory.

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "blocks": []
}
```

---

### Get Agent Tools
**HTTP Method:** `GET`

**API URL:** `/v1/agents/{agent_id}/tools`

**What it does:** Returns all tools currently attached to an agent.

**Request Payload:** `N/A`

**Response Payload:**
```json
[
  {
    "id": "tool-abc123",
    "name": "search_web",
    "description": "Search the web for information",
    "json_schema": {},
    "tags": [],
    "created_at": "2024-01-01T00:00:00Z"
  }
]
```

---

### Attach Tool to Agent
**HTTP Method:** `PATCH`

**API URL:** `/v1/agents/{agent_id}/tools/attach/{tool_id}`

**What it does:** Attaches a tool to an agent, making it available for the agent to call.

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "agent-123e4567",
  "tools": [
    {
      "id": "tool-abc123",
      "name": "search_web"
    }
  ]
}
```

---

### Detach Tool from Agent
**HTTP Method:** `PATCH`

**API URL:** `/v1/agents/{agent_id}/tools/detach/{tool_id}`

**What it does:** Removes a tool from an agent.

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "agent-123e4567",
  "tools": []
}
```

---

### Get Agent Sources
**HTTP Method:** `GET`

**API URL:** `/v1/agents/{agent_id}/sources`

**What it does:** Lists all data sources attached to the agent.

**Request Payload:** `N/A`

**Response Payload:**
```json
[
  {
    "id": "source-abc123",
    "name": "my_docs",
    "description": "Company documentation",
    "embedding_config": {},
    "created_at": "2024-01-01T00:00:00Z"
  }
]
```

---

### Attach Source to Agent
**HTTP Method:** `PATCH`

**API URL:** `/v1/agents/{agent_id}/sources/attach/{source_id}`

**What it does:** Attaches a data source to an agent, enabling archival memory search over its documents.

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "agent-123e4567",
  "sources": ["source-abc123"]
}
```

---

### Detach Source from Agent
**HTTP Method:** `PATCH`

**API URL:** `/v1/agents/{agent_id}/sources/detach/{source_id}`

**What it does:** Removes a data source from an agent.

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "agent-123e4567",
  "sources": []
}
```

---

### Get Agent Identities
**HTTP Method:** `GET`

**API URL:** `/v1/agents/{agent_id}/identities`

**What it does:** Returns all identities linked to an agent.

**Request Payload:** `N/A`

**Response Payload:**
```json
[
  {
    "id": "identity-abc123",
    "identifier_key": "user_123",
    "identity_type": "user",
    "name": "John Doe",
    "created_at": "2024-01-01T00:00:00Z"
  }
]
```

---

### Attach Identity to Agent
**HTTP Method:** `PATCH`

**API URL:** `/v1/agents/{agent_id}/identities/attach/{identity_id}`

**What it does:** Associates an identity with an agent.

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "agent-123e4567",
  "identity_ids": ["identity-abc123"]
}
```

---

### Detach Identity from Agent
**HTTP Method:** `PATCH`

**API URL:** `/v1/agents/{agent_id}/identities/detach/{identity_id}`

**What it does:** Removes an identity association from an agent.

**Request Payload:** `N/A`

**Response Payload:**
```json
{
  "id": "agent-123e4567",
  "identity_ids": []
}
```

---

### Migrate Agent
**HTTP Method:** `POST`

**API URL:** `/v1/agents/{agent_id}/migrate`

**What it does:** Migrates an agent to a new model or configuration.

**Request Payload:**
```json
{
  "to_llm_config": {
    "model": "gpt-4o",
    "model_endpoint_type": "openai"
  },
  "to_embedding_config": {
    "embedding_model": "text-embedding-ada-002"
  },
  "preserve_core_memories": true
}
```

**Response Payload:**
```json
{
  "id": "agent-123e4567",
  "llm_config": { "model": "gpt-4o" },
  "updated_at": "2024-01-02T00:00:00Z"
}
```

---

### Get Agent Versions
**HTTP Method:** `GET`

**API URL:** `/v1/agents/{agent_id}/versions`

**What it does:** Returns the version history of a block attached to the agent (block history).

**Request Payload:** `N/A`

**Response Payload:**
```json
[
  {
    "id": "agent-version-abc123",
    "agent_id": "agent-123e4567",
    "snapshot": {},
    "created_at": "2024-01-01T00:00:00Z"
  }
]
```

---

### Search Agents
**HTTP Method:** `POST`

**API URL:** `/v1/agents/search`

**What it does:** Advanced search for agents with multiple filter criteria.

**Request Payload:**
```json
{
  "query_text": "customer service",
  "tags": ["production"],
  "match_all_tags": false,
  "before": null,
  "after": null,
  "limit": 10
}
```

**Response Payload:**
```json
[
  {
    "id": "agent-123e4567",
    "name": "customer_service_agent",
    "tags": ["production"]
  }
]
```

---

## 2. Messages

### Send Message to Agent
**HTTP Method:** `POST`

**API URL:** `/v1/agents/{agent_id}/messages`

**What it does:** Sends a message to an agent and returns the agent's response. Supports streaming via SSE.

**Request Payload:**
```json
{
  "messages": [
    {
      "role": "user",
      "content": "Hello! How are you?"
    }
  ],
  "stream_steps": false,
  "stream_tokens": false,
  "return_message_object": false,
  "run_async": false,
  "assistant_message_tool_name": "send_message",
  "assistant_message_tool_kwarg": "message",
  "max_steps": null,
  "use_assistant_message": true
}
```

**Response Payload (non-streaming):**
```json
{
  "messages": [
    {
      "id": "message-abc123",
      "role": "assistant",
      "content": [
        {
          "type": "text",
          "text": "Hello! I'm doing great. How can I help you?"
        }
      ],
      "created_at": "2024-01-01T00:00:00Z"
    }
  ],
  "usage": {
    "completion_tokens": 20,
    "prompt_tokens": 500,
    "total_tokens": 520,
    "step_count": 1
  },
  "stop_reason": "end_turn"
}
```

**Response Payload (streaming, SSE):** Stream of `LettaMessage` events followed by a final `usage` event.

---

### List Agent Messages
**HTTP Method:** `GET`

**API URL:** `/v1/agents/{agent_id}/messages`

**What it does:** Retrieves the message history for an agent, with support for pagination and filtering.

**Query Parameters:**
- `before` (string, optional) — Cursor (message ID) for paginating backwards
- `after` (string, optional) — Cursor (message ID) for paginating forwards
- `limit` (integer, optional, default: 10) — Number of messages to return
- `msg_object` (boolean, optional) — Return full message objects
- `assistant_message_tool_name` (string, optional)
- `assistant_message_tool_kwarg` (string, optional)
- `use_assistant_message` (boolean, optional, default: true)
- `role` (string, optional) — Filter by role: `user`, `assistant`, `system`, `tool`

**Request Payload:** `N/A`

**Response Payload:**
```json

--- dependencies ---


# Dependency and Architecture Analysis: Letta

---

## Internal Modules

The following core internal packages are defined under the `letta/` source directory and are reused across multiple parts of the project.

---

### `letta/agents/`
**Agent Runtime Engine** — Contains the core execution logic for all agent types. Implements the primary stateful agent loop (`letta_agent.py`), versioned agent variants (`v2`, `v3`), batch processing agents, voice agents, ephemeral (stateless) agents, and the abstract base class defining the agent interface.

---

### `letta/services/`
**Business Logic / Service Layer** — The central manager layer implementing domain-specific business logic. Includes managers for every major domain entity: agents, messages, tools, memory blocks, sources, files, groups, jobs, providers, sandbox configurations, MCP servers, and more. Also contains sub-modules for tool execution (`tool_executor/`, `tool_sandbox/`), memory summarization (`summarizer/`), file ingestion (`file_processor/`), MCP protocol integration (`mcp/`), streaming (`streaming_service.py`), and LLM routing (`llm_router/`).

---

### `letta/server/`
**REST & WebSocket API Layer** — The FastAPI application host. Contains the main server setup (`server.py`), domain-specific REST route handlers (`rest_api/routers/`), WebSocket API endpoints (`ws_api/`), authentication middleware (`rest_api/auth/`), request middleware, and global exception handling.

---

### `letta/orm/`
**Data Access Layer (ORM Models)** — SQLAlchemy ORM model definitions mapping every domain entity to its database table (agents, messages, blocks, tools, sources, groups, jobs, providers, MCP servers, sandbox configs, etc.). Includes association/join tables, custom column types, and SQLite-specific compatibility functions.

---

### `letta/schemas/`
**Domain Data Models (Pydantic Schemas)** — Pydantic-based data transfer objects and API contract definitions for all domain entities. Serves as the canonical type layer between the API, service, and ORM layers. Includes provider-specific sub-schemas (`providers/`) and OpenAI-compatible schemas (`openai/`).

---

### `letta/llm_api/`
**LLM Provider Abstraction Layer** — Provider-specific client implementations behind a unified abstract interface (`llm_client_base.py`). Covers OpenAI, Anthropic, Google AI/Vertex, AWS Bedrock, Azure, Groq, Mistral, DeepSeek, Fireworks, Together AI, MiniMax, xAI, Baseten, SGLang, and others. Implements the Strategy pattern for pluggable LLM backends.

---

### `letta/groups/`
**Multi-Agent Orchestration** — Implements coordination patterns for multi-agent workflows including round-robin, supervisor-based, sleeptime (background agent), dynamic, and versioned sleeptime variants. Enables agent-to-agent delegation and parallel execution.

---

### `letta/otel/`
**Observability & Instrumentation** — OpenTelemetry integration layer providing distributed tracing (`tracing.py`), custom metrics registry (`metric_registry.py`, `metrics.py`), SQLAlchemy query instrumentation, database connection pool monitoring, and structured event emission.

---

### `letta/functions/`
**Tool & Function Management** — Handles registration, parsing, schema generation, and validation of agent tools (functions). Includes built-in tool collections (`function_sets/`), AST-based Python parsers, TypeScript tool parsing, MCP client integration (`mcp_client/`), and Composio tool support.

---

### `letta/prompts/`
**System Prompt Management** — Manages system prompt templates and generation logic. Includes prompt templates for agent personas, summarization requests, and system prompt files (`system_prompts/`).

---

### `letta/local_llm/`
**Local LLM Integration** — Handles integration with locally-hosted LLM servers (Ollama, vLLM, LM Studio, llama.cpp, KoboldCPP, WebUI). Provides chat completion wrappers, JSON/function parsing utilities, and grammar files for constrained decoding.

---

### `letta/interfaces/`
**LLM Streaming Interfaces** — Implements streaming response parsers for each provider's specific streaming protocol (OpenAI SSE, Anthropic streaming, Gemini streaming, parallel tool call streaming). Converts raw provider stream events into internal Letta message types.

---

### `letta/adapters/`
**Request/Response Adapters** — Translates between internal Letta request/response formats and provider-specific wire formats. Includes adapters for standard LLM requests, streaming responses, and SGLang native protocol.

---

### `letta/serialize_schemas/`
**Marshmallow Serialization Layer** — Marshmallow-based serialization schemas for ORM model serialization/deserialization (agents, blocks, messages, tools, tags). Used for agent state export/import (`.af` archive files).

---

### `letta/helpers/`
**Shared Utility Functions** — Cross-cutting utilities used throughout the codebase: JSON helpers, datetime formatting, cryptographic utilities, tool execution helpers, tool rule solvers, message helpers, Composio integration helpers, Pinecone utilities, and decorator utilities.

---

### `letta/jobs/`
**Background Job Scheduling** — Manages asynchronous and scheduled background jobs including LLM batch job polling, APScheduler-based task scheduling, and job type definitions.

---

### `letta/data_sources/`
**External Data Connectors** — Provides connectors for ingesting data from external sources into agent archival memory. Includes a Redis client for data source operations.

---

### `letta/monitoring/`
**Runtime Health & Readiness Monitoring** — Event loop watchdog for detecting hangs, readiness state tracking, and load gating for controlled server startup and health checks.

---

### `letta/plugins/`
**Plugin System** — Extensible plugin architecture allowing behavioral customization of the Letta server. Includes plugin registry, default plugin definitions, and the plugin loading mechanism.

---

### `letta/cli/`
**Command-Line Interface** — Typer-based CLI providing commands for running interactive agent sessions, starting the server, and loading data sources.

---

### `letta/model_specs/`
**LLM Model Specifications** — Stores LiteLLM-sourced model capability metadata (context windows, pricing, supported features) used for token budget calculations and model routing decisions.

---

---

## External Dependencies

The following dependencies are derived exclusively from the provided dependency files.

---

### Python Dependencies
**Source:** `/pyproject.toml`

| Official Name | Package | Primary Role |
|---|---|---|
| **Typer** | `typer` | CLI framework for building the `letta` command-line interface |
| **Questionary** | `questionary` | Interactive terminal prompts for CLI user input |
| **pytz** | `pytz` | Timezone handling and datetime localization |
| **tqdm** | `tqdm` | Progress bar display for long-running operations |
| **Black** | `black[jupyter]` | Python code formatter (used in tool schema generation and dev tooling) |
| **setuptools** | `setuptools` | Python packaging and build utilities |
| **PrettyTable** | `prettytable` | Tabular data formatting for CLI output |
| **docstring-parser** | `docstring-parser` | Parses Python docstrings to extract tool/function schemas |
| **HTTPX** | `httpx` | Async-capable HTTP client for outbound API requests |
| **NumPy** | `numpy` | Numerical operations, primarily for embedding vector handling |
| **demjson3** | `demjson3` | Lenient/optimistic JSON parser for handling malformed LLM JSON output |
| **PyYAML** | `pyyaml` | YAML configuration file parsing |
| **sqlalchemy-json** | `sqlalchemy-json` | JSON column type extension for SQLAlchemy |
| **Pydantic** | `pydantic` | Data validation and schema definition for all domain models |
| **html2text** | `html2text` | Converts HTML content to Markdown text |
| **SQLAlchemy** | `sqlalchemy[asyncio]` | ORM and database abstraction layer (sync and async) |
| **python-box** | `python-box` | Dot-notation dict access for configuration objects |
| **SQLModel** | `sqlmodel` | Pydantic + SQLAlchemy integration layer |
| **python-multipart** | `python-multipart` | Multipart form data parsing (file upload support in FastAPI) |
| **sqlalchemy-utils** | `sqlalchemy-utils` | Additional SQLAlchemy column types and utility functions |
| **pydantic-settings** | `pydantic-settings` | Settings/configuration management via Pydantic models |
| **httpx-sse** | `httpx-sse` | Server-Sent Events (SSE) streaming support over HTTPX |
| **NLTK** | `nltk` | Natural language processing (tokenization, text chunking) |
| **Alembic** | `alembic` | Database schema migration management |
| **pyhumps** | `pyhumps` | Case conversion (camelCase ↔ snake_case) for API serialization |
| **pathvalidate** | `pathvalidate` | File/path name validation and sanitization |
| **Sentry** | `sentry-sdk[fastapi]` | Error tracking and application monitoring |
| **Rich** | `rich` | Rich terminal output formatting for CLI and logging |
| **Brotli** | `brotli` | Brotli compression support for HTTP responses |
| **gRPC** | `grpcio` | gRPC protocol support (used with OpenTelemetry OTLP exporter) |
| **gRPC Tools** | `grpcio-tools` | gRPC code generation utilities |
| **LlamaIndex** | `llama-index` | Document indexing and retrieval framework for archival memory |
| **LlamaIndex OpenAI Embeddings** | `llama-index-embeddings-openai` | OpenAI embedding model integration for LlamaIndex |
| **Anthropic** | `anthropic` | Anthropic Claude API client |
| **Letta Client** | `letta-client` | Official Letta SDK client (used internally for SDK-level operations) |
| **OpenAI** | `openai[realtime]` | OpenAI API client including Realtime API (WebSocket voice) support |
| **OpenTelemetry API** | `opentelemetry-api` | OpenTelemetry instrumentation API |
| **OpenTelemetry SDK** | `opentelemetry-sdk` | OpenTelemetry SDK for trace/metric collection |
| **OpenTelemetry Requests Instrumentation** | `opentelemetry-instrumentation-requests` | Auto-instrumentation for HTTP requests tracing |
| **OpenTelemetry SQLAlchemy Instrumentation** | `opentelemetry-instrumentation-sqlalchemy` | Auto-instrumentation for SQLAlchemy database query tracing |
| **OpenTelemetry OTLP Exporter** | `opentelemetry-exporter-otlp` | Exports telemetry data to OpenTelemetry Collector via OTLP |
| **Faker** | `faker` | Synthetic test data generation |
| **Colorama** | `colorama` | Cross-platform terminal color output |
| **marshmallow-sqlalchemy** | `marshmallow-sqlalchemy` | Marshmallow schema generation from SQLAlchemy models (agent serialization) |
| **datamodel-code-generator** | `datamodel-code-generator[http]` | Generates Pydantic models from JSON Schema/OpenAPI specs |
| **MCP** | `mcp[cli]` | Model Context Protocol SDK for tool server integration |
| **Exa** | `exa-py` | Exa neural search API client (built-in agent web search tool) |
| **APScheduler** | `apscheduler` | In-process background job scheduling (batch job polling) |
| **aiomultiprocess** | `aiomultiprocess` | Async multiprocessing for parallel tool execution |
| **Matplotlib** | `matplotlib` | Plotting and data visualization (available as an agent tool) |
| **Tavily** | `tavily-python` | Tavily web search API client (built-in agent search tool) |
| **Temporal** | `temporalio` | Durable workflow orchestration (background job execution) |
| **Mistral** | `mistralai` | Mistral AI API client |
| **structlog** | `structlog` | Structured logging framework |
| **certifi** | `certifi` | Mozilla CA certificate bundle for TLS verification |
| **MarkItDown** | `markitdown[docx,pdf,pptx]` | Converts Office documents and PDFs to Markdown (file ingestion) |
| **orjson** | `orjson` | High-performance JSON serialization/deserialization |
| **Ruff** | `ruff` | Fast Python linter and formatter |
| **ty** | `ty` | Experimental Python type checker |
| **trafilatura** | `trafilatura` | Web page content extraction and scraping |
| **readability-lxml** | `readability-lxml` | Extracts readable content from HTML pages |
| **Google GenAI** | `google-genai` | Google Generative AI (Gemini) API client |
| **Datadog** | `datadog` | Datadog monitoring and metrics client |
| **psutil** | `psutil` | System and process resource monitoring |
| **FastMCP** | `fastmcp` | High-level MCP server framework for creating tool servers |
| **ddtrace** | `ddtrace` | Datadog APM distributed tracing client |
| **clickhouse-connect** | `clickhouse-connect` | ClickHouse database client (provider trace storage backend) |
| **aiofiles** | `aiofiles` | Async file I/O operations |
| **async-lru** | `async-lru` | Async-compatible LRU cache decorator |
| **pgvector** | `pgvector` | PostgreSQL pgvector extension client for vector similarity search |
| **pg8000** | `pg8000` | Pure-Python PostgreSQL driver |
| **psycopg2** | `psycopg2-binary` / `psycopg2` | PostgreSQL database adapter (sync) |
| **asyncpg** | `asyncpg` | High-performance async PostgreSQL driver |
| **Redis** | `redis` | Redis client for caching and pub/sub messaging |
| **Pinecone** | `pinecone[asyncio]` | Pinecone vector database client (optional archival memory backend) |
| **aiosqlite** | `aiosqlite` | Async SQLite driver |
| **sqlite-vec** | `sqlite-vec` | Vector similarity search extension for SQLite |
| **uvloop** | `uvloop` | High-performance async event loop replacement |
| **Granian** | `granian[uvloop,reload]` | High-performance ASGI/WSGI server (experimental alternative to uvicorn) |
| **WebSockets** | `websockets` | WebSocket protocol support |
| **FastAPI** | `fastapi` | Web framework for building the REST API server |
| **Uvicorn** | `uvicorn` | ASGI server for serving the FastAPI application |
| **Boto3** | `boto3` | AWS SDK (used for AWS Bedrock LLM access) |
| **aioboto3** | `aioboto3` | Async AWS SDK (async Bedrock client) |
| **pytest** | `pytest` | Primary test framework |
| **pytest-asyncio** | `pytest-asyncio` | Async test support for pytest |
| **pytest-order** | `pytest-order` | Test execution ordering for pytest |
| **pytest-mock** | `pytest-mock` | Mock/patch utilities for pytest |
| **pytest-json-report** | `pytest-json-report` | JSON output format for pytest results |
| **pexpect** | `pexpect` | CLI interaction testing (spawning and controlling subprocesses) |
| **pre-commit** | `pre-commit` | Git pre-commit hook management |
| **Pyright** | `pyright` | Static type checker for Python |
| **ipykernel** | `ipykernel` | Jupyter kernel support for notebook development |
| **ipdb** | `ipdb` | IPython-enhanced Python debugger |
| **E2B Code Interpreter** | `e2b-code-interpreter` | E2B cloud sandbox for secure remote tool code execution |
| **Modal** | `modal` | Modal Labs cloud compute platform for sandboxed tool execution |
| **Docker SDK** | `docker` | Docker API client for local container-based tool sandboxes |
| **LangChain** | `langchain` | LLM application framework (external tool integrations) |
| **Wikipedia** | `wikipedia` | Wikipedia API client (built-in agent search tool) |
| **LangChain Community** | `langchain-community` | Community integrations for LangChain tools |
| **TurboPuffer** | `turbopuffer` | TurboPuffer vector database client (optional archival memory backend) |
| **Locust** | `locust` | Load testing framework |
| **tiktoken** | `tiktoken` | OpenAI tokenizer for token counting |
| **Magika** | `magika` | Google's ML-based file type detection |

---

### JavaScript Dependencies
**Source:** `/sandbox/resources/server/package.json`

| Official Name | Package | Primary Role |
|---|---|---|
| **TypeScript** | `typescript` | TypeScript compiler used in the Node.js sandbox server (`sandbox/`) for TypeScript tool execution support |
| **Node.js Types** | `@types/node` | TypeScript type definitions for Node.js built-in APIs, used in the sandbox server |

---

### Infrastructure Dependencies
**Source:** `/Dockerfile`, `/scripts/docker-compose.yml`

| Official Name | Source Reference | Primary Role |
|---|---|---|
| **pgvector** (Docker image) | `pgvector/pgvector:0.8.1-pg15` (`/Dockerfile`) | Base Docker image providing PostgreSQL 15 with the pgvector extension pre-installed |
| **Redis** | `redis:alpine` (`/scripts/docker-compose.yml`) | In-memory data store for caching and pub/sub messaging, deployed as a sidecar container |
| **PostgreSQL with pgvector** | `ankane/pgvector` (`/scripts/docker-compose.yml`) | PostgreSQL database with vector similarity search extension, deployed as a sidecar container |
| **Node.js** | `NODE_VERSION=22` (`/Dockerfile`) | JavaScript runtime installed in the production Docker image to support TypeScript/JavaScript tool execution in sandboxes |
| **OpenTelemetry Collector** | `OTEL_VERSION=0.96.0` (`/Dockerfile`) | Standalone telemetry collector binary (`otelcol-contrib`) installed in the Docker image to receive, process, and export traces/metrics |
| **uv** | `ghcr.io/astral-sh/uv:latest` (`/Dockerfile`) | Fast Python package manager used to install and lock Python dependencies during the Docker build |

---

### Test-Specific Dependencies
**Source:** `/tests/mcp_tests/weather/requirements.txt`, `/tests/test_tool_sandbox/restaurant_management_system/requirements.txt`

| Official Name | Package | Primary Role |
|---|---|---|
| **MCP** | `mcp==1.7.1` | Model Context Protocol SDK used in the weather MCP test server fixture |
| **Uvicorn** | `uvicorn==0.34.2` | ASGI server used to run the test MCP server |
| **Pydantic** | `pydantic==2.11.4` | Data validation within the test MCP server |
| **pydantic-settings** | `pydantic-settings==2.9.1` | Settings management in the test MCP server |
| **Typer** | `typer==0.15.3` | CLI framework used by the test MCP server |
| **Rich** | `rich==14.0.0` | Terminal output formatting in the test MCP server |
| **HTTPX** | `httpx==0.28.1` | HTTP client within the test MCP server |
| **httpx-sse** | `httpx-sse==0.4.0` | SSE streaming support in the test MCP server |
| **python-dotenv** | `python-dotenv==1.1.0` | `.env` file loading in the test MCP server |
| **starlette** | `starlette==0.46.2` | ASGI web framework underlying the test MCP server |
| **sse-starlette** | `sse-starlette==2.3.3` | SSE response support for the Starlette-based test MCP server |
| **anyio** | `anyio==4.9.0` | Async concurrency library used by the test MCP server |
| **cowsay** | `cowsay` | ASCII art "cowsay" output used as a trivial test tool in the restaurant management sandbox test fixture |

--- core_entities ---


# Letta Project: Common Data Entities & Domain Models

## Overview

Letta is an AI agent framework with persistent memory, multi-agent orchestration, tool execution, and LLM provider management. The following entities form the core domain model, derived from the ORM layer (`letta/orm/`), schemas (`letta/schemas/`), and Alembic migrations.

---

## 1. Core Entities

---

### 1.1 `Agent` (AgentState)

The central entity — represents a stateful AI agent instance.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `name` | str | Agent display name |
| `description` | str | Optional description |
| `system` | str | System prompt |
| `agent_type` | enum | Type of agent (e.g., `letta`, `voice`, `sleeptime`) |
| `llm_config` | JSON | LLM configuration (model, temperature, etc.) |
| `embedding_config` | JSON | Embedding model configuration |
| `tools_used` | list | Tools available to the agent |
| `tool_rules` | JSON | Rules governing tool invocation order/constraints |
| `message_ids` | list | In-context message IDs |
| `last_stop_reason` | str | Last reason the agent loop stopped |
| `hidden` | bool | Whether agent is hidden (e.g., subagent) |
| `stateless` | bool | Whether agent has persistent state |
| `identifier_key` | str | External identifier key |
| `project_id` | str | Associated project |
| `template_id` | str | Template used to create agent |
| `timezone` | str | Agent's timezone |
| `metadata_` | JSON | Arbitrary metadata |
| `organization_id` | FK | Owning organization |
| `created_by_id` | FK | Creating user |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last update timestamp |

---

### 1.2 `Message`

Represents a single message in an agent's conversation history.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `role` | enum | `user`, `assistant`, `system`, `tool` |
| `content` | JSON/list | Message content (supports multimodal parts) |
| `name` | str | Optional sender name |
| `tool_calls` | JSON | Tool call requests (if assistant) |
| `tool_call_id` | str | ID linking tool result to tool call |
| `step_id` | FK | Associated execution step |
| `agent_id` | FK | Owning agent |
| `model` | str | Model that produced the message |
| `sender_id` | str | ID of the sender entity |
| `otid` | str | Optimistic transaction ID |
| `batch_item_id` | str | LLM batch item reference |
| `stop_reason` | str | Stop reason for this message |
| `feedback` | str | User feedback on the message |
| `approvals` | JSON | Human-in-the-loop approval data |
| `created_at` | datetime | Timestamp |

---

### 1.3 `Block`

A named, versioned memory block that can be attached to agents. The core unit of persistent memory.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `label` | str | Block label (e.g., `human`, `persona`) |
| `value` | str | Actual text content of the block |
| `limit` | int | Character limit |
| `is_template` | bool | Whether block is a reusable template |
| `preserve_on_migration` | bool | Retain across agent migrations |
| `read_only` | bool | Write-protected flag |
| `hidden` | bool | Internal/hidden block |
| `metadata_` | JSON | Arbitrary metadata |
| `description` | str | Human-readable description |
| `project_id` | str | Associated project |
| `organization_id` | FK | Owning organization |
| `created_by_id` | FK | Creating user |
| `created_at` / `updated_at` | datetime | Timestamps |

---

### 1.4 `BlockHistory`

Tracks historical versions of a `Block`'s content.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `block_id` | FK | Reference to parent Block |
| `value` | str | Historical content value |
| `sequence_number` | int | Version ordering |
| `created_at` | datetime | When this version was created |

---

### 1.5 `Tool`

A callable function or external integration available to agents.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `name` | str | Tool name (unique within org/project) |
| `description` | str | What the tool does |
| `tool_type` | enum | `python`, `composio`, `mcp`, `letta_core`, etc. |
| `source_code` | str | Python source implementation |
| `source_type` | str | Language/source type |
| `args_schema` | JSON | JSON Schema for tool arguments |
| `return_schema` | JSON | JSON Schema for return value |
| `tags` | list | Classification tags |
| `pip_requirements` | JSON | Python package dependencies |
| `npm_requirements` | JSON | Node.js package dependencies |
| `metadata_` | JSON | Arbitrary metadata |
| `enable_parallel_execution` | bool | Can run in parallel |
| `requires_approval` | bool | Needs human approval before execution |
| `project_id` | str | Associated project |
| `organization_id` | FK | Owning organization |

---

### 1.6 `Source`

A data source (e.g., uploaded documents) that can be attached to agents for archival memory retrieval.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `name` | str | Source name (unique per org) |
| `description` | str | Description |
| `instructions` | str | Processing/usage instructions |
| `embedding_config` | JSON | How passages are embedded |
| `metadata_` | JSON | Arbitrary metadata |
| `vector_db_provider` | str | External vector store provider |
| `organization_id` | FK | Owning organization |
| `created_by_id` | FK | Creating user |

---

### 1.7 `Passage`

A chunked, embedded text segment derived from a `Source` or stored as agent archival memory.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `text` | str | The passage content |
| `embedding` | vector | Dense embedding vector |
| `embedding_config` | JSON | Config used to generate embedding |
| `metadata_` | JSON | Arbitrary metadata |
| `source_id` | FK | Parent source (nullable for agent archival) |
| `agent_id` | FK | Owning agent (for agent archival memory) |
| `file_id` | FK | Source file this was derived from |
| `file_name` | str | Original file name |
| `organization_id` | FK | Owning organization |
| `created_at` | datetime | Timestamp |

---

### 1.8 `File`

A file uploaded and processed into a `Source`.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `file_name` | str | Original file name |
| `file_path` | str | Storage path |
| `file_type` | str | MIME type |
| `file_size` | int | Size in bytes |
| `processing_status` | enum | `pending`, `processing`, `completed`, `failed` |
| `total_chunks` | int | Total chunks generated |
| `chunks_embedded` | int | Chunks successfully embedded |
| `source_id` | FK | Parent source |
| `organization_id` | FK | Owning organization |
| `created_at` / `updated_at` | datetime | Timestamps |

---

### 1.9 `User`

A human user of the system.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `name` | str | Display name |
| `email` | str | Email address (unique) |
| `organization_id` | FK | Owning organization |
| `created_at` / `updated_at` | datetime | Timestamps |

---

### 1.10 `Organization`

The top-level tenancy/ownership entity.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `name` | str | Organization name |
| `privileged_tools` | JSON | Tools granted elevated access |
| `created_at` / `updated_at` | datetime | Timestamps |

---

### 1.11 `Job`

Represents a long-running background operation (e.g., file embedding, batch inference).

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `status` | enum | `pending`, `running`, `completed`, `failed`, `cancelled` |
| `job_type` | enum | Type of job |
| `stop_reason` | str | Why the job stopped |
| `callback_data` | JSON | Webhook callback payload |
| `callback_error` | str | Error from callback |
| `request_config` | JSON | Original request configuration |
| `metadata_` | JSON | Arbitrary metadata |
| `organization_id` | FK | Owning organization |
| `user_id` | FK | Requesting user |
| `created_at` / `updated_at` | datetime | Timestamps |

---

### 1.12 `Run`

Tracks an agent execution run (invocation of the agent loop).

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `status` | enum | Run status |
| `agent_id` | FK | Agent that ran |
| `job_id` | FK | Associated job (if any) |
| `usage` | JSON | Token/cost usage summary |
| `completion_tokens` | int | Output tokens |
| `duration_ms` | int | Total duration |
| `organization_id` | FK | Owning organization |
| `created_at` / `updated_at` | datetime | Timestamps |

---

### 1.13 `Step`

A single LLM inference step within a `Run`.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `run_id` | FK | Parent run |
| `agent_id` | FK | Agent executing |
| `provider_name` | str | LLM provider used |
| `model` | str | Model name |
| `model_endpoint` | str | Endpoint URL |
| `provider_category` | str | Provider classification |
| `context_window_size` | int | Context window used |
| `completion_tokens` | int | Tokens generated |
| `prompt_tokens` | int | Tokens in prompt |
| `prompt_tokens_details` | JSON | Breakdown (cached, etc.) |
| `stop_reason` | str | Why inference stopped |
| `trace_id` | str | Distributed trace ID |
| `request_id` | str | Provider request ID |
| `error` | str | Error if step failed |
| `build_request_latency_ms` | float | Time to build LLM request |
| `project_id` | str | Associated project |
| `created_at` | datetime | Timestamp |

---

### 1.14 `RunMetrics` / `StepMetrics`

Aggregated performance metrics for runs and steps.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `run_id` / `step_id` | FK | Parent entity |
| `tools_used` | JSON | Tools invoked |
| `total_tokens` | int | Aggregate token usage |
| `duration_ms` | int | Execution time |
| `created_at` | datetime | Timestamp |

---

### 1.15 `Group`

A multi-agent group enabling collaborative agent patterns.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `name` | str | Group name |
| `description` | str | Description |
| `manager_type` | enum | `round_robin`, `dynamic`, `supervisor`, `sleeptime` |
| `manager_agent_id` | FK | Designated manager agent |
| `ordered_agent_ids` | JSON | Ordered list of member agent IDs |
| `hidden` | bool | Internal/hidden flag |
| `organization_id` | FK | Owning organization |
| `project_id` | str | Associated project |
| `created_at` / `updated_at` | datetime | Timestamps |

---

### 1.16 `Identity`

Represents a persona or external user identity that can be associated with agents and blocks.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `identifier_key` | str | External unique key |
| `name` | str | Display name |
| `identity_type` | enum | Type of identity (e.g., `user`, `org`) |
| `properties` | JSON | Custom identity properties |
| `project_id` | str | Associated project |
| `organization_id` | FK | Owning organization |

---

### 1.17 `Provider`

An LLM/embedding API provider configuration (BYOK — Bring Your Own Key).

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `name` | str | Provider name |
| `provider_type` | enum | `openai`, `anthropic`, `azure`, `google`, etc. |
| `api_key` | str (encrypted) | API key |
| `base_url` | str | Custom endpoint URL |
| `api_version` | str | API version string |
| `access_key` / `secret_key` | str (encrypted) | AWS/Bedrock credentials |
| `last_synced` | datetime | Last model list sync |
| `organization_id` | FK | Owning organization |

---

### 1.18 `ProviderTrace`

Captures raw LLM request/response pairs for observability.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `step_id` | FK | Associated step |
| `agent_id` | FK | Agent that made the call |
| `provider_name` | str | Provider used |
| `model` | str | Model name |
| `request_json` | JSON | Raw LLM request payload |
| `response_json` | JSON | Raw LLM response payload |
| `source` | str | Trace source/context |
| `organization_id` | FK | Owning organization |
| `created_at` | datetime | Timestamp |

---

### 1.19 `SandboxConfig` / `SandboxEnvironmentVariable`

Configuration for isolated tool execution environments.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `sandbox_type` | enum | `local`, `e2b`, `modal` |
| `config` | JSON | Sandbox-specific configuration |
| `pip_requirements` | JSON | Python deps to install |
| `npm_requirements` | JSON | Node deps to install |
| `organization_id` | FK | Owning organization |

**SandboxEnvironmentVariable:**

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `key` | str | Variable name |
| `value` | str (encrypted) | Variable value |
| `sandbox_config_id` | FK | Parent sandbox config |

---

### 1.20 `MCPServer`

A Model Context Protocol server configuration.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `name` | str | Server name |
| `server_type` | enum | `sse`, `stdio`, `streamable_http` |
| `url` | str | Server URL |
| `token` | str (encrypted) | Auth token |
| `custom_headers` | JSON | HTTP headers |
| `organization_id` | FK | Owning organization |

---

### 1.21 `Conversation`

A logical conversation thread (distinct from per-agent message history).

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `name` | str | Conversation name |
| `description` | str | Description |
| `llm_config` | JSON | LLM config for this conversation |
| `last_message_at` | datetime | Last activity timestamp |
| `organization_id` | FK | Owning organization |
| `created_at` / `updated_at` | datetime | Timestamps |

---

### 1.22 `LLMBatchJob`

Represents a batched LLM inference job (e.g., Anthropic/OpenAI batch API).

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `letta_batch_job_id` | FK | Parent Letta job |
| `provider_batch_id` | str | Provider-side batch ID |
| `status` | enum | Batch status |
| `agent_ids` | JSON | Agents in this batch |
| `organization_id` | FK | Owning organization |
| `created_at` / `updated_at` | datetime | Timestamps |

---

### 1.23 `Archive`

An external vector database archive for archival memory.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `name` | str | Archive name |
| `vector_db_provider` | str | Vector DB provider (Pinecone, Turbopuffer, etc.) |
| `embedding_config` | JSON | Embedding configuration |
| `organization_id` | FK | Owning organization |

---

### 1.24 `Prompt`

A stored reusable system prompt or prompt template.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `name` | str | Prompt name |
| `text` | str | Prompt content |
| `organization_id` | FK | Owning organization |

---

### 1.25 `ProviderModel`

A cached model specification from a provider.

| Attribute | Type | Description |
|---|---|---|
| `id` | UUID/str | Primary key |
| `model_name` | str | Model identifier |
| `provider_name` | str | Provider name |
| `context_window` | int | Context window size |
| `supports_tools` | bool | Tool calling support |
| `supports_vision` | bool | Vision input support |
| `organization_id` | FK | Owning organization |

---

## 2. Association / Junction Tables

These represent many-to-many relationships:

| Table | Left Entity | Right Entity | Extra Attributes |
|---|---|---|---|
| `tools_agents` | Tool | Agent | `is_template` flag |
| `blocks_agents` | Block | Agent | `block_label`, `is_template` |
| `blocks_conversations` | Block | Conversation | — |
| `blocks_tags` | Block | Tag | — |
| `agents_tags` | Agent | Tag | — |
| `sources_agents` | Source | Agent | — |
| `files_agents` | File | Agent | `start`, `end`, `source_id`, `file_name` |
| `groups_agents` | Group | Agent | — |
| `groups_blocks` | Group | Block | — |
| `identities_agents` | Identity | Agent | — |
| `identities_blocks` | Identity | Block | — |
| `archives_agents` | Archive | Agent | — |
| `passage_tags` | Passage | Tag | — |
| `job_messages` | Job | Message | — |
| `conversation_messages` | Conversation | Message | — |

---

## 3. Entity Relationship Summary

```
Organization
  ├── 1:N ──► User
  ├── 1:N ──► Agent
  │             ├── M:N ──► Tool          (tools_agents)
  │             ├── M:N ──► Block         (blocks_agents)
  │             ├── M:N ──► Source        (sources_agents)
  │             ├── M:N ──► File          (files_agents)
  │             ├── M:N ──► Identity      (identities_agents)
  │             ├── M:N ──► Archive       (archives_agents)
  │             ├── M:N ──► Group         (groups_agents)
  │             ├── 1:N ──► Message
  │             ├── 1:N ──► Passage       (archival memory)
  │             └── 1:N ──► Run
  │                           └── 1:N ──► Step
  │                                         └── 1:1 ──► ProviderTrace
  ├── 1:N ──► Block
  │             ├── 1:N ──► BlockHistory
  │             ├── M:N ──► Identity      (identities_blocks)
  │             └── M:N ──► Conversation  (blocks_conversations)
  ├── 1:N ──► Tool
  ├── 1:N ──► Source
  │             └── 1:N ──► File
  │                           └── 1:N ──► Passage
  ├── 1:N ──► Group
  ├── 1:N ──► Job
  │             └── 1:N ──► LLMBatchJob
  ├── 1:N ──► Conversation
  │             └── M:N ──► Message       (conversation_messages)
  ├── 1:N ──► Provider
  ├── 1:N ──► ProviderModel
  ├── 1:N ──► SandboxConfig
  │             └── 1:N ──► SandboxEnvironmentVariable
  ├── 1:N ──► MCPServer
  ├── 1:N ──► Identity
  ├── 1:N ──► Archive
  └── 1:N ──► Prompt
```

---

## 4. Key Domain Observations

| Concern | Design Pattern |
|---|---|
| **Multi-tenancy** | Every entity is scoped to an `Organization` |
| **Memory Model** | `Block` (in-context) + `Passage` (archival) + `Archive` (external vector DB) form a layered memory system |
| **Observability** | `Step → ProviderTrace → ProviderTraceMetadata` provides full LLM call traceability |
| **Tool Isolation** | `SandboxConfig` enables isolated Python/Node execution environments per organization |
| **Versioning** | `BlockHistory` tracks memory mutations over time |
| **Batch Inference** | `LLMBatchJob` bridges Letta `Job` to provider-native batch APIs |
| **Multi-Agent** | `Group` + junction tables coordinate agent collaboration patterns |

--- module_deep_dive ---


# Letta Repository: Detailed Component Breakdown

---

## 1. `letta/agents/`

### Core Responsibility
The agent runtime layer — contains the core execution logic for AI agents. This is where the "thinking loop" lives: receiving messages, calling LLMs, processing tool calls, updating memory, and producing responses.

### Key Components

| File | Role |
|---|---|
| `base_agent.py` | Abstract base class defining the common agent interface and shared lifecycle methods |
| `base_agent_v2.py` | Updated base class with v2 protocol support |
| `letta_agent.py` | Primary production agent implementation; orchestrates the full LLM→tool→memory loop |
| `letta_agent_v2.py` | Refactored agent with improved streaming and message handling |
| `letta_agent_v3.py` | Latest iteration with additional protocol improvements |
| `letta_agent_batch.py` | Handles asynchronous batch LLM processing (async/offline inference) |
| `ephemeral_agent.py` | Stateless agent that doesn't persist memory between calls |
| `ephemeral_summary_agent.py` | Ephemeral agent specifically for summarization tasks |
| `voice_agent.py` | Agent variant optimized for voice/audio interactions |
| `voice_sleeptime_agent.py` | Voice agent with sleeptime (background memory consolidation) support |
| `agent_loop.py` | Core step-execution loop logic shared across agent types |
| `helpers.py` | Internal utility functions for the agent runtime |
| `exceptions.py` | Agent-specific exception classes |

### Dependencies & Interactions

**Internal modules:**
- `@letta/llm_api/` — Makes LLM API calls (OpenAI, Anthropic, etc.)
- `@letta/services/` — Calls `AgentManager`, `MessageManager`, `BlockManager`, `ToolManager` for persistence
- `@letta/schemas/` — Uses `AgentState`, `Message`, `LettaRequest`, `LettaResponse` schemas
- `@letta/interfaces/` — Uses streaming interfaces to handle streamed LLM responses
- `@letta/helpers/tool_rule_solver.py` — Enforces tool calling rules/sequences
- `@letta/helpers/message_helper.py` — Message formatting utilities
- `@letta/prompts/` — System prompt generation
- `@letta/functions/` — Tool function definitions and parsers
- `@letta/services/tool_sandbox/` — Delegates tool execution to sandbox
- `@letta/services/summarizer/` — Triggers memory summarization
- `@letta/otel/` — Emits tracing/metrics spans

**External services:**
- LLM providers (OpenAI, Anthropic, Google, etc.) via `letta/llm_api/`
- Indirectly: PostgreSQL (via service managers), Redis (via caching layer)

---

## 2. `letta/schemas/`

### Core Responsibility
The **data contract layer** of the application. Defines all Pydantic models used as API request/response bodies, internal data transfer objects (DTOs), and domain entity representations. Acts as the canonical type system for the entire codebase.

### Key Components

| File/Directory | Role |
|---|---|
| `agent.py` | `AgentState`, `CreateAgent`, `UpdateAgent` — core agent data models |
| `message.py` | `Message`, `MessageCreate` — message structure definitions |
| `memory.py` | `Memory`, `MemoryBlock` — in-context memory schemas |
| `block.py` | `Block`, `CreateBlock`, `UpdateBlock` — memory block schemas |
| `tool.py` | `Tool`, `ToolCreate` — tool definition schemas |
| `letta_request.py` | Incoming request schema for agent message sends |
| `letta_response.py` | Agent response envelope schema |
| `letta_message.py` | Typed message variants (assistant, tool call, tool result, etc.) |
| `letta_message_content.py` | Content part types (text, image, etc.) |
| `llm_config.py` | LLM provider configuration schema |
| `embedding_config.py` | Embedding model configuration |
| `sandbox_config.py` | Tool sandbox environment configuration |
| `group.py` | Multi-agent group schemas |
| `job.py` | Background job schemas |
| `source.py` | Data source schemas |
| `passage.py` | Archival memory passage schemas |
| `provider.py` / `providers/` | LLM provider schemas (24 provider-specific files) |
| `openai/` | OpenAI-compatible API schemas (chat completions format) |
| `step.py`, `run.py` | Execution step and run tracking schemas |
| `enums.py` | Shared enumeration types |
| `letta_base.py` | Base Pydantic model with shared config |
| `mcp.py`, `mcp_server.py` | MCP protocol schemas |
| `identity.py` | User/agent identity schemas |

### Dependencies & Interactions

**Internal modules:**
- Used by virtually every other module — `@letta/orm/`, `@letta/services/`, `@letta/server/`, `@letta/agents/`, `@letta/llm_api/`
- `@letta/helpers/` — Some schema validators call helper utilities

**External services:**
- No direct external calls; purely data definition
- Pydantic v2 for validation; some schemas mirror OpenAI's API format for compatibility

---

## 3. `letta/orm/`

### Core Responsibility
The **database persistence layer**. Contains SQLAlchemy ORM models mapping Python objects to database tables. Each file represents one domain entity or association table. This is the single source of truth for the database schema.

### Key Components

| File | Role |
|---|---|
| `base.py` | Base ORM class with common fields (id, created_at, updated_at) |
| `sqlalchemy_base.py` | SQLAlchemy declarative base setup |
| `mixins.py` | Reusable ORM mixin classes (e.g., soft-delete, org scoping) |
| `agent.py` | `AgentModel` — agents table |
| `message.py` | `MessageModel` — messages table |
| `block.py` | `BlockModel` — memory blocks table |
| `block_history.py` | Historical versions of memory blocks |
| `tool.py` | `ToolModel` — tools table |
| `source.py` | `SourceModel` — data sources table |
| `passage.py` | `PassageModel` — archival memory passages |
| `organization.py` | `OrganizationModel` — multi-tenancy organizations |
| `user.py` | `UserModel` — user accounts |
| `job.py` | `JobModel` — background jobs |
| `group.py` | `GroupModel` — multi-agent groups |
| `identity.py` | `IdentityModel` — agent/user identities |
| `sandbox_config.py` | Sandbox environment configurations |
| `provider.py`, `provider_model.py` | LLM provider registrations |
| `provider_trace.py`, `provider_trace_metadata.py` | LLM call traces for observability |
| `mcp_server.py`, `mcp_oauth.py` | MCP server registrations and OAuth state |
| `llm_batch_job.py`, `llm_batch_items.py` | Batch inference job tracking |
| `run.py`, `run_metrics.py`, `step.py`, `step_metrics.py` | Execution tracking |
| `*_agents.py`, `*_tags.py` | Many-to-many association/junction tables |
| `custom_columns.py` | Custom SQLAlchemy column types |
| `sqlite_functions.py` | SQLite-specific SQL function implementations |
| `errors.py` | ORM-level error classes |

### Dependencies & Interactions

**Internal modules:**
- `@letta/schemas/` — ORM models often map to/from corresponding schema models
- `@letta/database_utils.py` — Database connection/session management
- `@alembic/` — Migration scripts use these ORM models as the schema source

**External services:**
- **PostgreSQL** (primary, with `pgvector` for vector similarity columns)
- **SQLite** (alternate backend for testing/local use)
- SQLAlchemy ORM + Core

---

## 4. `letta/services/`

### Core Responsibility
The **business logic / service layer**. Manager classes implement CRUD operations, domain logic, and orchestration over the ORM models. Acts as the intermediary between the API layer and the database, encapsulating all business rules.

### Key Components

| File/Directory | Role |
|---|---|
| `agent_manager.py` | Agent lifecycle: create, read, update, delete agents; attach tools/blocks/sources |
| `message_manager.py` | Message persistence and retrieval |
| `block_manager.py` | Memory block CRUD; handles block sharing between agents |
| `tool_manager.py` | Tool registration, updates, and agent-tool associations |
| `source_manager.py` | Data source management (files, connectors) |
| `passage_manager.py` | Archival memory passage CRUD and vector search |
| `group_manager.py` | Multi-agent group management |
| `identity_manager.py` | Identity (user/agent persona) management |
| `job_manager.py` | Background job tracking and status updates |
| `run_manager.py` | Run lifecycle management |
| `step_manager.py` | Individual execution step recording |
| `organization_manager.py` | Organization/tenant management |
| `user_manager.py` | User account management |
| `provider_manager.py` | LLM provider registration and config management |
| `mcp_manager.py` | MCP tool discovery and invocation |
| `mcp_server_manager.py` | MCP server registration management |
| `sandbox_config_manager.py` | Sandbox environment config CRUD |
| `conversation_manager.py` | Conversation (thread) management |
| `file_manager.py` | File metadata management |
| `files_agents_manager.py` | File-to-agent association management |
| `llm_batch_manager.py` | LLM batch job creation and polling |
| `archive_manager.py` | Archival memory management |
| `streaming_service.py` | SSE/streaming response management |
| `webhook_service.py` | Outgoing webhook delivery |
| `telemetry_manager.py` | Telemetry data aggregation |
| `agent_generate_completion_manager.py` | Orchestrates LLM completion generation for agents |
| `agent_serialization_manager.py` | Agent import/export serialization |
| `credit_verification_service.py` | Usage credit/quota verification |
| `tool_schema_generator.py` | Generates JSON schemas from Python tool functions |
| `tool_sandbox/` | **Sandboxed tool execution** (Docker, Modal, local subprocess) |
| `summarizer/` | Memory summarization pipeline (triggers when context window fills) |
| `file_processor/` | File ingestion pipeline: parse → chunk → embed → store |
| `mcp/` | MCP client implementation for connecting to MCP servers |
| `memory_repo/` | Pluggable memory storage backends (pgvector, Turbopuffer, Pinecone) |
| `llm_router/` | LLM request routing logic |
| `context_window_calculator/` | Token counting and context window management |
| `provider_trace_backends/` | Pluggable trace storage backends (Clickhouse, file, etc.) |
| `lettuce/` | Internal utility service(s) |
| `helpers/` | Shared service-layer helper functions |

### Dependencies & Interactions

**Internal modules:**
- `@letta/orm/` — All managers query/mutate ORM models via SQLAlchemy sessions
- `@letta/schemas/` — Input/output types for all service methods
- `@letta/llm_api/` — `agent_generate_completion_manager.py` calls LLM clients
- `@letta/helpers/` — Utility functions (JSON, crypto, tool helpers)
- `@letta/otel/` — Traces service operations
- `@letta/functions/` — Tool schema generation and parsing
- `@letta/prompts/` — Prompt templates for summarization

**External services:**
- **PostgreSQL / SQLite** — via SQLAlchemy sessions
- **Redis** — caching, pub-sub for streaming
- **Docker SDK** — `tool_sandbox/` for containerized tool execution
- **Modal Labs API** — `tool_sandbox/` for cloud sandbox execution
- **MCP servers** — external tool servers via MCP protocol
- **Composio API** — third-party tool integrations
- **Pinecone / Turbopuffer** — external vector DB backends (optional)
- **Webhook endpoints** — outbound HTTP calls in `webhook_service.py`

---

## 5. `letta/server/`

### Core Responsibility
The **API presentation layer**. Houses the FastAPI application, all REST route handlers, WebSocket endpoints, middleware, authentication, and the main server setup. This is the entry point for all external HTTP/WS communication.

### Key Components

| File/Directory | Role |
|---|---|
| `server.py` | Main FastAPI `app` instance creation, middleware registration, router mounting |
| `db.py` | Database session factory and dependency injection for FastAPI |
| `constants.py` | Server-level constants (timeouts, limits, etc.) |
| `utils.py` | Server utility functions (response helpers, etc.) |
| `global_exception_handler.py` | Global FastAPI exception handler mapping errors to HTTP codes |
| `startup.sh` | Shell script for production server startup (Uvicorn) |
| `generate_openapi_schema.sh` | Script to export the OpenAPI JSON schema |
| `rest_api/routers/` | **Domain-specific route handlers** (one file per resource domain) |
| `rest_api/auth/` | Authentication middleware and token validation |
| `rest_api/middleware/` | HTTP middleware (logging, request ID, rate limiting, etc.) |
| `ws_api/` | WebSocket endpoint handlers for real-time agent streaming |

**Key Router Files in `rest_api/routers/`** (inferred from project structure):
- Agents router — `/agents` CRUD and message sending
- Tools router — `/tools` management
- Memory/Blocks router — `/blocks` management
- Sources router — `/sources` data ingestion
- Groups router — `/groups` multi-agent
- Jobs router — `/jobs` background job status
- Providers router — `/providers` LLM config
- Users/Orgs router — `/users`, `/organizations`
- MCP router — `/mcp-servers`
- Health/Readiness router

### Dependencies & Interactions

**Internal modules:**
- `@letta/services/` — All routers delegate business logic to manager classes
- `@letta/schemas/` — Request/response model types for all endpoints
- `@letta/agents/` — Directly instantiates and invokes agent execution
- `@letta/otel/` — Request tracing via middleware
- `@letta/monitoring/` — Health check / readiness state
- `@letta/server/db.py` — SQLAlchemy session injection
- `@letta/helpers/` — Auth helpers, validators

**External services:**
- **PostgreSQL** — via injected DB sessions
- **Redis** — for pub-sub streaming responses
- **OpenTelemetry collector** — spans emitted from middleware
- **OAuth providers** — via MCP OAuth flow

---

## 6. `letta/llm_api/`

### Core Responsibility
The **LLM provider abstraction layer**. Provides a unified interface for making LLM chat completion calls across all supported providers. Implements the Strategy pattern — each provider has its own client class with a common base interface.

### Key Components

| File | Role |
|---|---|
| `llm_client_base.py` | Abstract base class defining the `send_llm_request()` interface |
| `llm_client.py` | Factory/dispatcher that selects the correct provider client |
| `llm_api_tools.py` | High-level tools for making LLM calls with retry/error handling |
| `openai_client.py` | OpenAI API client (GPT-4, GPT-4o, o1, etc.) |
| `anthropic_client.py` | Anthropic API client (Claude models) |
| `google_ai_client.py` | Google AI Studio client (Gemini) |
| `google_vertex_client.py` | Google Vertex AI client |
| `bedrock_client.py` | AWS Bedrock client |
| `azure_client.py` | Azure OpenAI Service client |
| `groq_client.py` | Groq inference client |
| `together_client.py` | Together AI client |
| `fireworks_client.py` | Fireworks AI client |
| `mistral.py` | Mistral AI client |
| `deepseek_client.py` | DeepSeek client |
| `minimax_client.py` | MiniMax client |
| `xai_client.py` | xAI (Grok) client |
| `zai_client.py` | ZhipuAI client |
| `baseten_client.py` | Baseten client |
| `sglang_native_client.py` | SGLang native inference client |
| `chatgpt_oauth_client.py` | ChatGPT OAuth-authenticated client |
| `openai_ws_session.py` | OpenAI Realtime API WebSocket session (for voice) |
| `helpers.py` | Shared LLM utility functions (token counting, format conversion) |
| `error_utils.py` | Provider error parsing and normalization |
| `anthropic_constants.py`, `google_constants.py` | Provider-specific constants |
| `openai.py` | OpenAI-specific helpers and wrappers |
| `sample_response_jsons/` | Sample provider responses for testing |

### Dependencies & Interactions

**Internal modules:**
- `@letta/schemas/` — Uses `LLMConfig`, `Message`, provider-specific schemas
- `@letta/schemas/providers/` — Provider-specific request/response schemas
- `@letta/interfaces/` — Streaming response interfaces consumed here
- `@letta/helpers/` — JSON parsing, error handling utilities
- `@letta/otel/` — Traces LLM API calls

**External services (all direct):**
- **OpenAI API** (`api.openai.com`)
- **Anthropic API** (`api.anthropic.com`)
- **Google AI / Vertex AI**
- **AWS Bedrock**
- **Azure OpenAI**
- **Groq, Together AI, Fireworks, Mistral, DeepSeek, MiniMax, xAI, ZhipuAI, Baseten**
- **LiteLLM** — used as a routing/normalization layer for many providers
- **SGLang** — local/self-hosted inference

---

## 7. `letta/groups/`

### Core Responsibility
**Multi-agent orchestration**. Implements coordination patterns for running multiple agents together — defining how agents take turns, communicate results, and collaborate on tasks.

### Key Components

| File | Role |
|---|---|
| `round_robin_multi_agent.py` | Agents take turns in a fixed rotation; output of one feeds into the next |
| `supervisor_multi_agent.py` | A supervisor agent directs and delegates to worker agents |
| `sleeptime_multi_agent.py` | Background "sleeptime" agent runs memory consolidation between turns (v1) |
| `sleeptime_multi_agent_v2.py` | Sleeptime v2 with improved scheduling |
| `sleeptime_multi_agent_v3.py` | Sleeptime v3 |
| `sleeptime_multi_agent_v4.py` | Sleeptime v4 (latest iteration) |
| `dynamic_multi_agent.py` | Dynamically selects which agent to invoke based on context |
| `helpers.py` | Shared utility functions for group orchestration |

### Dependencies & Interactions

**Internal modules:**
- `@letta/agents/` — Instantiates and invokes individual agent instances
- `@letta/services/agent_manager.py` — Loads agent configurations
- `@letta/services/group_manager.py` — Persists group state
- `@letta/schemas/group.py` — Group configuration schemas
- `@letta/schemas/message.py` — Message passing between agents
- `@letta/otel/` — Distributed tracing across agent hops

**External services:**
- Indirectly uses LLM providers through the agents it orchestrates
- No direct external API calls

---

## 8. `letta/functions/`

### Core Responsibility
**Tool/function management and schema generation**. Handles the definition, parsing, validation, and schema generation for agent tools — both built-in Letta tools and externally-defined ones (Composio, MCP).

### Key Components

| File/Directory | Role |
|---|---|
| `functions.py` | Core tool function loading, registration, and lookup |
| `schema_generator.py` | Generates OpenAI-compatible JSON schemas from Python function signatures |
| `schema_validator.py` | Validates tool schemas against expected format |
| `ast_parsers.py` | AST-based Python source code parsing to extract function metadata |
| `typescript_parser.py` | Parses TypeScript function definitions for JS-based tools |
| `helpers.py` | Tool utility functions |
| `interface.py` | Tool interface/protocol definitions |
| `prompts.py` | Prompt templates related to tool usage instructions |
| `types.py` | Tool-related type definitions |
| `composio_helpers.py` | Composio-specific tool integration helpers |
| `async_composio_toolset.py` | Async wrapper for Composio tool execution |
| `function_sets/` | **Built-in tool collections**: core memory tools, archival memory tools, conversation search, etc. |
| `mcp_client/` | MCP protocol client for discovering and calling tools on MCP servers |

### Dependencies & Interactions

**Internal modules:**
- `@letta/schemas/tool.py` — Tool schema data models
- `@letta/services/tool_manager.py` — Registers/persists tools
- `@letta/helpers/tool_helpers.py` — Shared tool utilities
- `@letta/helpers/tool_execution_helper.py` — Execution helpers
- `@letta/services/mcp/` — MCP client connection management

**External services:**
- **Composio API** — fetches and executes Composio-hosted tools
- **MCP servers** — remote tool servers via the Model Context Protocol

---

## 9. `letta/local_llm/`

### Core Responsibility
**Local/self-hosted LLM support**. Provides adapters and wrappers for running inference against locally-hosted models (Ollama, vLLM, LM Studio, llama.cpp, KoboldCpp, etc.) that don't conform to the standard OpenAI API format.

### Key Components

| File/Directory | Role |
|---|---|
| `chat_completion_proxy.py` | Proxy that routes requests to the appropriate local backend |
| `function_parser.py` | Parses function/tool calls from local LLM text outputs (non-native tool calling) |
| `json_parser.py` | Optimistic JSON parser for handling malformed LLM JSON outputs |
| `utils.py` | Local LLM utility functions |
| `constants.py` | Local LLM constants |
| `llm_chat_completion_wrappers/` | **Format adapters** — converts Letta's request format to each local backend's expected format (9 wrapper files) |
| `vllm/` | vLLM-specific integration |
| `ollama/` | Ollama-specific integration |
| `lmstudio/` | LM Studio integration |
| `koboldcpp/` | Kob

--- events ---


I'll analyze the codebase systematically, focusing on event-related code — message brokers, queues, pub/sub systems, webhooks, and internal event buses.

Let me examine the key files:

**Key files to analyze:**
- `letta/services/webhook_service.py`
- `letta/jobs/` directory
- `letta/server/server.py`
- `letta/agents/` directory
- `letta/otel/events.py`
- `letta/data_sources/redis_client.py`
- `letta/schemas/job.py`, `letta/schemas/llm_batch_job.py`
- Various service files

After a thorough scan of the repository structure and all relevant source files, here is the complete event documentation:

---

## Event Documentation

---

### Event: Webhook — Agent Message Callback

* **Event Type:** HTTP Webhook (custom outbound HTTP POST)
* **Event Name/Topic/Queue:** Configurable per-job callback URL (stored in `job.callback_url`)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "status": "string (e.g., 'completed', 'failed')",
      "job_id": "string (UUID)",
      "agent_id": "string (UUID)",
      "messages": [
        {
          "id": "string",
          "role": "string (e.g., 'assistant', 'user', 'tool')",
          "content": "string | array",
          "created_at": "string (ISO 8601 datetime)",
          "agent_id": "string (UUID)"
        }
      ],
      "usage": {
        "prompt_tokens": "integer",
        "completion_tokens": "integer",
        "total_tokens": "integer"
      }
    }
    ```
* **Short explanation of what this event is doing:** When an asynchronous agent job (e.g., a `send_message` batch call) completes or fails, the server POSTs the result to a caller-supplied `callback_url`. This is the primary mechanism for notifying external systems of async job completion. Defined and dispatched in `letta/services/webhook_service.py`.

---

### Event: Webhook — Batch LLM Job Completion Callback

* **Event Type:** HTTP Webhook (custom outbound HTTP POST)
* **Event Name/Topic/Queue:** Configurable per-batch-job callback URL (stored on the `LLMBatchJob` record)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "status": "string (e.g., 'completed', 'failed')",
      "letta_batch_job_id": "string (UUID)",
      "agent_ids": ["string (UUID)"],
      "error": "string | null"
    }
    ```
* **Short explanation of what this event is doing:** After a batch LLM job (e.g., an Anthropic Batch API call) finishes polling and all agent steps have been processed, a webhook is fired to the registered callback URL. Handled in `letta/jobs/llm_batch_job_polling.py` and `letta/services/webhook_service.py`.

---

### Event: Redis Pub/Sub — Agent Run Step Notification

* **Event Type:** Redis Pub/Sub
* **Event Name/Topic/Queue:** `agent:{agent_id}:run` (dynamic channel name per agent)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "step_id": "string (UUID)",
      "agent_id": "string (UUID)",
      "status": "string (e.g., 'completed', 'error')",
      "timestamp": "string (ISO 8601 datetime)"
    }
    ```
* **Short explanation of what this event is doing:** When an agent completes a processing step, a notification is published to a Redis channel scoped to that agent. This enables real-time streaming/polling consumers (e.g., server-sent events endpoints) to detect when new output is available. Defined in `letta/data_sources/redis_client.py`.

---

### Event: Redis Pub/Sub — Agent Run Step Notification

* **Event Type:** Redis Pub/Sub
* **Event Name/Topic/Queue:** `agent:{agent_id}:run` (dynamic channel name per agent)
* **Direction:** Consuming
* **Event Payload:**
    ```json
    {
      "step_id": "string (UUID)",
      "agent_id": "string (UUID)",
      "status": "string (e.g., 'completed', 'error')",
      "timestamp": "string (ISO 8601 datetime)"
    }
    ```
* **Short explanation of what this event is doing:** The streaming service subscribes to the agent-scoped Redis channel to receive step completion signals. When a message arrives, it fetches the latest messages/steps from the database and forwards them to the connected HTTP streaming client (SSE). Defined in `letta/data_sources/redis_client.py` and consumed in `letta/services/streaming_service.py`.

---

### Event: Internal Scheduler — LLM Batch Job Polling Tick

* **Event Type:** Custom Internal Event Bus / APScheduler (scheduled job trigger)
* **Event Name/Topic/Queue:** `poll_llm_batch_jobs` (APScheduler job ID)
* **Direction:** Consuming
* **Event Payload:**
    N/A — timer-triggered, no payload
* **Short explanation of what this event is doing:** The APScheduler fires this recurring event on a configurable interval to trigger the `LLMBatchJobPollingManager`. The manager queries pending batch jobs (e.g., Anthropic Batch API), checks their remote status, processes completed items, and updates agent states accordingly. Defined in `letta/jobs/scheduler.py` and `letta/jobs/llm_batch_job_polling.py`.

---

### Event: Internal Scheduler — Sleeptime Agent Trigger

* **Event Type:** Custom Internal Event Bus / APScheduler (scheduled job trigger)
* **Event Name/Topic/Queue:** `sleeptime_agent_trigger` (APScheduler job ID)
* **Direction:** Consuming
* **Event Payload:**
    N/A — timer-triggered, no payload
* **Short explanation of what this event is doing:** A scheduled APScheduler event that periodically wakes "sleeptime" background agents. These are agents configured to run autonomously at set intervals (e.g., to perform background memory consolidation or proactive tasks). Defined in `letta/jobs/scheduler.py` and executed via `letta/groups/sleeptime_multi_agent.py`.

---

### Event: OpenTelemetry Span Event — LLM Request

* **Event Type:** OpenTelemetry (OTEL Tracing / Span Events)
* **Event Name/Topic/Queue:** `llm.request` (OTEL span event name)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "llm.model": "string",
      "llm.provider": "string",
      "llm.request.messages": "string (JSON-serialized array)",
      "llm.request.tools": "string (JSON-serialized array) | null",
      "llm.request.temperature": "number | null",
      "llm.request.max_tokens": "integer | null"
    }
    ```
* **Short explanation of what this event is doing:** Emitted as an OTEL span event each time an LLM API request is dispatched. Captures the full request context (model, provider, messages, tools, parameters) for observability, tracing, and debugging. Defined in `letta/otel/events.py` and emitted from `letta/llm_api/llm_api_tools.py`.

---

### Event: OpenTelemetry Span Event — LLM Response

* **Event Type:** OpenTelemetry (OTEL Tracing / Span Events)
* **Event Name/Topic/Queue:** `llm.response` (OTEL span event name)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "llm.response.content": "string (JSON-serialized response)",
      "llm.response.usage.prompt_tokens": "integer",
      "llm.response.usage.completion_tokens": "integer",
      "llm.response.usage.total_tokens": "integer",
      "llm.response.stop_reason": "string | null"
    }
    ```
* **Short explanation of what this event is doing:** Emitted as an OTEL span event upon receiving a response from the LLM provider. Captures response content and token usage metrics for observability, cost tracking, and performance analysis. Defined in `letta/otel/events.py`.

---

### Event: OpenTelemetry Span Event — Tool Call

* **Event Type:** OpenTelemetry (OTEL Tracing / Span Events)
* **Event Name/Topic/Queue:** `tool.call` (OTEL span event name)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "tool.name": "string",
      "tool.call_id": "string",
      "tool.arguments": "string (JSON-serialized object)",
      "agent.id": "string (UUID)"
    }
    ```
* **Short explanation of what this event is doing:** Emitted when an agent invokes a tool (function call). Used for distributed tracing to track which tools are being called with what arguments, enabling observability into agent decision-making. Defined in `letta/otel/events.py`.

---

### Event: OpenTelemetry Span Event — Tool Result

* **Event Type:** OpenTelemetry (OTEL Tracing / Span Events)
* **Event Name/Topic/Queue:** `tool.result` (OTEL span event name)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "tool.name": "string",
      "tool.call_id": "string",
      "tool.result": "string (serialized result)",
      "tool.error": "string | null"
    }
    ```
* **Short explanation of what this event is doing:** Emitted after a tool execution completes (or fails), recording the output or error. Paired with `tool.call` events for end-to-end tracing of tool invocations within agent loops. Defined in `letta/otel/events.py`.

---

### Event: OpenTelemetry Span Event — Agent Step

* **Event Type:** OpenTelemetry (OTEL Tracing / Span Events)
* **Event Name/Topic/Queue:** `agent.step` (OTEL span event name)
* **Direction:** Producing
* **Event Payload:**
    ```json
    {
      "agent.id": "string (UUID)",
      "step.id": "string (UUID)",
      "step.number": "integer",
      "step.stop_reason": "string | null"
    }
    ```
* **Short explanation of what this event is doing:** Emitted at each discrete agent reasoning step. Provides tracing visibility into the agent loop's progress — how many steps were taken, and why each step ended. Defined in `letta/otel/events.py`.

---

> **Note on scope:** This codebase does **not** use Kafka, SQS, EventBridge, RabbitMQ, Ably, Google Pub/Sub, or similar managed message brokers. The event infrastructure relies on: (1) outbound HTTP webhooks for async job completion notifications, (2) Redis Pub/Sub for real-time intra-service streaming coordination, (3) APScheduler for time-driven internal events, and (4) OpenTelemetry for observability span events exported to a configured OTEL collector backend.

--- deployment ---


# Deployment Pipeline Analysis: letta_a0979209

## 1. Deployment Overview

| Attribute | Value |
|-----------|-------|
| **Primary CI/CD Platform** | GitHub Actions |
| **Workflow Files** | 27 workflow files |
| **Environments** | Not explicitly defined (no environment protection rules visible) |
| **IaC** | Docker / Docker Compose (no Terraform/CloudFormation) |
| **Container Registry** | Docker Hub (implied by docker-image.yml) |
| **Package Registry** | PyPI (via Poetry publish workflows) |
| **SDK Distribution** | Fern-generated Python + TypeScript SDKs |

---

## 2. Deployment Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        TRIGGER EVENTS                               │
│  push(main/dev) │ PR events │ Tags(v*) │ Schedule │ Manual dispatch │
└────────┬────────┴─────┬─────┴────┬─────┴────┬──────┴───────┬────────┘
         │              │          │          │              │
         ▼              ▼          ▼          ▼              ▼
┌─────────────┐  ┌──────────┐ ┌────────┐ ┌────────┐  ┌──────────────┐
│  core-lint  │  │ PR Tests │ │Release │ │Nightly │  │Manual Trigger│
│  .yml       │  │          │ │ Tags   │ │ Build  │  │  Workflows   │
└──────┬──────┘  └────┬─────┘ └───┬────┘ └───┬────┘  └──────┬───────┘
       │              │           │           │              │
       ▼              ▼           ▼           ▼              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     QUALITY / VALIDATION STAGE                      │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────────────────┐ │
│  │ core-lint   │  │alembic-      │  │  fern-check (OpenAPI       │ │
│  │ (ruff,      │  │validation    │  │  schema validation)        │ │
│  │  formatting)│  │.yml          │  │                            │ │
│  └─────────────┘  └──────────────┘  └────────────────────────────┘ │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        TEST STAGE                                   │
│  ┌────────────────┐  ┌─────────────────┐  ┌───────────────────┐   │
│  │ Unit Tests     │  │ Integration     │  │ Docker Integration │   │
│  │ (SQLite + PG)  │  │ Tests (core)    │  │ Tests             │   │
│  │                │  │                 │  │                    │   │
│  │core-unit-      │  │core-integration │  │docker-integration- │   │
│  │sqlite-test.yaml│  │-tests.yml       │  │tests.yaml         │   │
│  │core-unit-      │  │                 │  │                    │   │
│  │test.yml        │  │                 │  │                    │   │
│  └────────────────┘  └─────────────────┘  └───────────────────┘   │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │         Specialized Integration Tests (on demand/schedule)   │  │
│  │  send-message │ test-ollama │ test-vllm │ test-lmstudio      │  │
│  │  model-sweep  │ test-pip-install │ migration-test            │  │
│  └──────────────────────────────────────────────────────────────┘  │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        BUILD STAGE                                  │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    docker-image.yml                          │  │
│  │  Build multi-arch Docker image (linux/amd64, linux/arm64)   │  │
│  │  Push to registry with tags (sha, version, latest)          │  │
│  └──────────────────────────────────────────────────────────────┘  │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    ▼                         ▼
┌──────────────────────────┐    ┌─────────────────────────────┐
│    PYPI PUBLISH          │    │     SDK PUBLISH             │
│                          │    │                             │
│  poetry-publish.yml      │    │  fern-sdk-python-publish    │
│  (on tag v*)             │    │  fern-sdk-typescript-publish│
│                          │    │  (on manual/tag trigger)    │
│  poetry-publish-         │    │                             │
│  nightly.yml             │    │  fern-docs-publish          │
│  (scheduled)             │    │  (docs deployment)          │
└──────────────────────────┘    └─────────────────────────────┘
```

---

## 3. CI/CD Platform Detection

**Platform:** GitHub Actions  
**Location:** `.github/workflows/` (27 workflow files)

---

## 4. Pipeline: Detailed Workflow Documentation

### Pipeline: `core-lint.yml`

**Location:** `.github/workflows/core-lint.yml`

**Triggers:**
- `push` to any branch
- `pull_request` events
- Manual dispatch (implied by lint-command.yml reusable workflow pattern)

**Stages/Jobs:**

1. **Stage: Lint**
   - **Purpose:** Enforce code style and formatting using `ruff`
   - **Steps:** Run `ruff check` and `ruff format --check`
   - **Tools:** `ruff` (configured in `pyproject.toml` with line-length=140, Python 3.12 target)
   - **Artifacts:** None (pass/fail gate only)

**Quality Gates:**
- `ruff` lint rules: E, W, F, I, RUF, FAST categories enforced
- Formatting enforced via `ruff format`

---

### Pipeline: `core-unit-test.yml`

**Location:** `.github/workflows/core-unit-test.yml`

**Triggers:**
- `push` to `main`/`develop` branches
- `pull_request` events

**Stages/Jobs:**

1. **Stage: Unit Tests (PostgreSQL)**
   - **Purpose:** Run unit test suite against PostgreSQL backend
   - **Steps:** Spin up PostgreSQL service container, install dependencies via `uv`, run `pytest`
   - **Dependencies:** PostgreSQL service
   - **Artifacts:** Test results (JSON report via `pytest-json-report`)

---

### Pipeline: `core-unit-sqlite-test.yaml`

**Location:** `.github/workflows/core-unit-sqlite-test.yaml`

**Triggers:**
- `push` to `main`/`develop` branches
- `pull_request` events

**Stages/Jobs:**

1. **Stage: Unit Tests (SQLite)**
   - **Purpose:** Run unit test suite against SQLite backend (faster, no external DB)
   - **Steps:** Install dependencies, run `pytest` with SQLite configuration
   - **Dependencies:** None (SQLite is embedded)
   - **Artifacts:** Test results

---

### Pipeline: `core-integration-tests.yml`

**Location:** `.github/workflows/core-integration-tests.yml`

**Triggers:**
- `pull_request` events
- Push to protected branches
- References `reusable-test-workflow.yml`

**Stages/Jobs:**

1. **Stage: Integration Tests**
   - **Purpose:** Full integration test suite against live services
   - **Steps:** Provision PostgreSQL + Redis, install app, run integration test files
   - **Dependencies:** PostgreSQL, Redis service containers
   - **Conditions:** Requires secrets (LLM API keys) to be present
   - **Artifacts:** Test results

**Notable:** Uses `reusable-test-workflow.yml` as a shared test execution pattern.

---

### Pipeline: `docker-image.yml`

**Location:** `.github/workflows/docker-image.yml`

**Triggers:**
- `push` to `main` branch
- On release tag (`v*`)
- Manual dispatch

**Stages/Jobs:**

1. **Stage: Build Docker Image**
   - **Purpose:** Build and push multi-architecture Docker image
   - **Steps:**
     1. Checkout code
     2. Set up Docker Buildx (multi-arch support)
     3. Login to container registry
     4. Build image with build args (`LETTA_VERSION`, `LETTA_ENVIRONMENT`)
     5. Push to registry with tags
   - **Artifacts:** Docker image pushed to registry
   - **Tags:** `sha-<commit>`, version tag, `latest` (on main)

**Multi-arch:** Builds for `linux/amd64` and `linux/arm64` (inferred from Dockerfile `TARGETARCH` arg)

---

### Pipeline: `docker-integration-tests.yaml`

**Location:** `.github/workflows/docker-integration-tests.yaml`

**Triggers:**
- `pull_request` events
- Push to `main`

**Stages/Jobs:**

1. **Stage: Build Test Image**
   - **Purpose:** Build Docker image for integration testing
   
2. **Stage: Run Docker Integration Tests**
   - **Purpose:** Run tests against a fully containerized Letta deployment
   - **Steps:** `docker compose up`, wait for health, run test suite
   - **Dependencies:** Full Docker Compose stack (app + postgres + redis)

---

### Pipeline: `alembic-validation.yml`

**Location:** `.github/workflows/alembic-validation.yml`

**Triggers:**
- `pull_request` events (specifically when migration files change)
- Push to `main`

**Stages/Jobs:**

1. **Stage: Validate Alembic Migrations**
   - **Purpose:** Ensure database migrations are valid and can be applied cleanly
   - **Steps:**
     1. Start PostgreSQL
     2. Run `alembic upgrade head`
     3. Verify schema state
   - **Dependencies:** PostgreSQL service container
   - **Critical:** Prevents broken migrations from merging

---

### Pipeline: `migration-test.yml`

**Location:** `.github/workflows/migration-test.yml`

**Triggers:**
- `pull_request` events
- Push to `main`

**Stages/Jobs:**

1. **Stage: Migration Test**
   - **Purpose:** Test migration upgrade and downgrade paths
   - **Steps:** Apply migrations forward and optionally backward
   - **Dependencies:** PostgreSQL

---

### Pipeline: `poetry-publish.yml`

**Location:** `.github/workflows/poetry-publish.yml`

**Triggers:**
- Git tag push matching `v*` pattern (release tags)

**Stages/Jobs:**

1. **Stage: Build Python Package**
   - **Purpose:** Build distributable Python package
   - **Steps:**
     1. Install Poetry/uv
     2. Build wheel and sdist
     3. Publish to PyPI

2. **Stage: Publish to PyPI**
   - **Purpose:** Upload `letta` package to PyPI
   - **Secrets Required:** `PYPI_TOKEN` or equivalent
   - **Artifacts:** PyPI package (`letta==<version>`)

---

### Pipeline: `poetry-publish-nightly.yml`

**Location:** `.github/workflows/poetry-publish-nightly.yml`

**Triggers:**
- Scheduled (nightly cron)
- Manual dispatch

**Stages/Jobs:**

1. **Stage: Nightly Build & Publish**
   - **Purpose:** Publish nightly/dev build to PyPI or test PyPI
   - **Steps:** Build with dev version suffix, publish

---

### Pipeline: `fern-sdk-python-publish.yml` / `fern-sdk-typescript-publish.yml`

**Location:** `.github/workflows/fern-sdk-python-publish.yml`

**Triggers:**
- Manual dispatch
- Release tags

**Stages/Jobs:**

1. **Stage: Generate SDK**
   - **Purpose:** Use Fern to generate Python/TypeScript SDK from `fern/openapi.json`
   - **Steps:**
     1. Run Fern CLI against `fern/openapi.json` and `fern/openapi-overrides.yml`
     2. Generate client code
     3. Publish to PyPI / npm

---

### Pipeline: `fern-sdk-python-preview.yml` / `fern-sdk-typescript-preview.yml`

**Location:** `.github/workflows/fern-sdk-python-preview.yml`

**Triggers:**
- `pull_request` events

**Stages/Jobs:**

1. **Stage: Preview SDK Generation**
   - **Purpose:** Preview SDK changes without publishing — generates diff/preview
   - **Artifacts:** SDK preview (comment on PR or artifact)

---

### Pipeline: `fern-docs-publish.yml`

**Location:** `.github/workflows/fern-docs-publish.yml`

**Triggers:**
- Push to `main`
- Manual dispatch

**Stages/Jobs:**

1. **Stage: Publish Docs**
   - **Purpose:** Generate and deploy documentation site using Fern
   - **Steps:** Run Fern docs generator, publish to docs hosting

---

### Pipeline: `fern-check.yml`

**Location:** `.github/workflows/fern-check.yml`

**Triggers:**
- `pull_request` events

**Stages/Jobs:**

1. **Stage: Fern Check**
   - **Purpose:** Validate OpenAPI spec (`fern/openapi.json`) is valid before merge
   - **Steps:** Run `fern check`

---

### Pipeline: `send-message-integration-tests.yml`

**Location:** `.github/workflows/send-message-integration-tests.yml`

**Triggers:**
- Manual dispatch
- Possibly scheduled

**Stages/Jobs:**

1. **Stage: Send Message Integration Tests**
   - **Purpose:** End-to-end tests for the core `send_message` endpoint against real LLM providers
   - **Requires:** LLM API keys as secrets

---

### Pipeline: `test-ollama.yml` / `test-vllm.yml` / `test-lmstudio.yml`

**Triggers:** Manual dispatch

**Purpose:** Provider-specific integration tests for local LLM backends (Ollama, vLLM, LM Studio)

---

### Pipeline: `model-sweep.yaml`

**Location:** `.github/workflows/model-sweep.yaml`

**Triggers:**
- Scheduled (periodic sweep)
- Manual dispatch

**Stages/Jobs:**

1. **Stage: Model Sweep**
   - **Purpose:** Test against multiple LLM model configurations
   - **Steps:** Uses scripts in `.github/scripts/model-sweep/`
   - **Matrix:** Runs against configs in `tests/model_settings/` (27 model config files)

---

### Pipeline: `letta-code-sync.yml`

**Location:** `.github/workflows/letta-code-sync.yml`

**Triggers:**
- Push to `main`
- Manual dispatch

**Purpose:** Sync code changes to an external repository (likely a private/cloud version of Letta)

---

### Pipeline: `notify-on-update.yaml`

**Triggers:** Push/release events  
**Purpose:** Send notifications when the codebase is updated (webhook/Slack/email)

---

### Pipeline: `close_stale_issues.yml` / `manually_clear_old_issues.yml`

**Triggers:** Scheduled (daily/weekly)  
**Purpose:** Repository maintenance — auto-close stale GitHub issues

---

### Pipeline: `warn_poetry_updates.yml`

**Triggers:** `pull_request` events  
**Purpose:** Warn when `pyproject.toml` changes affect Poetry lockfile compatibility

---

### Pipeline: `test-pip-install.yml`

**Triggers:** Manual dispatch, push to `main`  
**Purpose:** Verify the package installs cleanly via `pip install letta`

---

## 5. Build Process

### Build Tools

| Tool | Purpose |
|------|---------|
| `uv` | Fast Python package installer/resolver (replaces pip+virtualenv) |
| `hatchling` | Build backend for Python package (`pyproject.toml`) |
| Docker Buildx | Multi-architecture container builds |
| Fern | SDK and documentation generation from OpenAPI spec |

### Docker Build: Multi-Stage

**Location:** `Dockerfile`

**Stage 1: Builder**
```
FROM pgvector/pgvector:0.8.1-pg15 AS builder
```
- Base: pgvector with PostgreSQL 15
- Installs Python 3.11, build tools, libpq-dev
- Creates `/opt/venv` virtual environment
- Copies `pyproject.toml`, `uv.lock`
- Copies full application source
- Runs `uv sync --frozen --no-dev --all-extras --python 3.11`

**Stage 2: Runtime**
```
FROM pgvector/pgvector:0.8.1-pg15 AS runtime
```
- Base: same pgvector image (notably NOT a minimal base)
- Installs: `curl`, `python3`, `python3-venv`, `libpq-dev`, `redis-server`, `nodejs` (v22)
- Downloads and installs **OpenTelemetry Collector** binary (`otelcol-contrib` v0.96.0)
- Copies OTel configs: `file`, `clickhouse`, `signoz` variants
- Copies virtual environment and app from builder stage
- Copies `init.sql` to PostgreSQL init directory

**Exposed Ports:**
- `8283` — Letta API server
- `5432` — PostgreSQL
- `6379` — Redis
- `4317` — OTel gRPC receiver
- `4318` — OTel HTTP receiver

**Entry Point:**
- `ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]`
- `CMD ["./letta/server/startup.sh"]`

### Build Arguments
| Arg | Default | Purpose |
|-----|---------|---------|
| `LETTA_ENVIRONMENT` | `DEV` | Runtime environment flag |
| `LETTA_VERSION` | (none) | Injected version string |
| `NODE_VERSION` | `22` | Node.js version |
| `OTEL_VERSION` | `0.96.0` | OTel collector version |
| `TARGETARCH` | `amd64` | Target CPU architecture |

### Python Package Build
- **Format:** Wheel + sdist
- **Backend:** `hatchling`
- **Version:** Defined in `pyproject.toml` as `0.16.7`
- **Package name:** `letta`
- **Entry point:** `letta = "letta.main:app"`

---

## 6. Infrastructure as Code

### Docker Compose Files

Multiple Docker Compose configurations exist for different purposes:

| File | Purpose |
|------|---------|
| `compose.yaml` | Primary production/standard compose |
| `dev-compose.yaml` | Development environment |
| `development.compose.yml` | Alternative dev compose |
| `docker-compose-vllm.yaml` | vLLM local LLM backend |
| `scripts/docker-compose.yml` | Utility services (Redis + PostgreSQL only) |

**`scripts/docker-compose.yml` Services:**

```yaml
redis:
  image: redis:alpine
  ports: 6379:6379
  volumes: ./data/redis:/data
  healthcheck: redis-cli ping | grep PONG
  
postgres:
  image: ankane/pgvector
  ports: 5432:5432
  environment:
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: postgres  # ⚠️ Hardcoded credential
    POSTGRES_DB: letta
  volumes:
    - ./data/postgres:/var/lib/postgresql/data
    - ./scripts/postgres-db-init/init.sql:/docker-entrypoint-initdb.d/init.sql
```

### Database Infrastructure

**Tool:** Alembic  
**Location:** `alembic/` directory, `alembic.ini`

**Migration History:** 150+ migration files in `alembic/versions/`

**State Management:**
- State tracked in database (`alembic_version` table)
- No remote state store (not Terraform — this is schema migration, not IaC)
- Validated in CI via `alembic-validation.yml`

---

## 7. Testing in the Deployment Pipeline

### Test Organization

```
tests/
├── Unit Tests          → core-unit-test.yml / core-unit-sqlite-test.yaml
├── Integration Tests   → core-integration-tests.yml
├── Docker Tests        → docker-integration-tests.yaml
├── Provider Tests      → send-message-integration-tests.yml
│                         test-ollama.yml / test-vllm.yml / test-lmstudio.yml
├── Migration Tests     → migration-test.yml / alembic-validation.yml
├── Model Sweep         → model-sweep.yaml
└── Pip Install Test    → test-pip-install.yml
```

### Test Backends

| Suite | Database Backend | Trigger |
|-------|-----------------|---------|
| Unit (SQLite) | SQLite | PR + Push |
| Unit (PostgreSQL) | PostgreSQL | PR + Push |
| Integration | PostgreSQL + Redis | PR + Push |
| Docker Integration | Full Docker Compose | PR + Push |

### Test Framework Configuration

**Location:** `tests/pytest.ini` and `letta/pytest.ini`

```ini
[pytest.ini_options]
asyncio_mode = "auto"  # in pyproject.toml
```

**Dev dependencies** (`pyproject.toml`):
- `pytest`
- `pytest-asyncio>=0.24.0`
- `pytest-order>=1.2.0`
- `pytest-mock>=3.14.0`
- `pytest-json-report>=1.5.0`

### Reusable Test Workflow

**Location:** `.github/workflows/reusable-test-workflow.yml`

Referenced by `core-integration-tests.yml` — provides a shared pattern for:
- Service container setup
- Dependency installation
- Test execution
- Result reporting

---

## 8. Release Management

### Version Control

**Versioning Scheme:** Semantic Versioning (SemVer)  
**Current Version:** `0.16.7` (defined in `pyproject.toml`)

**Release Triggers:**
- Git tag `v*` → triggers `poetry-publish.yml` (PyPI release)
- Git tag `v*` → triggers `docker-image.yml` (Docker image release)
- Git tag `v*` → triggers Fern SDK publish workflows

### Artifact Distribution

| Artifact | Registry | Workflow |
|----------|----------|---------|
| Python package | PyPI | `poetry-publish.yml` |
| Nightly Python package | PyPI (nightly) | `poetry-publish-nightly.yml` |
| Docker image | Container registry | `docker-image.yml` |
| Python SDK | PyPI (letta-client) | `fern-sdk-python-publish.yml` |

--- DBs ---


# Database Analysis: letta_a0979209

---

## Database 1: PostgreSQL

* **Database Name/Type:** PostgreSQL (SQL - Relational)

* **Purpose/Role:** Primary transactional database for the Letta AI agent framework. Stores all core application data including agents, users, organizations, messages, memory blocks, tools, sources, files, jobs, runs, steps, sandbox configurations, providers, LLM batch jobs, identities, groups, conversations, passages (archival memory), and associated metadata. Serves as the authoritative persistent store for all agent state and operational data.

* **Key Technologies/Access Methods:**
  * **SQLAlchemy ORM** — used extensively for all model definitions, CRUD operations, and relationship management
  * **Alembic** — for database schema migrations
  * **asyncpg / psycopg2** — as underlying PostgreSQL drivers
  * **pgvector** — extension used for vector similarity search on passage embeddings (archival memory)
  * Raw SQL used in `init.sql` for initialization and in select migration scripts

* **Key Files/Configuration:**
  * `alembic.ini` — Alembic migration configuration
  * `alembic/env.py` — Alembic environment setup, DB URL resolution
  * `alembic/versions/` — 150+ migration scripts defining the full schema history
  * `letta/orm/` — All SQLAlchemy ORM model definitions
  * `letta/server/db.py` — Database engine creation and session management
  * `letta/database_utils.py` — Database utility functions
  * `letta/settings.py` — `pg_uri` / `database_url` configuration settings
  * `.env.example` — Environment variable template including `LETTA_PG_URI` / `DATABASE_URL`
  * `init.sql` — Initial SQL setup (e.g., `CREATE EXTENSION IF NOT EXISTS vector`)
  * `db/run_postgres.sh` — Script to spin up a local Postgres instance
  * `db/Dockerfile.simple` — Docker image for the Postgres service
  * `compose.yaml` / `dev-compose.yaml` — Docker Compose including `db` PostgreSQL service
  * `tests/clear_postgres_db.py` — Test utility to reset the Postgres database

* **Schema/Table Structure:**

  | Table | Key Columns |
  |---|---|
  | `users` | `id` (PK), `name`, `organization_id` (FK), `created_at`, `updated_at`, `is_deleted` |
  | `organizations` | `id` (PK), `name`, `slug`, `privileged_tools`, `created_at`, `updated_at`, `is_deleted` |
  | `agents` | `id` (PK), `organization_id` (FK), `name`, `agent_type`, `llm_config` (JSON), `embedding_config` (JSON), `system`, `message_ids`, `tool_rules`, `metadata_`, `project_id`, `template_id`, `identifier_key`, `last_run_at`, `last_stop_reason`, `timezone`, `stateless`, `hidden`, `compaction_settings`, `created_at`, `updated_at`, `is_deleted` |
  | `blocks` | `id` (PK), `organization_id` (FK), `name`, `label`, `value`, `limit`, `is_template`, `preserve_on_migration`, `read_only`, `description`, `metadata_`, `project_id`, `hidden`, `created_at`, `updated_at`, `is_deleted` |
  | `blocks_agents` | `id` (PK), `agent_id` (FK → `agents`), `block_id` (FK → `blocks`), `block_label`, `is_template_block`, `created_at` |
  | `block_history` | `id` (PK), `block_id` (FK → `blocks`), `agent_id` (FK → `agents`), `value`, `sequence_number`, `created_at` |
  | `messages` | `id` (PK), `organization_id` (FK), `agent_id` (FK → `agents`), `role`, `content` (JSON), `model`, `name`, `tool_calls` (JSON), `tool_call_id`, `step_id` (FK → `steps`), `batch_item_id`, `otid`, `sender_id`, `created_at` |
  | `tools` | `id` (PK), `organization_id` (FK), `name`, `description`, `source_code`, `source_type`, `tool_type`, `json_schema` (JSON), `args_schema` (JSON), `module`, `tags` (JSON), `pip_requirements`, `npm_requirements`, `metadata_`, `project_id`, `enable_parallel_execution`, `return_char_limit`, `created_at`, `updated_at`, `is_deleted` |
  | `tools_agents` | `id` (PK), `agent_id` (FK → `agents`), `tool_id` (FK → `tools`) |
  | `sources` | `id` (PK), `organization_id` (FK), `name`, `description`, `instructions`, `embedding_config` (JSON), `metadata_`, `vector_db_provider`, `created_at`, `updated_at`, `is_deleted` |
  | `source_passages` | `id` (PK), `organization_id` (FK), `agent_id`, `source_id` (FK → `sources`), `file_id` (FK → `files`), `text`, `embedding` (vector), `embedding_config` (JSON), `metadata_`, `file_name`, `is_deleted` |
  | `agent_passages` | `id` (PK), `organization_id` (FK), `agent_id` (FK → `agents`), `source_id`, `file_id`, `text`, `embedding` (vector), `embedding_config` (JSON), `metadata_`, `file_name`, `is_deleted` |
  | `passage_tags` | `id` (PK), `passage_id` (FK → passages), `tag`, `created_at` |
  | `sources_agents` | `id` (PK), `agent_id` (FK → `agents`), `source_id` (FK → `sources`) |
  | `files` | `id` (PK), `organization_id` (FK), `source_id` (FK → `sources`), `file_name`, `file_path`, `file_type`, `file_size`, `processing_status`, `total_chunks`, `chunks_embedded`, `start_byte`, `end_byte`, `created_at`, `updated_at`, `is_deleted` |
  | `files_agents` | `id` (PK), `agent_id` (FK → `agents`), `file_id` (FK → `files`), `source_id`, `file_name`, `start_byte`, `end_byte`, `created_at` |
  | `jobs` | `id` (PK), `organization_id` (FK), `user_id` (FK → `users`), `status`, `job_type`, `metadata_` (JSON), `request_config` (JSON), `callback_url`, `callback_data`, `callback_error`, `stop_reason`, `created_at`, `updated_at`, `completed_at`, `is_deleted` |
  | `job_messages` | `id` (PK), `job_id` (FK → `jobs`), `message_id` (FK → `messages`), `created_at` |
  | `runs` | `id` (PK), `organization_id` (FK), `agent_id` (FK → `agents`), `job_id` (FK → `jobs`), `status`, `usage` (JSON), `metadata_`, `created_at`, `updated_at`, `completed_at`, `duration_ms` |
  | `run_metrics` | `id` (PK), `run_id` (FK → `runs`), `tools_used` (JSON), `created_at` |
  | `agents_runs` | `id` (PK), `agent_id` (FK → `agents`), `run_id` (FK → `runs`) |
  | `steps` | `id` (PK), `organization_id` (FK), `agent_id` (FK → `agents`), `run_id` (FK → `runs`), `origin`, `model`, `model_endpoint`, `context_window_size`, `completion_tokens`, `prompt_tokens`, `total_tokens`, `prompt_tokens_details` (JSON), `steps_count`, `provider_name`, `provider_category`, `trace_id`, `request_id`, `error`, `error_type`, `stop_reasons` (JSON), `build_request_latency_ms`, `created_at` |
  | `step_metrics` | `id` (PK), `step_id` (FK → `steps`), `run_id` (FK → `runs`), `created_at` |
  | `sandbox_configs` | `id` (PK), `organization_id` (FK), `type`, `config` (JSON), `created_at`, `updated_at`, `is_deleted` |
  | `sandbox_env_vars` | `id` (PK), `organization_id` (FK), `sandbox_config_id` (FK → `sandbox_configs`), `key`, `value` (encrypted), `description`, `created_at`, `updated_at`, `is_deleted` |
  | `providers` | `id` (PK), `organization_id` (FK), `name`, `provider_type`, `api_key` (encrypted), `base_url`, `api_version`, `last_synced`, `created_at`, `updated_at`, `is_deleted` |
  | `provider_traces` | `id` (PK), `organization_id` (FK), `agent_id`, `step_id`, `request` (JSON), `response` (JSON), `source`, `created_at` |
  | `provider_trace_metadata` | `id` (PK), `provider_trace_id` (FK → `provider_traces`), `key`, `value`, `created_at` |
  | `identities` | `id` (PK), `organization_id` (FK), `identifier_key`, `identity_type`, `name`, `properties` (JSON/JSONB), `project_id`, `created_at`, `updated_at`, `is_deleted` |
  | `identities_agents` | `id` (PK), `agent_id` (FK → `agents`), `identity_id` (FK → `identities`) |
  | `identities_blocks` | `id` (PK), `block_id` (FK → `blocks`), `identity_id` (FK → `identities`) |
  | `groups` | `id` (PK), `organization_id` (FK), `name`, `description`, `agent_ids` (JSON), `manager_type`, `manager_config` (JSON), `ordered_agent_ids` (JSON), `hidden`, `created_at`, `updated_at`, `is_deleted` |
  | `groups_agents` | `id` (PK), `group_id` (FK → `groups`), `agent_id` (FK → `agents`) |
  | `groups_blocks` | `id` (PK), `group_id` (FK → `groups`), `block_id` (FK → `blocks`) |
  | `conversations` | `id` (PK), `organization_id` (FK), `name`, `description`, `metadata_` (JSON), `llm_config` (JSON), `last_message_at`, `created_at`, `updated_at`, `is_deleted` |
  | `conversation_messages` | `id` (PK), `conversation_id` (FK → `conversations`), `message_id` (FK → `messages`) |
  | `blocks_conversations` | `id` (PK), `conversation_id` (FK → `conversations`), `block_id` (FK → `blocks`), `block_label` |
  | `agents_tags` | `id` (PK), `agent_id` (FK → `agents`), `tag`, `created_at` |
  | `blocks_tags` | `id` (PK), `block_id` (FK → `blocks`), `tag`, `created_at` |
  | `llm_batch_jobs` | `id` (PK), `organization_id` (FK), `letta_batch_job_id` (FK → `jobs`), `status`, `llm_provider`, `create_batch_request` (JSON), `batch_response` (JSON), `created_at`, `updated_at` |
  | `llm_batch_items` | `id` (PK), `organization_id` (FK), `llm_batch_job_id` (FK → `llm_batch_jobs`), `agent_id` (FK → `agents`), `batch_request_item` (JSON), `batch_response_item` (JSON), `request_status`, `step_state` (JSON), `created_at`, `updated_at` |
  | `mcp_servers` | `id` (PK), `organization_id` (FK), `name`, `server_type`, `config` (JSON/encrypted), `token` (encrypted), `custom_headers` (JSON), `created_at`, `updated_at`, `is_deleted` |
  | `mcp_oauth` | `id` (PK), `organization_id` (FK), `mcp_server_id` (FK → `mcp_servers`), `provider`, `oauth_data` (JSON/encrypted), `created_at`, `updated_at` |
  | `prompts` | `id` (PK), `organization_id` (FK), `name`, `description`, `text`, `created_at`, `updated_at`, `is_deleted` |
  | `provider_models` | `id` (PK), `organization_id` (FK), `provider_id` (FK → `providers`), `model_name`, `config` (JSON), `created_at`, `updated_at`, `is_deleted` |
  | `archives` | `id` (PK), `organization_id` (FK), `name`, `description`, `embedding_config` (JSON), `vector_db_provider`, `created_at`, `updated_at`, `is_deleted` |
  | `archives_agents` | `id` (PK), `archive_id` (FK → `archives`), `agent_id` (FK → `agents`) |
  | `agent_environment_variables` | `id` (PK), `agent_id` (FK → `agents`), `key`, `value` (encrypted), `description`, `created_at`, `updated_at` |

* **Key Entities and Relationships:**
  * **Organization** — Top-level tenant; owns all other entities
  * **User** — Belongs to an Organization; initiates jobs and runs
  * **Agent** — Core AI agent entity; belongs to Organization; has many Blocks, Tools, Sources, Messages, Tags, Identities, Files; participates in Groups and Conversations
  * **Block** — Memory block (core memory); many-to-many with Agents; has history tracking
  * **Message** — Conversation message; belongs to Agent; many-to-many with Jobs; part of Conversations
  * **Tool** — Executable function; many-to-many with Agents; belongs to Organization
  * **Source** — Data source for archival memory; many-to-many with Agents; has Files and Passages
  * **Passage** — Vectorized text chunk (archival memory); stored with pgvector embedding; belongs to Source/Agent
  * **File** — File attached to a Source; has Passages; many-to-many with Agents
  * **Job** — Background job tracking; has Messages, an LLM Batch Job; belongs to Organization/User
  * **Run** — Agent execution run; belongs to Agent and Job; has Steps and Metrics
  * **Step** — Single LLM inference step within a Run; has Step Metrics
  * **Group** — Multi-agent group; many-to-many with Agents and Blocks
  * **Conversation** — Named conversation thread; many-to-many with Messages and Blocks
  * **Identity** — External identity (user persona); many-to-many with Agents and Blocks
  * **Provider** — LLM provider configuration (BYOK); has Provider Models and Provider Traces
  * **SandboxConfig** — Tool execution sandbox configuration; has SandboxEnvVars
  * **LLMBatchJob** → LLMBatchItems → Agents (batch inference pipeline)
  * **MCPServer** — MCP tool server config; has MCPOAuth records

  **Key Relationships:**
  - `Organization` (1) ──── (M) `Agent`, `User`, `Tool`, `Source`, `Block`, `Job`, `Provider`, `Identity`, `Group`, `Conversation`
  - `Agent` (M) ──── (M) `Block` via `blocks_agents`
  - `Agent` (M) ──── (M) `Tool` via `tools_agents`
  - `Agent` (M) ──── (M) `Source` via `sources_agents`
  - `Agent` (M) ──── (M) `File` via `files_agents`
  - `Agent` (M) ──── (M) `Identity` via `identities_agents`
  - `Agent` (M) ──── (M) `Group` via `groups_agents`
  - `Agent` (1) ──── (M) `Message`, `Step`, `Run`, `AgentPassage`
  - `Source` (1) ──── (M) `File`, `SourcePassage`
  - `Job` (1) ──── (M) `Run`; (M) ──── (M) `Message` via `job_messages`
  - `Run` (1) ──── (M) `Step`, `RunMetrics`
  - `Step` (1) ──── (M) `Message`
  - `LLMBatchJob` (1) ──── (M) `LLMBatchItem`
  - `Provider` (1) ──── (M) `ProviderModel`, `ProviderTrace`

* **Interacting Components:**
  * `letta/services/agent_manager.py` — Agent CRUD and lifecycle
  * `letta/services/message_manager.py` — Message persistence
  * `letta/services/block_manager.py` — Memory block management
  * `letta/services/tool_manager.py` — Tool management
  * `letta/services/source_manager.py` — Source and data ingestion management
  * `letta/services/passage_manager.py` — Archival memory / vector passage management
  * `letta/services/job_manager.py` — Background job tracking
  * `letta/services/run_manager.py` — Run tracking
  * `letta/services/step_manager.py` — Step tracking
  * `letta/services/organization_manager.py` — Organization management
  * `letta/services/user_manager.py` — User management
  * `letta/services/provider_manager.py` — LLM provider management
  * `letta/services/sandbox_config_manager.py` — Sandbox configuration
  * `letta/services/group_manager.py` — Multi-agent group management
  * `letta/services/identity_manager.py` — Identity management
  * `letta/services/conversation_manager.py` — Conversation management
  * `letta/services/file_manager.py` — File management
  * `letta/services/llm_batch_manager.py` — Batch LLM job management
  * `letta/services/mcp_server_manager.py` — MCP server management
  * `letta/server/server.py` — Main server, orchestrates all services
  * `letta/server/db.py` — Session factory and engine setup

---

## Database 2: SQLite

* **Database Name/Type:** SQLite (SQL - Relational, Embedded)

* **Purpose/Role:** Lightweight, file-based alternative to PostgreSQL used primarily for local development, unit testing, and single-user deployments. Uses the same SQLAlchemy ORM models and Alembic migrations as PostgreSQL, providing a zero-configuration database backend. A separate baseline migration (`2c059cad97cc_create_sqlite_baseline_schema.py`) exists specifically for SQLite compatibility.

* **Key Technologies/Access Methods:**
  * **SQLAlchemy ORM** — same ORM models as PostgreSQL; SQLite dialect used automatically based on `DATABASE_URL`
  * **Alembic** — same migration tooling; SQLite-specific baseline migration provided
  * `letta/orm/sqlite_functions.py` — Custom SQLite-compatible SQL functions (e.g., vector similarity implementations that substitute for pgvector)
  * Python's built-in `sqlite3` module as the underlying driver

* **Key Files/Configuration:**
  * `letta/settings.py` — `database_url` setting; defaults to a local SQLite file path when no PostgreSQL URI is provided (e.g., `sqlite:///~/.letta/letta.db`)
  * `letta/server/db.py` — Engine creation logic that branches between SQLite and PostgreSQL based on the database URL
  * `letta/orm/sqlite_functions.py` — SQLite-specific function overrides (e.g., vector operations)
  * `alembic/versions/2c059cad97cc_create_sqlite_baseline_schema.py` — SQLite baseline schema migration
  * `.github/workflows/core-unit-sqlite-test.yaml` — CI workflow specifically running unit tests against SQLite
  * `letta/pytest.ini` — Test configuration referencing SQLite for unit tests

* **Schema/Table Structure:**
  * Identical table definitions to PostgreSQL (see PostgreSQL section above), with the following key differences:
    * **No pgvector extension** — vector similarity search is handled by `letta/orm/sqlite_functions.py` which registers custom Python functions to perform cosine similarity in-process
    * **No native JSONB** — JSON columns stored as TEXT
    * **No `CONCURRENTLY` index creation** — some migration scripts have SQLite-specific conditional logic to skip unsupported DDL operations
    * The SQLite baseline migration (`2c059cad97cc`) provides the initial schema for fresh SQLite installations

* **Key Entities and Relationships:**
  * Same as PostgreSQL. All entities (Agent, Block, Message, Tool, Source, Passage, Job, Run, Step, etc.) and their relationships are identical, as the same ORM models are used.

* **Interacting Components:**
  * All the same service components as PostgreSQL (see above)
  * `letta/server/db.py` — Contains the connection logic that selects SQLite vs PostgreSQL
  * `letta/orm/sqlite_functions.py` — SQLite-specific runtime functions registered with the engine
  * All unit tests under `tests/` when run in SQLite mode

---

## Database 3: Redis

* **Database Name/Type:** Redis (NoSQL - Key-Value / Pub-Sub Store)

* **Purpose/Role:** Used as a distributed data source connector for ingesting data from Redis streams or key-value stores into Letta agent memory (archival memory / sources). Also referenced in test infrastructure. Based on `letta/data_sources/redis_client.py` and `tests/test_redis_client.py`, Redis acts as an **external data source** from which the Letta system reads data to populate agent knowledge bases, rather than as an internal caching or session layer.

* **Key Technologies/Access Methods:**
  * **`redis` Python client library** (`redis-py`) — used in `letta/data_sources/redis_client.py`
  * Connection via `redis://` URL, configurable via environment variables or settings
  * Data read from Redis keys/streams and transformed into `Passage` objects for storage in the primary SQL database

* **Key Files/Configuration:**
  * `letta/data_sources/redis_client.py` — Redis client wrapper and data source connector implementation
  * `tests/test_redis_client.py` — Unit/integration tests for the Redis data source client
  * `.env.example` — May contain `REDIS_URL` or similar environment variable

* **Schema/Table Structure (NoSQL - Key-Value):**
  * Redis is used as an **external data ingestion source**, not as a structured internal store. The specific key patterns depend on the external Redis instance being connected to.
  * The connector reads values from Redis keys and wraps

--- authorization ---


# Authorization Analysis: Letta Repository

## Executive Summary

After analyzing the Letta codebase, a **partially implemented authorization system** is present, centered primarily on **organization-based tenant isolation** with **API key authentication** driving authorization decisions. The system lacks formal RBAC, has minimal permission enforcement at the service layer, and has several notable security gaps.

---

## 1. Access Control Type

### Organization-Based Tenant Isolation (Primary Model)

**Location:** `letta/server/rest_api/auth/` and throughout service layer

The primary authorization model is organization-scoped resource access — all resources belong to an organization, and access is controlled by verifying the requesting user/actor belongs to the same organization.

```
letta/server/rest_api/auth/
├── (auth directory - token extraction and actor resolution)
letta/server/rest_api/middleware/
├── (middleware directory)
```

**Location:** `letta/orm/mixins.py`

The ORM mixin enforces organization scoping at the database query level:

```python
# letta/orm/mixins.py - OrganizationMixin
class OrganizationMixin:
    organization_id: Mapped[str] = mapped_column(...)
    
# All queries filter by organization_id
```

**Location:** `letta/services/agent_manager.py` (representative of all service managers)

```python
# All service methods accept actor: PydanticUser parameter
def get_agent_by_id(self, agent_id: str, actor: PydanticUser) -> PydanticAgentState:
    with self.db_session() as session:
        agent = AgentModel.read(db_session=session, identifier=agent_id, actor=actor)
```

---

## 2. Authentication → Authorization Pipeline

### API Key → Actor Resolution

**Location:** `letta/server/rest_api/auth/` directory

The core authorization flow: API key presented → resolved to a `PydanticUser` actor → actor's `organization_id` used to scope all resource access.

**Location:** `letta/server/server.py`

```python
# Actor resolution pattern used throughout
async def get_current_user(token: str = Depends(oauth2_scheme)) -> PydanticUser:
    # API key lookup → user resolution → organization assignment
```

**Location:** `letta/schemas/user.py`

```python
class PydanticUser(BaseModel):
    id: str
    organization_id: str  # Core authorization attribute
    # No role field present - no RBAC
```

**Location:** `letta/schemas/organization.py`

```python
class PydanticOrganization(BaseModel):
    id: str
    name: str
    privileged_tools: Optional[List[str]]  # Organization-level capability control
```

---

## 3. Permission Enforcement Points

### 3.1 ORM-Level Authorization (Primary Enforcement)

**Location:** `letta/orm/base.py` and `letta/orm/mixins.py`

The most consistent authorization enforcement occurs at the ORM read/write layer:

```python
# letta/orm/base.py - Base class read method
@classmethod
def read(cls, db_session: Session, identifier: str, actor: PydanticUser, **kwargs):
    result = db_session.get(cls, identifier)
    if result is None:
        raise NoResultFound(f"{cls.__name__} not found")
    # Organization check
    if hasattr(result, 'organization_id') and result.organization_id != actor.organization_id:
        raise NoResultFound(f"{cls.__name__} not found")  # Returns 404 not 403 (obscures existence)
    return result
```

**Coverage:** Agents, Tools, Sources, Blocks, Groups, Jobs, Messages, Passages, Identities, MCP Servers, Providers, Sandbox Configs

**Implementation Pattern:**
- Every ORM model with `OrganizationMixin` automatically scopes queries to the actor's organization
- Cross-organization access returns `NoResultFound` (not an authorization error — intentional obscuring)

### 3.2 Service Layer Authorization

**Location:** All files in `letta/services/`

Every service method accepts and enforces an `actor: PydanticUser` parameter:

| Service File | Authorization Pattern |
|---|---|
| `letta/services/agent_manager.py` | Actor org-scoped queries |
| `letta/services/tool_manager.py` | Actor org-scoped + tool type checks |
| `letta/services/source_manager.py` | Actor org-scoped queries |
| `letta/services/block_manager.py` | Actor org-scoped queries |
| `letta/services/user_manager.py` | Actor org-scoped queries |
| `letta/services/organization_manager.py` | Actor-based org access |
| `letta/services/job_manager.py` | Actor org-scoped queries |
| `letta/services/mcp_server_manager.py` | Actor org-scoped queries |
| `letta/services/sandbox_config_manager.py` | Actor org-scoped queries |

### 3.3 Router-Level Authorization (Middleware)

**Location:** `letta/server/rest_api/routers/`

Authorization is applied uniformly at the FastAPI dependency injection level — every protected route requires a valid actor:

```python
# Typical pattern across all routers
@router.get("/agents/{agent_id}")
async def get_agent(
    agent_id: str,
    actor: PydanticUser = Depends(get_current_user)  # Auth gate
):
    return server.agent_manager.get_agent_by_id(agent_id=agent_id, actor=actor)
```

**Location:** `letta/server/rest_api/routers/` — key router files:

```
v1/agents.py          - Agent CRUD + messaging
v1/tools.py           - Tool management  
v1/sources.py         - Data source management
v1/blocks.py          - Memory block management
v1/groups.py          - Multi-agent group management
v1/jobs.py            - Background job management
v1/users.py           - User management
v1/organizations.py   - Organization management
v1/providers.py       - LLM provider management
v1/identities.py      - Identity management
v1/runs.py            - Run management
v1/steps.py           - Step management
v1/health.py          - Health check (typically unprotected)
```

---

## 4. Privileged Operations & Tool Authorization

### 4.1 Privileged Tools System

**Location:** `letta/orm/organization.py`

```python
class OrganizationModel(OrganizationMixin, Base):
    privileged_tools: Mapped[Optional[List[str]]] = mapped_column(
        JSON, nullable=True, default=None
    )
```

**Location:** `letta/services/tool_manager.py`

```python
# Tool access checks for privileged tools
def get_tool_by_name(self, tool_name: str, actor: PydanticUser) -> PydanticTool:
    # Checks if tool is in organization's privileged_tools list
    # before allowing access
```

**Location:** Alembic migration `bdddd421ec41_add_privileged_tools_to_organization.py`

This is a capability-based authorization mechanism at the organization level — certain tools (likely with elevated system access) require explicit allowlisting per organization.

### 4.2 Tool Type-Based Authorization

**Location:** `letta/schemas/tool.py` and `letta/schemas/enums.py`

```python
class ToolType(str, Enum):
    CUSTOM = "custom"
    LETTA_CORE = "letta_core"          # Built-in Letta tools
    LETTA_MEMORY_CORE = "letta_memory_core"
    LETTA_MULTI_AGENT_CORE = "letta_multi_agent_core"
    LETTA_SLEEPTIME_CORE = "letta_sleeptime_core"
    LETTA_VOICE_SLEEPTIME_CORE = "letta_voice_sleeptime_core"
    COMPOSIO = "composio"
    MCP = "mcp"
    LETTA_FILES_CORE = "letta_files_core"
```

**Location:** `letta/services/tool_manager.py`

Tool type is used to determine visibility and access — system/core tools have different access patterns than custom tools.

---

## 5. Resource Ownership Model

### 5.1 Organization Ownership (Primary)

All resources are owned by organizations, not individual users. The `organization_id` is the fundamental access boundary.

**Location:** `letta/orm/mixins.py`

```python
class OrganizationMixin:
    """All resources belong to an organization"""
    organization_id: Mapped[str] = mapped_column(
        String, ForeignKey("organizations.id"), nullable=False, index=True
    )
```

### 5.2 User-Organization Mapping

**Location:** `letta/orm/user.py`

```python
class UserModel(Base):
    organization_id: Mapped[str]  # Each user belongs to one organization
    # No role field — flat user model within org
```

### 5.3 Agent Ownership Metadata

**Location:** `letta/orm/agent.py`

```python
class AgentModel(OrganizationMixin, Base):
    # created_by_id tracks creator but doesn't enforce ownership-based access
    # Access is purely org-scoped
```

---

## 6. Special Authorization Paths

### 6.1 Server-Mode vs. No-Auth Mode

**Location:** `letta/settings.py`

```python
class Settings(BaseSettings):
    letta_server_pass: Optional[str] = None  # If None, auth may be bypassed
    server_mode: bool = False
```

**Location:** `letta/server/rest_api/auth/` 

The server supports a mode where authentication (and therefore authorization) can be disabled entirely — designed for local/development use but creates risk if deployed without configuration.

### 6.2 Default Organization/User for Single-Tenant Mode

**Location:** `letta/server/server.py` and `letta/services/organization_manager.py`

```python
# DEFAULT_ORG_ID and DEFAULT_USER_ID constants used in single-tenant mode
DEFAULT_ORG_ID = "org-00000000-0000-4000-8000-000000000000"
DEFAULT_USER_ID = "user-00000000-0000-4000-8000-000000000000"
```

In single-tenant/development mode, all requests are attributed to a default user/org, effectively bypassing multi-tenant isolation.

### 6.3 Health Endpoint

**Location:** `letta/server/rest_api/routers/v1/health.py`

```python
# Health endpoint - no authentication required
@router.get("/health")
async def health():
    return {"status": "ok"}
```

---

## 7. Database Schema Authorization Tables

### Authorization-Relevant Tables

| Table | Purpose | Auth Relevance |
|---|---|---|
| `organizations` | Tenant container | Primary isolation boundary |
| `users` | User accounts | Actor identity |
| `agents` | AI agents | Org-scoped resource |
| `tools` | Tool definitions | Org-scoped + privileged_tools list |
| `sources` | Data sources | Org-scoped resource |
| `blocks` | Memory blocks | Org-scoped resource |
| `groups` | Multi-agent groups | Org-scoped resource |
| `sandbox_configs` | Execution environments | Org-scoped resource |
| `mcp_servers` | MCP server configs | Org-scoped resource |
| `providers` | LLM providers | Org-scoped resource |
| `identities` | External identities | Org-scoped resource |

**Notable:** No dedicated `roles`, `permissions`, `user_roles`, or `role_permissions` tables exist. Authorization is purely organizational.

### Key ORM Relationships for Authorization

**Location:** `letta/orm/`

```
organizations (1) ──── (many) users
organizations (1) ──── (many) agents
organizations (1) ──── (many) tools  
organizations (1) ──── (many) sources
organizations (1) ──── (many) blocks
organizations (1) ──── (many) groups
organizations (1) ──── (many) sandbox_configs
organizations (1) ──── (many) mcp_servers
organizations (1) ──── (many) providers
```

---

## 8. MCP OAuth Authorization

**Location:** `letta/orm/mcp_oauth.py` and `letta/services/mcp_manager.py`

```python
# MCP servers have OAuth tokens stored encrypted
# Access to MCP server configs is org-scoped
# OAuth credentials require org membership to access
```

**Location:** Alembic migration `f5d26b0526e8_add_mcp_oauth.py`

MCP server OAuth credentials are encrypted at rest and scoped to organizations.

---

## 9. Encrypted Credentials Authorization

**Location:** `letta/helpers/crypto_utils.py` and Alembic migrations `b6061da886ee_add_encrypted_columns.py`

```python
# BYOK (Bring Your Own Key) provider credentials
# Stored encrypted, access controlled by organization membership
# Fields: api_key_encrypted, aws_access_key_encrypted, etc.
```

Access to provider API keys requires org membership — no separate key-management role system exists.

---

## 10. Security Gaps & Vulnerabilities

### 🔴 Critical Gaps

#### Gap 1: No Role-Based Access Control Within Organizations

**Issue:** All users within an organization have identical permissions. No concept of admin vs. read-only vs. operator roles exists within an org.

**Impact:** Any user with an org API key can:
- Delete all agents in the organization
- Modify all tools including system tools
- Export all data sources
- Modify sandbox configurations (code execution environments)
- Manage all MCP server credentials

**Location:** `letta/schemas/user.py` — no `role` field present

#### Gap 2: Authentication Can Be Completely Disabled

**Location:** `letta/settings.py` and `letta/server/rest_api/auth/`

```python
# When letta_server_pass is not set and server_mode=False,
# authentication enforcement is minimal/absent
```

**Impact:** Deployments without explicit configuration of `letta_server_pass` may expose all API endpoints without any access control.

#### Gap 3: No Authorization on Server-Internal Calls

**Location:** `letta/server/server.py`

The `SyncServer` class has methods called directly (not via REST API) where actor parameters may use the default system user, bypassing organizational controls:

```python
# Internal server calls sometimes use default actor
# without validating the calling context
```

### 🟠 High Severity Gaps

#### Gap 4: Sandbox Configuration Authorization

**Location:** `letta/services/sandbox_config_manager.py`

Sandbox configurations control code execution environments. While org-scoped, any org member can modify execution environments affecting all agents in the org — a significant lateral movement risk.

#### Gap 5: No Field-Level Authorization

**Location:** All service managers

Sensitive fields (API keys, encrypted credentials, internal configurations) are returned in full to any authenticated org member. No field-level filtering based on sensitivity or user context.

**Example:** `letta/services/provider_manager.py` — provider objects containing BYOK credentials are returned to all org members.

#### Gap 6: Tool Execution Authorization Gap

**Location:** `letta/services/tool_executor/`

Tools are executed in agent context. While agent access is org-scoped, the authorization of what a specific agent can execute (beyond the tools attached to it) is not independently verified at execution time.

#### Gap 7: Missing Authorization on Batch Operations

**Location:** `letta/services/llm_batch_manager.py`

Batch LLM jobs have limited explicit authorization validation — relies primarily on the initial agent ownership check without re-validating at job status polling time.

### 🟡 Medium Severity Gaps

#### Gap 8: 404 vs 403 Confusion

**Location:** `letta/orm/base.py`

Cross-organization resource access returns `NoResultFound` (404) rather than a 403 Forbidden. While this is a deliberate security-through-obscurity pattern to prevent resource enumeration, it makes authorization failures invisible in audit logs.

#### Gap 9: No Audit Logging of Authorization Decisions

**Location:** Throughout the codebase

Authorization decisions (grants and denials) are not logged to any audit trail. OTel tracing exists (`letta/otel/`) but does not capture authorization events specifically.

#### Gap 10: Default User Privilege Escalation Risk

**Location:** `letta/server/server.py`

```python
# The default_user has organization_id = DEFAULT_ORG_ID
# In production multi-tenant mode, if any code path
# accidentally uses default_user as actor, it could
# access the default org's resources
```

#### Gap 11: No Rate Limiting Per User/Org

**Location:** `letta/server/rest_api/routers/`

No per-user or per-organization rate limiting exists in the authorization layer. The `nginx.conf` may provide some rate limiting at the infrastructure level, but application-level quotas are absent.

#### Gap 12: Privileged Tools List Not Validated at Runtime

**Location:** `letta/services/tool_manager.py` and `letta/orm/organization.py`

The `privileged_tools` list on organizations is stored as a JSON column. There is no validation that tools in this list are actually valid privileged tool names, nor is there runtime enforcement checking this list consistently across all tool access paths.

---

## 11. Security Observations

### What Is Implemented Well

| Mechanism | Assessment |
|---|---|
| Organization-level tenant isolation | ✅ Consistently applied via ORM mixins |
| API key-based authentication | ✅ Present across all protected routes |
| Encrypted credential storage | ✅ BYOK keys and MCP OAuth tokens encrypted at rest |
| Resource not found obscuring | ✅ 404 prevents org resource enumeration |
| Org-ID propagation | ✅ Actor passed to all service layer methods |

### What Is Missing

| Missing Control | Risk Level |
|---|---|
| Intra-org RBAC | 🔴 Critical |
| Authorization audit logging | 🟠 High |
| Field-level access control | 🟠 High |
| Rate limiting by user/org | 🟡 Medium |
| Privilege escalation detection | 🟡 Medium |
| Time-limited access | 🟡 Medium |
| Admin impersonation controls | 🟡 Medium |
| Cross-resource ownership validation | 🟡 Medium |

---

## 12. Summary Architecture Diagram

```
Request
  │
  ▼
FastAPI Router
  │
  ├─► Depends(get_current_user)
  │       │
  │       ▼
  │   API Key Lookup
  │       │
  │       ▼
  │   PydanticUser{organization_id}  ◄── SINGLE AUTH ATTRIBUTE
  │
  ▼
Service Layer (actor: PydanticUser)
  │
  ▼
ORM Layer
  │
  ├─► OrganizationMixin filter (WHERE organization_id = actor.organization_id)
  │
  └─► Data returned / NoResultFound (404) if cross-org access
```

**Key Finding:** The authorization system is effectively a **single-dimension access control** — organization membership is the sole authorization variable. This is appropriate for a strict single-tenant-per-org SaaS model but provides no granularity within an organization, creating significant risk if multiple users with different trust levels share an organization.

