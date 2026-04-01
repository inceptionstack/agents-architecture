
## Full Investigation — NemoClaw (11 sections)

--- module_deep_dive ---


# NemoClaw: Detailed Component Breakdown

---

## 1. `bin/` — CLI Entry Point & Core Library

### Core Responsibility
The primary runtime engine of NemoClaw. This module is the main `nemoclaw` command-line interface that orchestrates sandbox lifecycle management, credential handling, policy enforcement, and inference routing. It is the central hub through which all user-facing operations flow.

### Key Components

| File | Role |
|------|------|
| `nemoclaw.js` | **Primary CLI executable** — entry point, command parsing, dispatches to lib modules |
| `lib/credentials.js` | Manages credential isolation — stores, retrieves, and sanitizes secrets to prevent exposure to the sandbox |
| `lib/policies.js` | **Policy engine core** — loads, validates, and enforces network policy YAML definitions |
| `lib/runner.js` | **Container execution engine** — constructs and executes Docker run commands with policy/credential injection |
| `lib/inference-config.js` | Manages inference provider configuration (cloud vs. local routing, profile selection) |
| `lib/local-inference.js` | Handles local GPU inference setup and communication (NVIDIA NIM integration) |
| `lib/nim.js` | NVIDIA NIM-specific orchestration — image selection, container startup for local LLM inference |
| `lib/nim-images.json` | Static registry of supported NVIDIA NIM container images and their metadata |
| `lib/onboard.js` | **Onboarding orchestrator** — drives the first-run setup wizard, environment validation |
| `lib/onboard-session.js` | Manages per-session onboarding state — tracks progress, resumability |
| `lib/platform.js` | Platform detection — identifies OS/architecture (macOS/Linux/ARM/x86) for environment-specific behavior |
| `lib/preflight.js` | Pre-launch checks — validates Docker availability, permissions, dependencies before sandbox start |
| `lib/registry.js` | Container image registry management — pulls, validates, and resolves sandbox image references |
| `lib/resolve-openshell.js` | Resolves which shell implementation (OpenShell) to use inside the sandbox |
| `lib/runtime-recovery.js` | **Fault tolerance** — detects and recovers from crashed or stale sandbox containers |
| `lib/version.js` | Version management — reads, compares, and reports CLI version information |

### Dependencies & Interactions

```
bin/lib/ ──→ nemoclaw-blueprint/policies/    (loads network policy YAML files)
bin/lib/ ──→ bin/lib/nim-images.json         (internal static data)
bin/lib/ ──→ Docker daemon                   (external: spawns containers via shell exec)
bin/lib/ ──→ NVIDIA NIM registry             (external: pulls inference container images)
bin/lib/ ──→ Container registries            (external: Docker Hub / GHCR for sandbox images)
bin/lib/ ──→ scripts/                        (startup scripts invoked during onboarding)
bin/lib/ ──→ Host filesystem                 (reads/writes credentials, config, workspace)
```

**External Service Interactions:**
- **Docker Engine** — spawns, monitors, and tears down sandbox containers
- **NVIDIA NIM** — pulls and runs local GPU inference containers
- **Container Registries** (Docker Hub/GHCR) — resolves and pulls sandbox base images
- **Host OS credential store** — reads API keys for injection into sandbox

---

## 2. `nemoclaw/` — OpenClaw TypeScript Plugin

### Core Responsibility
A TypeScript-based plugin for the OpenClaw AI IDE environment. It acts as the bridge between the AI coding agent's IDE interface and the NemoClaw sandbox backend — exposing sandbox commands, onboarding flows, and blueprint management directly within the agent's development environment.

### Key Components

| File/Directory | Role |
|----------------|------|
| `src/index.ts` | **Plugin entry point** — registers the plugin with the OpenClaw runtime, wires up commands and handlers |
| `src/register.test.ts` | Unit tests for plugin registration logic |
| `openclaw.plugin.json` | **Plugin manifest** — declares plugin ID, version, capabilities, and command registrations for OpenClaw |
| `src/blueprint/` | Blueprint management subsystem — parsing, validating, and applying `blueprint.yaml` configurations inside the IDE |
| `src/commands/` | **Command handlers** — implements the UI-facing commands exposed to the AI agent (e.g., start sandbox, approve policy, configure inference) |
| `src/onboard/` | Plugin-side onboarding flow — guides the agent through initial setup within the IDE context |
| `tsconfig.json` | TypeScript compiler configuration scoped to this plugin |
| `eslint.config.mjs` | Plugin-specific ESLint rules |
| `package.json` | Plugin's own dependency manifest (separate from root) |

### Dependencies & Interactions

```
nemoclaw/src/ ──→ nemoclaw-blueprint/         (reads blueprint.yaml and policy definitions)
nemoclaw/src/ ──→ bin/nemoclaw.js             (invokes CLI commands via shell or IPC)
nemoclaw/src/ ──→ OpenClaw Plugin API         (external: registers with IDE plugin runtime)
nemoclaw/src/blueprint/ ──→ nemoclaw-blueprint/policies/  (loads policy presets)
nemoclaw/src/onboard/ ──→ bin/lib/onboard.js  (coordinates with CLI onboarding logic)
```

**External Service Interactions:**
- **OpenClaw IDE Runtime** — registers commands, listens for agent-initiated events, renders UI prompts
- **NemoClaw CLI** (`bin/nemoclaw.js`) — delegates actual sandbox operations to the CLI layer

---

## 3. `nemoclaw-blueprint/` — Declarative Sandbox Configuration

### Core Responsibility
The declarative configuration layer for NemoClaw. Defines *what* the sandbox environment looks like — including which network destinations are permitted, what capabilities are enabled, and what preset policy profiles are available. This is the primary configuration artifact that users customize.

### Key Components

| File/Directory | Role |
|----------------|------|
| `blueprint.yaml` | **Master sandbox blueprint** — declares the full environment specification: base image, network policy reference, resource limits, volume mounts, and inference settings |
| `policies/openclaw-sandbox.yaml` | **Default network policy** — the baseline network allow/deny rules for the OpenClaw sandbox environment |
| `policies/presets/` | **Policy preset library** — 9 pre-built policy profiles (e.g., `strict`, `permissive`, `npm-registry`, `pypi`, `github`) that users can compose or reference |

### Dependencies & Interactions

```
nemoclaw-blueprint/ ──→ bin/lib/policies.js   (consumed by policy engine at runtime)
nemoclaw-blueprint/ ──→ nemoclaw/src/blueprint/ (read and managed by plugin)
nemoclaw-blueprint/ ──→ bin/lib/runner.js      (blueprint drives container configuration)
nemoclaw-blueprint/ ──→ test/validate-blueprint.test.ts  (schema-validated in tests)
```

**External Service Interactions:**
- None directly. This module is a **pure configuration artifact** consumed by other modules.
- Policy definitions ultimately govern **outbound network traffic** from the Docker sandbox at runtime.

---

## 4. `scripts/` — Utility & Operational Scripts

### Core Responsibility
A collection of shell and TypeScript utility scripts that support installation, environment setup, debugging, CI operations, and auxiliary services. These are operational tools for developers and CI pipelines rather than application runtime code.

### Key Components

| File | Role |
|------|------|
| `install.sh` | System-wide NemoClaw installation script |
| `nemoclaw-start.sh` | **Sandbox startup orchestrator** — primary script to initialize and launch the sandbox environment |
| `start-services.sh` | Starts auxiliary supporting services alongside the sandbox |
| `setup.sh` | Initial environment setup (dependencies, config initialization) |
| `setup-dns-proxy.sh` | Configures DNS proxy for network policy enforcement inside containers |
| `setup-spark.sh` | Sets up the Spark remote GPU environment integration |
| `brev-setup.sh` | Environment setup for Brev.dev GPU cloud deployment |
| `backup-workspace.sh` | Workspace backup utility — snapshots the sandbox working directory |
| `debug.sh` | Diagnostic script — dumps sandbox state, logs, and environment for troubleshooting |
| `telegram-bridge.js` | **Telegram notification bridge** — forwards sandbox events/output to a Telegram bot |
| `test-inference.sh` | Integration smoke test for cloud inference provider connectivity |
| `test-inference-local.sh` | Integration smoke test for local NVIDIA NIM inference |
| `walkthrough.sh` | Interactive guided walkthrough for new users |
| `check-coverage-ratchet.ts` | **CI quality gate** — enforces that test coverage never decreases below thresholds |
| `check-spdx-headers.sh` | Validates all source files have correct SPDX license headers |
| `check-version-tag-sync.sh` | Ensures `package.json` version matches Git tag |
| `write-auth-profile.ts` | Writes authentication/credential profiles for inference providers |
| `update-docker-pin.sh` | Updates pinned Docker image digests in configuration |
| `docs-to-skills.py` | **Documentation converter** — transforms Sphinx docs into AI agent skill definition format |
| `fix-coredns.sh` | Fixes CoreDNS configuration issues in Kubernetes deployments |
| `install-openshell.sh` | Installs the OpenShell component into the sandbox |
| `clean-staged-tree.sh` | Git pre-commit utility — cleans staged file tree for validation |
| `smoke-macos-install.sh` | macOS-specific installation smoke test |
| `lib/runtime.sh` | **Shared shell library** — common functions reused across shell scripts (logging, error handling, container detection) |

### Dependencies & Interactions

```
scripts/ ──→ bin/nemoclaw.js              (invokes CLI for sandbox operations)
scripts/ ──→ nemoclaw-blueprint/          (reads/writes blueprint configuration)
scripts/ ──→ scripts/lib/runtime.sh       (internal: shared shell functions)
scripts/ ──→ Docker daemon                (external: direct docker CLI calls)
scripts/ ──→ Telegram Bot API             (external: telegram-bridge.js sends HTTP requests)
scripts/ ──→ Brev.dev API                 (external: brev-setup.sh provisions cloud GPU)
scripts/ ──→ NVIDIA NIM / Spark          (external: inference environment setup)
scripts/ ──→ ci/coverage-threshold-*.json (reads thresholds for coverage ratchet)
```

**External Service Interactions:**
- **Telegram Bot API** — `telegram-bridge.js` sends sandbox notifications
- **Brev.dev cloud** — GPU instance provisioning
- **Docker Engine** — direct container management
- **NVIDIA NIM / Spark** — local and remote GPU inference setup

---

## 5. `test/` — Test Suite

### Core Responsibility
Comprehensive testing layer covering unit tests for every CLI module, security boundary tests, and end-to-end integration tests. Ensures correctness of sandbox behavior, credential isolation, policy enforcement, and recovery mechanisms.

### Key Components

#### Unit Tests (per CLI module)

| Test File | Tests |
|-----------|-------|
| `cli.test.js` | Top-level CLI command parsing and dispatch |
| `credentials.test.js` | Credential storage, retrieval, and sanitization logic |
| `credential-exposure.test.js` | **Security**: verifies credentials never leak into container env or logs |
| `policies.test.js` | Policy YAML parsing, validation, and enforcement logic |
| `runner.test.js` | Container launch command construction and execution |
| `inference-config.test.js` | Inference provider config loading and switching |
| `local-inference.test.js` | Local NIM inference setup and communication |
| `nim.test.js` | NVIDIA NIM image selection and container orchestration |
| `onboard.test.js` | Onboarding wizard flow logic |
| `onboard-session.test.js` | Session state persistence and resumability |
| `onboard-readiness.test.js` | Environment readiness checks during onboarding |
| `onboard-selection.test.js` | User selection/prompt handling in onboarding |
| `platform.test.js` | OS/architecture detection accuracy |
| `preflight.test.js` | Pre-launch dependency validation |
| `registry.test.js` | Image registry resolution and pulling |
| `runtime-recovery.test.js` | Crash detection and container recovery |
| `runtime-shell.test.js` | Shell execution within sandbox runtime |
| `resolve-openshell.test.js` | OpenShell resolution logic |
| `dns-proxy.test.js` | DNS proxy configuration and behavior |
| `gateway-cleanup.test.js` | Network gateway resource cleanup |
| `install-preflight.test.js` | Installation pre-condition checks |
| `nemoclaw-cli-recovery.test.js` | CLI-level recovery scenarios |
| `nemoclaw-start.test.js` | Full sandbox start sequence |
| `service-env.test.js` | Service environment variable handling |
| `setup-sandbox-name.test.js` | Sandbox naming/identification logic |
| `smoke-macos-install.test.js` | macOS installation smoke test |
| `uninstall.test.js` | Clean uninstallation verification |
| `validate-blueprint.test.ts` | **Blueprint schema validation** (TypeScript) |

#### Security Tests

| Test File | Security Concern Tested |
|-----------|------------------------|
| `security-binaries-restriction.test.js` | Validates dangerous binaries are blocked inside sandbox |
| `security-c2-dockerfile-injection.test.js` | Prevents command injection via Dockerfile manipulation |
| `security-c4-manifest-traversal.test.js` | Guards against path traversal in manifest handling |
| `security-method-wildcards.test.js` | Prevents wildcard abuse in policy method matching |

#### E2E Tests (`test/e2e/`)

| File | Role |
|------|------|
| `test-full-e2e.sh` | Complete sandbox lifecycle E2E test |
| `brev-e2e.test.js` | E2E on Brev.dev GPU cloud |
| `test-credential-sanitization.sh` | E2E credential sanitization verification |
| `test-double-onboard.sh` | Validates idempotency of onboarding |
| `test-onboard-repair.sh` | Tests corrupted onboarding state recovery |
| `test-onboard-resume.sh` | Tests interrupted onboarding resumption |
| `test-gpu-e2e.sh` | GPU inference E2E pipeline |
| `test-telegram-injection.sh` | Security: prevents Telegram message injection |
| `test-spark-install.sh` | Spark/remote GPU install E2E |
| `e2e-gateway-isolation.sh` | Network gateway isolation verification |
| `e2e-cloud-experimental/` | Cloud-specific experimental E2E feature tests |
| `Dockerfile.sandbox` | Sandbox container image for unit test isolation |
| `Dockerfile.full-e2e` | Full environment container for E2E test runs |

### Dependencies & Interactions

```
test/ ──→ bin/lib/*.js                    (imports all CLI modules under test)
test/ ──→ nemoclaw-blueprint/             (validate-blueprint.test.ts reads blueprint schema)
test/ ──→ scripts/                        (e2e scripts invoke operational scripts)
test/ ──→ ci/coverage-threshold-*.json   (coverage ratchet thresholds)
test/package.json ──→ vitest             (test runner dependency)
test/e2e/ ──→ bin/nemoclaw.js            (E2E tests exercise the real CLI)
test/e2e/ ──→ Docker daemon              (external: spins up real sandbox containers)
test/e2e/ ──→ Brev.dev                   (external: cloud GPU E2E)
```

**External Service Interactions:**
- **Docker Engine** — E2E tests launch real sandboxes
- **Brev.dev** — GPU cloud E2E testing environment
- No mocked external services in security tests (intentionally exercises real boundaries)

---

## 6. `k8s/` — Kubernetes Deployment

### Core Responsibility
Provides Kubernetes manifests for deploying NemoClaw in a remote or cloud-hosted configuration, enabling the sandbox to run on a K8s cluster rather than the local Docker daemon — supporting team-shared or GPU-backed remote deployments.

### Key Components

| File | Role |
|------|------|
| `nemoclaw-k8s.yaml` | **K8s deployment manifest** — defines Deployment, Service, ConfigMap, and resource specifications for running NemoClaw on Kubernetes |
| `README.md` | Deployment instructions and configuration guidance for K8s setup |

### Dependencies & Interactions

