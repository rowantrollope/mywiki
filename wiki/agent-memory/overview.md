---
title: Agent Memory — Overview
tags: [agent-memory, overview, ai-agents, llm]
created: 2026-04-04
last_updated: 2026-04-04
---

## What Is Agent Memory?

Agent memory refers to the mechanisms by which an AI agent stores, retrieves, and reasons over information across time. Unlike a single-shot LLM call that processes a prompt and returns a response, an autonomous agent must maintain context across many interactions, learn from experience, and apply past knowledge to new situations.

The concept maps closely to human cognitive memory but is implemented through a mix of in-context storage (the prompt window), vector databases, key-value stores, relational databases, and increasingly, knowledge graphs. The challenge is not just *storing* information — it's knowing what to store, when to retrieve it, and how to keep it coherent.

## Why It Matters

Without memory, agents are amnesiac by default. Every interaction starts cold. A customer service agent forgets what the user said two messages ago. A coding assistant has no idea what architecture decisions were made last week. A personal assistant can't build a model of your preferences over time.

Memory is what turns an LLM into a persistent, improving entity. It enables:

- **Personalization** — agents that know your preferences, history, and context
- **Task continuity** — resuming work across sessions without re-explaining everything
- **Learning from feedback** — storing corrections and improving future behavior
- **Long-horizon planning** — retaining goals and sub-tasks over extended periods
- **Multi-agent collaboration** — shared memory between agents working on the same problem

The difference between a demo that impresses and a product that actually works often comes down to how well memory is implemented.

## The Memory Hierarchy

Modern agent architectures typically distinguish four types of memory, borrowed loosely from cognitive science:

1. **Working memory** — the active context window; what the agent is currently processing
2. **Episodic memory** — records of specific events and interactions ("user said X on Tuesday")
3. **Semantic memory** — generalized facts and knowledge ("user prefers Python over JavaScript")
4. **Procedural memory** — learned behaviors and skills ("how to format a report")

See [[concepts/types-of-memory]] for a detailed taxonomy.

## Current State of the Field (2024–2025)

The field has moved fast. A few years ago, "agent memory" mostly meant stuffing as much context as possible into the prompt. Today it encompasses:

- **RAG (Retrieval-Augmented Generation)** — retrieving relevant documents from a vector store and injecting them into context. Mature, widely deployed. See [[architectures/rag]].
- **MemGPT** — a 2023 paper from Berkeley that treats the context window like CPU registers and uses external storage like a paging system. Introduced virtual context management. See [[architectures/memgpt]].
- **mem0** — a commercial/OSS layer that provides a structured API for agent memory with automatic extraction, deduplication, and retrieval. See [[architectures/mem0]].
- **Knowledge graphs** — structured representations of entities and relationships; better than vectors for multi-hop reasoning. See [[architectures/knowledge-graphs]].
- **Generative Agents** — the Stanford/Google paper that simulated 25 agents with full episodic memory and showed emergent social behavior. A landmark. See [[research/key-papers]].

## The Core Tension

Every agent memory system faces a fundamental tradeoff: **completeness vs. relevance**. Storing everything is expensive and adds noise at retrieval time. Summarizing aggressively loses detail. The best systems use layered compression — raw logs → episode summaries → distilled facts — with different retrieval strategies for each layer.

There's also the **coherence problem**: as an agent accumulates memories from multiple sources and sessions, contradictions arise. Memory A says the user prefers dark mode; Memory B says they switched to light. Which is current? Systems that don't handle temporal ordering and conflict resolution degrade over time.

## See Also

- [[concepts/types-of-memory]]
- [[concepts/working-memory]]
- [[concepts/long-term-memory]]
- [[concepts/memory-retrieval]]
- [[architectures/rag]]
- [[architectures/memgpt]]
- [[research/key-papers]]
