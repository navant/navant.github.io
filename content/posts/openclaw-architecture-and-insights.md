---
title: "OpenClaw Architecture and Insights"
date: 2025-11-24T11:30:03+00:00
draft: false
tags: ["OpenClaw", "AI", "LLMs", "Architecture"]
author: "Navan Tirupathi"
showToc: true
TocOpen: false
hidemeta: false
comments: false
description: "A deep dive into OpenClaw (Claudbot) architecture: Gateway, channel adapters, inputs, agent runtime, context vs memory, plugins, and security—with deployment patterns for solo and team use."
canonicalURL: "https://navant.github.io/posts/openclaw-architecture-and-insights/"
disableHLJS: false
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>"
    alt: "<alt text>"
    caption: "<text>"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/navant/navant.github.io/tree/main/content/posts"
    Text: "Suggest Changes"
    appendFilePath: true
---

OpenClaw (a.k.a Claudbot) is an open-source personal AI assistant (MIT licensed) created by Peter Steinberger that has quickly gained traction with over 180,000 stars on GitHub at the time of writing this blogs. Lots of traction, internet buzz and usecases that evolve from here. I am going to cover something else, what makes this different and really powerful. Its the Architecture and how it was productised .

Lets dive into different aspects

1. The Gateway
2. Channel Adapters
3. Inputs: The Secret to "Aliveness"
4. Agent Runtime & The Loop
5. Context vs. Memory
6. The Brain: LLMs & Prompts
7. Access, Tools, Skills, and Canvas (A2UI)
8. Sessions & Security Boundaries
9. Multi-Agent Routing
10. Dashboard & Monitoring
11. Deployment Architectures
12. Registry & Plugins
13. Security as a Foundation to all above

![OpenClaw Architecture](/openclaw-architecture.png)

## **1. The Gateway**

The heart of OpenClaw is the **Gateway**, a long-running WebSocket server (typically on `localhost:18789`) that sits on your machine.

{{< collapse summary="Details" >}}
- **Role:** It does not think, reason, or decide. Its sole job is to accept inputs, manage connections, and route traffic.
- **Hub-and-Spoke Design:** It separates the interface layer (WhatsApp, Slack, CLI) from the intelligence layer. This ensures that if a channel fails, the agent remains alive.
- **Typed Protocol:** It enforces a strictly typed protocol. Every WebSocket frame is validated against a JSON schema to ensure stability before it ever reaches an agent.
- **Device Identity:** It handles security handshakes. Clients (like the CLI or Mobile Nodes) must cryptographically sign a challenge to prove their identity before connecting.
{{< /collapse >}}

## **2. Channel Adapters**

The Gateway does not natively understand specific platforms like WhatsApp or Discord. It relies on **Adapters** to normalise the messy reality of external APIs. Adapters perform four critical functions - Authentication, Normalisation,Access Control and Formatting

{{< collapse summary="Details" >}}
- **Authentication:** They handle specific login methods, such as QR pairing for WhatsApp (via Baileys), Bot Tokens for Discord, or native macOS integration for iMessage.
- **Normalisation:** They parse incoming data—extracting text, handling media attachments, and processing emoji reactions—into a standardised format the Gateway understands.
- **Access Control:** Security happens here first. Adapters check **Allowlists** (e.g., is this phone number allowed?) and **Pairing Policies** before a message is even processed.
- **Formatting:** They convert the agent's generic markdown response into the specific dialect required by the platform (e.g., converting bolding syntax for Slack vs. WhatsApp).
{{< /collapse >}}

## **3. Inputs: The Secret to "Aliveness"**

Most AI bots are reactive (waiting for you to type). OpenClaw feels proactive because it treats **Time** and **State** as inputs, just like text messages. There are six primary inputs: Messages , Heartbeats, Cronjobs, Hooks and Webhooks , Agent to Agent messages.

{{< collapse summary="Details" >}}
1. **Messages:** Standard chats from humans via adapters like Whatsapp, Telegram etc
2. **Heartbeats:** A timer (default: 30 mins) that prompts the agent to "check for tasks." If no action is needed, the system suppresses the response. This allows the agent to self-initiate work.
3. **Cron Jobs:** Scheduled events with specific instructions (e.g., "9:00 AM: Check my email"). This enables routine maintenance or daily briefings.
4. **Hooks:** Internal triggers fired by state changes, such as the system booting up or an agent finishing a task.
5. **Webhooks:** Notifications from external systems (e.g., a GitHub PR or Jira ticket) that trigger the agent to act immediately.
6. **Agent-to-Agent:** Messages sent between agents to collaborate. A "researcher" agent can finish a task and queue a job for a "writer" agent.
{{< /collapse >}}