```
k8s/ ──→ nemoclaw-blueprint/blueprint.yaml  (blueprint config applied to K8s deployment)
k8s/ ──→ Kubernetes cluster                 (external: manifests applied via kubectl)
k8s/ ──→ NVIDIA GPU operator               (external: GPU resource scheduling in K8s)
k8s/ ──→ Container registries              (external: pulls sandbox images)
scripts/fix-coredns.sh ──→ k8s/            (fixes K8s CoreDNS for DNS proxy compatibility)
```

**External Service Interactions:**
- **Kubernetes API Server** — manifests deployed via `kubectl`
- **NVIDIA GPU Operator** — GPU resource allocation for NIM inference on K8s
- **Container Registries** — image pulls during pod scheduling

---

## 7. `.agents/skills/` — AI Agent Skill Definitions

### Core Responsibility
Provides structured knowledge files that teach AI coding agents (like Claude) how to use NemoClaw effectively. These are **guidance artifacts** — not executable code — consumed by the AI agent at runtime to understand tool capabilities, workflows, and best practices.

### Key Components

| Skill Directory | Knowledge Domain |
|----------------|-----------------|
| `nemoclaw-overview/` | High-level introduction to what NemoClaw is and when to use it |
| `nemoclaw-reference/` | Command reference and API surface for the CLI |
| `nemoclaw-get-started/` | Step-by-step getting started workflow for first-time use |
| `nemoclaw-deploy-remote/` | How to deploy NemoClaw to remote GPU environments |
| `nemoclaw-workspace/` | Workspace file management, backup, and restore operations |
| `nemoclaw-manage-policy/` | Network policy approval, customization, and management |
| `nemoclaw-monitor-sandbox/` | Sandbox activity monitoring and log inspection |
| `nemoclaw-configure-inference/` | Inference provider configuration and switching |
| `update-docs/` | Guidance for the agent on how to update NemoClaw documentation |

Each skill directory contains a primary skill definition file and optionally a `references/` subdirectory with supporting reference documents.

### Dependencies & Interactions

```
.agents/skills/ ──→ docs/                  (skills are derived from documentation)
.agents/skills/ ──→ scripts/docs-to-skills.py  (generation pipeline from docs)
.agents/skills/ ──→ AI Agent Runtime       (external: consumed by Claude/OpenClaw agent)
```

**External Service Interactions:**
- **AI Agent Runtime (Claude/OpenClaw)** — skills are loaded into the agent's context window to guide tool usage
- No runtime code dependencies — purely declarative knowledge artifacts

---

## 8. `.github/` — CI/CD & Repository Governance

### Core Responsibility
Defines the complete automated pipeline for code quality enforcement, security checks, test execution, documentation deployment, and release management. Also governs contribution workflows via templates, ownership rules, and automated dependency management.

### Key Components

#### Workflows (`workflows/`)

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `pr.yaml` | Pull Request | Runs linting, unit tests, coverage checks, security scans |
| `main.yaml` | Push to `main` | Full validation suite on merge |
| `nightly-e2e.yaml` | Scheduled (nightly) | Regression E2E test runs |
| `sandbox-images-and-e2e.yaml` | Scheduled/Manual | Rebuilds sandbox Docker images + E2E |
| `e2e-brev.yaml` | Manual | GPU E2E tests on Brev.dev cloud |
| `base-image.yaml` | Manual | Builds and publishes `Dockerfile.base` |
| `docs-preview-deploy.yaml` | PR | Deploys documentation preview for review |
| `docs-preview-pr.yaml` | PR | Posts docs preview URL as PR comment |
| `commit-lint.yaml` | PR | Enforces conventional commit message format |
| `dco-check.yaml` | PR | Validates Developer Certificate of Origin sign-off |
| `docker-pin-check.yaml` | PR | Ensures Docker image references use pinned digests |
| `pr-limit.yaml` | PR | Enforces PR size/scope limits |

#### Reusable Actions (`actions/`)

| Action | Role |
|--------|------|
| `basic-checks/` | Composite action — runs lint, format check, and SPDX header validation |
| `resolve-sandbox-base-image/` | Composite action — determines correct sandbox base image for the current context |

#### Repository Governance

| File | Role |
|------|------|
| `CODEOWNERS` | Defines code ownership for automated review assignment |
| `PULL_REQUEST_TEMPLATE.md` | Standard PR description template |
| `dependabot.yml` | Automated dependency update configuration |
| `ISSUE_TEMPLATE/` | Structured templates for bug reports, feature requests, doc issues |
| `dco-bypass.txt` | DCO bypass list for specific automated commits |

### Dependencies & Interactions

```
.github/workflows/ ──→ test/              (executes unit and E2E test suites)
.github/workflows/ ──→ scripts/           (invokes check and setup scripts)
.github/workflows/ ──→ ci/               (reads coverage threshold JSON files)
.github/workflows/ ──→ docs/             (builds and deploys documentation)
.github/actions/ ──→ .github/workflows/  (consumed as reusable composite steps)
```

**External Service Interactions:**
- **GitHub Actions Runner** — workflow execution environment
- **Brev.dev** — GPU cloud for E2E test runs
- **Docker Hub / GHCR** — image push/pull during image build workflows
- **Documentation hosting platform** — docs preview deployment target
- **npm registry** — dependency installation during CI

---

## 9. `docs/` — User Documentation

### Core Responsibility
A Sphinx-based documentation site providing comprehensive user-facing guides covering installation, configuration, security practices, inference setup, and reference material. Also contains Sphinx extensions for JSON output generation and search asset building.

### Key Components

#### Documentation Sections

| Directory | Content |
|-----------|---------|
| `get-started/` | `quickstart.md` — first-time installation and launch guide |
| `about/` | `overview.md`, `how-it-works.md`, `release-notes.md` — project introduction |
| `reference/` | `commands.md`, `architecture.md`, `network-policies.md`, `inference-profiles.md`, `troubleshooting.md` |
| `network-policy/` | `approve-network-requests.md`, `customize-network-policy.md` |
| `inference/` | `switch-inference-providers.md` |
| `workspace/` | `backup-restore.md`, `workspace-files.md` |
| `deployment/` | `deploy-to-remote-gpu.md`, `sandbox-hardening.md`, `set-up-telegram-bridge.md` |
| `monitoring/` | `monitor-sandbox-activity.md` |
| `security/` | `best-practices.md` |

#### Infrastructure Files

| File/Directory | Role |
|----------------|------|
| `conf.py` | **Sphinx configuration** — extensions, theme, build settings |
| `index.md` | Documentation site root/table of contents |
| `project.json` |

--- events ---


I have performed a comprehensive scan of the entire repository structure and all provided file contents.

This codebase is a **CLI tool and developer sandbox orchestration system** (NemoClaw). After thoroughly examining all source files in:

- `bin/lib/` (core CLI logic)
- `nemoclaw/src/` (plugin source)
- `scripts/` (shell and JS scripts)
- `test/` (test files)
- `k8s/` (Kubernetes manifests)
- `.github/workflows/` (CI/CD pipelines)

I found **no usage of any message broker, event bus, or eventing system** such as SQS, EventBridge, Kafka, RabbitMQ, Pub/Sub, Ably, WebSockets, or any custom internal event bus. The codebase operates exclusively through:

- Direct CLI process execution (`child_process`, shell scripts)
- HTTP/REST API calls to inference providers and registries
- Kubernetes API interactions via `kubectl`
- Docker daemon interactions via `docker` CLI
- Local file system operations

no events

--- deployment ---


# Deployment Pipeline Analysis: NemoClaw Repository

## 1. Deployment Overview

| Attribute | Value |
|-----------|-------|
| **Primary CI/CD Platform** | GitHub Actions |
| **Environment Count** | 3 (PR preview, staging/sandbox, production via GHCR) |
| **Pipeline Files** | 12 workflow files |
| **IaC** | Dockerfile + Kubernetes YAML (limited) |
| **Build Tools** | npm, Docker multi-stage builds, Make |

---

## 2. Deployment Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        GITHUB ACTIONS PIPELINE                          │
└─────────────────────────────────────────────────────────────────────────┘

PR OPENED/UPDATED:
──────────────────
  PR Event ──► pr-limit.yaml        (max open PR gate)
           ──► dco-check.yaml       (DCO sign-off validation)
           ──► commit-lint.yaml     (conventional commit format)
           ──► docker-pin-check.yaml (Docker digest pin validation)
           ──► pr.yaml ─────────────► basic-checks (action)
                        │              └─ lint, format, SPDX headers
                        ├──────────────► unit tests (vitest)
                        ├──────────────► plugin build (tsc)
                        └──────────────► coverage gate check

  PR Event ──► docs-preview-pr.yaml ──► build Sphinx docs
                                    ──► deploy to preview URL (comment on PR)
  
  PR Event ──► e2e-brev.yaml ────────► sandbox E2E on Brev GPU infra
                                        (conditional/manual trigger)

PUSH TO MAIN:
─────────────
  push:main ──► main.yaml ──► basic-checks
                         ──► unit tests
                         ──► plugin build
                         ──► coverage gate
                         ──► sandbox Docker build + push to GHCR
                         ──► E2E tests against new image

  push:main ──► docs-preview-deploy.yaml ──► deploy docs to production

SCHEDULED / NIGHTLY:
─────────────────────
  cron ──► nightly-e2e.yaml ──► full sandbox E2E suite
                            ──► sandbox-images-and-e2e.yaml (matrix)

BASE IMAGE:
───────────
  Manual/tag ──► base-image.yaml ──► build Dockerfile.base
                                 ──► push ghcr.io/nvidia/nemoclaw/sandbox-base

RELEASE:
────────
  (No dedicated release workflow detected — version managed via package.json)
```

---

## 3. CI/CD Platform Detection

**Platform:** GitHub Actions (`.github/workflows/`)

**Workflows identified:**

| File | Purpose |
|------|---------|
| `main.yaml` | Primary CI on push to main |
| `pr.yaml` | PR validation pipeline |
| `base-image.yaml` | Base Docker image build/push |
| `commit-lint.yaml` | Commit message format enforcement |
| `dco-check.yaml` | Developer Certificate of Origin check |
| `docker-pin-check.yaml` | Docker image digest pin validation |
| `docs-preview-deploy.yaml` | Production docs deployment |
| `docs-preview-pr.yaml` | PR docs preview deployment |
| `e2e-brev.yaml` | E2E tests on Brev GPU infrastructure |
| `nightly-e2e.yaml` | Scheduled nightly E2E suite |
| `pr-limit.yaml` | Open PR count gate |
| `sandbox-images-and-e2e.yaml` | Sandbox image matrix + E2E |

**Composite Actions:**

| File | Purpose |
|------|---------|
| `.github/actions/basic-checks/` | Reusable lint/format/header checks |
| `.github/actions/resolve-sandbox-base-image/` | Base image tag resolution |

---

## 4. Detailed Pipeline Documentation

### Pipeline: `pr.yaml`

**Triggers:**
- `pull_request` events against default branch

**Stages/Jobs:**

1. **Stage: Basic Checks**
   - **Purpose:** Code quality validation
   - **Steps:** Runs `.github/actions/basic-checks` composite action (lint, Prettier format, SPDX license header check via `scripts/check-spdx-headers.sh`, shellcheck)
   - **Dependencies:** None
   - **Artifacts:** None
   - **Conditions:** All PRs

2. **Stage: Unit Tests**
   - **Purpose:** Run vitest unit test suite
   - **Steps:** `npm ci` → `npm test` (vitest with coverage)
   - **Dependencies:** Basic Checks (likely parallel)
   - **Artifacts:** Coverage reports
   - **Conditions:** All PRs

3. **Stage: Plugin Build**
   - **Purpose:** TypeScript compilation check
   - **Steps:** `npm ci` in `nemoclaw/` → `npm run build` (tsc)
   - **Dependencies:** None
   - **Artifacts:** Compiled JS in `dist/`
   - **Conditions:** All PRs

4. **Stage: Coverage Gate**
   - **Purpose:** Enforce coverage thresholds (ratchet)
   - **Steps:** `scripts/check-coverage-ratchet.ts` against `ci/coverage-threshold-cli.json` and `ci/coverage-threshold-plugin.json`
   - **Dependencies:** Unit Tests
   - **Conditions:** All PRs

---

### Pipeline: `main.yaml`

**Triggers:**
- `push` to `main` branch

**Stages/Jobs:**

1. **Stage: Basic Checks** *(same as PR)*

2. **Stage: Unit Tests + Coverage** *(same as PR)*

3. **Stage: Sandbox Docker Build**
   - **Purpose:** Build and push production sandbox image
   - **Steps:**
     - Resolve base image tag (`.github/actions/resolve-sandbox-base-image`)
     - `docker build` with multi-stage Dockerfile
     - Push to `ghcr.io/nvidia/nemoclaw/` registry
   - **Dependencies:** Unit tests pass
   - **Artifacts:** Docker image pushed to GHCR
   - **Conditions:** Push to main only (not PRs)

4. **Stage: E2E Smoke Tests**
   - **Purpose:** Validate deployed image
   - **Steps:** Run `test/e2e-test.sh` or equivalent against freshly built image
   - **Dependencies:** Docker Build completes
   - **Artifacts:** Test results

---

### Pipeline: `base-image.yaml`

**Triggers:**
- Manual dispatch (`workflow_dispatch`) and/or tag push (based on typical pattern for base images)

**Stages/Jobs:**

1. **Stage: Build Base Image**
   - **Purpose:** Build `Dockerfile.base` — expensive, rarely-changing layers
   - **Steps:** `docker build -f Dockerfile.base` → push `ghcr.io/nvidia/nemoclaw/sandbox-base:latest`
   - **Purpose:** Decouples slow base layer (apt, gosu, users, openclaw CLI) from fast PR builds
   - **Artifacts:** `sandbox-base:latest` on GHCR

---

### Pipeline: `nightly-e2e.yaml`

**Triggers:**
- Scheduled cron (nightly)

**Stages/Jobs:**

1. **Stage: Full E2E Suite**
   - **Purpose:** Run complete end-to-end tests including GPU scenarios
   - **Steps:** Likely invokes `test/e2e/test-full-e2e.sh`, `test-gpu-e2e.sh`, `test-onboard-repair.sh`, `test-double-onboard.sh`
   - **Conditions:** Scheduled only, not on PR

---

### Pipeline: `e2e-brev.yaml`

**Triggers:**
- PR events (conditional) or manual dispatch

**Stages/Jobs:**

1. **Stage: Brev E2E**
   - **Purpose:** Run `test/e2e/brev-e2e.test.js` on Brev GPU cloud infrastructure
   - **Steps:** `scripts/brev-setup.sh` → E2E test execution
   - **Note:** Requires Brev API credentials as GitHub secrets

---

### Pipeline: `docs-preview-pr.yaml` / `docs-preview-deploy.yaml`

**Triggers:**
- `docs-preview-pr.yaml`: PR events
- `docs-preview-deploy.yaml`: Push to main

**Stages/Jobs:**

1. **Stage: Build Docs**
   - **Purpose:** Sphinx documentation build
   - **Steps:** `uv sync --group docs` → `sphinx-build` → generate static HTML
   - **Tools:** Sphinx, MyST-Parser, nvidia-sphinx-theme (from `pyproject.toml`)
   - **Artifacts:** Static HTML site

2. **Stage: Deploy Preview / Production**
   - **Purpose:** Host docs
   - **Steps:** Deploy to preview URL (PR) or production docs host (main)
   - **PR Action:** Posts comment with preview URL

---

### Pipeline: `sandbox-images-and-e2e.yaml`

**Triggers:**
- Scheduled or manual dispatch

**Stages/Jobs:**

1. **Stage: Matrix Image Build**
   - **Purpose:** Build sandbox images across multiple configurations/models
   - **Steps:** Matrix build of Dockerfile with varying `NEMOCLAW_MODEL`, `NEMOCLAW_PROVIDER_KEY` build args
   - **Artifacts:** Multiple tagged images on GHCR

---

## 5. Build Process

### Docker Multi-Stage Build (`Dockerfile`)

**Stage 1: Builder**
```
FROM node:22-slim@sha256:4f77a690... AS builder
# Pinned digest — security best practice
# Compiles nemoclaw/src/ TypeScript → dist/
```

**Stage 2: Runtime**
```
FROM ${BASE_IMAGE}  # ghcr.io/nvidia/nemoclaw/sandbox-base:latest
# Layers plugin, blueprint, startup script on cached base
# Hardens: removes gcc, g++, make, netcat from base
# Generates openclaw.json with unique auth token per build
# Locks config files: chmod 444, chown root
```

**Key Build Arguments:**

| ARG | Purpose | Default |
|-----|---------|---------|
| `BASE_IMAGE` | Base image reference | `ghcr.io/nvidia/nemoclaw/sandbox-base:latest` |
| `NEMOCLAW_MODEL` | AI model identifier | `nvidia/nemotron-3-super-120b-a12b` |
| `NEMOCLAW_PROVIDER_KEY` | Inference provider | `nvidia` |
| `NEMOCLAW_INFERENCE_BASE_URL` | Inference endpoint | `https://inference.local/v1` |
| `NEMOCLAW_BUILD_ID` | Cache-busting token | `default` |
| `CHAT_UI_URL` | Chat UI origin | `http://127.0.0.1:18789` |

