---
title: Vector Stores for Agent Memory
tags: [agent-memory, vector-stores, embeddings, pinecone, chroma, pgvector, faiss]
created: 2026-04-04
last_updated: 2026-04-04
---

## Embeddings and Vector Search

A vector store is a database optimized for storing and querying high-dimensional vectors. In the context of agent memory, you embed text (memories, documents, facts) into dense vectors using an embedding model, store those vectors in the database, and at retrieval time embed the query and find the nearest stored vectors by some distance metric.

The core operation: **approximate nearest neighbor (ANN) search** at scale. Exact nearest neighbor search is O(n) linear scan — too slow for large corpora. ANN algorithms trade a small accuracy loss for dramatically faster search (sub-millisecond at millions of vectors).

## Embedding Models

The embedding model is as important as the vector store. It determines how well semantic similarity in the vector space maps to actual relevance.

**OpenAI**
- `text-embedding-3-small` — 1536 dimensions, cost-efficient, strong general-purpose
- `text-embedding-3-large` — 3072 dimensions, best quality, higher cost

**Open-Source**
- `bge-m3` (BAAI) — multilingual, strong retrieval, runs locally
- `gte-large-en-v1.5` (Alibaba) — strong English retrieval, 8192 token context
- `nomic-embed-text` — open-source, runs on Ollama, competitive with commercial models

**Specialized**
- `voyage-code-2` (Voyage) — code retrieval
- `cohere-embed-v3` — strong for RAG applications, supports int8 quantization

Choosing: for production with budget, OpenAI `text-embedding-3-small` is the default. For privacy or cost reasons, `bge-m3` self-hosted is excellent. Always evaluate on your specific domain data — general benchmarks don't always translate.

## Vector Database Comparison

### Pinecone
Managed cloud vector DB. The market leader for production RAG deployments.

**Strengths:**
- Zero infrastructure; fully managed
- Strong ANN performance at scale
- Rich metadata filtering (combine vector search with structured filters)
- Namespaces for multi-tenant isolation

**Weaknesses:**
- Cost at scale (per-vector pricing)
- No self-hosted option
- Less flexibility in index configuration

**Best for:** Production RAG applications where you want zero infra overhead and reliable performance.

### Chroma
Open-source, embeddable vector DB. Designed for developer experience — minimal setup, runs in-process or as a server.

```python
import chromadb
client = chromadb.Client()
collection = client.create_collection("agent_memory")
collection.add(documents=["user prefers Python"], ids=["fact_001"])
results = collection.query(query_texts=["programming language preference"], n_results=3)
```

**Strengths:**
- Extremely easy to get started
- Open-source, self-hosted
- Persistent or in-memory modes
- Good ecosystem (LangChain, LlamaIndex built-in support)

**Weaknesses:**
- Performance at very large scale (millions of vectors) is an open question
- Fewer advanced ANN index options than purpose-built DBs

**Best for:** Development, prototyping, small-to-medium deployments (<1M vectors).

### pgvector
PostgreSQL extension for vector similarity search. Store vectors alongside structured data in your existing Postgres instance.

```sql
CREATE TABLE memories (
  id SERIAL PRIMARY KEY,
  content TEXT,
  embedding vector(1536),
  created_at TIMESTAMPTZ,
  entity TEXT,
  importance FLOAT
);

-- Nearest neighbor search
SELECT content, 1 - (embedding <=> '[0.1, 0.2, ...]'::vector) AS similarity
FROM memories
WHERE entity = 'rowan'
ORDER BY embedding <=> '[0.1, 0.2, ...]'::vector
LIMIT 5;
```

**Strengths:**
- Unified storage for vectors + structured data (powerful hybrid queries)
- Leverage existing Postgres tooling, backups, replication
- ACID compliance
- HNSW and IVFFlat index support

**Weaknesses:**
- Not a purpose-built vector DB; performance at extreme scale (100M+ vectors) lags behind Pinecone
- Requires Postgres expertise for tuning

**Best for:** Teams already on Postgres who want vector search without a new service. Excellent for agent memory where you want to combine vector similarity with structured metadata filters.

### Qdrant
High-performance open-source vector DB written in Rust. Strong performance, rich filtering, supports quantization.

**Best for:** High-throughput, self-hosted deployments where performance matters and you don't want to pay Pinecone prices.

### Weaviate
Open-source, GraphQL-first vector DB with built-in vectorization modules. More complex but highly configurable.

### FAISS (Facebook AI Similarity Search)
Library (not a database) for ANN search. The foundation for many vector DBs. Fast, battle-tested, but no persistence or server — you build on top of it.

## Indexing Algorithms

**IVFFlat (Inverted File with Flat Quantization)**
Divides the vector space into clusters. Search is fast but recall degrades at low `nprobe` (number of clusters searched). Good for large datasets when you can tune accuracy vs. speed.

**HNSW (Hierarchical Navigable Small World)**
Graph-based ANN. Excellent recall at high speed, but higher memory usage. The preferred index for most modern deployments. Used by pgvector, Qdrant, and others.

**Scalar Quantization / Binary Quantization**
Compress vectors from float32 to int8 or 1-bit. Dramatically reduces memory footprint with modest accuracy loss. Important for cost management at scale.

## Practical Architecture for Agent Memory

For a production agent memory system with a vector store:

```
Agent Turn
  → Query: "what does Rowan prefer for database?"
  → Embed query (text-embedding-3-small)
  → Search vector store (top-5, entity='rowan', type='preference')
  → Inject retrieved chunks into context
  → Generate grounded response
  
Memory Write
  → Extract key facts from conversation
  → Embed fact text
  → Upsert to vector store with metadata
    {entity, type, source_session, created_at, importance}
```

## Key Takeaways

- Embedding model quality is as important as the vector store choice
- Pgvector is underrated for teams already on Postgres — hybrid queries are powerful
- Chroma for development, Pinecone/Qdrant for production at scale
- Always add rich metadata at write time to enable hybrid retrieval
- HNSW is the dominant index for recall/speed balance
- Quantization is essential for cost management at millions of vectors

## See Also

- [[rag]]
- [[knowledge-graphs]]
- [[mem0]]
- [[concepts/memory-retrieval]]
- [[concepts/long-term-memory]]
