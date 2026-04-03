# Multi-Channel

All 13 frameworks support multiple messaging channels — this is table-stakes for the domain. This page compares channel breadth, adapter architectures, and interesting channel-specific innovations.

Related: [Architecture Patterns](Architecture-Patterns.md), [Unique Innovations](Unique-Innovations.md).

---

## Channel Support Matrix

| Channel | CoPaw | Hermes | IronClaw | Letta | MicroClaw | Moltis | NanoClaw | Nanobot | NemoClaw | OpenFang | PicoClaw | ZeptoClaw |
|---------|-------|--------|----------|-------|-----------|--------|----------|---------|----------|----------|----------|-----------|
| **Telegram** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (docs) | ✓ | ✓ | ✓ |
| **Discord** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | ✓ | ✓ | ✓ |
| **Slack** | ✓ | ✓ | ✓ | — | ✓ | ✓ | ✓ | ✓ | — | ✓ | ✓ | ✓ |
| **WhatsApp** | ✓ | ✓ | — | — | ✓ | ✓ | ✓ | — | — | ✓ | — | ✓ |
| **Signal** | — | ✓ | — | — | — | — | — | — | — | — | — | — |
| **Matrix** | — | ✓ | — | — | ✓ | — | — | — | — | — | ✓ | — |
| **DingTalk** | ✓ | — | — | — | ✓ | — | — | — | — | — | ✓ | — |
| **Feishu/Lark** | ✓ | — | — | — | ✓ | — | — | — | — | — | ✓ | ✓ |
| **WeChat** | ✓ | — | — | — | ✓ | — | — | — | — | — | ✓ | — |
| **QQ** | ✓ | — | — | — | ✓ | — | — | — | — | — | — | — |
| **Email** | ✓ | — | — | — | ✓ | — | ✓ (Gmail) | — | — | — | — | ✓ |
| **IRC** | — | — | — | — | ✓ | — | — | — | — | — | — | — |
| **Nostr** | — | — | — | — | ✓ | — | — | — | — | — | — | — |
| **iMessage** | — | — | — | — | ✓ | — | — | — | — | — | — | — |
| **MS Teams** | — | — | — | — | — | ✓ | — | — | — | — | — | — |
| **MQTT** | — | — | — | — | — | — | — | — | — | — | — | ✓ |
| **Serial/USB** | — | — | — | — | — | — | — | — | — | — | — | ✓ |
| **Webhook** | ✓ | ✓ | ✓ | ✓ | — | — | — | ✓ | — | — | ✓ | ✓ |
| **Web/API** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | ✓ | — | ✓ | ✓ | ✓ |
| **X/Twitter** | — | — | — | — | — | — | ✓ | — | — | — | — | — |

**MicroClaw** has the broadest channel coverage, including niche channels (Nostr, IRC, iMessage). **ZeptoClaw** covers the most IoT channels (MQTT, Serial, USB). **CoPaw** has the deepest China ecosystem integration (WeChat, QQ, DingTalk, Feishu).

---

## Channel Adapter Architectures

### Registry Pattern
Channels register themselves with a central registry at startup. The gateway routes incoming messages to the appropriate registered handler.

| Agent | Registry Location | Pattern |
|-------|------------------|---------|
| MicroClaw | `src/channels/mod.rs` | Rust trait + registry |
| PicoClaw | `pkg/channels/` | Go interface + registry |
| ZeptoClaw | `src/channels/` | Rust feature-gated modules |
| OpenFang | `crates/openfang-channels/` | 48-file crate |
| Hermes Agent | `gateway/platforms/` | Python gateway adapters |

### Event-Driven Message Queue
Incoming channel messages are enqueued and processed asynchronously. NanoClaw uses a per-group queue (`src/group-queue.ts`) so messages for different groups don't block each other.

### Gateway Multiplexer (Hermes Agent)
Hermes Agent's `gateway/` is a dedicated service that:
1. Maintains persistent connections to all configured platforms
2. Normalizes platform messages to a canonical internal format
3. Routes to the appropriate agent instance
4. Handles back-pressure and retries per platform

Built-in gateway hooks (`gateway/builtin_hooks/`) allow lifecycle interception at the gateway level (before routing, after response, on error).

---