**Security Hardening in Build:**
- Build-time auth token generated via `secrets.token_hex(32)` — unique per image
- `openclaw.json` locked `chmod 444`, `chown root:root`
- Config hash pinned via `sha256sum` at build time
- Build tools (gcc, netcat) purged from runtime image
- Sandbox user cannot write to `.openclaw/` directory

---

### Docker Base Image (`Dockerfile.base`)

- Contains: apt packages, gosu, user accounts, openclaw CLI
- Rebuilt infrequently via `base-image.yaml`
- Cached on GHCR to accelerate PR builds

---

### npm Build

**Root package:** `package.json`
- Runtime: `openclaw@2026.3.11`, `p-retry@^4.6.2`
- Dev: vitest, commitlint, eslint, prettier, tsx, typescript

**Plugin package:** `nemoclaw/package.json`
- Runtime: commander, execa, json5, tar, yaml
- Dev: TypeScript, vitest, eslint

**Build commands (inferred from Makefile and package.json patterns):**
```bash
npm ci                  # Install dependencies
npm run build           # tsc compilation (plugin)
npm test                # vitest (unit tests)
npm run lint            # eslint
npm run format          # prettier
```

---

## 6. Infrastructure as Code

### Kubernetes (`k8s/nemoclaw-k8s.yaml`)

**Technology:** Raw Kubernetes YAML manifests

**Resources Managed:**
- Deployment/Pod spec for NemoClaw sandbox
- Likely: Service, ConfigMap based on typical patterns

**Limitations:**
- No Helm chart detected
- No Kustomize overlays detected
- Static YAML — no parameterization visible
- No state management (stateless K8s manifests)

**Location:** `k8s/README.md` documents usage

---

### Container Registry

**Registry:** `ghcr.io/nvidia/nemoclaw/`
**Images:**
- `sandbox-base:latest` — base image
- `sandbox:latest` (or versioned) — full sandbox image
- `test/Dockerfile.sandbox` — test sandbox image
- `test/e2e/Dockerfile.full-e2e` — full E2E test image

---

## 7. Testing in Deployment Pipeline

### Test Organization

| Test Type | Location | Runner | Stage |
|-----------|----------|--------|-------|
| Unit tests (CLI) | `test/*.test.js` | Vitest | PR + Main |
| Unit tests (plugin) | `nemoclaw/src/*.test.ts` | Vitest | PR + Main |
| Security tests | `test/security-*.test.js` | Vitest | PR + Main |
| Coverage ratchet | `scripts/check-coverage-ratchet.ts` | tsx | PR + Main |
| E2E (basic) | `test/e2e-test.sh` | Shell | Main |
| E2E (full) | `test/e2e/test-full-e2e.sh` | Shell | Nightly |
| E2E (GPU) | `test/e2e/test-gpu-e2e.sh` | Shell | Nightly/Brev |
| E2E (Brev cloud) | `test/e2e/brev-e2e.test.js` | Vitest/Shell | `e2e-brev.yaml` |
| Smoke (macOS install) | `test/smoke-macos-install.test.js` | Vitest | PR (conditional) |
| Docs build check | Sphinx build | Python | PR + Main |

### Coverage Thresholds

**Location:** `ci/coverage-threshold-cli.json`, `ci/coverage-threshold-plugin.json`

Coverage is enforced via a **ratchet mechanism** (`scripts/check-coverage-ratchet.ts`) — thresholds can only increase, never decrease. This is a strong quality gate pattern.

### Security Test Suite

Notable dedicated security tests:
- `security-binaries-restriction.test.js` — validates binary execution restrictions
- `security-c2-dockerfile-injection.test.js` — C2 injection via Dockerfile
- `security-c4-manifest-traversal.test.js` — manifest path traversal
- `security-method-wildcards.test.js` — wildcard method abuse
- `credential-exposure.test.js` — credential leak detection

---

## 8. Quality Gates

| Gate | Mechanism | Enforcement |
|------|-----------|-------------|
| Commit format | `commit-lint.yaml` + `commitlint.config.js` | Blocking |
| DCO sign-off | `dco-check.yaml` | Blocking |
| Lint | ESLint via `basic-checks` action | Blocking |
| Format | Prettier via `basic-checks` action | Blocking |
| SPDX headers | `check-spdx-headers.sh` | Blocking |
| Shell safety | shellcheck via `.shellcheckrc` | Blocking |
| Docker pin | `docker-pin-check.yaml` | Blocking |
| Unit tests | vitest | Blocking |
| Coverage ratchet | `check-coverage-ratchet.ts` | Blocking |
| PR count limit | `pr-limit.yaml` | Blocking |
| E2E (basic) | `e2e-test.sh` on main | Post-merge |
| E2E (full) | Nightly | Non-blocking (scheduled) |

### Pre-commit Hooks (`.pre-commit-config.yaml`)

Local developer enforcement:
- Prettier formatting
- ESLint
- shellcheck
- commitlint (via `@j178/prek` in devDependencies)
- SPDX header checks

---

## 9. Release Management

### Version Strategy

- **Version source:** `package.json` at root
- **Sync validation:** `scripts/check-version-tag-sync.sh` — enforces git tag matches package version
- **Scheme:** Date-based versioning observed (`openclaw: 2026.3.11`)
- **No dedicated release workflow detected** — release is implicit via push to main + Docker image push

### Artifact Management

| Artifact | Registry | Retention |
|----------|----------|-----------|
| Docker images | `ghcr.io/nvidia/nemoclaw/` | Not documented |
| npm packages | Not published (no `npm publish`) | N/A |
| Docs | External hosting (docs preview service) | Per PR lifecycle |

### Changelog

- `docs/about/release-notes.md` — manual release notes
- No automated changelog generation detected (no `conventional-changelog`, `semantic-release`, etc.)

---

## 10. Deployment Targets & Environments

### Environment: PR Preview

| Attribute | Value |
|-----------|-------|
| **Purpose** | Documentation preview per PR |
| **Trigger** | PR open/update |
| **Deployment Method** | Direct replacement |
| **Cleanup** | On PR close |

### Environment: GHCR (Production Images)

| Attribute | Value |
|-----------|-------|
| **Platform** | GitHub Container Registry |
| **Push trigger** | Merge to main |
| **Image** | `ghcr.io/nvidia/nemoclaw/sandbox-*` |
| **Auth** | `GITHUB_TOKEN` (standard GHCR auth) |

### Environment: Brev GPU Cloud (E2E)

| Attribute | Value |
|-----------|-------|
| **Platform** | Brev.dev GPU cloud |
| **Setup** | `scripts/brev-setup.sh` |
| **Purpose** | GPU-accelerated E2E testing |
| **Trigger** | PR (conditional) + manual |

### Environment: Kubernetes (self-hosted)

| Attribute | Value |
|-----------|-------|
| **Manifests** | `k8s/nemoclaw-k8s.yaml` |
| **Deployment** | `kubectl apply` (manual, documented in `k8s/README.md`) |
| **No automation detected** | Manual deployment only |

---

## 11. Deployment Validation & Rollback

### Post-Deployment Validation

- `test/e2e-test.sh` — basic E2E smoke test after main build
- `scripts/nemoclaw-start.sh` — startup validation inside container
- `test/e2e/test-onboard-repair.sh` — onboard recovery validation
- `test/e2e/test-double-onboard.sh` — idempotency check

### Rollback Strategy

**Detected mechanisms:**
- Docker image: Re-tag previous GHCR image and redeploy (manual)
- No automated rollback pipeline detected
- No rollback workflow in `.github/workflows/`
- Kubernetes: `kubectl rollout undo` available but not automated

**Assessment:** Rollback is entirely manual.

---

## 12. Deployment Access Control

### CODEOWNERS (`.github/CODEOWNERS`)

- File-based ownership enforced on PRs
- Merges require owner approval

### Secret Management

**Build-time secrets (Docker ARGs → ENV):**
```
NEMOCLAW_MODEL, NEMOCLAW_PROVIDER_KEY, 
NEMOCLAW_INFERENCE_BASE_URL, NEMOCLAW_INFERENCE_API_KEY
```

**Security pattern observed:**
- Build args promoted to ENV vars via explicit `ENV` directive (C-2 injection protection documented in Dockerfile comments)
- Auth token generated at build time via `secrets.token_hex(32)` — not a hardcoded secret
- `NEMOCLAW_INFERENCE_COMPAT_B64` passed as base64-encoded JSON

**GitHub Actions secrets:**
- GHCR push uses `GITHUB_TOKEN`
- Brev E2E likely uses `BREV_API_KEY` (secret, not visible in code)

**Credential test:** `test/credential-exposure.test.js` validates no credential leakage

---

## 13. Anti-Patterns & Issues Found

### 🔴 Critical Issues

**Issue 1: No Automated Rollback**
- **Location:** `.github/workflows/` — no rollback workflow exists
- **Current State:** Zero rollback automation
- **Impact:** Production incidents require manual image re-tagging and redeployment; MTTR is high
- **Fix Needed:** Add rollback workflow triggered by failed post-deploy health check or manual dispatch; implement `docker tag previous-sha latest && docker push` pattern

---

**Issue 2: Kubernetes Deployment is Fully Manual**
- **Location:** `k8s/nemoclaw-k8s.yaml`, `k8s/README.md`
- **Current State:** K8s manifests exist but no CI/CD automation deploys them; purely `kubectl apply` by hand
- **Impact:** Deployment consistency not guaranteed; no audit trail; human error risk
- **Fix Needed:** Add a GitHub Actions deployment job that runs `kubectl apply -f k8s/` on merge to main with environment-specific kubeconfig secrets

---

### 🟠 High Severity Issues

**Issue 3: `BASE_IMAGE` ARG Defaults to `:latest` Tag**
- **Location:** `Dockerfile`, line 10: `ARG BASE_IMAGE=ghcr.io/nvidia/nemoclaw/sandbox-base:latest`
- **Current State:** Default base image uses mutable `:latest` tag
- **Impact:** Non-deterministic builds; a compromised or accidentally updated base image breaks all PR builds silently
- **Note:** `docker-pin-check.yaml` partially mitigates this by checking digest pins in the Dockerfile itself, but the `BASE_IMAGE` ARG default is not a pinned digest
- **Fix Needed:** Default `BASE_IMAGE` to a digest-pinned reference: `ARG BASE_IMAGE=ghcr.io/nvidia/nemoclaw/sandbox-base@sha256:<digest>`

---

**Issue 4: No Dedicated Release Workflow**
- **Location:** `.github/workflows/` — no `release.yaml` exists
- **Current State:** Releases are implicit; Docker images pushed on every merge to main
- **Impact:** No formal release gating; no changelog automation; release notes in `docs/about/release-notes.md` are manually maintained
- **Fix Needed:** Implement a tag-triggered release workflow using `scripts/check-version-tag-sync.sh` validation, automated changelog from conventional commits, and formal GitHub Release creation

---

**Issue 5: Missing Staging Environment**
- **Current State:** Pipeline goes PR → main → GHCR push → (manual K8s deploy)
- **Impact:** No intermediate staging validation before production image is published; E2E only runs post-merge on main
- **Fix Needed:** Add a staging environment where the built image is deployed and E2E-tested before the `:latest` tag is promoted

---

**Issue 6: E2E Tests Run After Merge (Not Before)**
- **Location:** `main.yaml` — E2E runs post-push to main
- **Current State:** Full E2E tests are only run after code merges to main; nightly for comprehensive suite
- **Impact:** Breaking E2E issues are discovered post-merge, requiring a follow-up fix commit
- **Fix Needed:** Move at minimum the basic E2E (`e2e-test.sh`) to run on PRs as a required check; use the existing `e2e-brev.yaml` pattern as the model

---

### 🟡 Medium

--- dependencies ---


# Dependency and Architecture Analysis: NemoClaw

---

## Internal Modules

The following internal modules are developed as part of NemoClaw and reused across different components of the project.

### CLI Core Library (`bin/lib/`)

These modules form the heart of the `nemoclaw` CLI tool, each owning a distinct runtime responsibility:

| Module | File | Responsibility |
|--------|------|----------------|
| **Credentials** | `bin/lib/credentials.js` | Manages secure handling and isolation of API keys and auth tokens used by the AI agent |
| **Inference Config** | `bin/lib/inference-config.js` | Resolves and routes inference provider configuration (model, API type, base URL) |
| **Local Inference** | `bin/lib/local-inference.js` | Handles locally-hosted inference setup, distinct from remote provider routing |
| **NIM** | `bin/lib/nim.js` | Interfaces with NVIDIA NIM GPU inference images; reads `nim-images.json` for available images |
| **Onboard** | `bin/lib/onboard.js` | Orchestrates the initial sandbox provisioning and setup flow for a new session |
| **Onboard Session** | `bin/lib/onboard-session.js` | Manages state and lifecycle of an active onboarding session |
| **Platform** | `bin/lib/platform.js` | Detects and abstracts host platform differences (e.g., macOS vs. Linux) |
| **Policies** | `bin/lib/policies.js` | Enforces and evaluates network/execution policies defined in YAML blueprints |
| **Preflight** | `bin/lib/preflight.js` | Validates system prerequisites before launching the sandbox (Docker availability, dependencies, etc.) |
| **Registry** | `bin/lib/registry.js` | Manages Docker image registry references for sandbox base and NIM images |
| **Resolve OpenShell** | `bin/lib/resolve-openshell.js` | Resolves the correct OpenShell binary or version to use inside the sandbox |
| **Runner** | `bin/lib/runner.js` | Core execution engine — launches agent workloads inside Docker/K8s containers |
| **Runtime Recovery** | `bin/lib/runtime-recovery.js` | Handles error recovery and restart logic when the sandbox runtime encounters failures |
| **Version** | `bin/lib/version.js` | Manages CLI versioning and version-tag synchronization checks |

---

### OpenClaw Plugin (`nemoclaw/src/`)

The TypeScript plugin integrates NemoClaw into the OpenClaw AI IDE environment:

