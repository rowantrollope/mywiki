---
title: "CoALA: Cognitive Architectures for Language Agents"
tags: [agent-memory, papers, arxiv, coala, cognitive-architecture, taxonomy, memory-types]
created: 2026-04-04
last_updated: 2026-04-04
---

## Overview

**Paper:** Cognitive Architectures for Language Agents
**Authors:** Theodore R. Sumers, Shunyu Yao, Karthik Narasimhan, Thomas L. Griffiths (Princeton)
**arXiv:** [2309.02427](https://arxiv.org/abs/2309.02427) — September 2023, published in TMLR
**GitHub:** [awesome-language-agents](https://github.com/ysymyth/awesome-language-agents)

## The Core Idea

Before CoALA, the LLM agent field had a terminology problem: different papers used "memory," "context," "state," and "knowledge" inconsistently. CoALA provides a **unified cognitive architecture framework** that maps language agent components to well-established concepts from cognitive science and symbolic AI — giving the field a shared vocabulary.

The paper draws on classical cognitive architectures (SOAR, ACT-R) and cognitive science's memory taxonomy to build a structured framework for understanding, comparing, and designing language agents.

## The CoALA Framework

CoALA describes every language agent along three dimensions:

### 1. Memory Components

CoALA maps cognitive science's memory taxonomy to LLM agent architecture:

| Cognitive Science Term | CoALA Definition | LLM Implementation |
|----------------------|-----------------|-------------------|
| **Working memory** | Information held in active context during reasoning | The context window; current observations and intermediate thoughts |
| **Episodic memory** | Record of specific past events and experiences | Conversation logs, interaction history, experience buffers |
| **Semantic memory** | General world knowledge, facts, concepts | Knowledge bases, RAG corpora, pre-trained model weights |
| **Procedural memory** | Knowledge of how to perform tasks; skills | System prompts, fine-tuned weights, learned action sequences |

This four-way taxonomy has become the field's standard vocabulary. When someone says "episodic memory for an LLM agent" they almost certainly mean the CoALA definition.

### 2. Action Space

Agents act in one of four domains:
- **Memory actions:** Read/write to working, episodic, semantic, or procedural memory
- **Execution actions:** Execute code, call APIs, interact with tools
- **Grounding actions:** Interact with external world (browse web, search databases, use sensors)
- **Reasoning actions:** Internal chain-of-thought, deliberation, planning

### 3. Decision-Making Process

CoALA models the agent's decision cycle as:
1. **Observe:** Update working memory with new inputs
2. **Retrieve:** Pull relevant memories from long-term stores
3. **Reason:** Deliberate over available information
4. **Act:** Execute chosen action
5. **Learn:** Update long-term memory stores

## How CoALA Organizes Existing Work

One of CoALA's most useful contributions is retrospectively organizing the landscape of existing agent papers:

| System | Working Memory | Episodic Memory | Semantic Memory | Procedural Memory |
|--------|---------------|----------------|----------------|------------------|
| **MemGPT** | Context window | Archival + recall storage | External DB | System prompt |
| **Generative Agents** | Context window | Memory stream | Reflection memories | Planning prompts |
| **ReAct** | Context window + scratchpad | None (single-session) | Tool descriptions | ReAct prompt template |
| **Reflexion** | Context window | Episodic memory buffer | None | Updated policy via reflection |
| **AutoGPT** | Context window | Task history | Web search | Task-specific prompts |

## Prospective Contributions

Beyond organizing existing work, CoALA identifies gaps:
- Most systems have no **semantic memory consolidation** (learning general facts from experience)
- **Procedural memory updates** (actually getting better at tasks) are rare
- **Cross-agent memory sharing** is nearly unexplored
- Memory lifecycle (creation, update, forgetting) is underspecified

## Why This Matters

1. **Shared vocabulary:** The CoALA taxonomy eliminated confusion about what "memory" means in agent systems. Papers published after CoALA use its terms consistently.
2. **Gap identification:** By mapping the space, CoALA showed which combinations of memory components were underexplored.
3. **Design tool:** The framework helps practitioners think through what their agent needs — "do I need episodic or semantic memory? both?"
4. **Bridge to cognitive science:** Grounds LLM agent design in 50+ years of cognitive architecture research (SOAR, ACT-R, CLARION)

## Limitations

- The taxonomy is useful but not perfect — the boundaries between memory types can be blurry in practice
- The framework describes architectures but doesn't prescribe how to implement them
- Does not cover multi-agent memory sharing or distributed architectures

## Key Takeaways

- **Four memory types:** working (context), episodic (experiences), semantic (knowledge), procedural (skills)
- **Action space taxonomy:** memory / execution / grounding / reasoning actions
- **Unified vocabulary** for the entire field of language agents
- **Gap analysis:** most agents lack semantic consolidation and procedural learning
- **Cognitive science bridge:** connects LLM agent design to ACT-R, SOAR, and 50 years of cognitive architecture research

## See Also

- [[papers/memgpt]]
- [[papers/generative-agents]]
- [[concepts/types-of-memory]]
- [[concepts/episodic-vs-semantic]]
- [[research/key-papers]]
