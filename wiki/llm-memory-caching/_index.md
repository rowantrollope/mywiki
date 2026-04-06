---
title: LLM Memory and Caching — Topic Index
tags: [llm, memory, caching, kv-cache, prompt-caching, rag, index]
created: 2026-04-06
last_updated: 2026-04-06
---

## LLM Memory and Caching Wiki

This section covers the full stack of how LLMs store, reuse, and retrieve information — from the transformer attention mechanism's KV cache up through API-level prompt caching, context management strategies, and full-scale memory architectures.

---

## Overview

| Article | Summary |
|---------|---------|
| [[overview]] | What LLM memory and caching means, why it matters, and how the pieces fit together |

---

## Concepts

| Article | Summary |
|---------|---------|
| [[concepts/kv-cache]] | KV cache mechanics: how transformers avoid recomputing attention for every token |
| [[concepts/prompt-caching]] | Anthropic, OpenAI, and Google's prompt caching APIs — cost savings, TTLs, gotchas |
| [[concepts/context-window-management]] | Strategies for fitting more useful information into a finite context window |
| [[concepts/memory-types]] | In-context, external, and parametric memory — a unified taxonomy |

---

## Patterns

| Article | Summary |
|---------|---------|
| [[patterns/cache-optimization]] | How to structure prompts for maximum cache hits across all major providers |
| [[patterns/memory-architectures]] | RAG, vector stores, episodic memory — choosing and combining patterns in production |

---

## Research

| Article | Summary |
|---------|---------|
| [[research/key-papers]] | Annotated bibliography of landmark papers on LLM memory and caching |
| [[research/state-of-the-art-2026]] | Current SOTA: long-context models, KV compression, memory-augmented LLMs, open problems |

---

## Backlinks

This index is referenced from [[/_global_index]].

Individual articles link back here via their `## See Also` sections.

---

## How to Navigate

- **New to this?** Start with [[overview]], then [[concepts/memory-types]]
- **Reducing API costs?** Go to [[concepts/prompt-caching]] → [[patterns/cache-optimization]]
- **Building agents?** Read [[patterns/memory-architectures]] and [[concepts/context-window-management]]
- **Optimizing inference infra?** Deep dive into [[concepts/kv-cache]]
- **Researching?** Start with [[research/key-papers]], then [[research/state-of-the-art-2026]]

---

## Quick Reference: Memory Layer → Mechanism → Cost/Latency

| Layer | Mechanism | Typical Cost Impact |
|-------|-----------|---------------------|
| KV Cache (model) | Attention reuse within one inference | Reduces compute; memory bandwidth bound |
| Prompt Cache (API) | Reuse prefix computation across calls | 50–90% input token cost reduction |
| Context Window | In-context storage, no external lookup | Fixed per model; pays per token |
| External (RAG) | Vector search + retrieval | DB cost + embedding; avoids long contexts |
| Parametric | Fine-tuned into weights | One-time training cost; zero inference overhead |

## Related Topics

- [[wiki/agent-memory/_index|Agent Memory]] — overlapping topic; focuses on agent architectures rather than model mechanics
