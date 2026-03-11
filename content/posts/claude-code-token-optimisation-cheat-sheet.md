---
title: "Claude Code – Token Optimisation Cheat Sheet"
date: 2025-02-16T11:30:03+00:00
draft: false
tags: ["Claude", "AI", "Developer Tools", "Tokens"]
author: "Navan Tirupathi"
showToc: true
TocOpen: false
hidemeta: false
comments: false
description: "A practical checklist to reduce Claude Code token usage and cost: RTK, language servers, ccusage, .claudeignore, skills, hooks, and session habits. Save 30–50% on costs."
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

AI coding tools like Claude Code can dramatically speed up development, but costs can grow quickly if context and token usage aren’t managed—especially if you don’t use locally running models. Most token waste comes from long conversations, large logs, and unnecessary files being loaded into context.

I’ve collated a short checklist, from one-time setup to daily workflow habits, to reduce token usage, lower cost, and keep Claude responses fast and reliable. These can save **30–50%** of your Claude Code cost.

![Claude Code Token Optimisation Cheat Sheet](/cc_token_optimise.png)

## Checklist: Setup and Workflow

| Action | Setup / Example | Why |
|--------|-----------------|-----|
| **Output compression (RTK)** | `brew install rtk` → `rtk gain` → `rtk init --global`. Ref: [rtk-ai/rtk](https://github.com/rtk-ai/rtk) | Free CLI proxy that compresses outputs for Claude Code, Cursor, or Aider; cuts tokens in console/CLI or VS Code (not web). |
| **Language server** | e.g. `npm install -g pyright` or `npm install -g typescript-language-server`, then in Claude open `/plugin` and select it | Gives Claude type info, definitions, and references without reading large files. |
| **ccusage** | `npx ccusage@latest` → `npx ccusage daily --project myapp --breakdown` | Shows where tokens are spent by project, model, and time period. |
| **Claude monitor** | `pip install claude-monitor` → `claude-monitor --plan pro --refresh-rate 5` | Live visibility into token usage while you work. |
| **`.claudeignore`** | Add `node_modules/`, `logs/`, `dist/`, `build/`, `coverage/` | Stops Claude from scanning large generated folders. |
| **`CLAUDE.md` + skills** | Keep `CLAUDE.md` small (`wc -l CLAUDE.md`); move large workflows from `.claude/rules/` to `.claude/skills/` | Reduces always-loaded context; large instructions become on-demand. |
| **Quick reference file** | Create `docs/QUICK_REF.md` with commands like `npm run dev`, `npm run test`, `npm run build` | Lets Claude use a small cheat sheet instead of large docs. |
| **Hooks** | `.claude/hooks/session-start.sh` → `git branch --show-current`; `.claude/hooks/pre-tool.sh` → `grep "ERROR" logs/app.log` | Injects useful context and filters noisy logs before they reach Claude. |
| **Session management** | `/clear`, `/compact`, `/rename <name>`, `/resume <name>`, `/cost`, `/model haiku`, `/mcp disable <tool>` | Keeps context small, reduces cost, avoids carrying stale conversations. |
| **Prompt and workflow habits** | e.g. “Fix race in src/auth.ts lines 42–60”; “Done when: tests pass, no lint errors”; use subagents for multi-file work; use `Shift + Tab` before implementation | Reduces unnecessary reading, follow-ups, and large-context investigations. |
| **Scripts for repeated tasks** | `format.sh` → `npm run lint && npm run format && npm run typecheck` | Moves repeated work out of chat and into local automation. |

## The Main Causes of Token Waste

| Source | Typical Fix |
|--------|-------------|
| Large CLI outputs | Use **RTK compression** |
| Large generated files | Configure **`.claudeignore`** |
| Long conversations | Use **`/clear`** and **`/compact`** regularly |

Optimising these three areas alone typically reduces token usage by **30–50%** while making Claude responses faster and more reliable.
