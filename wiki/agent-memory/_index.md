---
title: Agent Memory — Topic Index
tags: [agent-memory, index]
created: 2026-04-04
last_updated: 2026-04-04
---

## Agent Memory Wiki

This section covers everything about how AI agents store, retrieve, and reason over memory — from foundational concepts to production architectures to cutting-edge research.

---

## Overview

| Article | Summary |
|---------|---------|
| [[overview]] | What agent memory is, why it matters, and the current state of the field |

---

## Concepts

| Article | Summary |
|---------|---------|
| [[concepts/types-of-memory]] | Taxonomy of the four memory types: episodic, semantic, procedural, working |
| [[concepts/episodic-vs-semantic]] | The episodic/semantic distinction, consolidation pipeline, practical implementations |
| [[concepts/working-memory]] | Context window as working memory; limits, attention patterns, management strategies |
| [[concepts/long-term-memory]] | External stores, layered compression, retention policies, coherence challenges |
| [[concepts/memory-retrieval]] | Similarity search, recency weighting, importance scoring, failure modes |

---

## Architectures

| Article | Summary |
|---------|---------|
| [[architectures/rag]] | Retrieval-Augmented Generation: how it works, when to use it, advanced patterns |
| [[architectures/vector-stores]] | Embeddings, vector databases (Pinecone, Chroma, pgvector), indexing algorithms |
| [[architectures/knowledge-graphs]] | Graph memory: entities + relations, multi-hop queries, when graphs beat vectors |
| [[architectures/mem0]] | mem0.ai: managed memory layer with automatic extraction and conflict resolution |
| [[architectures/memgpt]] | MemGPT/Letta: virtual context management, the OS analogy, agent-driven memory |

---

## Research

| Article | Summary |
|---------|---------|
| [[research/key-papers]] | Annotated bibliography: MemGPT, Generative Agents, CoALA, RAG, ReAct, Reflexion |
| [[research/open-problems]] | Unsolved: catastrophic forgetting, coherence, hallucination, privacy, relevance scoring |

---

## Backlinks

This index is referenced from [[/_global_index]].

Individual articles link back to this index via their `## See Also` sections.

---

## How to Navigate

- **New to the topic?** Start with [[overview]], then [[concepts/types-of-memory]]
- **Building something?** Go to [[architectures/rag]] or [[architectures/vector-stores]]
- **Evaluating tools?** Compare [[architectures/mem0]] vs [[architectures/memgpt]]
- **Researching?** Start with [[research/key-papers]] reading order recommendations
- **Debugging a system?** Check [[research/open-problems]] for known failure modes

---

## Quick Reference: Memory Type → Storage → Retrieval

| Memory Type | Typical Storage | Typical Retrieval |
|-------------|----------------|-------------------|
| Working | Context window | Direct (in-context) |
| Episodic | Log DB + vector store | Recency + semantic similarity |
| Semantic | KV store + vector store + graph | Entity lookup + semantic similarity |
| Procedural | System prompt + fine-tuned weights | Implicit (baked in) |