## Channel Adapter Architecture Deep-Dives

### NanoClaw: Container-per-Task with Group Routing
NanoClaw's channel architecture is unique: messages from any channel are routed to a **group**, and each group maintains a queue of tasks. Tasks run in fresh containers. The group's `CLAUDE.md` file configures the agent's behavior for that group.

```
Telegram/Slack/Discord/etc.
        ↓
    Channel Adapter
        ↓
    Router (src/router.ts)
        ↓
    Group Queue (per group)
        ↓
    Container Runner (fresh Docker/Apple Container)
        ↓
    Claude AI inside container
        ↓
    IPC auth → Result back to channel
```

The sender allowlist prevents untrusted senders from triggering containers. The mount allowlist controls what directories the container can see.

### Moltis: Dedicated Crates per Platform
Moltis has separate Rust crates for major platforms:
- `moltis-telegram` — Telegram Bot API
- `moltis-discord` — Discord gateway
- `moltis-slack` — Slack Events API
- `moltis-whatsapp` — WhatsApp Cloud API
- `moltis-msteams` — Microsoft Teams

Each crate owns its platform-specific logic, authentication, and message types. The `moltis-gateway` crate ties them together. This separation enables independent testing and deployment.

### ZeptoClaw: IoT-First Channels
ZeptoClaw extends the channel concept to IoT protocols:
- **MQTT** (`src/channels/mqtt.rs`) — subscribe to MQTT topics; agent responds via MQTT publish
- **Serial** (`src/channels/serial.rs`) — serial port as a bidirectional channel
- **ACP** (`src/channels/acp.rs`) — Agent Communication Protocol

This makes ZeptoClaw the only framework where a hardware sensor (e.g., temperature sensor publishing to MQTT) can directly trigger agent actions.

### CoPaw: China Ecosystem
CoPaw's channel adapters reflect the Chinese market:
- WeChat (企业微信 / WeCom)
- QQ
- DingTalk (钉钉)
- Feishu/Lark (飞书)

The React frontend uses `react-i18next` with Chinese (ZH), Japanese (JA), English (EN), and Russian (RU) translations. Font packages for Chinese characters are included. Base Docker image from Alibaba Cloud Registry.

---

## Webhook Architecture

Most frameworks support inbound webhooks as a generic channel — accepting HTTP POST requests from any source. This enables integration with:
- GitHub (code events)
- CI/CD systems
- Custom automations
- IoT sensors

**ZeptoClaw** (`src/channels/webhook.rs`) and **Hermes Agent** support incoming webhooks with configurable authentication (HMAC signatures, bearer tokens).

---

## Multi-Agent Protocols as Channels

Some frameworks treat inter-agent communication as just another channel:

| Protocol | Agents | Transport |
|----------|--------|-----------|
| **A2A** (Agent-to-Agent) | MicroClaw, ZeptoClaw | HTTP |
| **ACP** (Agent Communication Protocol) | MicroClaw, ZeptoClaw, PicoClaw | stdio |
| **MCP** (as agent transport) | Nanobot | HTTP/stdio |

**MicroClaw's Nostr channel** is architecturally interesting: Nostr is a decentralized, censorship-resistant social protocol where messages are cryptographically signed. Running an agent on Nostr means it can receive messages without a central server, making it uniquely resilient to service disruptions.

---

## Channel Security

| Agent | Channel Auth | Injection Prevention | Rate Limiting |
|-------|-------------|---------------------|---------------|
| NemoClaw | Policy-gated | Telegram injection tests | Via container |
| ZeptoClaw | Per-channel token | Input sanitization | Gateway rate limit |
| NanoClaw | Sender allowlist + IPC auth | — | Queue depth |
| IronClaw | Multi-tenant scope isolation | — | — |
| Hermes Agent | Platform SDK auth | Prompt injection tests | — |
| Others | Platform SDK auth | — | — |

**NemoClaw** explicitly tests for Telegram injection attacks (crafting malicious Telegram messages to inject commands into the agent). This is a real attack vector: if the agent processes Telegram messages as commands, a malicious user could craft a message that hijacks the agent.

---

*Back to [Home](Home.md) | See also: [Architecture Patterns](Architecture-Patterns.md) | [Security Models](Security-Models.md)*
