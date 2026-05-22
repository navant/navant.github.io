---
title: "Claude Code – Token Optimisation Cheat Sheet"
date: 2026-05-21T00:00:00+00:00
draft: false
tags: ["Claude", "AI", "Developer Tools"]
author: "Navan Tirupathi"
showToc: true
TocOpen: false
hidemeta: false
comments: false
description: "A practical checklist to reduce Claude Code token usage and cost. Part 1: built-in features with no installs. Part 2: external tools and repositories. Save 30–70% on costs."
canonicalURL: "https://navant.github.io/posts/claude-code-token-optimisation-cheat-sheet/"
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

A practical checklist to reduce Claude Code token usage and cost.

Part 1 covers what you can do with zero installs. Part 2 covers external tools and repositories. Save **30–70%** on costs.

---

Claude Code can dramatically speed up development, but costs grow quickly if context and token usage aren’t managed. Most token waste comes from long conversations, large logs, unnecessary files loaded into context, and Claude re-learning your codebase from scratch every session.

I’ve collated a checklist in two parts: first, everything you can do with Claude Code’s built-in features (no installs, no external repos), then external tools that layer on top.

## Part 1: Built-in Optimisation

Everything here ships with Claude Code or requires only a config file edit. No installs, no external dependencies.

### CLAUDE.md hygiene

Your `CLAUDE.md` file loads into every single message. Every line costs tokens on every turn.

| Action | Why |
|--------|-----|
| Keep `CLAUDE.md` under 150 lines | Every line multiplies across every message in every session. `wc -l CLAUDE.md` to check. |
| Move large workflows to `.claude/skills/` | Skills load on-demand when triggered. Rules in `.claude/rules/` and content in `CLAUDE.md` load every time. |
| Use `@file` references sparingly | Each `@`-referenced file adds its full content to context. Reference only what Claude needs for the current task. |
| Write concise instructions | “Fix lint errors before committing” beats a 20-line explanation of your lint setup. Claude already knows ESLint, Prettier, etc. |

