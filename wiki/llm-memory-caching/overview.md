---
title: LLM Memory and Caching — Overview
tags: [llm, memory, caching, overview]
created: 2026-04-06
last_updated: 2026-04-06
---

## What Is LLM Memory and Caching?

"Memory" in the context of LLMs refers to the mechanisms by which these models store, access, and reuse information — both within a single inference run and across multiple calls. "Caching" refers to storing intermediate or repeated computations so they don't need to be recalculated, cutting cost and latency.

These two concepts are deeply intertwined but operate at different levels of abstraction:

- **Model-level caching** happens inside the transformer during inference. The KV (key-value) cache stores attention computations for tokens already processed, so each new token doesn't recompute the entire history. This is automatic and invisible to users.
- **API-level caching** (prompt caching) happens at the inference server. If you send a long system prompt that's identical across many calls, the provider can cache the processed representation and skip recomputing it. Anthropic, OpenAI, and Google all now offer this.
- **Application-level memory** is what developers build: retrieval-augmented generation (RAG), vector stores, conversation buffers, summarization pipelines. This extends the model's effective memory far beyond the context window.

## Why It Matters

The naive approach to LLM applications is to pack everything into the context: the full conversation history, all system instructions, relevant documents. This works until it doesn't. Context windows are finite, expensive, and — despite recent expansions to 128K or even 1M tokens — not free to use.

The economics are stark. As of 2025–2026, Claude Sonnet charges roughly $3/M input tokens. A 100K-token prompt with a 5K system prompt repeated across 10,000 daily requests costs ~$3,000/day *just for the system prompt tokens*. Prompt caching that prefix can reduce that by 90% — $270/day instead.

Beyond cost, there's latency. Attention over a 100K context is quadratically more expensive than attention over 1K tokens. KV caching and smart context management aren't just about money — they're about whether your application is actually fast enough to use.

And at the architecture level, memory defines what kind of intelligence is possible. A stateless LLM is a brilliant amnesiac. Memory is what enables:

- **Personalization** — knowing user preferences, history, context
- **Long-horizon task completion** — maintaining goals across sessions
- **Iterative refinement** — building on prior work without re-explaining it
- **Multi-agent coordination** — sharing state between autonomous agents

## The Memory Stack

Think of LLM memory as a layered stack, from fastest/most-expensive to slowest/cheapest:

```
┌─────────────────────────────────────────────────────────────┐
│  PARAMETRIC MEMORY (model weights)                          │
│  - Knowledge baked into model at training time              │
│  - Zero inference cost, zero latency                        │
│  - Cannot be updated without retraining/fine-tuning         │
├─────────────────────────────────────────────────────────────┤
│  KV CACHE (inside the transformer)                          │
│  - Attention key-value pairs for processed tokens           │
│  - Per-inference, stored in GPU memory                      │
│  - Automatic; not configurable at API level                 │
├─────────────────────────────────────────────────────────────┤
│  IN-CONTEXT MEMORY (prompt / context window)                │
│  - What the model currently "sees"                          │
│  - 4K–1M tokens depending on model                         │
│  - Costs per token; no persistence across calls             │
├─────────────────────────────────────────────────────────────┤
│  PROMPT CACHE (API-level prefix cache)                      │
│  - Reused prefix computation across API calls               │
│  - 5 min–1 hour TTL depending on provider                   │
│  - 50–90% cost reduction on cached tokens                   │
├─────────────────────────────────────────────────────────────┤
│  EXTERNAL MEMORY (RAG, vector stores, databases)            │
│  - Unlimited storage; retrieved on demand                   │
│  - Adds latency for lookup; requires embedding pipeline     │
│  - Survives across sessions, users, and deployments         │
└─────────────────────────────────────────────────────────────┘
```

Most production systems use at least three of these five layers simultaneously.

## The Current State of Play (2025–2026)

The field has converged on a few clear truths:

**Context windows are bigger but not free.** Models like Gemini 1.5 Pro (1M tokens) and Claude with 200K windows have largely eliminated the "doesn't fit" problem for most tasks. But fitting and *effectively using* long context are different things — attention degrades in the middle of long contexts (the "lost in the middle" phenomenon, documented by Liu et al., 2023). And the cost still adds up.

**Prompt caching is now table stakes.** All major providers have it. If you have a long static prefix (system prompt, documents, examples), and you're not caching it, you're leaving money on the table. The patterns are well-understood; see [[patterns/cache-optimization]].

**RAG is mature but not solved.** Standard RAG works well for simple retrieval. But multi-hop reasoning, contradiction handling, and freshness management remain hard. Hybrid approaches (RAG + long context + prompt caching) are increasingly common.

**KV cache compression is an active research frontier.** GPU memory is the bottleneck for large batch sizes. Research into KV cache eviction, quantization, and offloading is moving quickly. See [[research/state-of-the-art-2026]].

## See Also

- [[concepts/kv-cache]] — deep dive into transformer KV caching mechanics
- [[concepts/prompt-caching]] — API-level prompt caching across providers
- [[concepts/memory-types]] — full taxonomy of in-context, external, parametric memory
- [[concepts/context-window-management]] — strategies for using context windows effectively
- [[patterns/cache-optimization]] — practical guide to maximizing cache hits
- [[patterns/memory-architectures]] — RAG, vector stores, and episodic memory patterns
- [[research/key-papers]] — foundational papers
- [[wiki/agent-memory/overview|Agent Memory Overview]] — related topic focusing on agent architectures