## **4. Agent Runtime & The Loop**

The **Agent Runtime** executes the intelligence loop using an **RPC Streaming** model.

- **The Queue:** If multiple inputs arrive while the agent is busy, they are queued and processed in order. The agent finishes one "thought" before starting the next.
- **The Loop:**
    1. **Resolve Session:** Determine the security context (Main vs. Group).
    2. **Assemble Context:** Load rules, personality, and relevant memories.
    3. **Stream & Intercept:** The model's response is streamed. If the model generates a tool call (e.g., `bash`), the runtime intercepts it, executes the code, and feeds the result back into the stream.
    4. **Persist:** The updated state is written to disk.

## **5. Context vs. Memory**

OpenClaw draws a hard line between **Context** (temporary, limited by token window) and **Memory** (persistent, stored on disk).

**Context** is what the model sees *right now*.

It includes:

- system instructions
- recent messages
- tool outputs

Context is:

- temporary
- limited by token windows
- expensive

**Memory** is what lives on disk.

It includes:

- daily notes
- long-term preferences
- past decisions
- prior session summaries

Memory is:

- persistent
- unbounded
- cheap to store
- searchable

Example

You tell OpenClaw:

> "We decided last week to use PostgreSQL instead of MySQL."

That sentence is **not guaranteed** to be in context tomorrow.

But if it was written to memory:

- it survives restarts
- it survives compaction
- it can be retrieved weeks later

Context forgets.

Memory endures.

{{< collapse summary="Additional Details" >}}
- **Markdown Storage:** Memory is stored in plain text files you can read and edit (`MEMORY.md` for facts, `2026-01-26.md` for daily logs).
- **Two-Layer System:**
  - *Daily Memory:* An append-only log of everything that happened today.
  - *Long-Term Memory:* Curated facts (e.g., "User prefers TypeScript").
- **Hybrid Search:** It retrieves memory using both **Semantic Search** (vector embeddings for meaning) and **Keyword Search** (exact matches for specific IDs or keys).
  - Example: Memory contains: "Chose PostgreSQL for production database." Later, you ask — semantic search finds it. "What is `POSTGRES_URL`?" — keyword search finds it.
  - Hybrid retrieval handles **both intent and precision** — something pure vector systems often miss.
- **Memory Flush & Compaction:** To manage context limits, the system "compacts" (summarizes) old logs. Crucially, it performs a **Memory Flush** first—extracting key facts to persistent storage—before the raw logs are compressed, ensuring data isn't lost.
- **Debounced Indexing: Why Memory Doesn't Thrash**
  - Memory files change frequently: multiple writes per minute, tool edits, manual edits.
  - Indexing after *every keystroke* would be wasteful. OpenClaw waits briefly before indexing changes. If multiple edits happen quickly, they are **collapsed into one indexing pass**.
  - Example: Between 2:00–2:03 PM — three notes appended, one preference updated, a summary written. Instead of indexing five times, OpenClaw indexes **once**.
  - This enables: fast interaction, stable embeddings, low compute overhead.
{{< /collapse >}}

## **6. The Brain: LLMs & Prompts**

OpenClaw is model-agnostic (supporting Claude, OpenAI, etc.). It constructs the "Brain" via **Composite Prompting**:

- `AGENTS.md`: Core operational rules.
- `SOUL.md`: Personality and tone instructions.
- `TOOLS.md`: User-defined conventions for tool usage.
- **Skill Injection:** It does not dump every available skill into the context window. It discovers skills at runtime and injects *only* the relevant ones to save tokens and reduce hallucinations.

## **7.Access, Tools, Skills, and Canvas (A2UI)**

- **Access:** OpenClaw feels "alive" because it has **deep access to your system**. It can run shell commands, read/write files, execute scripts, and control your browser. While this enables powerful automation, it also poses significant risks (e.g., prompt injection or credential exposure) if not run in an isolated environment like a container.
- **Core Tools:** Agents have deep system access, including Bash terminal, File System, and Browser Automation (via Chrome DevTools Protocol).
- **Tools:** These are capabilities like email access, calendar access, or the ability to browse Twitter.
- **Skills:** There is a marketplace of over 31,000 skills available for agents. However, a security analysis has described this as a "security nightmare". Risks include malicious skills executing code or deleting files.Consider like Apps in the Appstore. Some Apps are trustworthy some are not
- **Canvas (A2UI):** This stands for **Agent-to-UI**. The agent can generate HTML with special attributes (e.g., `<button a2ui-action="approve">`). The client renders this as a real UI. When you click it, a tool call is sent back to the agent. This allows agents to build their own interactive dashboards on the fly.

