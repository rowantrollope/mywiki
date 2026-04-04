---
title: Long-Term Memory in AI Agents
tags: [agent-memory, long-term-memory, external-storage, compression, forgetting]
created: 2026-04-04
last_updated: 2026-04-04
---

## What Is Long-Term Memory for Agents?

Long-term memory is everything the agent stores *outside* the context window and can retrieve in future sessions. It's the mechanism for continuity across time — the difference between an agent that treats every conversation as its first and one that actually knows you.

Unlike working memory (immediate, volatile, capacity-constrained), long-term memory is:
- **Persistent** — survives session restarts, model updates, and infrastructure changes
- **Scalable** — can grow to millions of records without breaking retrieval
- **Queryable** — retrieved by content, entity, time, or relevance rather than direct access

The implementation is almost always a combination of external storage systems: vector databases for semantic search, relational or document databases for structured facts, and increasingly knowledge graphs for relational queries.

## What to Store in Long-Term Memory

Not everything deserves long-term storage. The signal-to-noise ratio of your memory store determines retrieval quality. Storing indiscriminately is the fastest path to a degraded, noisy system.

**High value for long-term storage:**
- User preferences and behavioral patterns
- Key decisions and their rationale
- Factual knowledge specific to the user/domain
- Successful strategies and patterns (procedural)
- Important relationships between entities
- Commitments, agreements, and follow-ups

**Low value (often not worth storing):**
- Routine pleasantries and filler conversation
- Information that's trivially available elsewhere (public knowledge)
- Transient context that won't matter next session
- Intermediate reasoning steps that led nowhere
- Duplicate or redundant facts already captured

## Storage Architecture Patterns

### Vector Store (Primary Layer)
The most common long-term memory substrate for modern agents. Text (or structured data) is embedded into a high-dimensional vector and stored in a vector database. Retrieval is by nearest-neighbor similarity to a query embedding.

Good for: semantic search, when you know roughly what you're looking for but not the exact key.
Poor for: multi-hop relational queries, strict consistency requirements.

See [[architectures/vector-stores]] for implementation details.

### Relational/Document DB (Structured Facts Layer)
Structured semantic facts stored in a traditional database. Supports precise lookups ("what is Rowan's preferred language?"), filtering, and joins. More rigid schema but more predictable retrieval.

```sql
SELECT fact_value FROM user_facts 
WHERE entity = 'rowan' AND fact_type = 'preference' 
ORDER BY updated_at DESC LIMIT 1;
```

### Knowledge Graph (Relational Layer)
Stores entities and typed relationships. Enables multi-hop queries that flat vector retrieval handles poorly: "What projects involve team members who have worked with Rowan on Redis?"

See [[architectures/knowledge-graphs]].

### Hierarchical Log Store (Episodic Layer)
Raw session logs and event records. Time-indexed. Used for audit trails and as the source material for semantic extraction.

## Compression: The Critical Problem

Raw episodic storage grows without bound. The solution is multi-level compression:

### Level 1: Session Summarization
After each session, compress the raw transcript into a structured summary:
- Key topics discussed
- Decisions made
- Follow-up actions
- New facts learned about entities

### Level 2: Periodic Consolidation
Weekly or monthly: review session summaries, extract persistent semantic facts, update the knowledge store, and archive or delete redundant raw records.

### Level 3: Semantic Distillation
Abstract from facts to patterns: "Rowan consistently prefers brevity in responses" is more valuable than 50 individual instances of asking for shorter answers.

This mirrors human memory consolidation — the hippocampus replays and compresses episodic memories during sleep, extracting patterns into neocortical long-term storage.

## What to Forget: The Retention Policy

Memory systems need explicit forgetting policies. Without them, stale, contradictory, and low-value memories accumulate and degrade retrieval quality.

**Time-based decay:**
Facts get a confidence weight that decreases over time. A preference stated 18 months ago is less reliable than one stated last week.

**Supersession:**
When a new fact contradicts an old one about the same entity+attribute, the old fact is marked superseded rather than deleted (for audit trail).

**Relevance pruning:**
Facts that have never been retrieved in N months are candidates for archival or deletion. If nothing ever finds it relevant, it's probably not.

**Explicit forgetting:**
Allow users and agents to explicitly mark facts for deletion. "I no longer work at that company" should immediately suppress related facts from retrieval.

## The Coherence Challenge

Long-term memory accrues from many sources over time. Contradictions are inevitable. Managing coherence means:

1. **Detecting conflicts** — new facts are checked against existing beliefs about the same entity
2. **Resolving conflicts** — usually by recency (newer wins) but sometimes by confidence or source reliability
3. **Versioning** — keeping fact history so the agent can reason about change over time ("you used to prefer X, but recently switched to Y")

This is one of the hardest open problems. See [[research/open-problems]].

## Key Takeaways

- Long-term memory = everything stored outside the context window, persistent across sessions
- Be selective: storing everything degrades retrieval quality; filter for high-signal information
- Use a layered architecture: vector store for semantic search, structured DB for facts, graph for relations
- Compress episodic records aggressively — summarize sessions, consolidate periodically
- Implement explicit forgetting: time decay, supersession, relevance pruning
- Coherence under update is the hardest part — fact conflicts need detection and resolution

## See Also

- [[types-of-memory]]
- [[working-memory]]
- [[episodic-vs-semantic]]
- [[memory-retrieval]]
- [[architectures/vector-stores]]
- [[architectures/knowledge-graphs]]
- [[research/open-problems]]