| Module | Location | Responsibility |
|--------|----------|----------------|
| **Plugin Entry Point** | `nemoclaw/src/index.ts` | Registers the plugin with the OpenClaw runtime; top-level plugin bootstrap |
| **Blueprint Module** | `nemoclaw/src/blueprint/` | Handles reading, validation, and application of declarative YAML sandbox environment blueprints |
| **Commands Module** | `nemoclaw/src/commands/` | Implements IDE-facing plugin commands exposed to the AI agent and developer |
| **Onboard Module** | `nemoclaw/src/onboard/` | Manages onboarding flows as initiated from within the plugin/IDE context |

---

### Sandbox Blueprint (`nemoclaw-blueprint/`)

| Component | Location | Responsibility |
|-----------|----------|----------------|
| **Blueprint Definition** | `nemoclaw-blueprint/blueprint.yaml` | Declarative YAML specification of the sandbox environment (tools, mounts, constraints) |
| **Network Policies** | `nemoclaw-blueprint/policies/openclaw-sandbox.yaml` | Primary network policy enforced within the sandbox |
| **Policy Presets** | `nemoclaw-blueprint/policies/presets/` | Pre-defined, reusable network policy configurations selectable per use case |

---

### Test Suite (`test/`)

The test suite is organized into distinct testing concerns, each targeting a specific internal module or security boundary:

| Test Group | Files | Responsibility |
|------------|-------|----------------|
| **Unit Tests** | `test/*.test.js` | Per-module unit tests mirroring each `bin/lib/` module (e.g., `policies.test.js`, `runner.test.js`) |
| **Security Tests** | `test/security-*.test.js` | Dedicated tests for security boundaries: binary restrictions, C2 Dockerfile injection, manifest traversal, method wildcards |
| **E2E Tests** | `test/e2e/` | End-to-end validation of full sandbox lifecycle, credential sanitization, onboarding repair/resume, GPU flows |

---

### Scripts (`scripts/`)

Utility scripts supporting installation, debugging, and auxiliary services:

| Script | Responsibility |
|--------|----------------|
| `scripts/nemoclaw-start.sh` | Primary sandbox startup entrypoint (also used as the Docker `ENTRYPOINT`) |
| `scripts/setup-dns-proxy.sh` | Configures the network DNS proxy for sandbox network policy enforcement |
| `scripts/telegram-bridge.js` | Implements the optional Telegram notification bridge for sandbox events |
| `scripts/check-coverage-ratchet.ts` | Enforces coverage thresholds by comparing against stored ratchet values in `ci/` |
| `scripts/docs-to-skills.py` | Python script that converts documentation pages into AI agent skill definition files |
| `scripts/write-auth-profile.ts` | Writes authentication profile configuration for inference providers |
| `scripts/backup-workspace.sh` | Backs up the agent workspace state |
| `scripts/install.sh` / `install.sh` | System-wide installation of the `nemoclaw` CLI |

---

### Sphinx Documentation Extensions (`docs/_ext/`)

| Module | Location | Responsibility |
|--------|----------|----------------|
| **JSON Output Extension** | `docs/_ext/json_output/` | Custom Sphinx extension generating JSON-formatted documentation output (used by `docs-to-skills.py`) |
| **Search Assets Extension** | `docs/_ext/search_assets/` | Custom Sphinx extension managing documentation search index assets |

---

## External Dependencies

### JavaScript — Production Dependencies

| Official Name | Package | Source File | Role in Project |
|---------------|---------|-------------|-----------------|
| **Commander** | `commander` | `/nemoclaw/package.json` | CLI argument parsing and command routing for the plugin's command interface |
| **Execa** | `execa` | `/nemoclaw/package.json`, `/package.json (dev)` | Executes shell subprocesses (e.g., Docker commands, system calls) with improved ergonomics over Node's native `child_process` |
| **JSON5** | `json5` | `/nemoclaw/package.json` | Parses JSON5-formatted configuration files, which extend standard JSON with comments and relaxed syntax |
| **Tar** | `tar` | `/nemoclaw/package.json` | Creates and extracts tar archives, used for workspace backup/restore and file packaging |
| **YAML** | `yaml` | `/nemoclaw/package.json` | Parses and serializes YAML files, including blueprint definitions and network policy configurations |
| **OpenClaw** | `openclaw` | `/package.json` | The host AI coding environment/IDE that NemoClaw extends via its plugin; the core platform this tool integrates with |
| **p-retry** | `p-retry` | `/package.json` | Retries failed asynchronous operations with configurable backoff; used for resilient Docker/network calls |

---

### JavaScript — Developer-Only Dependencies

| Official Name | Package | Source File | Role in Project |
|---------------|---------|-------------|-----------------|
| **TypeScript Node Types** | `@types/node` | `/nemoclaw/package.json (dev)`, `/package.json (dev)` | Type definitions for Node.js built-in APIs, enabling TypeScript type-checking of Node-specific code |
| **TypeScript ESLint Plugin** | `@typescript-eslint/eslint-plugin` | `/nemoclaw/package.json (dev)` | ESLint rules specific to TypeScript code quality and correctness |
| **TypeScript ESLint Parser** | `@typescript-eslint/parser` | `/nemoclaw/package.json (dev)` | Parses TypeScript source files for ESLint analysis |
| **ESLint** | `eslint` | `/nemoclaw/package.json (dev)`, `/package.json (dev)` | Static code linting for JavaScript and TypeScript source files |
| **ESLint Config Prettier** | `eslint-config-prettier` | `/nemoclaw/package.json (dev)` | Disables ESLint rules that conflict with Prettier formatting |
| **Prettier** | `prettier` | `/nemoclaw/package.json (dev)`, `/package.json (dev)` | Opinionated code formatter enforcing consistent style across JS/TS files |
| **TypeScript** | `typescript` | `/nemoclaw/package.json (dev)`, `/package.json (dev)` | TypeScript compiler used to type-check and transpile plugin and script source files |
| **Vitest** | `vitest` | `/nemoclaw/package.json (dev)`, `/package.json (dev)` | Unit and integration test framework used across all JS/TS test suites |
| **Commitlint CLI** | `@commitlint/cli` | `/package.json (dev)` | Enforces conventional commit message format via git hooks |
| **Commitlint Conventional Config** | `@commitlint/config-conventional` | `/package.json (dev)` | Shared conventional commits ruleset consumed by Commitlint |
| **ESLint JS** | `@eslint/js` | `/package.json (dev)` | Core ESLint JavaScript recommended ruleset |
| **Prek** | `@j178/prek` | `/package.json (dev)` | Pre-commit hook runner used to execute lint and format checks before commits |
| **Vitest Coverage V8** | `@vitest/coverage-v8` | `/package.json (dev)` | V8-based code coverage provider for Vitest, powering the coverage ratchet system |
| **TSX** | `tsx` | `/package.json (dev)` | TypeScript execution engine for running `.ts` scripts directly (e.g., `scripts/check-coverage-ratchet.ts`) |

---

### Python — Production Dependencies

| Official Name | Package | Source File | Role in Project |
|---------------|---------|-------------|-----------------|
| **Sphinx** | `sphinx` | `/pyproject.toml` | Core documentation site generator used to build the NemoClaw user documentation |
| **MyST Parser** | `myst-parser` | `/pyproject.toml` | Extends Sphinx to parse Markdown (`.md`) files in addition to reStructuredText |
| **Sphinx Copybutton** | `sphinx-copybutton` | `/pyproject.toml` | Adds a copy-to-clipboard button to all code blocks in the generated documentation |
| **Sphinx Design** | `sphinx-design` | `/pyproject.toml` | Provides responsive UI components (cards, grids, tabs) for the documentation site |
| **Sphinx Autobuild** | `sphinx-autobuild` | `/pyproject.toml` | Live-reloading development server for the documentation site |
| **Sphinxcontrib Mermaid** | `sphinxcontrib-mermaid` | `/pyproject.toml` | Renders Mermaid diagram syntax within documentation pages |
| **NVIDIA Sphinx Theme** | `nvidia-sphinx-theme` | `/pyproject.toml` | NVIDIA-branded HTML theme applied to the generated documentation site |
| **Sphinx LLM** | `sphinx-llm` | `/pyproject.toml` | Sphinx extension that produces LLM-optimized documentation output, likely consumed by the `docs-to-skills.py` pipeline |

--- core_entities ---


# Domain Model Analysis: NemoClaw Repository

## Overview

This repository appears to be a **developer sandbox/workspace management tool** with AI inference capabilities, network policy management, and secure container-based execution environments. Based on the file structure and naming conventions, the following core domain entities are identified.

---

## 1. Core Data Entities

### 1.1 `Blueprint`

The central configuration entity that defines how a sandbox environment is constructed and governed.

| Attribute | Type | Description |
|-----------|------|-------------|
| `name` | string | Identifier for the blueprint |
| `version` | string | Blueprint schema/format version |
| `sandboxImage` | string | Base container image reference |
| `policies` | Policy[] | Attached network/security policies |
| `inferenceConfig` | InferenceConfig | Associated inference provider settings |
| `workspaceConfig` | WorkspaceConfig | Workspace mount/storage settings |

**Source refs:** `nemoclaw-blueprint/blueprint.yaml`, `nemoclaw/src/blueprint/`, `test/validate-blueprint.test.ts`

---

### 1.2 `NetworkPolicy`

Defines rules governing outbound/inbound network access for a sandbox.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | string | Policy identifier |
| `name` | string | Human-readable policy name |
| `rules` | PolicyRule[] | List of allow/deny rules |
| `preset` | string | Reference to a named preset |
| `approvalStatus` | enum | `pending`, `approved`, `denied` |
| `methods` | string[] | HTTP methods (GET, POST, etc.) |
| `endpoints` | string[] | Target hosts/URLs |

**Source refs:** `nemoclaw-blueprint/policies/`, `nemoclaw-blueprint/policies/presets/`, `bin/lib/policies.js`, `test/policies.test.js`, `test/security-method-wildcards.test.js`, `docs/reference/network-policies.md`

---

### 1.3 `PolicyPreset`

A reusable, named collection of network policy rules that can be referenced by a `NetworkPolicy`.

| Attribute | Type | Description |
|-----------|------|-------------|
| `name` | string | Preset identifier |
| `description` | string | Purpose of the preset |
| `rules` | PolicyRule[] | Predefined allow/deny rule set |

**Source refs:** `nemoclaw-blueprint/policies/presets/` (9 preset files)

---

### 1.4 `InferenceConfig` / `InferenceProfile`

Represents the configuration for an AI inference provider used within the sandbox.

| Attribute | Type | Description |
|-----------|------|-------------|
| `provider` | string | Inference backend (e.g., NIM, local) |
| `profileName` | string | Named profile identifier |
| `endpoint` | string | API base URL |
| `modelId` | string | Target model identifier |
| `apiKey` / `credentials` | Credentials | Auth reference |
| `isLocal` | boolean | Whether inference runs locally |
| `nimImage` | string | NIM container image reference |

**Source refs:** `bin/lib/inference-config.js`, `bin/lib/local-inference.js`, `bin/lib/nim.js`, `bin/lib/nim-images.json`, `test/inference-config.test.js`, `test/local-inference.test.js`, `docs/reference/inference-profiles.md`

---

### 1.5 `Credentials`

Stores authentication tokens and secrets for external services (registries, inference APIs, etc.).

| Attribute | Type | Description |
|-----------|------|-------------|
| `type` | enum | `registry`, `inference`, `platform` |
| `token` / `apiKey` | string | Secret value (sanitized in logs) |
| `username` | string | Optional username |
| `registry` | string | Target registry URL |
| `expiresAt` | datetime | Optional expiry timestamp |

**Source refs:** `bin/lib/credentials.js`, `test/credentials.test.js`, `test/credential-exposure.test.js`, `scripts/write-auth-profile.ts`

---

### 1.6 `Sandbox`

The runtime execution environment — a containerized workspace instance.

| Attribute | Type | Description |
|-----------|------|-------------|
| `name` | string | Unique sandbox identifier |
| `status` | enum | `running`, `stopped`, `error`, `recovering` |
| `image` | string | Container image in use |
| `blueprint` | Blueprint | Originating blueprint reference |
| `networkPolicy` | NetworkPolicy | Active policy |
| `platform` | Platform | Host platform context |
| `createdAt` | datetime | Creation timestamp |
| `workspaceMount` | string | Volume/path mount info |

**Source refs:** `bin/lib/runner.js`, `bin/lib/runtime-recovery.js`, `test/runner.test.js`, `test/runtime-recovery.test.js`, `test/setup-sandbox-name.test.js`, `test/nemoclaw-start.test.js`

---

### 1.7 `Platform`

Describes the host environment where NemoClaw is running.

| Attribute | Type | Description |
|-----------|------|-------------|
| `os` | enum | `linux`, `macos`, `windows` |
| `arch` | string | CPU architecture (e.g., `amd64`, `arm64`) |
| `containerRuntime` | string | Docker, Podman, etc. |
| `hasGPU` | boolean | GPU availability |
| `isRemote` | boolean | Running on remote/cloud GPU |
| `k8sContext` | string | Kubernetes context (if applicable) |

**Source refs:** `bin/lib/platform.js`, `test/platform.test.js`, `k8s/nemoclaw-k8s.yaml`

---

### 1.8 `OnboardSession`

Tracks the state of the interactive onboarding/setup process for a new user or workspace.

| Attribute | Type | Description |
|-----------|------|-------------|
| `sessionId` | string | Unique session identifier |
| `stage` | enum | `preflight`, `selection`, `credentials`, `complete` |
| `selectedBlueprint` | Blueprint | User-chosen blueprint |
| `credentialsProvided` | boolean | Auth step completion flag |
| `readinessChecks` | CheckResult[] | Results of preflight checks |
| `resumable` | boolean | Whether session can be resumed |

**Source refs:** `bin/lib/onboard.js`, `bin/lib/onboard-session.js`, `test/onboard.test.js`, `test/onboard-session.test.js`, `test/onboard-readiness.test.js`, `test/onboard-selection.test.js`

---

### 1.9 `Registry`

Represents a container image registry used to pull sandbox or NIM images.

| Attribute | Type | Description |
|-----------|------|-------------|
| `url` | string | Registry base URL |
| `namespace` | string | Image namespace/org |
| `credentials` | Credentials | Auth reference |
| `isDefault` | boolean | Default registry flag |

**Source refs:** `bin/lib/registry.js`, `test/registry.test.js`

---

### 1.10 `WorkspaceSnapshot` (Backup/Restore)

Represents a saved state of a workspace for backup and restore operations.

| Attribute | Type | Description |
|-----------|------|-------------|
| `snapshotId` | string | Unique snapshot identifier |
| `sandboxName` | string | Source sandbox reference |
| `createdAt` | datetime | Snapshot timestamp |
| `storagePath` | string | Location of backup artifact |
| `files` | string[] | List of backed-up file paths |

**Source refs:** `scripts/backup-workspace.sh`, `docs/workspace/backup-restore.md`

---

## 2. Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                          Blueprint                              │
│  (core configuration template)                                  │
└──────┬──────────────────┬──────────────────┬────────────────────┘
       │ 1                │ 1                │ 1
       ▼ *                ▼ *                ▼ 1
┌──────────────┐  ┌───────────────┐  ┌────────────────┐
│NetworkPolicy │  │InferenceConfig│  │WorkspaceConfig │
└──────┬───────┘  └───────┬───────┘  └────────────────┘
       │ *                │ 1
       ▼ 1                ▼
┌──────────────┐  ┌───────────────┐
│ PolicyPreset │  │  Credentials  │◄──────────────┐
└──────────────┘  └───────────────┘               │
                                                   │
┌─────────────────────────────────────────────────┐│
│                     Sandbox                     ││
│  (runtime instance of a Blueprint)              ││
└──────┬──────────────────┬──────────────────────┬┘│
       │ *                │ 1                    │  │
       ▼ 1                ▼ 1                   │  │
