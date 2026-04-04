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
| [[concepts/memory-consolidation]] | How agents consolidate short-term episodic experiences into stable long-term knowledge |
| [[concepts/forgetting-mechanisms]] | Intentional forgetting, relevance decay, Ebbinghaus-inspired pruning, retention policies |
| [[concepts/reflection-and-abstraction]] | Higher-order memory synthesis, multi-level reflection, abstraction patterns |

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

## Papers (Detailed Article per Paper)

| Article | Summary |
|---------|---------|
| [[papers/_index]] | Annotated bibliography of all surveyed papers with reading order recommendations |
| [[papers/memgpt]] | MemGPT: virtual context management via OS-inspired memory hierarchy (arXiv: 2310.08560) |
| [[papers/generative-agents]] | Generative Agents: memory stream, reflection, and emergent social behavior (arXiv: 2304.03442) |
| [[papers/coala]] | CoALA: unified taxonomy mapping cognitive science memory types to LLM agents (arXiv: 2309.02427) |
| [[papers/a-mem]] | A-MEM: Zettelkasten-inspired agentic memory with dynamic linking and evolution (arXiv: 2502.12110) |
| [[papers/hipporag]] | HippoRAG: hippocampus-inspired graph-indexed RAG with Personalized PageRank (arXiv: 2405.14831) |
| [[papers/memorybank]] | MemoryBank: Ebbinghaus forgetting curve for LLM long-term memory (arXiv: 2305.10250) |
| [[papers/reflexion]] | Reflexion: verbal self-reflection as episodic memory for agent self-improvement (arXiv: 2303.11366) |
| [[papers/raise]] | RAISE: scratchpad + examples working memory for conversational agents (arXiv: 2401.02777) |
| [[papers/agemem]] | AgeMem: unified LTM/STM management via end-to-end RL with step-wise GRPO (arXiv: 2601.01885, 2026) |
| [[papers/autorefine]] | AutoRefine: dual-form experience patterns (subagents + skills) for continual agent refinement (arXiv: 2601.22758, 2026) |
| [[papers/memory-agent-bench]] | MemoryAgentBench: four-competency memory evaluation benchmark via incremental multi-turn interactions (arXiv: 2507.05257, 2026) |

---

## Blogs and Industry Coverage

| Article | Summary |
|---------|---------|
| [[blogs/lilian-weng-agents]] | Lilian Weng's canonical LLM agents post — memory section analysis and influence |
| [[blogs/recent-developments]] | Synthesis of 2024-2025 blog coverage: LangChain, Anthropic, mem0, OpenAI, LlamaIndex |

---

## Research

| Article | Summary |
|---------|---------|
| [[research/key-papers]] | Annotated bibliography: MemGPT, Generative Agents, CoALA, RAG, ReAct, Reflexion, and 7 more |
| [[research/open-problems]] | Unsolved: catastrophic forgetting, coherence, hallucination, privacy, evaluation gaps |
| [[research/state-of-the-art-2025]] | SOTA 2025–2026: production maturity, reference architecture, key systems, trends, 2026 paper updates |

---

## Backlinks

This index is referenced from [[/_global_index]].

Individual articles link back to this index via their `## See Also` sections.

---

## How to Navigate

- **New to the topic?** Start with [[overview]], then [[concepts/types-of-memory]]
- **Building something?** Go to [[architectures/rag]] or [[architectures/vector-stores]]
- **Evaluating tools?** Compare [[architectures/mem0]] vs [[architectures/memgpt]]
- **Researching?** Start with [[papers/_index]] for reading order recommendations
- **Reading papers?** Browse [[papers/memgpt]], [[papers/generative-agents]], etc. for detailed summaries
- **Debugging a system?** Check [[research/open-problems]] for known failure modes
- **Want the big picture?** Read [[research/state-of-the-art-2025]]

---

## Quick Reference: Memory Type → Storage → Retrieval

| Memory Type | Typical Storage | Typical Retrieval |
|-------------|----------------|-------------------|
| Working | Context window | Direct (in-context) |
| Episodic | Log DB + vector store | Recency + semantic similarity |
| Semantic | KV store + vector store + graph | Entity lookup + semantic similarity |
| Procedural | System prompt + fine-tuned weights | Implicit (baked in) |
