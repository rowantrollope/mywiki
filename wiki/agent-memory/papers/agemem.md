---
title: "AgeMem: Learning Unified Long-Term and Short-Term Memory Management for LLM Agents"
tags: [agent-memory, papers, arxiv, 2026, reinforcement-learning, memory-management]
arxiv: "2601.01885"
created: 2026-04-04
last_updated: 2026-04-04
---

## Overview

LLM agents working on long-horizon tasks face a fundamental tension: their context windows are finite, yet the information required to solve complex, extended tasks is not. Most prior systems deal with this by treating long-term memory (LTM) and short-term memory (STM) as separate components with separate controllers — the LTM module handles persistence, the STM module handles in-context working space, and a heuristic coordinator decides what moves where.

AgeMem (Agentic Memory) challenges this separation. The core argument: if you want the agent to use memory *well*, the memory operations need to be part of the agent's policy — not a bolted-on subsystem. An agent that learns to store, retrieve, summarize, and discard memories as first-class actions can optimize those operations end-to-end toward task success.

---

## Key Contributions

- **Unified LTM/STM framework:** Rather than separate components with separate controllers, AgeMem exposes all memory operations (store, retrieve, update, summarize, discard) as tool-based actions that the LLM agent calls directly — integrating memory management into the main policy loop.
- **Three-stage progressive reinforcement learning:** A training curriculum that incrementally introduces complexity: (1) basic retrieval, (2) write/update operations, (3) lifecycle management (summarization and forgetting). This prevents the agent from skipping difficult memory operations early in training.
- **Step-wise GRPO:** Standard RL reward signals for memory are sparse and discontinuous — memory written in step 3 may only pay off in step 47. The paper introduces a step-wise Group Relative Policy Optimization variant that assigns credit to individual memory operations based on their downstream effect.
- **Benchmark breadth:** Evaluated across five long-horizon benchmarks covering QA, multi-document reasoning, and sequential decision-making, with multiple LLM backbone models.

---

## Architecture / Method

The AgeMem agent sees memory operations as tools in its action space — analogous to web search or code execution. At each step, the agent can:

- **STORE(content, key):** Write to long-term memory with a retrieval key
- **RETRIEVE(query):** Fetch from long-term memory by semantic similarity
- **UPDATE(key, content):** Revise existing memory entries
- **SUMMARIZE(scope):** Compress a set of short-term observations into a compact long-term note
- **DISCARD(key):** Remove stale or irrelevant memories

Training uses the three-stage curriculum:
1. **Stage 1 — Retrieval only:** The agent learns when and what to retrieve. LTM is pre-populated; the agent just retrieves.
2. **Stage 2 — Read/Write:** The agent both retrieves and writes memories. Learns to store task-relevant information for later use.
3. **Stage 3 — Full lifecycle:** The agent manages the entire lifecycle: stores, updates, summarizes, and discards. Step-wise GRPO assigns per-operation credit.

The step-wise GRPO adapts standard GRPO by computing group-relative rewards at individual decision steps rather than episode boundaries, allowing credit assignment for memory operations whose payoff is temporally distant.

---

## Results

AgeMem consistently outperforms memory-augmented baselines across all five long-horizon benchmarks:

- Outperforms both heuristic-controller baselines and separate LTM/STM approaches
- Achieves higher task completion rates with *fewer* total context tokens — demonstrating more efficient context utilization
- Improvement is consistent across different LLM backbones (not backbone-specific)
- Long-term memory quality (measured by retrieval precision and coherence) improves vs. baselines

Specific benchmark numbers are available in the full paper ([arXiv:2601.01885](https://arxiv.org/abs/2601.01885)).

---

## Key Takeaways

- **End-to-end beats modular for memory:** Training memory operations as part of the agent policy outperforms heuristic coordination between separate LTM and STM modules. The agent learns *when* memory operations matter, not just *how* to execute them.
- **Curriculum matters for RL-trained memory:** Introducing lifecycle complexity progressively (retrieve → write → lifecycle) is essential. Attempting full lifecycle training from scratch leads to instability.
- **Step-wise credit assignment is the crux:** Sparse rewards from temporally-delayed memory payoffs are the core challenge in memory RL. GRPO adapted for per-step credit solves this practically. This technique is likely to transfer beyond memory to other long-horizon RL problems.

---

## See Also

- [[papers/autorefine]] — complementary work on extracting reusable expertise from trajectories
- [[papers/memgpt]] — the original framework for agent-driven memory operations as tool calls
- [[papers/reflexion]] — using episodic memory for self-improvement via verbal RL
- [[concepts/memory-consolidation]] — the consolidation problem AgeMem addresses via summarize/discard
- [[research/state-of-the-art-2025]] — broader context of the 2025-2026 memory research frontier
