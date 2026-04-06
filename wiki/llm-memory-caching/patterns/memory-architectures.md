---
title: Memory Architectures — RAG, Vector Stores, and Episodic Memory Patterns
tags: [memory-architectures, rag, vector-stores, episodic-memory, patterns, production]
created: 2026-04-06
last_updated: 2026-04-06
---

## Choosing an Architecture

LLM memory architecture is not one-size-fits-all. The right pattern depends on:
- **What you're remembering:** Documents? Conversations? User preferences? World facts?
- **How often it changes:** Static knowledge vs. continuously updated state
- **How it's accessed:** Exact lookup vs. semantic search vs. relationship traversal
- **Scale:** One user vs. millions; kilobytes vs. terabytes
- **Latency budget:** Synchronous retrieval adds round-trip time

This article maps the major patterns, when to use each, and how to combine them.

---

## Pattern 1: Basic RAG (Retrieval-Augmented Generation)

**What it is:** Documents are chunked, embedded, and stored in a vector database. At query time, the user's message is embedded and used to retrieve the most similar chunks. Retrieved chunks are injected into the prompt before generation.

```
User message
    ↓ embed
Query vector → [Vector DB] → Top-K chunks
                                  ↓
            [System Prompt + Retrieved Chunks + User Message] → LLM → Response
```

**When to use:**
- Knowledge base Q&A (product docs, internal wikis, legal documents)
- Document analysis at scale
- Any task where "here are the relevant facts" improves the answer

**Key implementation choices:**

*Chunking strategy:* Fixed-size (e.g., 512 tokens) is simple but may split related content. Sentence or paragraph chunking is more semantically coherent. Hierarchical chunking (parent + child chunks) retrieves parents for context but scores on children for precision.

*Embedding model:* OpenAI `text-embedding-3-large`, Cohere Embed v3, or open-source models (BGE-M3, E5-Mistral). Larger models embed better but cost more. Use the same model for indexing and retrieval — model mismatch causes silent degradation.

*Vector DB:* Pinecone and Weaviate for managed; pgvector for Postgres-native; Chroma for local/dev. At large scale, Milvus or Qdrant.

*Top-K:* Typical values are 3–10 chunks. More chunks = more context but higher cost and potential noise. Use MMR (Maximal Marginal Relevance) to avoid retrieving near-duplicate chunks.

**Failure modes:**
- **Retrieval miss:** The right chunk isn't returned (wrong embedding model, bad chunking, low similarity threshold)
- **Context dilution:** Too many retrieved chunks add noise
- **Stale index:** Documents updated but embeddings not re-indexed

---

## Pattern 2: Hybrid RAG (Dense + Sparse)

Standard RAG uses dense vector retrieval (semantic similarity). Hybrid RAG combines this with **sparse retrieval (BM25/keyword search)**:

```
Query → [Dense retrieval] → semantic matches
      → [Sparse retrieval] → keyword matches
              ↓ RRF (Reciprocal Rank Fusion)
      → Re-ranked combined results
```

Dense retrieval handles paraphrase and semantic matching. Sparse retrieval handles exact terminology, product codes, named entities, and uncommon words. Hybrid consistently outperforms either alone.

**Tools:** Weaviate, Pinecone, Qdrant all support hybrid retrieval natively. LangChain and LlamaIndex have hybrid retrievers.

---

## Pattern 3: Episodic Memory

**What it is:** Structured storage of conversation turns and agent events, indexed by time. Used to give agents memory of prior interactions with a specific user or about a specific task.

```
Session 1:
  User: "I prefer Python over JavaScript"
  → Stored as episode with timestamp, user_id, topic

Session 2 (days later):
  Agent startup → retrieve recent episodes for user
  → Inject: "From prior sessions: user prefers Python"
```

**Key components:**
1. **Log store:** Append-only record of events (conversations, tool calls, observations). Could be a simple database table.
2. **Memory extractor:** After each session, extract salient facts using an LLM call. "What should I remember from this conversation?" → stores condensed facts.
3. **Retrieval layer:** At session start, retrieve relevant memories (by recency, topic, or importance score).
4. **Injection:** Relevant memories prepended to system prompt.

**Frameworks implementing this:**
- **MemGPT/Letta** — full OS-style virtual memory management with agent-controlled memory operations
- **mem0** — managed API that handles extraction, deduplication, and retrieval automatically
- **Zep** — session memory layer with automatic fact extraction and graph storage

**Granularity:** Store memories at the right level of abstraction:
- Too granular: "User said 'hi' at 3pm" — noise
- Right level: "User is a senior engineer at a startup, prefers brevity, dislikes explanations of basics"
- Too coarse: "User is technical" — too vague to be useful

---

## Pattern 4: Long-Term User Profiles

A specific form of episodic memory focused on stable user-level attributes.