## **8. Sessions & Security Boundaries**

A "Session" in OpenClaw is a **security boundary**, not just a chat log.

- **Main Session (agent:main):** The operator (you). Has full permissions to run tools directly on the host machine.
- **DM/Group Sessions:** Untrusted interactions. These are **Sandboxed** by default.
- **Docker Sandboxing:** Untrusted sessions are forced into ephemeral Docker containers. If a user in a group chat tricks the bot into running `rm -rf /`, it only destroys the temporary container, not your laptop.
- **Pairing:** You must explicitly "pair" devices and approve new DM contacts via a challenge-response system.

## **9. Multi-Agent Routing**

You can configure specific **Personas** for different channels:

- **Routing:** A Discord bot can be configured to use `Claude Sonnet` with a "Moderator" personality, while your personal Telegram DM uses `GPT-4` with an "Executive Assistant" personality.
- **Collaboration:** Agents use tools like `sessions_send` to delegate work. A research agent can finish a task and queue a job for a separate writing agent.

## **10. Dashboard & Monitoring**

The system includes a self-contained control plane:

- **Web UI:** Served directly from the Gateway (`localhost:18789`). It allows you to view chats, health status, and logs.
- **Clients:**

◦ *CLI:* For developers to manage the gateway (`openclaw gateway`).

◦ *macOS App:* A native menu-bar app that manages the lifecycle and supports Voice Wake.

◦ *Mobile Nodes:* iOS/Android apps that connect as "nodes," allowing the agent to access the phone's camera or location.

## **11. Deployment Architectures**

OpenClaw is designed to run on infrastructure you control:

- **Local:** Runs on `localhost` for development.
- **VPS (Remote):** Runs on a Linux server. Accessed securely via:

◦ **SSH Tunnel:** Forwarding the local port to the remote server (Recommended).

◦ **Tailscale:** Using "Serve" (Tailnet-only) or "Funnel" (Public internet) modes.

