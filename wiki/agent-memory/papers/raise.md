---
title: "RAISE: Reasoning and Acting through Scratchpad and Examples"
tags: [agent-memory, papers, arxiv, raise, scratchpad, working-memory, react, conversational-agents]
created: 2026-04-04
last_updated: 2026-04-04
---

## Overview

**Paper:** From LLM to Conversational Agent: A Memory Enhanced Architecture with Fine-Tuning of Large Language Models
**Short name:** RAISE (Reasoning and Acting through Scratchpad and Examples)
**Authors:** Liangyu Chen et al.
**arXiv:** [2401.02777](https://arxiv.org/abs/2401.02777) — January 2024
**Base framework:** Extension of ReAct

## The Core Idea

RAISE enhances the ReAct agent framework with a **dual-component memory system** that mirrors human short-term and long-term memory. The fundamental problem it addresses: ReAct agents are stateless within a conversation — they don't maintain a working scratchpad of intermediate reasoning, and they don't have access to relevant past examples. RAISE adds both.

## The Four-Component Working Memory

RAISE defines **working memory** with four distinct components:

### Dialogue-Level Components (Persist across turns)
1. **Conversation History:** Full log of the current multi-turn dialogue. Standard for conversational agents but explicitly managed as a memory component.
2. **Scratchpad:** A running workspace for the agent's intermediate reasoning, plans, and observations during the current session. Unlike the conversation history (which captures the agent-user exchange), the scratchpad is the agent's private workspace.

### Turn-Level Components (Reset each turn)
3. **Task Trajectory:** The sequence of actions and observations for the current task. Tracks what the agent has done and observed in the current turn.
4. **Examples (Few-Shot):** Retrieved relevant examples from a few-shot example bank that are injected into context for the current turn.

This taxonomy — especially the distinction between conversation history (dialogue-level) and task trajectory (turn-level), and between public dialogue and private scratchpad — clarifies the different roles of working memory components in a conversational agent.

## Agent Construction Pipeline

Beyond the memory architecture, RAISE proposes a **comprehensive agent construction scenario** for building and fine-tuning conversational agents:

1. **Conversation Selection:** Identify high-quality example conversations from production data
2. **Scene Extraction:** Extract the key scenarios, intents, and contexts from selected conversations
3. **CoT Completion:** Use a capable LLM to generate chain-of-thought reasoning traces explaining each agent action
4. **Scene Augmentation:** Augment the training data with diverse variations of each scene
5. **LLM Training:** Fine-tune the base LLM on the resulting high-quality annotated conversations

This pipeline creates a virtuous cycle: the fine-tuned model is better at reasoning through its scratchpad, which creates better trajectories, which can be used for further fine-tuning.

## The Scratchpad as Working Memory

The **scratchpad** is RAISE's most conceptually interesting contribution. In standard ReAct, the agent's "Thought" field is ephemeral — it appears once and is consumed. RAISE makes the scratchpad **persistent and accumulating** within a session.

The scratchpad can contain:
- Current task decomposition
- Partial results from tool calls
- Hypotheses the agent is testing
- Notes about what has been tried and failed
- Plans for upcoming steps

This gives the agent a genuine working memory — a space for thinking-in-progress that accumulates over the session rather than resetting each turn.

## Dual Memory System

RAISE's explicit design mirrors cognitive science's short-term / long-term distinction:

| Component | Memory Type | Scope | Persistence |
|-----------|-------------|-------|-------------|
| Scratchpad | Short-term working memory | Session | Resets after session |
| Conversation history | Short-term episodic | Session | Resets after session |
| Few-shot example bank | Long-term semantic | Cross-session | Persistent |
| Fine-tuned weights | Long-term procedural | Permanent | Permanent |

## Application: Real Estate Sales Agent

RAISE was evaluated in a **real estate sales context** — a domain requiring:
- Multi-turn conversations spanning multiple sessions
- Domain knowledge lookup (property details, pricing)
- Context continuity (remembering client preferences across turns)
- Adaptable communication style

Results show RAISE advantages over traditional conversational agents in controllability and adaptability in complex multi-turn scenarios.

## Key Takeaways

- **Four-component working memory:** conversation history + scratchpad (dialogue-level) + task trajectory + examples (turn-level)
- **Scratchpad as persistent workspace:** accumulates intermediate reasoning throughout a session
- **Dual memory analogy:** scratchpad/history = short-term; few-shot bank + fine-tuned weights = long-term
- **Agent construction pipeline:** Conversation Selection → Scene Extraction → CoT Completion → Augmentation → Fine-tuning
- **ReAct extension:** builds on the ReAct framework by giving the agent richer working memory

## Limitations

- Scratchpad grows throughout a session and can overflow the context window
- Few-shot example retrieval quality depends on example bank curation
- The construction pipeline requires significant human annotation effort
- Evaluated only in one domain (real estate); generalization is unclear

## See Also

- [[papers/reflexion]]
- [[papers/memgpt]]
- [[concepts/working-memory]]
- [[concepts/episodic-vs-semantic]]
- [[research/key-papers]]
