---
title: "MemoryBank: Enhancing LLMs with Long-Term Memory"
tags: [agent-memory, papers, arxiv, memorybank, long-term-memory, forgetting, ebbinghaus, personalization]
created: 2026-04-04
last_updated: 2026-04-04
---

## Overview

**Paper:** MemoryBank: Enhancing Large Language Models with Long-Term Memory
**Authors:** Wanjun Zhong et al.
**arXiv:** [2305.10250](https://arxiv.org/abs/2305.10250) — May 2023
**Application:** SiliconFriend — an LLM-based long-term AI companion

## The Core Problem

Standard LLMs have no memory between sessions. Every conversation starts fresh. This fundamentally limits applications that require sustained interaction: personal companions, mental health support, personal assistants that learn preferences over time. Users must re-introduce themselves every session; the system can't build on past interactions; personalization is impossible.

MemoryBank proposes a memory mechanism tailored specifically for **long-term LLM interactions** that enables models to:
- Remember relevant past experiences
- Continuously update memory based on new interactions
- Understand and adapt to individual user personality
- Selectively forget less important information (like humans do)

## The Ebbinghaus Forgetting Curve

The most distinctive feature of MemoryBank is its **biologically-inspired memory forgetting mechanism** based on Ebbinghaus's forgetting curve theory.

Hermann Ebbinghaus (1885) discovered that memory retention follows a predictable exponential decay:

```
R = e^(-t/S)
```

Where R is retention, t is time elapsed, and S is the memory's "strength" (a function of how many times it's been recalled).

MemoryBank operationalizes this:
- Each memory has a **strength score** that increases when recalled
- Strength decays exponentially with time since last recall
- Memories with strength below a threshold are "forgotten" (pruned from active memory)
- Frequently recalled memories are reinforced (like spaced repetition)

This creates human-like memory dynamics: important, frequently-accessed memories persist; rarely-accessed memories fade. The result is a self-managing memory bank that doesn't grow indefinitely.

## Architecture

MemoryBank has three core components:

### Memory Storage
Memories are stored as structured records containing:
- The memory content (natural language summary of an event or fact)
- Timestamp
- Strength score (updated on access)
- User-related metadata

### Memory Retrieval
Retrieval combines:
- **Recency:** recent memories score higher
- **Relevance:** semantic similarity to current query
- **Strength:** memories reinforced through repeated access score higher

### Memory Update
After each conversation turn:
- New memories extracted from the interaction are written to storage
- Accessed memories have their strength reinforced
- Decayed memories (below strength threshold) are pruned

### Personality Understanding
MemoryBank also maintains a **user profile** built by summarizing patterns across episodic memories. This enables the system to understand user personality traits, preferences, and communication style — and adapt responses accordingly.

## SiliconFriend Application

The paper demonstrates MemoryBank through **SiliconFriend**, an LLM-based companion application focused on emotional support and long-term relationship building. Key features:
- Fine-tuned on psychological dialogue datasets for empathy
- Uses MemoryBank for persistent user context across sessions
- Adapts communication style based on user personality profile

Evaluation: ChatGPT acts as users with diverse personalities, generating long-term dialogue contexts. SiliconFriend with MemoryBank shows stronger long-term consistency and more contextually appropriate responses than baseline systems.

## Compatibility

MemoryBank is designed to work with **both closed-source and open-source models**:
- Closed-source: ChatGPT (via API)
- Open-source: ChatGLM

This separation of memory mechanism from the underlying LLM is a key design principle — MemoryBank wraps any LLM with persistent memory capabilities.

## Comparison to Related Work

| Feature | MemGPT | Generative Agents | MemoryBank |
|---------|--------|-------------------|------------|
| Forgetting mechanism | None | None | Ebbinghaus decay |
| Personality modeling | No | Limited | Yes |
| Application focus | General | Social simulation | Long-term companion |
| Memory reinforcement | No | Importance score | Strength + recall reinforcement |

## Why This Matters

1. **Principled forgetting:** Most agent memory systems grow indefinitely. MemoryBank provides the first biologically-grounded forgetting mechanism for LLM agents.
2. **Personalization:** The personality modeling layer enables genuinely adaptive agents.
3. **Production viability:** Works with closed-source APIs — no need to modify the LLM.
4. **Long-term interactions:** Designed specifically for the multi-session, multi-week use case.

## Key Takeaways

- **Ebbinghaus forgetting curve:** memory strength decays over time; recalled memories are reinforced — creates human-like memory dynamics
- **Selective forgetting:** memories below strength threshold are pruned — prevents unbounded growth
- **Personality modeling:** builds user profiles from episodic patterns to enable adaptive responses
- **Plug-and-play:** works with any LLM via API (ChatGPT, ChatGLM)
- **Application-driven:** designed for the long-term companion use case where persistent personalization is essential

## See Also

- [[papers/memgpt]]
- [[papers/generative-agents]]
- [[concepts/forgetting-mechanisms]]
- [[concepts/long-term-memory]]
- [[research/key-papers]]
