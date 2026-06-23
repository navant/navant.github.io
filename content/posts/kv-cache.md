---
title: "The KV Cache: The Hidden Engine Behind Every AI Response"
date: 2026-06-23T00:00:00+00:00
draft: false
tags: ["AI", "LLM", "Developer Tools"]
author: "Navan Tirupathi"
showToc: true
TocOpen: false
hidemeta: false
comments: false
description: "Why your Claude or Cursor session suddenly slows to a crawl, and the simple habits that keep it fast. A practical guide to the KV cache."
canonicalURL: "https://navant.github.io/posts/kv-cache/"
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

*Why your Claude or Cursor session suddenly slows to a crawl, and the simple habits that keep it fast.*

---

You're deep in a Cursor session. First few replies stream back almost instantly. Then, twenty messages in, you hit send and the screen just... stares at you. Ten seconds. Fifteen. A spinner.

Nothing changed on your end. Same laptop, same wifi, same model. What happened is you tripped the KV cache, and the AI had to relearn your entire codebase from scratch.

Once you understand why, the freezes stop.

## What a KV cache actually is

Language models write text one token at a time. To write word 50, the model needs to "look at" words 1 through 49. To write word 1,000, it looks at everything before it.

Without any optimisation, that compute grows quadratically. Word 10,000 means re-reading 9,999 words. This is why early inference was painfully slow.

The KV cache is the fix. As the model reads your prompt, it computes a set of mathematical vectors (Keys and Values) for each token and saves them in GPU memory. On the next token, instead of recalculating from scratch, it just reads from those saved vectors.

Think of it like a detective's notebook. Without notes, they re-read 500 pages every time a new clue comes in. With notes, they glance at the page and move on. A cache miss means the notebook got lost. Back to page one.

The speed gap is not subtle: a cache hit takes roughly 0.1 seconds. A cache miss on a long context takes 3 to 15 seconds. That blank screen isn't the AI thinking. It's the AI re-reading.

<details>
<summary><strong>Deep dive: what's actually happening inside the model</strong> (for engineers; skip if you don't care about the internals)</summary>

### The attention mechanism

Every transformer layer runs self-attention. For each token, the model asks: "how much should I attend to every other token?"

The score between token `i` and token `j`:

```
score(i,j) = dot(Q_i, K_j) / sqrt(d_k)
```

These scores get softmaxed across the full sequence, then used to take a weighted sum of the Value vectors. That result passes to the next layer.

Without caching, generating token N requires computing Q, K, V for all N tokens and running attention across all N×N pairs, at every layer. For a 40-layer model on a 10k-token context: `40 × 10,000 × 10,000` attention ops before the feed-forward layers even run.

### What the cache stores

During the forward pass, K and V vectors are computed for every token at every layer and stored. Shape:

```
[num_layers, 2, seq_len, num_heads, head_dim]
```

For Llama 3 70B (80 layers, 64 heads, head_dim 128):

```
80 × 2 × 64 × 128 × 2 bytes (fp16) ≈ 2.6 MB per token
100k-token context = ~260 GB VRAM just for the cache.
```

This is why long-context inference is a hardware problem, not just a software one.

### Why one changed token invalidates everything downstream

The cache is positional. Each K and V vector encodes the token's meaning **and** its position (via RoPE embeddings).

Change token 3 → its K/V vectors change → every token after it attended to the old token 3 → their outputs are now wrong. No partial reuse. Recompute from the edit point to the end.

### How production systems handle it

| System | Technique | What it does |
|---|---|---|
| **PagedAttention** (vLLM) | Paged KV cache | Splits the cache into fixed pages like virtual memory. Requests share GPU memory non-contiguously, so less fragmentation and more concurrent sessions. |
| **RadixAttention** (SGLang) | Radix tree of prefixes | 1,000 users with the same system prompt → KV tensors stored once, shared across all. This is why identical system prompts matter at scale. |
| **Prefix caching** (Anthropic / OpenAI) | Commercial prefix reuse | You pay full price once; matching requests after are heavily discounted. Anthropic ~10% of normal input cost for cache reads, OpenAI ~50%. |

</details>

## What a cache miss actually costs you

Most people assume this only matters if you're running your own inference server. It doesn't. If you use Claude Pro or Cursor Pro, the KV cache governs your daily experience.

**Speed.** Cache miss on a large context means the backend re-ingests your entire conversation history before generating a single new word. That's the lag.

**Rate limits.** You have rolling usage caps on paid plans. Cache misses force the backend to re-process your context, and that compute counts against your quota. Consistent cache misses can drain your fast-usage window 3 to 5× faster than a clean session.