Check out this trending [CLAUDE.md](https://github.com/multica-ai/andrej-karpathy-skills/blob/main/CLAUDE.md) from Andrej Karpathy’s skills repo — it’s just 64 lines.

### .claudeignore (one-time setup)

Create a `.claudeignore` file in your project root. Same syntax as `.gitignore`. Stops Claude from scanning or reading matched paths.

```
node_modules/
dist/
build/
coverage/
logs/
*.min.js
*.map
*.lock
__pycache__/
.next/
```

Without this, Claude may read thousands of generated files during exploration, burning tokens on content that provides no value.

### Model switching

Not every task needs Opus. Picking the right model per task is one of the easiest cost wins.

| Model | When to use | Command |
|-------|-------------|---------|
| **Haiku** | Formatting, renaming, file moves, simple grep-and-replace, generating boilerplate, quick lookups, mechanical refactors | `/model haiku` |
| **Sonnet** | Default daily driver. Code review, bug fixes, multi-file changes, test writing, architectural questions | `/model sonnet` |
| **Opus** | Complex debugging, large refactors spanning 10+ files, nuanced design decisions, security review, when Sonnet gives wrong answers twice | `/model opus` |

Rule: start with Sonnet. Drop to Haiku for mechanical work. Escalate to Opus only when Sonnet struggles. Most teams spend 60%+ on Opus when Sonnet handles 80% of tasks fine.

### Session management commands

These are built into Claude Code. Use them actively, not as a last resort.

| Command | When to use | Why |
|---------|-------------|-----|
| `/compact` | At ~70% context usage | Don’t wait for auto-compact at 95%. Tell Claude what to preserve first: *”Before we compact, note we decided X...”* |
| `/clear` | After completing a distinct task | Starts fresh context. Cheaper than carrying stale conversation forward. |
| `/cost` | After each major task | Check spending so you can adjust behaviour. |
| `/context` | When things feel slow or expensive | Shows what’s actually filling your context. Diagnose before optimising blindly. |
| `/resume <name>` and `/rename <name>` | Between work sessions | Resume a named session instead of re-explaining everything. |

### MCP server discipline

Each enabled MCP server adds 100–500 tokens of tool definitions to every message. If you have 10 servers enabled but only use 2, you’re paying for 8 on every turn.

**Know what you have before changing anything:**

| Command | What it shows |
|---------|---------------|
| `/mcp` | Lists all configured MCP servers and their current status (enabled/disabled) |
| `/mcp details <server>` | Shows tools exposed by that server (this is what costs tokens per message) |
| `/mcp disable <server>` | Disables server, removes its tool definitions from context |
| `/mcp enable <server>` | Re-enables when needed |

Workflow: run `/mcp` first to see the full list. Check which servers you actually used this week. Disable the rest. Re-enable when needed.

### Prompt craft

The single biggest lever you control. A vague prompt causes Claude to explore broadly (reading many files, producing long responses). A precise prompt gets a precise answer.

| Habit | Example | Token impact |
|-------|---------|--------------|
| Precise scope | `”Fix the race condition in src/auth.ts lines 42-60”` | Prevents Claude from reading unrelated files |
| Done criteria | `”Done when: tests pass, no lint errors, no new warnings”` | Stops Claude from continuing past the goal |
| Plan before implement | Press `Shift+Tab` or ask “outline your plan first” | Catches wrong approaches before Claude reads 20 files |
| One task per session | Don’t chain unrelated tasks | Each topic adds context that persists and compounds |
| Reference specific files | `”Update the handler in api/users.ts, not the route”` | Eliminates exploration time |

### Subagents for exploration

When Claude needs to explore many files (searching for usages, understanding a module), use the Task tool or ask Claude to use subagents. The subagent reads 20 files but returns only a summary to your main session. Your main context stays clean.

You don’t need to install anything for this. Ask: *”Use a subagent to find all callers of processPayment, then summarize what you found.”*

### Environment variables

Set these in `~/.claude/settings.json` or as shell env vars:

| Variable | Value | Why |
|----------|-------|-----|
| `MAX_THINKING_TOKENS` | `10000` | Caps extended thinking burn on simple tasks. Default can be much higher. |
| `CLAUDE_CODE_SUBAGENT_MODEL` | `haiku` | Uses cheaper Haiku for subagent work that doesn’t need full reasoning. |

### Hooks (config only, no external tools)

Hooks are shell commands in `.claude/settings.json` that run on events. No plugins required.

| Hook | Example | Why |
|------|---------|-----|
| Session start | `git branch --show-current` | Injects current branch so Claude doesn’t need to ask or run git. |
| Pre-tool guard | Warn when a file >100KB is about to be read | Prevents accidental full reads of generated or binary files. |
| Output filter | `grep “ERROR” logs/app.log \| tail -20` | Sends only relevant log lines instead of full log files. |

### Scripts for repeated tasks

If you run the same sequence often (lint, format, typecheck), put it in a script. Claude calls one script instead of three commands, and the output is shorter than three separate tool results.

```bash
# scripts/check.sh
npm run lint && npm run format && npm run typecheck
```

### Quick reference file

Create `docs/QUICK_REF.md` with your most-used commands, architecture notes, and conventions. Keep it under 50 lines. Claude reads this small file instead of large documentation.

---

> **Callout:** Never go past 12% of your context window (~120K out of 1M tokens). Above that you enter the *context rot zone* where every extra token costs more for lower-quality output. The 1M window is insurance for emergencies, not a target.

## Part 2: External Tools & Repositories

These require third-party installation but provide capabilities beyond what’s built in.

### Output compression

| Tool | Install | Stars | Why |
|------|---------|-------|-----|
| **caveman** | `/plugin marketplace add JuliusBrussee/caveman` · [github](https://github.com/JuliusBrussee/caveman) | 61K | 65% mean output token reduction. Strips narrative filler while keeping code intact. One of the highest-leverage changes you can make. |
| **RTK** | `brew install rtk` → `rtk gain` → `rtk init --global` · [github](https://github.com/rtk-ai/rtk) | — | Compresses CLI outputs before they reach Claude. Up to 90% reduction on noisy commands. |

### Persistent memory

| Tool | Install | Stars | Why |
|------|---------|-------|-----|
| **claude-mem** | `npx claude-mem install` · [github](https://github.com/thedotmack/claude-mem) | 75K | Captures session work, compresses with AI, injects relevant context into future sessions. ChromaDB vector search + local SQLite. Eliminates re-explaining your architecture every session. |

### Context engineering

| Tool | Install | Stars | Why |
|------|---------|-------|-----|
| **GSD** | `npx gsd install` · [github](https://github.com/gsd-build/get-shit-done) | 63K | Spec-driven development. Subagents get exactly the context they need. Solves context drift on sessions longer than ~30 mins. |
| **Language server** | `npm install -g pyright` or `typescript-language-server` | — | Gives Claude type info, definitions, and references without reading large files. |
| **code-review-graph** | `pip install code-review-graph` · [github](https://github.com/tirth8205/code-review-graph) | ~1K | Tree-sitter knowledge graph in SQLite. Claude queries the graph instead of scanning files. 6.8x average token reduction on reviews. |
| **claude-token-efficient** | `git clone github.com/drona23/claude-token-efficient` · [github](https://github.com/drona23/claude-token-efficient) | — | Drop-in optimised CLAUDE.md. 63% output token reduction in benchmarks. |
| **claude-token-optimizer** | [github](https://github.com/nadimtuhin/claude-token-optimizer) | — | Restructures your docs directory so Claude loads only what it needs. Typical: 11,000 → 1,300 tokens on session start. |

### Measure usage

| Tool | Install | Why |
|------|---------|-----|
| **ccusage** | `npx ccusage@latest` → `npx ccusage daily --project myapp --breakdown` · [github](https://github.com/ryoppippi/ccusage) | Shows where tokens are spent by project, model, and time period. |
| **claude-monitor** | `pip install claude-monitor` → `claude-monitor --plan pro --refresh-rate 5` | Live visibility into token usage while you work. |
| **token-optimizer** | `/plugin marketplace add alexgreensh/token-optimizer` · [github](https://github.com/alexgreensh/token-optimizer) | Finds *ghost tokens*: skills you never use, orphaned MEMORY.md files, decisions lost on compaction. |
| **claude-token-lens** | `npm install -g claude-token-lens` | Real-time token attribution showing which tool or MCP server is burning quota. |

### Star-ranked tool summary

| Tool | Stars | Cost impact | Install |
|------|-------|-------------|---------|
| everything-claude-code | 185K | High: strategic-compact + monitoring | `npx ecc install` |
| claude-mem | 75K | High: eliminates session re-learning | `npx claude-mem install` |
| ccswitch | 73K | Medium: model routing + MCP management | [farion1231/cc-switch](https://github.com/farion1231/cc-switch) |
| GSD | 63K | High: context engineering, no drift | `npx gsd install` |
| caveman | 61K | Very high: 65% output reduction | `/plugin marketplace add JuliusBrussee/caveman` |
| claude-squad | 7.4K | Medium: parallel agents per worktree | `brew install claude-squad` |
| code-review-graph | ~1K | Very high on large codebases (6–49x) | `pip install code-review-graph` |

---

## The Main Causes of Token Waste

| Source | Built-in Fix | External Fix |
|--------|-------------|--------------|
| Verbose Claude responses | Concise prompts, done criteria | caveman (65% reduction) |
| Large CLI outputs | Hooks to filter logs | RTK compression |
| Large generated files | `.claudeignore` | — |
| Long conversations | `/clear`, `/compact` at 70% | — |
| Re-explaining codebase | `/resume`, QUICK_REF.md | claude-mem |
| Claude reading irrelevant files | `.claudeignore`, precise prompts | code-review-graph |
| Unused MCP servers | `/mcp`, `/mcp disable` | — |
| Overspending on model tier | Start Sonnet, Haiku for mechanical, Opus only when needed | — |

---

Optimising these areas typically reduces token usage by **30–70%** while making Claude responses faster and more reliable.
