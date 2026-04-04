---
title: Memory Retrieval in AI Agents
tags: [agent-memory, retrieval, similarity-search, recency, importance-scoring, rag]
created: 2026-04-04
last_updated: 2026-04-04
---

## The Retrieval Problem

Having a memory store is table stakes. The real challenge is retrieval: given the current context, which memories are relevant *right now*? Poor retrieval is as bad as no memory — the agent surfaces stale, irrelevant, or contradictory information and acts on it.

Retrieval is where most agent memory systems fail in practice. The embedding-based semantic search that works beautifully in demos degrades when the memory store grows to millions of entries, when queries are ambiguous, or when the most important memory is recent but semantically distant from the query.

## The Three Signals

Most sophisticated retrieval systems combine three signals:

### 1. Semantic Similarity
The workhorse of modern retrieval. The query and each stored memory are embedded into high-dimensional vectors; retrieval is by nearest-neighbor search (cosine similarity or dot product).

**Strengths:** Works for paraphrase, synonyms, and conceptual similarity. No exact keyword match required.
**Weaknesses:** Embedding models have limited context; complex multi-sentence facts can embed poorly. Two semantically similar memories may have very different relevance. Similarity ≠ relevance.

### 2. Recency
Recent memories are often more relevant than older ones. A preference stated yesterday likely supersedes one from six months ago. A task completed last session is probably more relevant than one from three months ago.

Recency can be weighted additively:
```
score = α * semantic_similarity + β * recency_score
recency_score = exp(-λ * days_since_created)
```

The decay rate `λ` is task-specific. For volatile preferences, high decay. For stable facts (name, timezone), near-zero decay.

### 3. Importance
Not all memories are equally important. A memory tagged as "critical decision" should score higher than one tagged "casual mention." Importance can be assigned:
- At write time by the agent ("this seems important, flag it")
- At write time by user action ("remember this")
- At read time by access frequency (retrieved many times = probably important)
- By LLM-based importance scoring at write time

The Generative Agents paper (Park et al., 2023) formalized this as a weighted combination:
```
retrieval_score = α*recency + β*importance + γ*relevance
```
This triplet has become something of a standard reference in the field.

## Retrieval Modes

### Query-Time Retrieval
The most common pattern. On each agent turn, before generating a response, run a retrieval query against the memory store using the current message (and possibly conversation context) as the query. Inject top-K results into the context window.

Straightforward but has latency cost and requires careful K selection. Too few results miss relevant memories; too many add noise.

### Proactive Retrieval
The agent proactively decides *when* to retrieve, not just *what*. Before taking a complex action, the agent queries for relevant past decisions, similar situations, or known constraints. More agentic but requires the model to have good metacognition about when retrieval adds value.

### Streaming Retrieval
For long agentic sessions, retrieval happens continuously in the background. A secondary process monitors the conversation and pushes relevant memories into the context as they become relevant, rather than waiting for a query.

### Retrieval Augmented Generation (RAG)
The classic retrieval pattern: retrieve relevant documents/memories, inject them into the prompt, generate a grounded response. Covered in detail in [[architectures/rag]].

## Chunking and Indexing Strategy

How you store memories fundamentally affects retrieval quality:

**Chunk size**: Too small and individual chunks lack context. Too large and you retrieve more noise than signal. Common sweet spot: 200-500 token chunks with overlap for continuity.

**Metadata indexing**: Attach rich metadata at write time — entity tags, time, source, memory type, importance score. Enables hybrid retrieval: semantic search + metadata filtering.

**Hierarchical indexing**: Index summaries at the top level, raw content below. First-pass retrieval over summaries (fast, low noise), second-pass drilling into relevant raw content.

**Parent-child chunking**: Store small, precise chunks for retrieval, but inject the parent document for context. Retrieve by the child (precise match), serve the parent (rich context).

## Common Failure Modes

### Semantic Drift
Embedding similarity is imperfect. Queries for recent events may retrieve conceptually similar but temporally irrelevant memories. Always add recency weighting.

### Retrieval Hallucination
The retrieval returns something plausible-sounding but wrong, and the model accepts it without skepticism. Mitigation: include source metadata in retrieved content; train the model to cite and verify.

### Recency Bias Without Decay
If every new memory has maximum recency score, the system quickly degrades to only using new information. Implement proper exponential decay.

### Context Window Stuffing
Retrieving top-20 results and injecting all of them bloats the context and dilutes signal. Typically top-3 to top-5 with a relevance threshold is better than top-20 unconstrained.

### Cold Start
A new user or topic has no relevant memories. Build fallback strategies: use default context, ask clarifying questions, or acknowledge the absence of relevant history.

## Key Takeaways

- Retrieval is where most memory systems fail; similarity ≠ relevance
- Combine three signals: semantic similarity, recency, importance
- Implement exponential decay for recency; not all memories age equally
- Metadata indexing at write time dramatically improves retrieval precision
- Less is more: top-3 high-relevance results beat top-20 noisy ones
- Plan for failure modes: semantic drift, hallucination, cold start

## See Also

- [[types-of-memory]]
- [[long-term-memory]]
- [[episodic-vs-semantic]]
- [[architectures/rag]]
- [[architectures/vector-stores]]
- [[research/key-papers]]
