---
title: "MemGPT: Towards LLMs as Operating Systems"
tags: [agent-memory, papers, arxiv, memgpt, virtual-context, memory-tiers]
created: 2026-04-04
last_updated: 2026-04-04
---

## Overview

**Paper:** MemGPT: Towards LLMs as Operating Systems
**Authors:** Charles Packer, Sarah Wooders, Kevin Lin, Vivian Fang, Shishir G. Patil, Ion Stoica, Joseph E. Gonzalez (UC Berkeley)
**arXiv:** [2310.08560](https://arxiv.org/abs/2310.08560) — published October 2023
**Productionized as:** [Letta](https://letta.ai) (formerly MemGPT)

## The Core Idea

MemGPT treats LLM context management like an operating system treats memory. The key insight: modern OS kernels manage a hierarchy of memory tiers (CPU registers → RAM → disk → network storage) through paging, eviction, and virtual memory — creating the illusion that programs have unlimited fast memory even though physical memory is finite.

Applied to LLMs: the context window is like RAM (small, fast, expensive), and external databases are like disk (large, slow, cheap). MemGPT builds a virtual context management system that intelligently moves data between these tiers via **agent-driven memory operations**.

## Architecture

MemGPT introduces a structured memory hierarchy:

- **Main Context (working memory):** The active context window the LLM sees. Contains the system prompt, working scratchpad, and selected memories.
- **External Context (archival memory):** A large-scale external store (vector DB). The agent explicitly queries this via function calls.
- **Recall Storage:** Conversation history with search capabilities. A searchable log of past interactions.

The agent controls its own memory via **function calls**:
- `archival_memory_insert(content)` — write a new long-term memory
- `archival_memory_search(query)` — retrieve relevant memories
- `conversation_search(query)` — search conversation history
- `core_memory_replace(key, value)` — update structured "always-present" facts

## The OS Analogy in Depth

| OS Concept | MemGPT Equivalent |
|-----------|-------------------|
| CPU registers | Active reasoning in context |
| RAM (main memory) | Context window |
| Disk storage | Archival/external DB |
| Page fault interrupt | Context overflow → eviction + retrieval |
| Virtual memory | Virtual context (agent manages what's "in" context) |
| OS kernel | The MemGPT system prompt and memory functions |

When the context window fills up, MemGPT triggers an **interrupt** — analogous to a page fault — and the agent decides what to evict to external storage and what to load from archival storage.

## Key Results

- **Document analysis:** MemGPT can analyze documents that far exceed the underlying LLM's context window by paging sections in/out as needed
- **Multi-session chat:** MemGPT agents maintain continuity across thousands of conversational turns by storing and retrieving from archival memory
- Demonstrated with GPT-4 as the underlying LLM

## Why This Matters

Before MemGPT, most systems treated memory retrieval as **preprocessing** — you retrieve relevant content and stuff it into the context before calling the LLM. MemGPT shifted to **agent-driven memory** — the agent itself decides when to read/write memory as part of the reasoning loop.

This is a fundamental shift: memory becomes a *first-class action* rather than an external preprocessing step.

## Limitations and Critiques

- Extra function calls add latency and token cost
- The agent must learn when to invoke memory operations — this requires good prompting
- Archival memory search is only as good as the embedding model
- The OS analogy is illuminating but imperfect (LLMs don't have hard eviction policies)

## Key Takeaways

- **Virtual context management:** create the illusion of unlimited memory by paging data in/out of the context window
- **Agent-driven memory:** the LLM actively controls its own memory via function calls, not just passively consuming retrieved content
- **Interrupt-based control flow:** context overflow triggers memory management, analogous to OS page faults
- **Tiered architecture:** working memory (context) → recall storage (conversation history) → archival storage (long-term vector DB)
- **Productionized:** became Letta, a production agent memory framework used by thousands of developers

## See Also

- [[research/key-papers]]
- [[papers/generative-agents]]
- [[papers/coala]]
- [[architectures/memgpt]]
- [[concepts/working-memory]]
- [[concepts/long-term-memory]]
