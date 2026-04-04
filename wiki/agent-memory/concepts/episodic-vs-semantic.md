---
title: Episodic vs. Semantic Memory
tags: [agent-memory, episodic, semantic, memory-types, implementation]
created: 2026-04-04
last_updated: 2026-04-04
---

## The Core Distinction

Episodic and semantic memory are the two most practically important memory types for production AI agents, and they're often confused. The distinction is simple but consequential:

- **Episodic**: *What happened.* Grounded in time, context, and specific events. "On Tuesday, the user mentioned they were refactoring the authentication service."
- **Semantic**: *What is true.* Abstracted from events. "The user is working on an authentication service refactor."

The episodic record is the raw input. The semantic fact is the derived output. Good memory systems build semantic knowledge *from* episodic records and then decide how long to keep the raw source.

In cognitive neuroscience, the hippocampus handles episodic consolidation — replaying experiences during sleep to extract generalizations into the neocortex. Agent memory architectures increasingly mirror this: a background process periodically reviews episodic logs and extracts or updates semantic facts.

## Why the Distinction Matters in Practice

Treating all memory as one undifferentiated blob is a common mistake. It leads to:

1. **Retrieval noise** — retrieving stale, contradictory, or low-signal episodic events when you needed a clean semantic fact
2. **Storage explosion** — keeping raw logs forever when only the distilled insight matters
3. **Temporal confusion** — the system can't tell whether "user prefers dark mode" is a current belief or an old note that's since been overridden

Separating the two allows for different storage strategies, different retrieval mechanisms, and different update logic.

## Episodic Memory: Implementation Patterns

### Raw Conversation Storage
The simplest episodic implementation: store every turn of every conversation with timestamps. Useful for audit trails and debugging. Poor for retrieval at scale — too much noise.

### Session Summarization
After each session ends, a background process runs a summarization prompt over the raw transcript. The summary is stored as an episodic record. Retrieval is over summaries rather than raw turns.

```
Episode: 2026-03-15 session with Rowan
Summary: Discussed Redis cluster migration. User concerned about downtime window. 
Decided to use slot-based migration. Next step: draft migration runbook.
Key entities: Rowan, Redis, slot migration, migration runbook
```

### Event Extraction
More structured: extract discrete events with entity tags, timestamps, and sentiment/importance scores. Enables fine-grained temporal queries ("what did we discuss about Redis in the last 30 days?").

## Semantic Memory: Implementation Patterns

### Structured Key-Value Facts
Store beliefs about entities as structured data:
```json
{
  "entity": "Rowan",
  "fact": "prefers Python over JavaScript",
  "confidence": 0.9,
  "source_episodes": ["ep_20260315", "ep_20260318"],
  "last_updated": "2026-03-18"
}
```

### Vector-Indexed Knowledge
Embed semantic facts and store in a vector DB. Retrieval is by content similarity — useful when you don't know the exact entity but can describe what you're looking for contextually.

### Knowledge Graphs
Store entities and typed relationships: `Rowan --PREFERS--> Python`. Enables multi-hop queries that vectors handle poorly. See [[architectures/knowledge-graphs]].

## The Consolidation Pipeline

The most sophisticated agent memory systems run a consolidation process — analogous to sleep consolidation in humans:

1. **Accumulate** episodic records during active operation
2. **Review** recent episodes in a background process
3. **Extract** candidate semantic facts using an LLM
4. **Deduplicate and merge** with existing semantic memory, resolving conflicts
5. **Archive or delete** raw episodic records based on retention policy

This pipeline is at the core of systems like [[architectures/mem0]], which automates steps 3-4 with an extraction LLM and a graph-based conflict resolver.

## Temporal Semantics: The Hard Part

The trickiest part of semantic memory is handling time correctly. Facts change. "Rowan is working on an authentication refactor" was true in March but false in April. Systems that don't track the *currency* of semantic facts produce stale, misleading retrievals.

Three approaches:
1. **Timestamps + decay** — facts get confidence-weighted by age; old facts fade
2. **Explicit contradiction detection** — new facts are checked against existing ones; conflicts trigger a resolution step
3. **Event sourcing** — never mutate semantic facts; append new events and derive current state on-the-fly

Approach 3 is the most robust but most expensive. Most production systems use approach 1 or 2.

## Key Takeaways

- Episodic memory = specific events with temporal context; semantic memory = distilled, generalized facts
- Don't store everything as one undifferentiated blob — the retrieval quality degrades
- Build semantic facts *from* episodic records via a consolidation process
- Temporal semantics are the hardest part: track when facts were true, not just whether they are
- Systems like mem0 and MemGPT automate consolidation to varying degrees

## See Also

- [[types-of-memory]]
- [[working-memory]]
- [[long-term-memory]]
- [[memory-retrieval]]
- [[architectures/mem0]]
- [[architectures/knowledge-graphs]]