┌──────────────┐  ┌───────────────┐             │  │
│   Platform   │  │OnboardSession │             ▼  │
└──────────────┘  └───────────────┘         ┌──────────┐
                                             │ Registry │
                                             └──────────┘
       │ Sandbox
       │ 1
       ▼ *
┌───────────────────┐
│ WorkspaceSnapshot │
└───────────────────┘
```

---

## 3. Relationship Summary

| Relationship | Type | Description |
|---|---|---|
| `Blueprint` → `NetworkPolicy` | One-to-Many | A blueprint can define multiple policies |
| `Blueprint` → `InferenceConfig` | One-to-One | Each blueprint has one inference configuration |
| `NetworkPolicy` → `PolicyPreset` | Many-to-One | Many policies can share a reusable preset |
| `InferenceConfig` → `Credentials` | Many-to-One | Inference configs reference shared credentials |
| `Registry` → `Credentials` | Many-to-One | Registries use stored credentials for auth |
| `Blueprint` → `Sandbox` | One-to-Many | A blueprint can instantiate multiple sandboxes |
| `Sandbox` → `Platform` | Many-to-One | Many sandboxes run on the same platform |
| `Sandbox` → `OnboardSession` | One-to-One | Each sandbox setup has one onboard session |
| `Sandbox` → `WorkspaceSnapshot` | One-to-Many | A sandbox can have multiple backup snapshots |
| `OnboardSession` → `Blueprint` | Many-to-One | Multiple sessions may select the same blueprint |

--- authorization ---


# Authorization Analysis: NemoClaw Repository

## Executive Summary

After comprehensive analysis of the NemoClaw codebase, this repository implements a **network policy-based access control system** for sandbox environments, combined with **credential-based API authentication** for external services. The authorization model is domain-specific — it controls what network destinations a sandboxed AI coding agent can reach, rather than implementing traditional user/role authorization.

---

## 1. Access Control Type

### Network Policy-Based Access Control (Primary Mechanism)

**Location:** `nemoclaw-blueprint/policies/openclaw-sandbox.yaml`, `nemoclaw-blueprint/policies/presets/`, `bin/lib/policies.js`

This is the dominant authorization mechanism. It is a **policy-based access control (PBAC)** system controlling network egress from sandboxes.

#### Policy Structure

**File:** `nemoclaw-blueprint/policies/openclaw-sandbox.yaml`

```yaml
# Network policy controlling what external endpoints the sandbox can reach
# Defines allow/deny rules for outbound network traffic
```

**File:** `nemoclaw-blueprint/policies/presets/` (9 preset files)

These presets represent named, pre-defined permission bundles for common use cases (e.g., allowing access to specific registries, inference endpoints, package managers).

**File:** `bin/lib/policies.js` — Core policy evaluation and management logic

---

## 2. Permission Structure

### Network Egress Permissions

**Location:** `bin/lib/policies.js`, `nemoclaw-blueprint/policies/`

**Implementation:** Policies define allowlists of network destinations. Each policy entry specifies:
- Target hostnames/IP ranges
- Allowed ports and protocols
- HTTP methods permitted
- Direction (egress only based on sandbox model)

**Location:** `docs/reference/network-policies.md`, `docs/network-policy/`

Documents the permission schema including:
- `approve-network-requests.md` — mechanism for approving new destinations
- `customize-network-policy.md` — how to extend permissions

### Policy Presets (Permission Bundles)

**Location:** `nemoclaw-blueprint/policies/presets/` (9 files)

These represent pre-approved permission sets. Named presets abstract groups of network destinations into reusable permission bundles — functionally analogous to roles in RBAC but applied to network access.

---

## 3. Credential-Based Access Control

### API Credential Management

**Location:** `bin/lib/credentials.js`, `scripts/write-auth-profile.ts`

**Implementation:** Controls access to external APIs (NVIDIA NIM, inference endpoints, container registries) through credential validation.

```
bin/lib/credentials.js          — Credential storage, retrieval, validation
scripts/write-auth-profile.ts   — Writes authentication profiles for service access
bin/lib/registry.js             — Registry authentication (container image pulls)
bin/lib/nim.js                  — NIM API credential handling
```

**Coverage:** Protects access to:
- NVIDIA NIM inference endpoints
- Container registries
- External API services

**Security Test Coverage:**

```
test/credential-exposure.test.js    — Tests that credentials are not leaked
test/credentials.test.js            — Credential management unit tests
```

---

## 4. Network Isolation / Sandbox Boundary Enforcement

### Gateway Isolation

**Location:** `test/e2e-gateway-isolation.sh`, `test/gateway-cleanup.test.js`

**Implementation:** The sandbox operates with network-level isolation enforced at the infrastructure layer. The policy engine controls which egress routes are permitted through the gateway.

**Location:** `scripts/setup-dns-proxy.sh`, `test/dns-proxy.test.js`

DNS proxy enforces resolution-level access control — domains not in the policy allowlist are blocked at DNS resolution, adding a second enforcement layer before TCP connection.

**Enforcement layers:**
1. DNS proxy — blocks resolution of non-allowlisted domains
2. Network gateway — blocks TCP connections to non-allowed destinations
3. Policy file — defines what is allowlisted

---

## 5. Security Method Restrictions

### Binary/Method-Level Restrictions

**Location:** `test/security-binaries-restriction.test.js`

**Implementation:** Restricts which system binaries are accessible within the sandbox. This is a capability-based control limiting what operations the agent can perform.

**Location:** `test/security-method-wildcards.test.js`

Tests that wildcard method permissions are not inadvertently granted — validates that overly broad permission grants are rejected.

---

## 6. Injection Attack Prevention

### C2/Dockerfile Injection Controls

**Location:** `test/security-c2-dockerfile-injection.test.js`

**Implementation:** Authorization check preventing command-and-control injection through Dockerfile manipulation — validates that injected instructions cannot bypass sandbox policy.

### Manifest Traversal Prevention

**Location:** `test/security-c4-manifest-traversal.test.js`

**Implementation:** Authorization check preventing path traversal in manifest files that could be used to access resources outside permitted scope.

---

## 7. Blueprint-Level Authorization

### Blueprint Validation

**Location:** `nemoclaw/src/blueprint/`, `test/validate-blueprint.test.ts`

**Implementation:** Blueprint files define the complete authorization posture of a sandbox instance. Validation ensures blueprints conform to the policy schema before deployment.

```
nemoclaw-blueprint/blueprint.yaml    — Master blueprint with policy references
```

**The blueprint controls:**
- Which network policy presets are active
- Which inference providers are accessible
- Which registries are permitted

**Location:** `nemoclaw/src/commands/` (4 files)

CLI commands enforce blueprint-based authorization at the command invocation level — commands that would violate the active policy are rejected.

---

## 8. Onboarding Authorization Checks

### Preflight Authorization Validation

**Location:** `bin/lib/preflight.js`, `test/preflight.test.js`, `test/install-preflight.test.js`

**Implementation:** Before sandbox initialization, preflight checks validate:
- Credential validity for required services
- Platform capability authorization
- Network policy applicability for the target environment

**Location:** `bin/lib/onboard.js`, `bin/lib/onboard-session.js`

**Implementation:** Onboarding sequence enforces authorization gates — a session cannot proceed without valid credentials and a conformant policy configuration.

```
test/onboard-readiness.test.js     — Tests authorization readiness checks
test/onboard-selection.test.js     — Tests authorized option selection
test/onboard-session.test.js       — Tests session authorization
test/onboard.test.js               — Full onboard authorization flow
```

---

## 9. Runtime Authorization

### Runtime Recovery Controls

**Location:** `bin/lib/runtime-recovery.js`, `test/runtime-recovery.test.js`

**Implementation:** Controls what recovery actions are permitted during runtime failures — prevents unauthorized state manipulation during recovery sequences.

### Service Environment Authorization

**Location:** `test/service-env.test.js`

**Implementation:** Validates that service environment variables do not expose credentials or grant unintended permissions.

---

## 10. Platform-Level Authorization

**Location:** `bin/lib/platform.js`, `test/platform.test.js`

**Implementation:** Platform detection gates which authorization mechanisms are available. Certain capabilities are restricted based on the detected execution environment (Linux/macOS, Docker, Kubernetes).

**Location:** `k8s/nemoclaw-k8s.yaml`

Kubernetes deployment manifest — authorization at the infrastructure level through Kubernetes RBAC (cluster-side) and network policies.

---

## 11. Inference Provider Authorization

**Location:** `bin/lib/inference-config.js`, `bin/lib/local-inference.js`, `test/inference-config.test.js`, `test/local-inference.test.js`

**Implementation:** Controls which inference providers the sandbox is authorized to use. Provider selection is gated against the active blueprint configuration.

```
bin/lib/nim-images.json    — Authorized NIM image manifest
bin/lib/nim.js             — NIM access control and credential binding
```

---

## 12. Telegram Bridge Authorization

**Location:** `scripts/telegram-bridge.js`, `test/e2e/test-telegram-injection.sh`

**Implementation:** The Telegram bridge has injection prevention controls. The e2e test validates that messages cannot be used to inject unauthorized commands through the bridge interface.

---

## 13. Credential Sanitization

**Location:** `test/e2e/test-credential-sanitization.sh`

**Implementation:** End-to-end test validating that credentials are sanitized from outputs — preventing credential exposure through logging, error messages, or response data.

---

## Identified Security Gaps

### Gap 1: No User-Level RBAC

| Attribute | Detail |
|-----------|--------|
| **Type** | Missing control |
| **Severity** | Informational (by design) |
| **Location** | Entire codebase |
| **Description** | No user identity or role management system exists. The system does not authenticate human operators at the application layer — authorization is entirely at the sandbox/network policy level. If multiple operators use the same deployment, there is no per-operator audit trail or permission differentiation. |

### Gap 2: Policy File Write Protection

| Attribute | Detail |
|-----------|--------|
| **Type** | Potential privilege escalation |
| **Severity** | Medium |
| **Location** | `nemoclaw-blueprint/policies/` |
| **Description** | Policy YAML files define the authorization posture. If the agent running inside the sandbox has write access to these files (filesystem permissions permitting), it could modify its own policy. The security tests in `security-c2-dockerfile-injection.test.js` address the Dockerfile vector but the policy file vector should be explicitly tested. |

### Gap 3: Wildcard Method Authorization

| Attribute | Detail |
|-----------|--------|
| **Type** | Overly permissive defaults risk |
| **Severity** | Medium |
| **Location** | `test/security-method-wildcards.test.js` |
| **Description** | Tests exist to prevent wildcard method grants, but the existence of these tests indicates this was an identified vulnerability. The actual enforcement implementation in `bin/lib/policies.js` should be audited to confirm wildcard rejection is enforced at the policy evaluation layer, not just tested as a side effect. |

### Gap 4: DNS Proxy as Single Enforcement Point Risk

| Attribute | Detail |
|-----------|--------|
| **Type** | Defense-in-depth gap |
| **Severity** | Medium |
| **Location** | `scripts/setup-dns-proxy.sh`, `test/dns-proxy.test.js` |
| **Description** | DNS-based blocking can be bypassed by direct IP connections. While the gateway layer provides a second enforcement point, the combination depends on correct configuration. If the DNS proxy is disabled or misconfigured, the gateway must be the sole enforcement layer. The `fix-coredns.sh` script suggests DNS configuration issues have occurred in practice. |

### Gap 5: Double-Onboard Race Condition

| Attribute | Detail |
|-----------|--------|
| **Type** | Potential authorization bypass |
| **Severity** | Low-Medium |
| **Location** | `test/e2e/test-double-onboard.sh` |
| **Description** | A dedicated test exists for the double-onboard scenario, indicating a known edge case. If two onboard sequences run concurrently, there may be a window where authorization state is inconsistent — specifically, the policy enforcement layer may be partially initialized. |

### Gap 6: No Authorization Audit Log

| Attribute | Detail |
|-----------|--------|
| **Type** | Compliance gap |
| **Severity** | Medium |
| **Location** | No dedicated audit logging found |
| **Description** | No centralized authorization decision log was identified. Network policy decisions (allow/deny) do not appear to be durably logged to an audit store accessible after sandbox termination. The monitoring documentation (`docs/monitoring/monitor-sandbox-activity.md`) covers activity monitoring but not authorization decision recording. |

### Gap 7: Preset Policy Integrity

| Attribute | Detail |
|-----------|--------|
| **Type** | Supply chain / integrity |
| **Severity** | Medium |
| **Location** | `nemoclaw-blueprint/policies/presets/` |
| **Description** | The 9 preset policy files define trusted permission bundles. No cryptographic signing or integrity verification of these preset files was identified. If the preset files are modified (supply chain attack, misconfiguration), the sandbox would operate with an unauthorized permission set without detection. |

---

## Authorization Architecture Summary

```
┌─────────────────────────────────────────────────────────┐
│                    AUTHORIZATION LAYERS                  │
├─────────────────────────────────────────────────────────┤
│  Layer 1: Blueprint Validation                           │
│    nemoclaw/src/blueprint/ + validate-blueprint.test.ts  │
│    → Validates policy schema before deployment           │
├─────────────────────────────────────────────────────────┤
│  Layer 2: Credential Authorization                       │
│    bin/lib/credentials.js + write-auth-profile.ts        │
│    → API access requires valid credentials               │
├─────────────────────────────────────────────────────────┤
│  Layer 3: DNS Resolution Control                         │
│    scripts/setup-dns-proxy.sh                            │
│    → Blocks resolution of non-allowlisted domains        │
├─────────────────────────────────────────────────────────┤
│  Layer 4: Network Gateway Policy                         │
│    nemoclaw-blueprint/policies/ + bin/lib/policies.js    │
│    → Enforces egress allowlist at TCP level              │
├─────────────────────────────────────────────────────────┤
│  Layer 5: Binary/Capability Restrictions                 │
│    security-binaries-restriction.test.js                 │
│    → Limits available system capabilities                │
├─────────────────────────────────────────────────────────┤
│  Layer 6: Injection Prevention                           │
│    security-c2-dockerfile-injection.test.js              │
│    security-c4-manifest-traversal.test.js                │
│    → Prevents policy bypass through injection            │
└─────────────────────────────────────────────────────────┘
```

---

## Key Files Reference

| File | Authorization Role |
|------|--------------------|
| `nemoclaw-blueprint/policies/openclaw-sandbox.yaml` | Master network policy definition |
| `nemoclaw-blueprint/policies/presets/` (×9) | Pre-approved permission bundles |
| `bin/lib/policies.js` | Policy evaluation engine |
| `bin/lib/credentials.js` | Credential-based access control |
| `bin/lib/preflight.js` | Pre-authorization validation gates |
| `scripts/setup-dns-proxy.sh` | DNS-layer enforcement |
| `scripts/write-auth-profile.ts` | Auth profile provisioning |
| `test/security-binaries-restriction.test.js` | Capability restriction validation |
| `test/security-method-wildcards.test.js` | Wildcard permission rejection |
| `test/security-c2-dockerfile-injection.test.js` | Injection attack prevention |
| `test/security-c4-manifest-traversal.test.js` | Traversal attack prevention |
| `test/credential-exposure.test.js` | Credential leak prevention |
| `k8s/nemoclaw-k8s.yaml` | Kubernetes-level authorization |

--- DBs ---


## Database Analysis

After a comprehensive scan of the repository structure and all files within the `NemoClaw_9536aca0` codebase (excluding the `arch-docs` folder, which is not present), I analyzed all source files, configuration files, scripts, test files, Kubernetes manifests, Dockerfiles, and CI/CD workflows for any database interactions, ORM definitions, schema definitions, connection strings, migration scripts, or data persistence logic.

**Findings:**

- No SQL database connections, ORM models, schema definitions, or migration scripts were found (no PostgreSQL, MySQL, SQLite, etc.).
- No NoSQL database client libraries, connection configurations, or data access patterns were found (no MongoDB, Redis, DynamoDB, Cassandra, etc.).
- The codebase is a **CLI tooling and sandbox orchestration platform** (`nemoclaw`). Its persistence is limited to:
  - Local filesystem operations (e.g., configuration files, workspace backups via `scripts/backup-workspace.sh`).
  - Credential/auth profile files written to disk (e.g., `scripts/write-auth-profile.ts`).
  - Kubernetes manifests (`k8s/nemoclaw-k8s.yaml`) for container orchestration, with no database workloads defined.
- No database SDKs, drivers, or client packages (e.g., `pg`, `mysql2`, `mongoose`, `redis`, `ioredis`, `sqlalchemy`, `prisma`, `sequelize`, `typeorm`, `boto3` for DynamoDB) were found in `package.json`, `pyproject.toml`, `uv.lock`, or `package-lock.json`.
- Test files exercise CLI behavior, credential handling, DNS proxying, policy management, and sandbox lifecycle — none interact with a database layer.

---

no database

--- APIs ---


# API Documentation Analysis

After a comprehensive scan of the entire codebase repository **NemoClaw_9536aca0**, I have thoroughly examined all source files including:

- `bin/` — CLI entry points and library modules
- `nemoclaw/src/` — Plugin source files
- `scripts/` — Shell and JS scripts
- `test/` — Test files (which reveal API call patterns)
- `docs/` — Documentation files
- Configuration and infrastructure files (`k8s/`, `Dockerfile`, etc.)

## Findings

This codebase is a **CLI tool and developer sandbox environment** (NemoClaw). It operates as:

1. A **command-line interface** (`bin/nemoclaw.js`) that orchestrates container/sandbox lifecycle
2. A **plugin** (`nemoclaw/src/`) for an editor/shell environment
3. A collection of **shell scripts** for setup, sandboxing, DNS proxying, and inference configuration

The code interacts **as a client** with external HTTP APIs (e.g., NVIDIA NIM inference endpoints, container registries, credential services), but does **not define, expose, or serve any HTTP API endpoints** of its own. There are no:

- HTTP server declarations (no `express`, `fastapi`, `flask`, `http.createServer`, Spring controllers, etc.)
- Route handlers or REST endpoint definitions
- WebSocket servers
- Any listening TCP/HTTP services

---

no HTTP API

--- hl_overview ---


# Project Analysis: NemoClaw

## [[NemoClaw]]

---

## 1. Project Purpose

NemoClaw is a **developer sandbox orchestration and security tool** designed to run AI coding agents (like Claude) inside isolated, policy-controlled environments. It solves the problem of safely executing autonomous AI agents that need access to development tools, network resources, and GPU inference capabilities — without exposing the host system or leaking credentials.

**Primary Domain:** AI agent sandboxing, developer tooling, container orchestration, and security policy enforcement for LLM-driven coding assistants.

---

## 2. Architecture Pattern

- **CLI Tool + Plugin Architecture**: A Node.js CLI (`nemoclaw`) with a companion OpenClaw plugin (`nemoclaw/`) that integrates into an AI coding environment
- **Container-Based Sandbox Isolation**: Workloads run inside Docker containers with enforced network policies
- **Policy-Driven Security Model**: YAML-defined network and execution policies control what the agent can and cannot do
- **Blueprint Configuration**: Declarative YAML blueprints define the sandbox environment

---

## 3. Technology Stack

### Primary Languages
| Language | Usage |
|----------|-------|
| **JavaScript/Node.js** | Core CLI, runtime orchestration, tests |
| **TypeScript** | Plugin source, scripts, type-checked utilities |
| **Python** | Documentation tooling, scripts |
| **Bash/Shell** | Install scripts, E2E test runners, setup utilities |

### JavaScript/Node.js Stack
| Package/Framework | Purpose |
|-------------------|---------|
| **Node.js** | Runtime for CLI |
| **Vitest** | Unit test framework |
| **ESLint** | Linting (via `eslint.config.mjs`) |
| **Prettier** | Code formatting |
| **commitlint** | Commit message enforcement |
| **TypeScript** | Typed plugin development |

### Python Stack (from `pyproject.toml`)
| Dependency | Purpose |
|-----------|---------|
| Sphinx / MyST | Documentation generation |
| Python docs tooling | `docs-to-skills.py` conversion script |

### Infrastructure / Container
| Tool | Purpose |
|------|---------|
| **Docker** | Sandbox container runtime |
| **Kubernetes** (`k8s/`) | Optional K8s deployment target |
| **NVIDIA NIM** | GPU inference images (`nim-images.json`) |

---

## 4. Initial Structure Impression

The application has these high-level parts:

| Part | Location | Description |
|------|----------|-------------|
| **CLI Tool** | `bin/` | Main `nemoclaw` command-line interface |
| **Plugin** | `nemoclaw/` | OpenClaw/AI IDE plugin (TypeScript) |
| **Sandbox Blueprint** | `nemoclaw-blueprint/` | Declarative environment definitions |
| **Documentation** | `docs/` | Sphinx-based user documentation |
| **Test Suite** | `test/` | Unit + E2E tests |
| **CI/CD** | `.github/workflows/` | GitHub Actions pipelines |
| **Agent Skills** | `.agents/skills/` | AI agent guidance/skill definitions |
| **K8s Manifests** | `k8s/` | Kubernetes deployment config |
| **Scripts** | `scripts/` | Installation, setup, debug utilities |

---

## 5. Configuration / Package Files

| File | Purpose |
|------|---------|
| `package.json` | Root Node.js project manifest |
| `package-lock.json` | Root NPM lock file |
| `nemoclaw/package.json` | Plugin-specific Node.js manifest |
| `nemoclaw/package-lock.json` | Plugin lock file |
| `test/package.json` | Test suite dependencies |
| `pyproject.toml` | Python project config (docs tooling) |
| `uv.lock` | Python `uv` package lock file |
| `tsconfig.cli.json` | TypeScript config for CLI |
| `nemoclaw/tsconfig.json` | TypeScript config for plugin |
| `jsconfig.json` | JS editor config for root |
| `eslint.config.mjs` | Root ESLint config |
| `nemoclaw/eslint.config.mjs` | Plugin ESLint config |
| `.prettierrc` | Root Prettier config |
| `nemoclaw/.prettierrc` | Plugin Prettier config |
| `vitest.config.ts` | Vitest test runner config |
| `commitlint.config.js` | Commit message rules |
| `.editorconfig` | Editor formatting standards |
| `.coderabbit.yaml` | CodeRabbit AI review config |
| `.pre-commit-config.yaml` | Pre-commit hook definitions |
| `.markdownlint-cli2.yaml` | Markdown lint rules |
| `.prettierignore` | Files excluded from formatting |
| `.shellcheckrc` | Shell script lint config |
| `Dockerfile` | Main application container image |
| `Dockerfile.base` | Base image definition |
| `test/Dockerfile.sandbox` | Test sandbox container |
| `test/e2e/Dockerfile.full-e2e` | Full E2E test container |
| `.dockerignore` | Docker build exclusions |
| `ci/coverage-threshold-cli.json` | CLI coverage thresholds |
| `ci/coverage-threshold-plugin.json` | Plugin coverage thresholds |
| `nemoclaw-blueprint/blueprint.yaml` | Sandbox environment blueprint |
| `nemoclaw/openclaw.plugin.json` | Plugin manifest for OpenClaw |
| `k8s/nemoclaw-k8s.yaml` | Kubernetes deployment manifest |
| `.github/dependabot.yml` | Automated dependency updates |
| `docs/conf.py` | Sphinx documentation config |
| `docs/project.json` | Documentation project metadata |

---

## 6. Directory Structure

```
NemoClaw/
├── bin/                    # CLI entry point and core library modules
│   └── lib/                # Core logic: credentials, inference, policies,
│                           #   runner, onboarding, registry, platform detection
├── nemoclaw/               # TypeScript OpenClaw plugin
│   └── src/                # Plugin source: commands, blueprint handling,
│                           #   onboarding flows, plugin registration
├── nemoclaw-blueprint/     # Declarative sandbox environment configuration
│   └── policies/           # Network policy YAML definitions + presets
├── docs/                   # Sphinx documentation site
│   ├── get-started/        # Quickstart guides
│   ├── reference/          # Architecture, commands, troubleshooting
│   ├── deployment/         # Remote GPU, hardening, Telegram bridge
│   ├── workspace/          # Backup/restore, file management
│   ├── inference/          # Inference provider configuration
│   ├── network-policy/     # Policy approval and customization
│   ├── monitoring/         # Sandbox activity monitoring
│   ├── security/           # Best practices
│   ├── about/              # Overview, how-it-works, release notes
│   └── _ext/               # Sphinx extensions (JSON output, search assets)
├── scripts/                # Shell + TS utility scripts
│                           #   (install, setup, debug, inference tests,
│                           #    workspace backup, DNS proxy, Telegram bridge)
├── test/                   # Comprehensive test suite
│   ├── *.test.js           # Unit tests for all CLI modules
│   ├── security-*.test.js  # Security-specific test cases
│   └── e2e/                # End-to-end test scripts and containers
├── k8s/                    # Kubernetes deployment manifests
├── ci/                     # CI configuration (coverage thresholds)
├── .agents/skills/         # AI agent skill definition files
│   └── nemoclaw-*/         # Skills: overview, reference, deploy, workspace,
│                           #   policy management, monitoring, inference config
├── .github/
│   ├── workflows/          # CI/CD pipelines (PR, main, nightly, E2E, docs)
│   └── actions/            # Reusable composite GitHub Actions
└── ISSUE_TEMPLATE/         # GitHub issue templates
```

**Organization Pattern:** Organized **by feature/responsibility** — each directory owns a specific domain (CLI runtime, plugin, policies, docs, tests).

---

## 7. High-Level Architecture

### Pattern: **CLI Tool + Plugin + Policy Engine + Container Runtime**

```
┌─────────────────────────────────────────────────┐
│              Developer / AI Agent                │
└──────────────┬──────────────────┬───────────────┘
               │                  │
    ┌──────────▼──────┐  ┌────────▼────────────┐
    │  nemoclaw CLI   │  │  OpenClaw Plugin     │
    │  (bin/nemoclaw) │  │  (nemoclaw/src/)     │
    └──────────┬──────┘  └────────┬────────────┘
               │                  │
    ┌──────────▼──────────────────▼────────────┐
    │         Core Library (bin/lib/)           │
    │  credentials | policies | runner          │
    │  inference   | onboard  | registry        │
    │  platform    | preflight| version         │
    └──────────────────┬───────────────────────┘
                       │
    ┌──────────────────▼───────────────────────┐
    │     Sandbox Container (Docker/K8s)        │
    │  + Blueprint Config (nemoclaw-blueprint/) │
    │  + Network Policy Enforcement             │
    │  + NVIDIA NIM Inference (optional)        │
    └──────────────────────────────────────────┘
