---
title: Working Memory in AI Agents
tags: [agent-memory, working-memory, context-window, llm, in-context-learning]
created: 2026-04-04
last_updated: 2026-04-04
---

## The Context Window as Working Memory

Working memory is whatever the model can see right now. Every token in the current context window — the system prompt, conversation history, retrieved documents, tool call results, intermediate reasoning — is working memory. It's the only information the model has *direct, zero-latency access* to.

This is both the most powerful and most constrained type of memory. Powerful because anything in context is immediately available for reasoning without a retrieval step. Constrained because context windows, even million-token ones, have real limits and non-trivial costs.

Understanding working memory isn't just about knowing context window sizes. It's about understanding how models actually use what's in context, and engineering accordingly.

## Context Window Sizes: The Current Landscape

As of 2025, frontier model context windows:

| Model | Context Window |
|-------|---------------|
| GPT-4o | 128K tokens |
| Claude 3.5 Sonnet | 200K tokens |
| Claude 3 Opus | 200K tokens |
| Gemini 1.5 Pro | 1M tokens |
| Gemini 1.5 Flash | 1M tokens |
| Llama 3.1 405B | 128K tokens |

Bigger isn't always better. The "lost in the middle" problem — documented in Liu et al. 2023 — shows that retrieval accuracy for information buried in the middle of long contexts drops significantly. Models tend to attend most strongly to the beginning (primacy) and end (recency) of the context.

## What Belongs in Working Memory

Not everything should be crammed into context. Working memory should be reserved for:

1. **The system prompt** — agent role, personality, constraints, core instructions
2. **Active task context** — the current goal, current state, constraints
3. **Recent conversation history** — enough turns for coherence, not entire history
4. **Freshly retrieved memories** — semantic facts and episodic summaries retrieved for this turn
5. **Tool results** — outputs from recent tool calls needed for current reasoning
6. **Intermediate reasoning** — chain-of-thought, scratch work

What *shouldn't* be in working memory: entire knowledge bases, full conversation history from weeks ago, raw document dumps, or anything that won't affect the next response.

## Strategies for Managing Working Memory

### Conversation Compression
Naive approach: keep all turns in context. This works until the context fills up, then you truncate the oldest turns — losing potentially critical early context.

Better: periodically compress older conversation history into a rolling summary. Keep the last N turns verbatim and replace older history with a compressed summary:

```
[Summary of previous conversation]
Rowan and the agent have been working on a Redis migration plan since March 15. 
Key decisions made: use slot-based migration, target 2am maintenance window, 
Rowan owns the runbook draft.

[Recent turns - verbatim]
Rowan: Can you draft the rollback procedure?
Agent: Sure, here's a rollback plan...
```

### Hierarchical Context Management
MemGPT formalizes this into a paging system with explicit in-context and out-of-context storage. The agent has a function to load/unload memory blocks, analogous to virtual memory paging. See [[architectures/memgpt]].

### Dynamic Retrieval
Rather than pre-loading memory, retrieve on demand. Each turn, retrieve the most relevant memories from external storage and inject them. This keeps working memory lean while making relevant knowledge available. See [[architectures/rag]] and [[concepts/memory-retrieval]].

### Structured Working Memory Slots
Instead of free-form context, assign structured slots:
- **Goal**: current task objective
- **Plan**: current plan steps
- **State**: what's been completed
- **Relevant context**: key retrieved facts
- **Recent history**: last 5 turns

This reduces the cognitive load on the model and makes context management more predictable.

### Tool Call Buffering
In agentic loops with many tool calls, results accumulate quickly. Strategies:
- Summarize tool results after each call ("retrieved 20 documents, key finding: X")
- Clear intermediate results once incorporated into reasoning
- Store detailed results externally and keep only references in context

## The Attention Distribution Problem

Attention in transformer models isn't uniform over context. Research shows:

- **Primacy bias**: early context gets disproportionate attention
- **Recency bias**: recent context gets strong attention
- **Middle neglect**: content in the middle of very long contexts is under-attended

Practical implications:
- Put your most important instructions at the **beginning** of the system prompt
- Put time-sensitive, immediately-relevant context **at the end** (just before the current turn)
- Don't rely on critical information being reliably retrieved from the middle of a million-token context

## Working Memory vs. External Memory: The Tradeoff

| Dimension | Working Memory | External Memory |
|-----------|---------------|-----------------|
| Access latency | Zero | 50-500ms |
| Relevance | Manual/curated | Retrieved (imperfect) |
| Capacity | Hard limit | Effectively unlimited |
| Cost | Per-token pricing | Storage + retrieval cost |
| Reliability | Perfect (if in context) | Depends on retrieval quality |

The right balance is system-specific. High-stakes, low-context tasks (quick Q&A) can work with minimal working memory. Long-horizon agentic tasks with rich history need robust external memory with smart retrieval.

## Key Takeaways

- Working memory = the context window; everything the model sees right now
- "Lost in the middle" is real: don't bury critical information in the middle of long contexts
- Compress conversation history; don't accumulate indefinitely
- Structure your context intentionally — slots beat free-form accumulation
- Dynamic retrieval keeps working memory lean without sacrificing relevance
- The goal is maximum signal density per token in context

## See Also

- [[types-of-memory]]
- [[long-term-memory]]
- [[memory-retrieval]]
- [[architectures/memgpt]]
- [[architectures/rag]]