- **Cloud:** Deployed via Docker containers on services like [Fly.io](http://Fly.io) for 24/7 availability.

Example of two setups illustrated here

{{< collapse summary="Solo User Setup" >}}
**Goal**: Deeply personal, always-available assistant on your Mac (or Linux) with full host access, iMessage/voice wake, proactive heartbeats, and secure phone access.

### Architecture (Solo)

```
[ You + Phone + Devices ]
           │
           ▼
Multi-Channel Inputs (WhatsApp, iMessage, Telegram, CLI, Web)
           │
           ▼
Channel Adapter (normalize, auth, media)
           │
           ▼
Gateway Server (native, port 18789, runs as daemon)
      ┌─────────────┴─────────────┐
      ▼                           ▼
Lane Queue (serial per session)   Control Plane (Menu Bar + Dashboard)
           │
           ▼
Brain / Agent Runner (LLM selector, prompt builder, context guard)
           ↻
     Agentic Loop (think → act → observe)
           │
           ▼
Tool Execution (full host access – sandbox disabled)
├── Browser (semantic a11y)
├── Filesystem (full ~/)
├── Shell (allowlist optional)
├── Code REPL, APIs, etc.
           │
           ▼
Persistent Memory (~/clawd/ + MEMORY.md + hybrid search)
           │
Security & Guardrails (light – trusted main session)
           │
Response Path (streaming back to channels)
```

**Key decisions**:

- Native install (not Docker) for lowest latency and native macOS features.
- Sandbox disabled or permissive (you trust yourself).
- Tailscale for secure remote access.
- Full filesystem & proactive features enabled.

### Step-by-Step Setup (Solo)

1. **Install OpenClaw** — `curl -fsSL https://openclaw.ai/install.sh | bash`
2. **Run onboarding wizard** — `openclaw onboard --install-daemon` (sets up API keys, default LLM, LaunchAgent on macOS).
3. **Set up Tailscale** — Install on Mac and phone, same tailnet; expose gateway: `sudo tailscale serve https://openclaw:18789 --https=443`
4. **Configure trusted main session** — Edit `~/.openclaw/config.json`: sandbox disabled for main, heartbeatInterval, dailyBriefing.
5. **Enable channels** — iMessage, WhatsApp/Telegram bridges; test with a message.
6. **Start / verify** — `openclaw gateway status` and `openclaw dashboard`

**Adding Skills & Plugins (Solo)** — From ClawHub: chat "install skill research-paper" or `openclaw skills install research-paper`. From GitHub: `openclaw skills install https://github.com/user/my-skill`. Create your own: add `SKILL.md` in `~/.openclaw/skills/<name>/`, then `openclaw skills reload`. Plugins: `openclaw plugins install @openclaw/msteams` or drop into `~/.openclaw/plugins/`. Skills run with full host privileges — review before enabling.

**Security Tips (Solo)** — Use Tailscale only; `openclaw skills audit` before installing; keep credentials in a separate workspace; regular `openclaw update`.
{{< /collapse >}}

{{< collapse summary="Small Enterprise Setup (SMB)" >}}
**Goal**: 24/7 centralized team brain with strict isolation, shared company memory, and controlled access for 5–50 users.

### Architecture (SMB)

```
[ Team Members (Slack/Discord/Teams) ]
           │
           ▼
Public Channels + Webhooks → Gateway (Docker container)
           │
           ▼
Lane Queues (per user / per channel session)
           │
           ▼
Brain / Agent Runner (multi-persona routing)
           ↻
     Agentic Loop
           │
           ▼
Tool Execution Engine (strict Docker sandbox per turn)
├── Ephemeral containers (no host access)
├── Browser, Filesystem (rw in /workspace only)
├── Shell (very strict allowlist)
           │
           ▼
Persistent Shared Memory (mounted volume: MEMORY.md + company skills)
           │
Security Layer (allowlists, domain auth, budgets, audit)
           │
           ▼
Response Path (streaming + typing indicators)
           │
Control Plane (accessible only via SSH tunnel / Tailscale)
```

**Key decisions**:

- Dockerized on Linux VPS for 24/7 uptime + easy scaling.
- Strict sandbox for every non-admin session.
- Persistent volume for shared company brain.
- Admin-only skill/plugin approval.

### Step-by-Step Setup (SMB)

1. **Provision a VPS** (Ubuntu 22.04+, 2 vCPU / 4 GB RAM, 20+ GB SSD) – Hetzner, DigitalOcean, Fly.io, etc.
2. **Install Docker & prerequisites** — `sudo apt update && sudo apt install curl git docker.io docker-compose-plugin` and `sudo usermod -aG docker $USER`
3. **Create project & persistent volume** — `mkdir ~/openclaw-team && cd ~/openclaw-team`
4. **Deploy with official Docker setup** — [docs](https://docs.openclaw.ai/install/docker): `curl -fsSL https://docs.openclaw.ai/install/docker.sh | bash`
5. **Configure strict sandbox & multi-user settings** — Edit mounted config.json (sandbox mode non-main, security allowFrom, channels.requireMention).
6. **Connect team channels** (Slack/Discord bot tokens) via onboarding inside container.
7. **Secure control plane** — Dashboard/admin only via SSH tunnel or Tailscale; optional Tailscale Funnel for webhooks.
8. **Start & monitor** — `docker compose up -d` and `docker compose logs -f`

**Adding Skills & Plugins (SMB – governed)** — Admin-only: `openclaw skills install --admin research-paper`. Company skills: private Git repo, mount at /skills/company, config skills.sources. Team flow: request in DM → admin reviews → install → announce. Plugins: same admin-only process.

**Security & Governance (SMB)** — Audit every skill with `openclaw skills audit`; resource budgets per user/session; append-only audit logs; capability tokens; regular container restarts and volume backups.
{{< /collapse >}}

{{< collapse summary="Quick Comparison" >}}

| Feature | Solo (Iron Man) | SMB (Team Ops) |
| --- | --- | --- |
| Runtime | Native on Mac/Linux | Docker on VPS |
| Sandbox | Disabled/permissive | Strict (ephemeral containers) |
| Uptime | Machine-dependent | True 24/7 |
| Adding skills | Instant (just ask) | Admin-approved |
| Access | Tailscale + native apps | Slack/Discord + tunneled UI |
| Best for | Personal power user | Team / company brain |

{{< /collapse >}}

## 12. Registry & Plugins

OpenClaw is architected as an operating system. Just as an OS needs drivers for new hardware and applications for new tasks, OpenClaw uses a **Plugin System** to add new capabilities, messaging platforms, and ai model providers.

{{< collapse summary="Details" >}}
**The Plugin Architecture**

The system relies on a **discovery-based model** located in the `extensions/` directory. It does not require you to hardcode imports into the main server file.

- **The Loader:** The plugin loader (`src/plugins/loader.ts`) scans the workspace packages.
- **Identification:** It looks for a specific `openclaw.extensions` field inside the `package.json` of installed packages.
- **Validation & Hot-Loading:** It validates the plugin against declared JSON schemas and "hot-loads" them when the corresponding configuration is present. This means you can drop a plugin into the folder, configure it, and it works immediately.

**The Four Types of Plugins**

OpenClaw supports extensibility across four distinct layers of the architecture:

1. **Channel Plugins:** *Purpose:* To connect OpenClaw to messaging platforms not supported out of the box. *Examples:* Microsoft Teams, Matrix, Mattermost, Signal. They implement the standard "Adapter" interface (auth, parsing, formatting).
2. **Tool Plugins:** *Purpose:* To give the agent new capabilities beyond the standard Bash/Browser tools. *Examples:* Spotify controller, Jira integration, Home Assistant connector.
3. **Memory Plugins:** *Purpose:* To swap out the underlying storage engine. *Examples:* Vector Database (Pinecone, Weaviate) or Knowledge Graph.
4. **Provider Plugins:** *Purpose:* To change the "brain" of the agent. *Examples:* Self-hosted LLMs (Ollama) or new API providers.

**The Skill Registry & Marketplace**

While Plugins extend the *system*, **Skills** extend the *agent's knowledge*.

- **The Marketplace:** Over **31,000 skills** — like "apps" for your agent.
- **Structure:** A skill is defined by a `SKILL.md` file in `skills/<skill>/`. It acts as a playbook or SOP.
- **Link:** Plugins = capability (code). Skills = instruction (knowledge). Plugins give the employee new equipment; Skills give the handbook on how to use it.

**The Distinction: Code vs. Markdown**

- **Plugins (Code):** Live in `extensions/`, TypeScript/JavaScript, register tools via `api.registerTool`.
- **Skills (Prompts):** Live in `skills/` as `SKILL.md`, Markdown SOPs/playbooks.

**The Workflow:** (1) Plugin Load → registers e.g. `jira_create_ticket`. (2) Skill Discovery → finds "How to manage projects". (3) Injection → when you say "file a bug", relevant Skill is injected. (4) Execution → agent calls the tool from the Plugin.
{{< /collapse >}}

## 13 Security

Because OpenClaw gives an AI agent **shell access** to your machine, security cannot be an afterthought. It is the primary architectural constraint. OpenClaw implements a "Defense in Depth" strategy with five distinct layers of protection.

{{< collapse summary="Details" >}}
**Layer 1: Network Isolation (The "Localhost" Default)** — By default the Gateway binds to `127.0.0.1`. Remote access: **SSH Tunneling** (e.g. `ssh -N -L 18789:127.0.0.1:18789`) or **Tailscale** (Serve for Tailnet-only, Funnel for public with password).

**Layer 2: Device Identity & Cryptographic Handshakes** — Device Pairing for CLI, Mobile Nodes, Web UI. Challenge-response; new devices need admin approval and get a device token.

**Layer 3: Channel Access Control** — DM Pairing (unknown users get pairing code; block until `openclaw pairing approve`). Allowlists for specific numbers/usernames. Group policies e.g. `requireMention: true`.

**Layer 4: Docker Sandboxing** — Main session (you) runs on host. Untrusted sessions (dm/group) run in ephemeral Docker containers; e.g. `rm -rf /` only hits the container. Configurable granularity and strictness.

**Layer 5: Memory Safeguards** — Secret detection/redaction before writing to memory. Fact verification (claims vs verified facts) to limit memory poisoning.

**Risks and Mitigations**

| Risk | Description | Mitigation |
| --- | --- | --- |
| **Prompt Injection** | Attacker tricks LLM into ignoring rules | Context isolation & sandboxing; untrusted sessions in Docker |
| **Malicious Skills** | Vulnerable or malicious community skills (e.g. 26% vulnerable) | Tool sandboxing; vigilance for Main session skills |
| **Memory Poisoning** | False info stored in long-term memory | Claim classification; scoped access (group vs Main memory) |
| **Credential Exposure** | Agent pastes `.env` into chat | Output filtering; sandbox limits filesystem access |
| **Unauthorized Access** | Guessed password/port | Device identity: valid cryptographic key required |

{{< /collapse >}}

### References

- https://github.com/openclaw/openclaw
- [OpenClaw System Architecture Overview (ppaolo.substack.com)](https://ppaolo.substack.com/p/openclaw-system-architecture-overview)
- https://www.youtube.com/watch?v=CAbrRTu5xcw
- https://manthanguptaa.in/posts/clawdbot_memory/