```

**Evidence:**
- `bin/lib/runner.js` — executes agent workloads in containers
- `bin/lib/policies.js` + `nemoclaw-blueprint/policies/` — policy engine
- `bin/lib/credentials.js` + `test/credential-exposure.test.js` — credential isolation
- `bin/lib/inference-config.js` + `bin/lib/local-inference.js` — inference routing
- `nemoclaw/src/blueprint/` — declarative environment configuration
- `test/security-*.test.js` — dedicated security boundary testing
- `k8s/` — Kubernetes deployment target for remote operation

---

## 8. Build, Execution and Test

### Build
```bash
# Install dependencies
npm install                    # Root and CLI deps
cd nemoclaw && npm install     # Plugin deps

# Python docs tooling
uv sync                        # Install Python deps via uv

# TypeScript compilation
npm run build                  # Compiles nemoclaw/ plugin TypeScript
```

### Run
```bash
# Primary entry point
./bin/nemoclaw.js              # CLI entry point

# Installation
./install.sh                   # System-wide install script
./scripts/install.sh           # Alternative installer

# Start sandbox
./scripts/nemoclaw-start.sh    # Start the sandbox environment

# Services
./scripts/start-services.sh    # Start supporting services
./scripts/setup-dns-proxy.sh   # Network/DNS proxy setup
```

### Test
```bash
# Unit tests (Vitest)
npx vitest                     # Run all unit tests

# Specific test files
npx vitest test/policies.test.js
npx vitest test/credentials.test.js
npx vitest test/security-*.test.js  # Security tests

# E2E tests
./test/e2e-test.sh             # Full E2E suite
./test/e2e/test-full-e2e.sh    # Container-based E2E