**Worse answers.** When a model has to cold-start re-digest your project files, it's more likely to hallucinate variable names, forget constraints you established earlier, and give inconsistent answers. The cache affects answer quality, not only speed.

## The rule everything else follows

The KV cache builds left to right. Change anything near the beginning, even one word, and every cached token after that point gets thrown away.

This is why editing an old message and resubmitting feels so much slower than sending a correction as a new message. You didn't fix one line. You invalidated hundreds of tokens the server had already worked through.

## Threads are not interchangeable

This one catches people constantly.

You have a long session in Thread A about your auth system. You open Thread B to ask something related. Jump back to Thread A. Then Thread B again.

Every switch is a full cache miss. Thread A and Thread B have completely separate caches. The server has no memory of the other conversation. Each time you land in a thread, it re-ingests everything from zero.

Same problem when you switch models mid-conversation. Claude Sonnet and Claude Opus run on different architectures. Switching mid-thread guarantees a 100% cache miss. Pick your thread, pick your model, stay there.

## What happens on long-horizon tasks

This is where most people building with AI agents have no idea they're destroying their own cache on every loop.

The interactive view below walks through it. Flip between **agent loops**, **user edits**, **rules / AGENTS.md**, and the **full impact table**:

<iframe src="/kv-cache-longhorizon.html" title="KV Cache: Long Horizon Tasks (interactive)" loading="lazy" style="width:100%;height:820px;border:1px solid var(--border, #1e2433);border-radius:8px;margin:18px 0;" scrolling="auto"></iframe>

## Why compact summaries mid-task matter so much

When an agent runs a long task (reading files, calling tools, browsing), raw outputs pile up fast. A single `read_file` on a large codebase might dump 8,000 tokens into context. A web scrape adds another 5,000. By loop 6 you've got 40,000 tokens of raw tool output sitting in the sequence.

Two problems hit at once. First, VRAM. At ~2.6 MB per token for a large model, 40,000 tokens of tool output is ~100 GB of cache pressure. The server starts evicting older entries to make room, including your cached AGENTS.md prefix. Second, if you then delete to recover space, you break the sequence and everything cached after the gap recomputes anyway.

Compact summaries solve both: tiny so VRAM pressure stays low, and appended rather than substituted so the sequence stays intact.

```
# Instead of keeping this in context:
[tool_result: <html><head><title>...</title>... 5,000 tokens of raw HTML]

# Replace with this:
[ref#2: scraped pricing page, 3 tiers found: Starter $9, Pro $29, Enterprise custom]

# Model still knows what was found.
# Cache timeline unbroken.
# VRAM cost: 20 tokens instead of 5,000.
```

Think of it like a git commit message vs the full diff. You don't keep every diff in working memory. You keep a one-liner that tells you what changed and where to look if you need the detail.

## Rules, skills, and AGENTS.md are your most fragile cache dependency

Your rules file (`AGENTS.md`, `CLAUDE.md`, `.cursorrules`) sits at the very front of the prefix and loads on every single request. If it's byte-for-byte identical, you pay to process it once, then it's free. Every edit, however small, forces a full re-ingestion next session. So does loading your skills in a different order, or putting a timestamp at the top: one dynamic token at an early position invalidates everything after it.

Treat the prefix like a database schema: change it intentionally and in batches, not casually.

## Recommendations

| Situation | What to do | Why |
|---|---|---|
| Made a typo in a previous message | Send correction as new message | Editing invalidates everything downstream |
| Working across multiple files in Cursor | Attach all files at thread start | Cached once, referenced free for the rest of the session |
| Two related tasks | One thread per task | Every tab switch is a full cache miss |
| Switching models mid-chat | Don't; start a fresh thread | Different architectures share no cache state |
| System prompt with timestamp at the top | Move dynamic values to the bottom | One changing token at position 1 kills the entire cache |
| Long document, many questions | Keep all questions in one thread | Document cached on first read, free on every follow-up |
| Agent with large tool outputs | Replace with compact summary tags | Keeps VRAM low without breaking the sequence |
| AGENTS.md or rules file | Edit in batches, not casually | Every edit = full re-ingestion next session |
| Skills loaded in different order each session | Fix the order, always same sequence | Different token positions = no prefix match |
| Reusing prompts across many users (API) | Keep system prompt byte-for-byte identical | Shared infra serves the cached prefix once for all users |

---

*Next: how to structure long-document RAG so the model caches individual chapters rather than re-reading the whole book on every query.*

*If this was useful, pass it to one person who complains about slow AI responses.*
