---
title: RAG — Retrieval-Augmented Generation
tags: [agent-memory, rag, retrieval, architecture, vector-store]
created: 2026-04-04
last_updated: 2026-04-04
---

## What Is RAG?

Retrieval-Augmented Generation (RAG) is an architecture pattern where a language model's generation is grounded by dynamically retrieved external knowledge. Instead of relying solely on what the model learned during training, RAG retrieves relevant documents, facts, or memories at inference time and injects them into the prompt.

The term was coined by Lewis et al. (Meta, 2020) in the paper "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks." It's since become the dominant paradigm for building knowledge-grounded AI applications.

## How It Works

The basic RAG pipeline has three stages:

### 1. Indexing (Offline)
Documents are split into chunks, embedded using an embedding model (e.g., OpenAI `text-embedding-3-small`, Cohere Embed, or open-source models like `bge-m3`), and stored in a vector database alongside their original text and metadata.

```
Document → Chunker → Embedding Model → Vector DB
```

### 2. Retrieval (Online)
At query time, the user query is embedded using the same model, and the vector DB performs a nearest-neighbor search to find the most semantically similar chunks.

```
Query → Embedding Model → Nearest Neighbor Search → Top-K Chunks
```

### 3. Generation (Online)
The retrieved chunks are injected into the LLM's context alongside the query. The model generates a response grounded in the retrieved content.

```
[System Prompt] + [Retrieved Chunks] + [User Query] → LLM → Response
```

## When to Use RAG

RAG is the right tool when:

- Your knowledge base changes frequently (training the model on it would be impractical)
- You need the model to cite sources or show its work
- The relevant knowledge is too large to fit in context
- You need to ground the model in domain-specific, proprietary, or recent knowledge
- You want to reduce hallucination by anchoring responses to specific retrieved facts

RAG is a particularly good fit for:
- Documentation Q&A ("how do I configure X?")
- Customer support over a knowledge base
- Agent memory retrieval (episodic and semantic memories as the retrieval corpus)
- Enterprise search and synthesis

## Limitations and Failure Modes

### Retrieval Failure
If the retrieval step doesn't surface the right chunks, the generation step is working with bad inputs. "Garbage in, garbage out" — but worse, because the model will often still produce a confident-sounding answer.

Common causes: poor chunking strategy, mismatch between query embedding and document embedding space, query too abstract for keyword-similar retrieval.

### Context Stuffing
More retrieved chunks ≠ better answers. Injecting 20 chunks into context adds noise and can push important information into the "lost in the middle" zone. Typically top-3 to top-5 chunks with a relevance threshold outperforms naively maximizing retrieved context.

### Stale Index
The retrieval corpus can become stale if the indexing pipeline doesn't run continuously. A fast-moving codebase, knowledge base, or memory store needs a near-real-time indexing strategy to keep RAG fresh.

### Semantic Gap
Embedding models capture semantic similarity, but similarity isn't always the same as relevance. A query about "database performance" might retrieve chunks about PostgreSQL performance when the user meant Redis. Hybrid retrieval (semantic + keyword/BM25) helps close this gap.

### Multi-Hop Reasoning
Standard RAG retrieves documents that are directly relevant to the query. If the answer requires chaining through multiple documents ("what is X, and how does X relate to Y, which affects Z?"), naive RAG often fails. This is where knowledge graphs have an advantage. See [[knowledge-graphs]].

## Advanced RAG Patterns

### Hybrid Retrieval (BM25 + Dense)
Combine traditional keyword search (BM25) with dense vector retrieval. Keyword search catches exact matches that semantic similarity misses; dense retrieval catches conceptual matches that keyword search misses. Reciprocal Rank Fusion (RRF) is a common merging strategy.

### Re-ranking
After initial retrieval, apply a cross-encoder re-ranker (e.g., Cohere Rerank, bge-reranker) to re-score the top-K chunks. Cross-encoders jointly attend to query and document, producing much more accurate relevance scores than embedding similarity. Computationally expensive, so only applied to the top-K retrieval pool.

### HyDE (Hypothetical Document Embeddings)
Instead of embedding the raw query, prompt the LLM to generate a hypothetical answer to the query, then embed *that*. The hypothesis often embeds closer to relevant documents than the original question. Improves recall for complex queries.

### Query Expansion
Expand the query into multiple variants before retrieval to improve recall. "What's the Redis connection timeout?" → also retrieve for "Redis timeout config", "connection pool settings", "client timeout parameter".

### Parent-Child Chunking
Store small precise chunks for retrieval, but when injecting, serve the full parent document for context richness. Retrieve by child (precise match), inject parent (full context).

## RAG vs. Fine-Tuning

| Dimension | RAG | Fine-Tuning |
|-----------|-----|-------------|
| Knowledge update | Real-time (re-index) | Retraining required |
| Transparency | Citable sources | Black box |
| Cost | Per-query retrieval | Training compute |
| Customization | Content, not behavior | Behavior and style |
| Hallucination risk | Lower (grounded) | Higher for domain knowledge |

Fine-tune for *behavior*; RAG for *knowledge*. They're complementary, not competing.

## See Also

- [[vector-stores]]
- [[knowledge-graphs]]
- [[mem0]]
- [[memgpt]]
- [[concepts/memory-retrieval]]
- [[research/key-papers]]
