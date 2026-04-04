---
title: Key Papers in Agent Memory Research
tags: [agent-memory, research, papers, bibliography, memgpt, generative-agents, coala]
created: 2026-04-04
last_updated: 2026-04-04
---

## Annotated Bibliography

This is a curated reading list for agent memory research, organized by theme with annotations on what each paper contributes and why it matters.

---

## Foundational Memory Architecture Papers

### MemGPT: Towards LLMs as Operating Systems (2023)
**Authors:** Packer, Fang, Wooders, Patil, Dao, Stoica, Gonzalez (UC Berkeley)
**Link:** https://arxiv.org/abs/2310.08560

The paper that introduced the OS analogy for LLM memory management. Proposes treating the context window like CPU registers and external databases like hierarchical memory (RAM/disk). The agent controls its own memory via function calls — loading, evicting, and summarizing content.

**Why it matters:** Shifted the paradigm from "retrieval as pre-processing" to "retrieval as agent action." The structured block architecture (main context + archival + recall) has influenced nearly every serious agent memory design since. Has been productionized as Letta.

**Key contribution:** Agent-driven memory management; virtual context management.

---

### Generative Agents: Interactive Simulacra of Human Behavior (2023)
**Authors:** Park, O'Brien, Cai, Morris, Liang, Bernstein (Stanford/Google)
**Link:** https://arxiv.org/abs/2304.03442

Simulated 25 LLM-powered agents in a social environment ("Smallville") with full episodic memory, planning, and social behavior. Each agent maintains a memory stream — a log of observations and reflections — and retrieves from it using a composite score of recency, importance, and relevance.

**Why it matters:** The memory retrieval formula (α*recency + β*importance + γ*relevance) has become a standard reference. Showed that emergent social behavior (parties, gossip, collaboration) arises from proper memory + planning. First large-scale demonstration of persistent agent identity across extended simulation.

**Key contribution:** Memory stream architecture; composite retrieval scoring; emergent social behavior from memory.

---

### Cognitive Architectures for Language Agents (CoALA, 2023)
**Authors:** Sumers, Yao, Narasimhan, Griffiths (Princeton)
**Link:** https://arxiv.org/abs/2309.02427

A systematic survey and taxonomy of language agent architectures. Maps LLM agent components to cognitive science concepts: working memory, episodic memory, semantic memory, procedural memory. Provides a unified framework for comparing and designing agent systems.

**Why it matters:** The CoALA taxonomy has become the standard vocabulary for discussing agent memory. Before this paper, researchers used inconsistent terminology. CoALA gave the field a shared conceptual framework.

**Key contribution:** Unified taxonomy mapping cognitive science memory types to LLM agent architectures.

---

## Retrieval-Augmented Generation

### Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks (2020)
**Authors:** Lewis, Perez, Piktus, Petroni, Karpukhin, Goyal, et al. (Meta AI)
**Link:** https://arxiv.org/abs/2005.11401

The paper that named RAG. Combines a retrieval component (dense passage retrieval over Wikipedia) with a seq2seq generator. Fine-tunes both components end-to-end.

**Why it matters:** Established RAG as a first-class architecture pattern. Showed that parametric knowledge (model weights) + non-parametric knowledge (retrieval) dramatically outperforms either alone on knowledge-intensive tasks.

**Key contribution:** RAG as an architecture; joint training of retrieval + generation.

---

### Lost in the Middle: How Language Models Use Long Contexts (2023)
**Authors:** Liu, Lin, Hewitt, Paranjape, Bevilacqua, Petroni, Liang (Stanford/Meta)
**Link:** https://arxiv.org/abs/2307.03172

Systematic study of how LLMs use information at different positions in long contexts. Found significant performance degradation for information placed in the middle of long contexts (primacy and recency bias).

**Why it matters:** Fundamental constraint for working memory design. Explains why you can't just stuff everything into a 1M-token context and expect reliable retrieval. Motivates structured context management.

**Key contribution:** Empirical evidence for the "lost in the middle" phenomenon; implications for context window utilization.

---

## Long-Term Memory and Personalization

### A Survey on the Memory Mechanism of Large Language Model Based Agents (2024)
**Authors:** Zhang, Bo, Feng, Li, Wang, et al.
**Link:** https://arxiv.org/abs/2404.13501

Comprehensive survey of memory mechanisms in LLM-based agents. Covers storage types, retrieval methods, and memory operations. Good entry point for the field.

**Why it matters:** Most comprehensive single-document overview as of 2024. Useful for understanding the breadth of approaches.

---

### Beyond the Context Window: Memory-Augmented Neural Networks for Efficient Reasoning (various)

The field of memory-augmented neural networks (MANNs) predates the LLM era. Key works:
- **Neural Turing Machines** (Graves et al., 2014) — external differentiable memory with read/write heads
- **Memory Networks** (Weston et al., 2015, Facebook) — external memory for QA tasks
- **Differentiable Neural Computer** (Graves et al., 2016) — improved NTM with dynamic allocation

**Why they matter:** These older papers established that external memory is architecturally separable from computation — a key insight that RAG and agent memory systems operationalize at the LLM level.

---

## Agent Planning and Memory Integration

### ReAct: Synergizing Reasoning and Acting in Language Models (2022)
**Authors:** Yao, Zhao, Yu, Du, Shafran, Narasimhan, Cao (Princeton/Google)
**Link:** https://arxiv.org/abs/2210.03629

Introduces the ReAct pattern: interleave chain-of-thought reasoning with tool use. Memory retrieval becomes a tool action in the ReAct loop.

**Why it matters:** ReAct is the backbone of most agent memory implementations. The agent reasons → decides to retrieve → retrieves → updates reasoning → acts. Memory is integrated into the agent loop, not just pre-loaded.

---

### Reflexion: Language Agents with Verbal Reinforcement Learning (2023)
**Authors:** Shinn, Cassano, Berman, Gopalan, Song, Narasimhan
**Link:** https://arxiv.org/abs/2303.11366

Agents learn from failures by reflecting on past attempts and storing verbal feedback as episodic memory. No gradient updates — learning is entirely through stored linguistic self-reflection.

**Why it matters:** Shows that procedural memory can be updated through natural language rather than fine-tuning. The agent stores "what went wrong and why" as episodic memory and retrieves it to avoid repeating mistakes.

---

## Reading Order Recommendations

1. **Start with CoALA** — get the vocabulary and taxonomy
2. **Then Generative Agents** — see a complete memory system in action
3. **Then MemGPT** — understand the architectural innovations
4. **Then ReAct** — understand how retrieval integrates with action
5. **Then RAG (Lewis 2020)** — the retrieval foundation
6. **Then Lost in the Middle** — understand context window constraints
7. **Then Reflexion** — memory as self-improvement

## See Also

- [[open-problems]]
- [[architectures/memgpt]]
- [[architectures/rag]]
- [[concepts/memory-retrieval]]
- [[concepts/types-of-memory]]
