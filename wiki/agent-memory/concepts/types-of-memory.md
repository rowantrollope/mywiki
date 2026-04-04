---
title: Types of Agent Memory
tags: [agent-memory, taxonomy, episodic, semantic, procedural, working-memory]
created: 2026-04-04
last_updated: 2026-04-04
---

## Taxonomy of Agent Memory

The dominant framework for classifying AI agent memory borrows from cognitive psychology's taxonomy of human memory. While the mapping isn't perfect, it's operationally useful: it tells you *where* to store different kinds of information and *how* to retrieve it.

## Working Memory

Working memory is the agent's active scratchpad — the context window. It holds the current conversation, the task being executed, retrieved documents, and any intermediate reasoning steps. It is fast, directly accessible, and temporary.

Modern LLMs have context windows ranging from 8K to 1M+ tokens, but effective working memory is usually far smaller due to attention degradation in long contexts (the "lost in the middle" problem). Critically, working memory is not persistent — when the context ends, everything in it is gone unless explicitly saved.

Key properties:
- Directly readable by the model (no retrieval step)
- Limited capacity
- Volatile (lost at end of session)
- Includes both the prompt and prior turns

See [[working-memory]] for a deep dive on strategies for managing context effectively.

## Episodic Memory

Episodic memory stores records of specific events: what happened, when, and in what context. In human cognition, episodic memory is autobiographical — "I remember the meeting last Tuesday where we decided on the API schema."

In agents, episodic memory is typically implemented as:
- **Raw conversation logs** — full interaction transcripts
- **Event summaries** — compressed representations of what happened in a session
- **Action traces** — records of tool calls, decisions, and their outcomes

Episodic memory is time-indexed and often retrieved by recency or temporal proximity. It's the foundation of continuity: the agent that "remembers" your last interaction is drawing on episodic memory.

The main challenge is storage cost and noise. Raw logs balloon quickly. Most production systems compress episodic records into summaries after a cooldown period.

## Semantic Memory

Semantic memory stores generalized, context-free facts and knowledge. Unlike episodic memory ("on March 3rd the user said they prefer Python"), semantic memory holds the distilled inference ("user prefers Python"). The specific event that generated this belief is stripped away; only the fact remains.

In agents, semantic memory is typically implemented as:
- **Key-value stores** — structured facts about entities (users, projects, preferences)
- **Vector stores** — embedding-indexed knowledge bases
- **Knowledge graphs** — entities and typed relationships

Semantic memory is retrieved by content similarity or entity lookup, not by time. It's the primary substrate for personalization — an agent knowing your preferences, your team structure, your project context.

The challenge with semantic memory is **coherence under update**: when new information contradicts existing beliefs, the system needs to detect the conflict and resolve it. See [[concepts/long-term-memory]] and [[research/open-problems]].

## Procedural Memory

Procedural memory encodes *how to do things* — skills, workflows, and behaviors. In human cognition, this is implicit and automatic (riding a bike). In agents, it's more explicit: stored prompts, tool-use patterns, successful strategies, and fine-tuned model weights.

Implementations include:
- **System prompt refinements** — lessons learned encoded into the agent's instructions
- **Tool usage patterns** — cached examples of successful tool call sequences
- **Fine-tuned weights** — when learning is baked into the model itself (less common in production)
- **Skill libraries** — reusable sub-agent routines for common tasks

Procedural memory is typically the *least* dynamic at runtime — it's updated through deliberate learning cycles, not on every interaction. It's also the hardest to inspect and debug.

## Memory in Practice: A Layered View

Most real agent systems use all four types simultaneously:

| Memory Type | Storage | Retrieval | Update Frequency |
|-------------|---------|-----------|-----------------|
| Working | Context window | Direct | Every turn |
| Episodic | Log DB / vector store | Recency + similarity | Every session |
| Semantic | KV store / graph | Entity lookup + similarity | When beliefs change |
| Procedural | System prompt / weights | Implicit | Deliberate learning cycles |

The art is in knowing what to promote between layers: which episodic events become semantic facts, which semantic facts get promoted to procedural rules.

## Key Takeaways

- **Working memory** is the context window — fast, limited, volatile
- **Episodic memory** records specific events — the basis of continuity
- **Semantic memory** holds generalized facts — the basis of personalization
- **Procedural memory** encodes skills and behaviors — the hardest to update
- Real agents layer all four; the challenge is managing promotion and eviction between layers

## See Also

- [[overview]]
- [[episodic-vs-semantic]]
- [[working-memory]]
- [[long-term-memory]]
- [[memory-retrieval]]
