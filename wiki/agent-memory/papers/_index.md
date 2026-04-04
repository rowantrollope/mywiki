---
title: "Agent Memory Papers — Annotated Bibliography"
tags: [agent-memory, papers, bibliography, index]
created: 2026-04-04
last_updated: 2026-04-04
---

## Annotated Bibliography

Papers on agent memory, organized by contribution area. Each entry includes the arXiv ID, a one-sentence summary, and a link to the full article.

---

## Foundational Architectures

### MemGPT: Towards LLMs as Operating Systems (2023)
**arXiv:** [2310.08560](https://arxiv.org/abs/2310.08560) | **Article:** [[papers/memgpt]]

Packer et al. treat the LLM context window as "RAM" and external databases as "disk," building a virtual context management system where the agent pages memories in/out via explicit function calls. Productionized as Letta.

---

### Generative Agents: Interactive Simulacra of Human Behavior (2023)
**arXiv:** [2304.03442](https://arxiv.org/abs/2304.03442) | **Article:** [[papers/generative-agents]]

Park et al. demonstrate 25 LLM-powered agents in a social simulation, using a memory stream with composite retrieval scoring (recency + importance + relevance) and periodic reflection to produce believable emergent social behavior.

---

### Cognitive Architectures for Language Agents (CoALA, 2023)
**arXiv:** [2309.02427](https://arxiv.org/abs/2309.02427) | **Article:** [[papers/coala]]

Sumers et al. provide a unified taxonomy mapping LLM agent components to cognitive science memory types (working, episodic, semantic, procedural), giving the field a shared conceptual vocabulary.

---

## Long-Term Memory and Personalization

### MemoryBank: Enhancing LLMs with Long-Term Memory (2023)
**arXiv:** [2305.10250](https://arxiv.org/abs/2305.10250) | **Article:** [[papers/memorybank]]

Zhong et al. propose a memory mechanism for long-term AI companions, featuring Ebbinghaus-inspired forgetting (memory strength decays with time, reinforced on recall), personality modeling, and compatibility with both closed and open-source LLMs.

---

### A-MEM: Agentic Memory for LLM Agents (2025)
**arXiv:** [2502.12110](https://arxiv.org/abs/2502.12110) | **Article:** [[papers/a-mem]]

Xu et al. propose Zettelkasten-inspired memory where new experiences are stored as richly-linked notes, and new memories can trigger updates to existing ones — creating an evolving knowledge network rather than an isolated collection.

---

## Retrieval-Augmented Memory

### HippoRAG: Neurobiologically Inspired Long-Term Memory (2024)
**arXiv:** [2405.14831](https://arxiv.org/abs/2405.14831) | **Article:** [[papers/hipporag]]

Jimenez Gutierrez et al. model the hippocampus/neocortex division of labor: an LLM extracts entities into a knowledge graph (hippocampal index), and Personalized PageRank enables multi-hop retrieval — outperforming standard RAG by up to 20% at 10-30× lower cost.

---

## Self-Improvement via Memory

### Reflexion: Language Agents with Verbal Reinforcement Learning (2023)
**arXiv:** [2303.11366](https://arxiv.org/abs/2303.11366) | **Article:** [[papers/reflexion]]

Shinn et al. reinforce agents through linguistic self-reflection rather than gradient updates: failed attempts generate verbal lessons stored as episodic memory, which are retrieved on subsequent attempts — achieving 91% on HumanEval coding benchmark.

---

## Working Memory Architecture

### RAISE: Reasoning and Acting through Scratchpad and Examples (2024)
**arXiv:** [2401.02777](https://arxiv.org/abs/2401.02777) | **Article:** [[papers/raise]]

Chen et al. extend ReAct with a four-component working memory (scratchpad + conversation history + task trajectory + few-shot examples), providing the agent with a persistent workspace for multi-turn reasoning.

---

## Context Scaling

### LongAgent: Scaling Language Models to 128k Context (2024)
**arXiv:** [2402.11550](https://arxiv.org/abs/2402.11550)

Zhao et al. scale LLaMA to 128k context through multi-agent collaboration: a leader agent decomposes long documents and coordinates member agents, with an inter-member communication mechanism to resolve hallucination-caused conflicts.

---

## Embodied Agent Memory

### Ghost in the Minecraft (GITM, 2023)
**arXiv:** [2305.17144](https://arxiv.org/abs/2305.17144)

Zhu et al. create generally capable Minecraft agents using LLMs with text-based knowledge and memory — handling hundreds of tasks without GPU-based training, showing memory-driven task decomposition works for long-horizon embodied tasks.

---

## Recent Surveys

### Memory in the Age of AI Agents: A Survey (2025)
**arXiv:** [2512.13564](https://arxiv.org/abs/2512.13564)

Hu, Liu et al. — comprehensive 2025 survey covering memory mechanisms across individual and multi-agent systems, taxonomy of memory operations, evaluation benchmarks, and open research directions.

---

## 2026 Papers

### AgeMem: Unified Long-Term and Short-Term Memory Management via RL (2026)
**arXiv:** [2601.01885](https://arxiv.org/abs/2601.01885) | **Article:** [[papers/agemem]]

Yu et al. train LTM and STM as unified policy actions via RL with three-stage progressive curriculum and step-wise GRPO, outperforming all heuristic-controller baselines on five long-horizon benchmarks.

---

### AutoRefine: Dual-Form Experience Patterns for Continual LLM Agent Refinement (2026)
**arXiv:** [2601.22758](https://arxiv.org/abs/2601.22758) | **Article:** [[papers/autorefine]]

Ye et al. extract procedural subagents and declarative skill patterns from execution trajectories with continuous maintenance (scoring, pruning, merging), exceeding manually-designed systems on TravelPlanner (27.1% vs 12.1%).

---

### MemoryAgentBench: Evaluating Memory in LLM Agents via Incremental Multi-Turn Interactions (2025, updated Mar 2026)
**arXiv:** [2507.05257](https://arxiv.org/abs/2507.05257) | **Article:** [[papers/memory-agent-bench]]

Hu, Wang et al. introduce the first benchmark covering four core memory competencies (accurate retrieval, test-time learning, long-range understanding, selective forgetting), revealing that no current memory agent masters all four simultaneously.

---

## Reading Order Recommendations

**For newcomers:**
1. [[papers/coala]] — get the vocabulary and taxonomy first
2. [[papers/generative-agents]] — see a complete memory system in action
3. [[papers/memgpt]] — understand the key architectural pattern
4. [[papers/reflexion]] — memory as self-improvement

**For practitioners:**
1. [[papers/raise]] — working memory for conversational agents
2. [[papers/hipporag]] — retrieval that goes beyond cosine similarity
3. [[papers/memorybank]] — production-viable long-term memory
4. [[papers/a-mem]] — next-generation dynamic memory

## See Also

- [[research/key-papers]]
- [[research/state-of-the-art-2025]]
- [[architectures/memgpt]]
- [[architectures/rag]]