```python
# Profile schema
{
  "user_id": "u_123",
  "preferences": {
    "language": "Python",
    "style": "terse",
    "expertise": "senior engineer"
  },
  "history": [
    {"project": "API redesign", "context": "Stripe integration"},
    {"project": "data pipeline", "context": "pandas + Parquet"}
  ],
  "facts": [
    {"fact": "Works at a series B fintech startup", "confidence": 0.95},
    {"fact": "Has a morning standup at 9am PT", "confidence": 0.7}
  ]
}
```

**Storage:** Key-value store (Redis, DynamoDB) keyed by user_id. Inject relevant fields into system prompt at session start.

**Challenge:** Contradictions and staleness. User said "I'm on AWS" in January; in March said "we migrated to GCP." Systems need temporal ordering and conflict resolution. mem0 and Letta handle this; DIY implementations often don't.

---

## Pattern 5: Knowledge Graph Memory

**What it is:** Entities and relationships stored as a graph. Better than vectors for queries involving multiple hops across relationships.

```
Person(Rowan) -[works_at]-> Company(Redis)
Company(Redis) -[competitor_of]-> Company(Momento)
Person(Rowan) -[manages]-> Project(VectorDB launch)
```

Query: "What projects are managed by people at Redis?" — graph traversal, not vector similarity.

**When to prefer over vectors:**
- Multi-hop reasoning ("find all X related to Y through Z")
- Structured factual queries ("what did X say about topic Y?")
- Entity deduplication (same person mentioned multiple ways)

**Tools:** Neo4j, Amazon Neptune, or lightweight options like NetworkX for smaller scales. LlamaIndex has a KnowledgeGraphIndex; LangChain has experimental graph memory.

**Challenge:** Knowledge graph construction is hard. Extracting structured triples from unstructured conversation is imperfect. Most production systems combine graph storage for key entities with vector storage for unstructured content.

---

## Pattern 6: Hierarchical Memory (MemGPT-style)

**What it is:** Tiered memory management inspired by OS virtual memory. The agent actively manages what information is in each tier:

```
┌────────────────────────────────┐
│  Main Context (working memory) │  ← active attention, fastest
│  ~2-8K tokens                  │
├────────────────────────────────┤
│  External Storage              │  ← retrieved on demand
│  - Archival memory (vector)    │
│  - Conversation history (DB)   │
└────────────────────────────────┘
```

The agent has explicit memory management tools (like system calls):
- `core_memory_append(key, value)` — add to persistent core memory
- `archival_memory_insert(content)` — write to long-term storage
- `archival_memory_search(query)` — retrieve from long-term storage

The agent decides when to write to and read from each tier, mimicking how a human decides to take notes.

**When to use:** Long-lived conversational agents (personal assistants, therapist-style agents, software development agents that work over weeks). High complexity — use a framework (Letta) rather than building from scratch.

---

## Combining Patterns: Production Architecture

Most real systems combine multiple patterns:

```
                       ┌─────────────────┐
User message ───────── │  Agent Context  │
                        │                 │
System prompt  ────────►│  + User profile │ (from KV store)
(prompt cached) ───────►│  + Agent rules  │
                        │  + Session hist │ (last N turns)
Retrieved docs ────────►│  + Top-K chunks │ (from vector DB)
                        └────────┬────────┘
                                 │
                                 ▼
                          LLM Inference
                        (prompt cached prefix)
                                 │
                                 ▼
                           Response + new facts
                                 │
                     ┌───────────┴────────────┐
                     ▼                        ▼
              Update user profile        Append to log
              (KV store)                 (episodic store)
```

**Layer assignment:**
| Content | Pattern | Storage |
|---------|---------|---------|
| Agent instructions | Prompt cache | API cache prefix |
| User preferences | User profile | KV store |
| Domain knowledge | Basic RAG | Vector DB |
| Conversation history | Sliding window | In-context |
| Historical summaries | Episodic memory | DB + vector |

## See Also

- [[concepts/memory-types]] — the taxonomy of what each pattern stores
- [[concepts/kv-cache]] — hardware-level caching that backs all in-context memory
- [[concepts/prompt-caching]] — caching static parts of the architecture
- [[concepts/context-window-management]] — managing context budget across patterns
- [[wiki/agent-memory/architectures/rag|Agent Memory: RAG]] — deeper RAG dive
- [[wiki/agent-memory/architectures/memgpt|Agent Memory: MemGPT]] — MemGPT architecture deep dive

## References

- Lewis et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.* https://arxiv.org/abs/2005.11401
- Packer et al. (2024). *MemGPT: Towards LLMs as Operating Systems.* https://arxiv.org/abs/2310.08560
- Izacard et al. (2022). *Few-shot Learning with Retrieval Augmented Language Models.* https://arxiv.org/abs/2208.03299
- Shi et al. (2023). *REPLUG: Retrieval-Augmented Language Model Pre-Training.* https://arxiv.org/abs/2301.12652
- Ma et al. (2023). *Query Rewriting for Retrieval-Augmented Large Language Models.* EMNLP. https://arxiv.org/abs/2305.14283