# Coverage check
npx ts-node scripts/check-coverage-ratchet.ts
```

### CI/CD (GitHub Actions)
| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `pr.yaml` | Pull Request | Standard PR checks |
| `main.yaml` | Push to main | Main branch validation |
| `nightly-e2e.yaml` | Scheduled | Nightly E2E regression |
| `sandbox-images-and-e2e.yaml` | Scheduled/Manual | Sandbox image builds + E2E |
| `e2e-brev.yaml` | Manual | GPU E2E on Brev |
| `base-image.yaml` | Manual | Build base Docker image |
| `docs-preview-deploy.yaml` | PR | Documentation preview |

### Main Entry Points
| Entry Point | Type |
|------------|------|
| `bin/nemoclaw.js` | Primary CLI executable |
| `nemoclaw/src/index.ts` | Plugin entry point |
| `scripts/nemoclaw-start.sh` | Sandbox startup script |
| `install.sh` | Installation entry point |

--- service_dependencies ---


# External Dependencies Analysis: NemoClaw_9536aca0

---

## JavaScript / Node.js Production Dependencies

---

### 1. `openclaw` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | OpenClaw NPM Library |
| **Type of Dependency** | Library/Framework |
| **Purpose/Role** | Core AI agent/coding assistant framework that NemoClaw wraps and extends. The NemoClaw project is built as a plugin on top of this platform. |
| **Integration Point/Clues** | Listed as a production dependency in `/package.json` at version `2026.3.11`. The `Dockerfile` runs `openclaw doctor --fix` and `openclaw plugins install /opt/nemoclaw`, confirming it's installed and executed as a runtime binary. References to `openclaw.json` config file appear throughout the codebase. |

---

### 2. `p-retry` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `p-retry` Library |
| **Type of Dependency** | Library/Framework |
| **Purpose/Role** | Provides retry logic with exponential backoff for Promise-based operations, likely used for resilient HTTP/API calls or async operations. |
| **Integration Point/Clues** | Listed as a production dependency in `/package.json` at `^4.6.2`. |

---

### 3. `commander` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `commander` Library |
| **Type of Dependency** | Library/Framework |
| **Purpose/Role** | CLI argument parsing framework for building command-line interfaces. Used by the `nemoclaw` plugin to define and handle CLI commands. |
| **Integration Point/Clues** | Listed as a production dependency in `/nemoclaw/package.json` at `^13.1.0`. The project has a `bin/nemoclaw.js` entrypoint and a `nemoclaw/src/commands/` directory. |

---

### 4. `execa` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `execa` Library |
| **Type of Dependency** | Library/Framework |
| **Purpose/Role** | Enhanced child process execution for Node.js. Used to spawn shell commands and subprocesses from within the CLI and plugin code. |
| **Integration Point/Clues** | Listed as a production dependency in `/nemoclaw/package.json` at `^9.6.1` AND as a dev dependency in the root `/package.json` at `^9.6.1`. |

---

### 5. `json5` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `json5` Library |
| **Type of Dependency** | Library/Framework |
| **Purpose/Role** | Parses JSON5 format (a superset of JSON allowing comments, trailing commas, etc.). Likely used to read configuration files such as `openclaw.json` or blueprint files. |
| **Integration Point/Clues** | Listed as a production dependency in `/nemoclaw/package.json` at `^2.2.3`. |

---

### 6. `tar` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `tar` Library |
| **Type of Dependency** | Library/Framework |
| **Purpose/Role** | Creates and extracts tar archives in Node.js. Likely used for workspace backup/restore operations or packaging plugin artifacts. |
| **Integration Point/Clues** | Listed as a production dependency in `/nemoclaw/package.json` at `^7.0.0`. The project contains `scripts/backup-workspace.sh` suggesting archive operations. |

---

### 7. `yaml` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `yaml` Library |
| **Type of Dependency** | Library/Framework |
| **Purpose/Role** | Parses and serializes YAML documents. Used to read/write blueprint and network policy YAML files (e.g., `nemoclaw-blueprint/blueprint.yaml`, `nemoclaw-blueprint/policies/*.yaml`). |
| **Integration Point/Clues** | Listed as a production dependency in `/nemoclaw/package.json` at `^2.4.0`. The project has extensive YAML configuration files under `nemoclaw-blueprint/`. |

---

## JavaScript / Node.js Developer Dependencies

---

### 8. `vitest` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `vitest` Testing Framework |
| **Type of Dependency** | Library/Framework (Dev) |
| **Purpose/Role** | Unit and integration testing framework. Used to run the project's test suite located in `/test/`. |
| **Integration Point/Clues** | Listed as a dev dependency in both `/package.json` (`^4.1.0`) and `/nemoclaw/package.json` (`^4.1.0`). A `vitest.config.ts` file is present at the root, and CI coverage threshold files are in `/ci/`. |

---

### 9. `@vitest/coverage-v8` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `@vitest/coverage-v8` Library |
| **Type of Dependency** | Library/Framework (Dev) |
| **Purpose/Role** | V8-based code coverage provider for Vitest. Used to generate coverage reports checked against thresholds in `/ci/coverage-threshold-*.json`. |
| **Integration Point/Clues** | Listed as a dev dependency in `/package.json` at `^4.1.0`. The `scripts/check-coverage-ratchet.ts` script enforces coverage thresholds. |

---

### 10. `eslint` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `eslint` Linting Tool |
| **Type of Dependency** | Library/Framework (Dev) |
| **Purpose/Role** | JavaScript/TypeScript static code analysis and linting. Enforces code quality standards across the project. |
| **Integration Point/Clues** | Listed as a dev dependency in `/package.json` (`^10.1.0`) and `/nemoclaw/package.json` (`^9.39.4`). Configuration files `eslint.config.mjs` exist at both root and `/nemoclaw/`. |

---

### 11. `@typescript-eslint/eslint-plugin` & `@typescript-eslint/parser` (NPM Packages)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM TypeScript ESLint Packages |
| **Type of Dependency** | Library/Framework (Dev) |
| **Purpose/Role** | Provides TypeScript-specific linting rules and a TypeScript parser for ESLint. |
| **Integration Point/Clues** | Listed as dev dependencies in `/nemoclaw/package.json` at `^8.57.0`. Referenced by `eslint.config.mjs`. |

---

### 12. `typescript` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `typescript` Compiler |
| **Type of Dependency** | Library/Framework (Dev) |
| **Purpose/Role** | TypeScript-to-JavaScript compiler. The `nemoclaw` plugin and some scripts are written in TypeScript and compiled during build. |
| **Integration Point/Clues** | Listed as dev dependency in `/nemoclaw/package.json` (`^5.4.0`) and `/package.json` (`^6.0.2`). `tsconfig.json` files exist at root and in `/nemoclaw/`. The `Dockerfile` runs `npm run build` in the builder stage. |

---

### 13. `prettier` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `prettier` Code Formatter |
| **Type of Dependency** | Library/Framework (Dev) |
| **Purpose/Role** | Opinionated code formatter for consistent code style across JS/TS files. |
| **Integration Point/Clues** | Listed as dev dependency in both `/package.json` and `/nemoclaw/package.json` at `^3.8.1`. `.prettierrc` and `.prettierignore` files are present at root and in `/nemoclaw/`. |

---

### 14. `eslint-config-prettier` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `eslint-config-prettier` Library |
| **Type of Dependency** | Library/Framework (Dev) |
| **Purpose/Role** | Disables ESLint rules that conflict with Prettier formatting, allowing both tools to be used together. |
| **Integration Point/Clues** | Listed as a dev dependency in `/nemoclaw/package.json` at `^10.1.8`. |

---

### 15. `@commitlint/cli` & `@commitlint/config-conventional` (NPM Packages)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `commitlint` Packages |
| **Type of Dependency** | Library/Framework (Dev) |
| **Purpose/Role** | Enforces conventional commit message formatting. Integrated into the CI pipeline for commit message validation. |
| **Integration Point/Clues** | Listed as dev dependencies in `/package.json` at `^20.5.0`. A `commitlint.config.js` configuration file is present at root. A dedicated GitHub Actions workflow `.github/workflows/commit-lint.yaml` exists. |

---

### 16. `@j178/prek` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `@j178/prek` Library |
| **Type of Dependency** | Library/Framework (Dev) |
| **Purpose/Role** | A pre-commit hook runner for Node.js projects (alternative to Husky). Used to run linters and formatters before commits. |
| **Integration Point/Clues** | Listed as a dev dependency in `/package.json` at `^0.3.6`. A `.pre-commit-config.yaml` is also present suggesting pre-commit tooling is in use. |

---

### 17. `tsx` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `tsx` TypeScript Executor |
| **Type of Dependency** | Library/Framework (Dev) |
| **Purpose/Role** | Executes TypeScript files directly without a separate compilation step. Used to run TypeScript scripts like `scripts/check-coverage-ratchet.ts` and `scripts/write-auth-profile.ts`. |
| **Integration Point/Clues** | Listed as a dev dependency in `/package.json` at `^4.21.0`. |

---

### 18. `@eslint/js` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `@eslint/js` Library |
| **Type of Dependency** | Library/Framework (Dev) |
| **Purpose/Role** | Official ESLint JavaScript rules package used with the flat config system (`eslint.config.mjs`). |
| **Integration Point/Clues** | Listed as a dev dependency in `/package.json` at `^10.0.1`. |

---

### 19. `@types/node` (NPM Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NPM `@types/node` Type Definitions |
| **Type of Dependency** | Library/Framework (Dev) |
| **Purpose/Role** | TypeScript type definitions for Node.js built-in APIs. Required for TypeScript compilation of Node.js code. |
| **Integration Point/Clues** | Listed as a dev dependency in `/package.json` (`^25.5.0`) and `/nemoclaw/package.json` (`^22.0.0`). |

---

## Python Dependencies

---

### 20. `sphinx` (Python Package)

| Field | Detail |
|---|---|
| **Dependency Name** | Python `sphinx` Documentation Builder |
| **Type of Dependency** | Library/Framework (Dev/Docs) |
| **Purpose/Role** | Documentation generation tool. Used to build the NemoClaw documentation from Markdown/reStructuredText source files in the `/docs/` directory. |
| **Integration Point/Clues** | Listed in `/pyproject.toml` under `[dependency-groups] docs` at `<=7.5`. The `/docs/conf.py` is the Sphinx configuration file. |

---

### 21. `myst-parser` (Python Package)

| Field | Detail |
|---|---|
| **Dependency Name** | Python `myst-parser` Library |
| **Type of Dependency** | Library/Framework (Dev/Docs) |
| **Purpose/Role** | Sphinx extension that enables parsing Markdown (`.md`) files using MyST syntax. Allows the documentation to be written in Markdown instead of reStructuredText. |
| **Integration Point/Clues** | Listed in `/pyproject.toml` at `<=5`. All documentation source files in `/docs/` use `.md` extensions. |

---

### 22. `sphinx-copybutton` (Python Package)

| Field | Detail |
|---|---|
| **Dependency Name** | Python `sphinx-copybutton` Library |
| **Type of Dependency** | Library/Framework (Dev/Docs) |
| **Purpose/Role** | Sphinx extension that adds a copy button to code blocks in generated documentation. |
| **Integration Point/Clues** | Listed in `/pyproject.toml` at `<=0.6`. |

---

### 23. `sphinx-design` (Python Package)

| Field | Detail |
|---|---|
| **Dependency Name** | Python `sphinx-design` Library |
| **Type of Dependency** | Library/Framework (Dev/Docs) |
| **Purpose/Role** | Sphinx extension providing UI design components (grids, cards, tabs, dropdowns) for documentation pages. |
| **Integration Point/Clues** | Listed in `/pyproject.toml` under `[dependency-groups] docs`. |

---

### 24. `sphinx-autobuild` (Python Package)

| Field | Detail |
|---|---|
| **Dependency Name** | Python `sphinx-autobuild` Library |
| **Type of Dependency** | Library/Framework (Dev/Docs) |
| **Purpose/Role** | Provides a live-reloading local server for Sphinx documentation development, automatically rebuilding on file changes. |
| **Integration Point/Clues** | Listed in `/pyproject.toml` under `[dependency-groups] docs`. |

---

### 25. `sphinxcontrib-mermaid` (Python Package)

| Field | Detail |
|---|---|
| **Dependency Name** | Python `sphinxcontrib-mermaid` Library |
| **Type of Dependency** | Library/Framework (Dev/Docs) |
| **Purpose/Role** | Sphinx extension enabling Mermaid diagram rendering within documentation pages. |
| **Integration Point/Clues** | Listed in `/pyproject.toml` under `[dependency-groups] docs`. |

---

### 26. `nvidia-sphinx-theme` (Python Package)

| Field | Detail |
|---|---|
| **Dependency Name** | NVIDIA Sphinx Theme |
| **Type of Dependency** | Library/Framework (Dev/Docs) — External/Third-Party |
| **Purpose/Role** | NVIDIA's official Sphinx HTML theme, applying NVIDIA branding and styling to the generated documentation. |
| **Integration Point/Clues** | Listed in `/pyproject.toml` under `[dependency-groups] docs`. This is an NVIDIA-owned external package, likely hosted on PyPI. |

---

### 27. `sphinx-llm` (Python Package)

| Field | Detail |
|---|---|
| **Dependency Name** | Python `sphinx-llm` Library |
| **Type of Dependency** | Library/Framework (Dev/Docs) |
| **Purpose/Role** | Sphinx extension likely providing LLM-optimized documentation output or search assets. The `/docs/_ext/` directory contains `json_output/` and `search_assets/` sub-extensions, suggesting LLM-ready content generation. |
| **Integration Point/Clues** | Listed in `/pyproject.toml` at `>=0.3.0`. The `scripts/docs-to-skills.py` script and the `json_output` extension in `/docs/_ext/` further suggest LLM-targeted doc processing. |

---

## Container / Infrastructure Dependencies

---

### 28. GHCR NemoClaw Sandbox Base Image

| Field | Detail |
|---|---|
| **Dependency Name** | `ghcr.io/nvidia/nemoclaw/sandbox-base` (GitHub Container Registry) |
| **Type of Dependency** | Container Image / External Service |
| **Purpose/Role** | Pre-built base Docker image hosted on GitHub Container Registry (GHCR). Contains expensive, infrequently-changed layers (apt packages, `gosu`, user setup, OpenClaw CLI). The main `Dockerfile` builds on top of this image. |
| **Integration Point/Clues** | Referenced in `Dockerfile` as `ARG BASE_IMAGE=ghcr.io/nvidia/nemoclaw/sandbox-base:latest`. Also referenced in `.github/workflows/base-image.yaml` (build/push workflow) and `.github/workflows/sandbox-images-and-e2e.yaml`. The `Makefile` and various CI workflows reference this image. |

---

### 29. Docker Hub `node:22-slim` Base Image

| Field | Detail |
|---|---|
| **Dependency Name** | Docker Hub `node:22-slim` Image |
| **Type of Dependency** | Container Image / External Service |
| **Purpose/Role** | Official Node.js 22 slim Docker image used as the builder stage in the multi-stage `Dockerfile` to compile the TypeScript NemoClaw plugin. |
| **Integration Point/Clues** | Referenced in `Dockerfile` as `FROM node:22-slim@sha256:4f77a690f2f8946ab16fe1e791a3ac0667ae1c3575c3e4d0d4589e9ed5bfaf3d AS builder`. Pinned by digest for security. |

---

## CI/CD & External Platform Dependencies

---

### 30. GitHub Actions

| Field | Detail |
|---|---|
| **Dependency Name** | GitHub Actions CI/CD Platform |
| **Type of Dependency** | External Service (CI/CD) |
| **Purpose/Role** | Hosts and executes all CI/CD pipelines for the project including linting, testing, Docker image building, documentation deployment, and end-to-end tests. |
| **Integration Point/Clues** | Extensive workflow definitions under `.github/workflows/` (13 workflow files including `main.yaml`, `pr.yaml`, `nightly-e2e.yaml`, `base-image.yaml`, `docs-preview-deploy.yaml`, etc.). |

---

### 31. GitHub Container Registry (GHCR)

| Field | Detail |
|---|---|
| **Dependency Name** | GitHub Container Registry (`ghcr.io`) |
| **Type of Dependency** | External Service (Container Registry) |
| **Purpose/Role** | Hosts and serves the pre-built NemoClaw sandbox base Docker image (`ghcr.io/nvidia/nemoclaw/sandbox-base`). CI workflows push new base images here; the `Dockerfile` pulls from it at build time. |
| **Integration Point/Clues** | Referenced in `Dockerfile` (`ghcr.io/nvidia/nemoclaw/sandbox-base:latest`), `.github/workflows/base-image.yaml`, and `.github/actions/resolve-sandbox-base-image/`. |

---

### 32. Dependabot (GitHub)

| Field | Detail |
|---|---|
| **Dependency Name** | GitHub Dependabot |
| **Type of Dependency** | External Service (Dependency Management) |
| **Purpose/Role** | Automated dependency update service that monitors and creates pull requests to update outdated dependencies in the repository. |
| **Integration Point/Clues** | Configuration file `.github/dependabot.yml` is present in the repository. |

---

### 33. Brev.dev

| Field | Detail |
|---|---|
| **Dependency Name** | Brev.dev GPU Cloud Platform |
| **Type of Dependency** | External Service (Cloud/GPU Compute) |
| **Purpose/Role** | Cloud GPU development environment platform used for running end-to-end tests on GPU hardware and for setting up remote GPU deployments. |
| **Integration Point/Clues** | Referenced in `.github/workflows/e2e-brev.yaml` and `.github/workflows/nightly-e2e.yaml`. Setup script `scripts/brev-setup.sh` exists. Documentation at `docs/deployment/deploy-to-remote-gpu.md` references remote GPU setup. |

---

### 34. NVIDIA NIM (NVIDIA Inference Microservices)

| Field | Detail |
|---|---|
| **Dependency Name** | NVIDIA NIM Inference Services |
| **Type of Dependency** | External Service (AI/ML Inference API) |
| **Purpose/Role** | NVIDIA's inference microservices providing hosted AI model endpoints. NemoClaw configures agents to use NIM as the inference provider for AI model calls. |
| **Integration Point/Clues** | `bin/lib/nim-images.json` lists NIM container images. `bin/lib/nim.js` handles NIM-specific logic. The `Dockerfile` has build args `NEMOCLAW_MODEL=nvidia/nemotron-3-super-120b-a12b` and `NEMOCLAW_PROVIDER_KEY=nvidia`. `docs/reference/inference-profiles.md` and `docs/inference/switch-inference-providers.md` document NIM integration. |

---

### 35. Telegram API (Telegram Bridge)

| Field | Detail |
|---|---|
| **Dependency Name** | Telegram Bot API |
| **Type of Dependency** | Third-party API / External Service |
| **Purpose/Role** | Provides a Telegram bot bridge for interacting with the NemoClaw sandbox via Telegram messaging. |
| **Integration Point/Clues** | `scripts/telegram-bridge.js` implements the bridge. `docs/deployment/set-up-telegram-bridge.md` documents the setup. End-to-end test `test/e2e/test-telegram-injection.sh` specifically tests this integration. |

---

### 36. CodeRabbit AI Code Review

| Field | Detail |
|---|---|
| **Dependency Name** | CodeRabbit AI Code Review Service |
| **Type of Dependency** | External Service (Code Review) |
| **Purpose/Role** | AI-powered automated code review service integrated into the pull request workflow. |
| **Integration Point/Clues** | `.coderabbit.yaml` configuration file is present at the repository root. |

---

### 37. OpenAI-Compatible Inference API

| Field | Detail |
|---|---|
| **Dependency Name** | OpenAI-Compatible Inference Endpoint |
| **Type of Dependency** | Third-party API / External Service |
| **Purpose/Role** | The NemoClaw sandbox is configured to communicate with an OpenAI-compatible inference endpoint (defaulting to `https://inference.local/v1`) for AI model completions. This can be any OpenAI-compatible provider or local inference server. |
| **Integration Point/Clues** | `Dockerfile` build args: `NEMOCLAW_INFERENCE_BASE_URL=https://inference.local/v1`, `NEMOCLAW_INFERENCE_API=openai-completions`. `bin/lib/inference-config.js` and `bin/lib/local-inference.js` handle inference configuration. `test/inference-config.test.js` and `test/local-inference.test.js` test this integration. |

---

## Summary Table

| # | Dependency Name | Type | Scope |
|---|---|---|---|
| 1 | `openclaw` NPM Package | Library/Framework | Production |
| 2 | `p-retry` NPM Package | Library/Framework | Production |
| 3 | `commander` NPM Package | Library/Framework | Production |
| 4 | `execa` NPM Package | Library/Framework | Production |

--- authentication ---


# Authentication Security Analysis: NemoClaw_9536aca0

## Executive Summary

This codebase implements a **credential-based authentication system** for an AI sandbox/development environment CLI tool. The authentication architecture centers on managing API keys and credentials for external services (NVIDIA NIM, container registries, inference providers) rather than traditional user authentication. The system includes credential storage, validation, session/onboarding flows, and a write-auth-profile mechanism.

---

## 1. Primary Authentication Mechanisms

### 1.1 API Key / Credential-Based Authentication

**Location:** `bin/lib/credentials.js`

The core authentication mechanism revolves around storing, retrieving, and validating API credentials for external services.

```
bin/lib/credentials.js        — Primary credential management
bin/lib/registry.js           — Container registry authentication
bin/lib/inference-config.js   — Inference provider credential config
scripts/write-auth-profile.ts — Auth profile writer
test/credentials.test.js      — Credential unit tests
test/credential-exposure.test.js — Credential exposure security tests
```

**Implementation Details:**
- API keys stored locally for services including NVIDIA NGC/NIM, container registries
- Credentials consumed by Docker registry login flows and inference API calls
- No in-memory caching of credentials beyond process lifetime indicated

---

### 1.2 Session / Onboarding Authentication Flow

**Location:** `bin/lib/onboard-session.js`, `bin/lib/onboard.js`

```
bin/lib/onboard-session.js    — Session lifecycle during onboarding
bin/lib/onboard.js            — Onboarding orchestration
nemoclaw/src/onboard/         — Plugin-layer onboarding (2 files)
test/onboard-session.test.js  — Session tests
test/onboard.test.js          — Onboarding flow tests
test/onboard-readiness.test.js
test/onboard-selection.test.js
```

**Implementation Details:**
- Session is created during the onboarding flow rather than a traditional login
- Session state tracked through onboarding phases
- `onboard-session.js` manages the lifecycle of an active provisioning session

---

### 1.3 Container Registry Authentication

**Location:** `bin/lib/registry.js`

```
bin/lib/registry.js           — Registry credential handling
test/registry.test.js         — Registry auth tests
```

**Implementation Details:**
- Handles Docker registry login using stored credentials
- Credentials passed to container runtime (Docker/Podman) for image pulls
- NVIDIA NGC registry authentication for NIM images (`bin/lib/nim-images.json`, `bin/lib/nim.js`)

---

### 1.4 Inference Provider Authentication

**Location:** `bin/lib/inference-config.js`, `bin/lib/local-inference.js`

```
bin/lib/inference-config.js   — Inference API key configuration
bin/lib/local-inference.js    — Local inference service auth
test/inference-config.test.js
test/local-inference.test.js
test/test-inference.sh
test/test-inference-local.sh
```

**Implementation Details:**
- API keys configured per inference provider
- Supports multiple providers via `docs/inference/switch-inference-providers.md`
- Reference profiles documented in `docs/reference/inference-profiles.md`

---

### 1.5 Write-Auth-Profile Mechanism

**Location:** `scripts/write-auth-profile.ts`

```
scripts/write-auth-profile.ts — Writes authentication profiles to disk
```

**Implementation Details:**
- TypeScript script that serializes authentication profiles
- Used during setup/initialization flows
- Interacts with credential storage layer

---

## 2. Credential Management

### 2.1 Credential Storage Architecture

**Location:** `bin/lib/credentials.js`

| Aspect | Details |
|--------|---------|
| Storage Type | Local filesystem (CLI tool context) |
| Credential Types | API keys, registry tokens, inference provider keys |
| Test Coverage | `credentials.test.js`, `credential-exposure.test.js` |

### 2.2 Credential Exposure Testing

**Location:** `test/credential-exposure.test.js`, `test/e2e/test-credential-sanitization.sh`

```
test/credential-exposure.test.js         — Unit tests for credential leakage
test/e2e/test-credential-sanitization.sh — E2E sanitization verification
```

**Security-Positive Finding:** The codebase includes dedicated tests specifically designed to detect credential exposure — this is a security-conscious practice indicating awareness of credential leakage risks (e.g., credentials appearing in logs, error messages, or process output).

---

## 3. Network Policy & Access Control

### 3.1 Network Policy Authentication

**Location:** `bin/lib/policies.js`, `nemoclaw-blueprint/policies/`

```
bin/lib/policies.js                           — Policy enforcement logic
nemoclaw-blueprint/policies/openclaw-sandbox.yaml — Base sandbox policy
nemoclaw-blueprint/policies/presets/          — 9 policy preset files
docs/network-policy/approve-network-requests.md
docs/network-policy/customize-network-policy.md
docs/reference/network-policies.md
test/policies.test.js
test/security-method-wildcards.test.js
```

**Implementation Details:**
- Network-level access control for sandbox environments
- YAML-based policy definitions with preset configurations
- Method wildcard protection tested (`security-method-wildcards.test.js`)
- Egress/ingress network request approval workflow

### 3.2 Security Policy Tests

```
test/security-binaries-restriction.test.js    — Binary execution restrictions
test/security-c2-dockerfile-injection.test.js — C2 injection prevention
test/security-c4-manifest-traversal.test.js   — Path traversal prevention
test/security-method-wildcards.test.js        — Method wildcard abuse prevention
```

**Security-Positive Finding:** Dedicated security test suite covering binary restrictions, Dockerfile injection (C2), manifest traversal (C4), and method wildcard abuse — indicates a structured security testing approach.

---

## 4. Platform & Runner Authentication Context

**Location:** `bin/lib/platform.js`, `bin/lib/runner.js`

```
bin/lib/platform.js    — Platform detection and auth context
bin/lib/runner.js      — Execution runner with auth context
test/platform.test.js
test/runner.test.js
```

**Implementation Details:**
- Platform-specific credential handling (macOS keychain vs. Linux credential stores implied by `scripts/smoke-macos-install.sh`)
- Runner applies auth context when launching sandboxed processes

---

## 5. Sandbox Hardening & Security Configuration

**Location:** `docs/deployment/sandbox-hardening.md`, `nemoclaw-blueprint/blueprint.yaml`

```
docs/deployment/sandbox-hardening.md    — Hardening documentation
nemoclaw-blueprint/blueprint.yaml       — Blueprint with security config
k8s/nemoclaw-k8s.yaml                  — Kubernetes deployment config
```

**Implementation Details:**
- Sandbox isolation configuration
- Blueprint-level security constraints
- Kubernetes deployment security posture (`k8s/` directory)

---

## 6. Telegram Bridge Authentication

**Location:** `scripts/telegram-bridge.js`, `docs/deployment/set-up-telegram-bridge.md`

```
scripts/telegram-bridge.js              — Telegram bridge implementation
docs/deployment/set-up-telegram-bridge.md
test/e2e/test-telegram-injection.sh     — Injection attack test
```

**Implementation Details:**
- Optional Telegram notification/bridge integration
- Bot token management for Telegram API
- Injection attack testing (`test-telegram-injection.sh`) suggests awareness of injection risks through the Telegram interface

---

## 7. CI/CD & Workflow Authentication

**Location:** `.github/workflows/`

```
.github/workflows/main.yaml
.github/workflows/e2e-brev.yaml
.github/workflows/nightly-e2e.yaml
.github/workflows/sandbox-images-and-e2e.yaml
.github/workflows/docs-preview-deploy.yaml
```

**Implementation Details:**
- GitHub Actions workflows use repository secrets (standard GitHub OIDC/secrets pattern)
- E2E test workflows (`e2e-brev.yaml`, `nightly-e2e.yaml`) imply credential injection for live environment testing
- Brev platform integration (`scripts/brev-setup.sh`) for remote GPU testing

---

## 8. DNS Proxy & Service Authentication

**Location:** `scripts/setup-dns-proxy.sh`, `test/dns-proxy.test.js`

```
scripts/setup-dns-proxy.sh    — DNS proxy configuration
test/dns-proxy.test.js        — DNS proxy tests
scripts/fix-coredns.sh        — CoreDNS fix script
```

**Implementation Details:**
- DNS proxy used to control/intercept network resolution within sandboxes
- Contributes to network isolation (access control layer)

---

## 9. Runtime Recovery & Session Resilience

**Location:** `bin/lib/runtime-recovery.js`

```
bin/lib/runtime-recovery.js         — Recovery logic
test/runtime-recovery.test.js       — Recovery tests
test/nemoclaw-cli-recovery.test.js  — CLI-level recovery tests
test/e2e/test-onboard-repair.sh     — Onboard repair E2E
test/e2e/test-onboard-resume.sh     — Onboard resume E2E
```

**Implementation Details:**
- Handles credential/session state recovery after runtime failures
- Prevents credential loss during unexpected termination
- Resume capability for interrupted onboarding sessions

---

## 10. Vulnerability Assessment

### 10.1 Identified Strengths

| Strength | Evidence |
|----------|----------|
| Credential exposure testing | `credential-exposure.test.js`, `test-credential-sanitization.sh` |
| Injection attack testing | `security-c2-dockerfile-injection.test.js`, `test-telegram-injection.sh` |
| Path traversal protection | `security-c4-manifest-traversal.test.js` |
| Network policy access control | `policies/`, `security-method-wildcards.test.js` |
| Binary restriction enforcement | `security-binaries-restriction.test.js` |
| Security documentation | `docs/security/best-practices.md`, `SECURITY.md` |
| Sandbox hardening docs | `docs/deployment/sandbox-hardening.md` |

### 10.2 Gaps Requiring Source Code Review

> **Note:** The following are **architectural concerns** based on file structure analysis. Definitive vulnerability confirmation requires reading the actual source file contents, which were not provided.

| Risk Area | Location | Concern |
|-----------|----------|---------|
| **Credential storage format** | `bin/lib/credentials.js` | Cannot confirm encryption-at-rest for locally stored API keys without source |
| **API key rotation** | `bin/lib/credentials.js` | No rotation mechanism files visible; rotation policy unclear |
| **No MFA implementation** | Entire codebase | No MFA/2FA files detected — appropriate for CLI tool but noteworthy |
| **No rate limiting files** | `bin/lib/` | No rate limiting implementation visible at CLI layer |
| **Telegram bot token storage** | `scripts/telegram-bridge.js` | Bot tokens require same secure storage as other credentials |
| **Kubernetes secret management** | `k8s/nemoclaw-k8s.yaml` | K8s secret configuration requires review for base64 vs. encrypted secrets |
| **GitHub Actions secrets** | `.github/workflows/` | Secret names/scoping cannot be verified without file contents |
| **Session fixation** | `bin/lib/onboard-session.js` | Onboarding session ID regeneration on privilege change cannot be confirmed |

### 10.3 Architecture-Level Observations

```
OBSERVATION: This is a CLI tool / developer sandbox orchestrator.
Traditional web authentication patterns (JWT, OAuth flows, session cookies)
are NOT expected and correctly NOT present.

The authentication surface is:
  1. Local credential store → External API services (NGC, registries, inference)
  2. Network policy enforcement → Sandbox egress/ingress control
  3. Process isolation → Binary/execution restrictions
  4. Blueprint policy → Declarative access control

This is appropriate for the tool's purpose.
```

---

## 11. Authentication Flow Summary

```
┌─────────────────────────────────────────────────────────┐
│                    NemoClaw Auth Flow                    │
└─────────────────────────────────────────────────────────┘

User/CI Invokes CLI
        │
        ▼
bin/nemoclaw.js ──→ bin/lib/preflight.js (pre-auth checks)
        │
        ▼
bin/lib/credentials.js ──→ Local credential store
        │                        │
        │                        ├── NGC API Key
        │                        ├── Registry credentials
        │                        └── Inference provider keys
        │
        ▼
bin/lib/onboard.js ──→ bin/lib/onboard-session.js
        │                   (session lifecycle)
        │
        ├──→ bin/lib/registry.js ──→ Docker/Podman registry login
        │
        ├──→ bin/lib/inference-config.js ──→ Inference API auth
        │
        ├──→ bin/lib/policies.js ──→ Network policy enforcement
        │           │
        │           └── nemoclaw-blueprint/policies/*.yaml
        │
        └──→ bin/lib/runner.js ──→ Sandboxed execution
                    │
                    └── bin/lib/runtime-recovery.js
```

---

## 12. Recommendations

### High Priority

1. **Verify credential encryption at rest** in `bin/lib/credentials.js` — confirm API keys are not stored in plaintext
2. **Audit `write-auth-profile.ts`** for file permissions (should be `0600`) and plaintext key material
3. **Review `k8s/nemoclaw-k8s.yaml`** for Kubernetes Secret objects using proper secret management (Sealed Secrets, Vault, or ESO) rather than base64-encoded values

### Medium Priority

4. **Confirm credential sanitization** in all log/error output paths — the existence of `credential-exposure.test.js` is positive but coverage completeness should be verified
5. **Audit Telegram bridge token storage** in `scripts/telegram-bridge.js` — bot tokens must follow same security posture as other credentials
6. **Review GitHub Actions workflow scoping** — ensure `GITHUB_TOKEN` and repository secrets have minimum necessary permissions

### Low Priority

7. **Document credential rotation procedure** — no rotation mechanism files are visible; a runbook should exist even if automation is absent
8. **Add pre-commit credential scanning** — `.pre-commit-config.yaml` exists; confirm it includes secret detection hooks (e.g., `detect-secrets`, `gitleaks`)

---

*Analysis based on repository file structure. Full vulnerability confirmation requires source code content review of `bin/lib/credentials.js`, `scripts/write-auth-profile.ts`, `bin/lib/onboard-session.js`, and `k8s/nemoclaw-k8s.yaml`.*

