---
title: Memory Types — In-Context, External, and Parametric
tags: [memory-types, in-context, external, parametric, rag, taxonomy]
created: 2026-04-06
last_updated: 2026-04-06
---

## A Unified Taxonomy of LLM Memory

LLM memory doesn't map cleanly to a single concept — it operates across multiple abstraction levels, from bits in GPU memory to facts baked into model weights. Understanding the distinctions is necessary for designing systems that are both capable and cost-effective.

The most useful taxonomy divides memory into three categories based on where information lives and how it's accessed:

1. **Parametric memory** — encoded in model weights at training time
2. **In-context memory** — present in the current prompt/context window
3. **External memory** — stored outside the model; retrieved on demand

Each has different characteristics for capacity, cost, updateability, and reliability.

---

## 1. Parametric Memory

**What it is:** Knowledge and skills baked into the model's weights during pre-training and fine-tuning. When you ask a model what the capital of France is, it doesn't look that up — it retrieves the answer from parametric memory.

**Characteristics:**
- **Capacity:** Effectively unlimited (bounded by model size; billion-parameter models store significant world knowledge)
- **Access latency:** Zero — no lookup required
- **Update cost:** Expensive — requires fine-tuning or full retraining
- **Staleness:** Frozen at training cutoff
- **Reliability:** Prone to hallucination; models confabulate when parametric knowledge is incomplete or conflicting

**What's stored:** Factual knowledge, language patterns, reasoning strategies, coding skills, behavioral tendencies (from RLHF), world models.

**What's NOT stored:** Information from after the training cutoff, user-specific information, private organizational data (unless explicitly fine-tuned in).

**Fine-tuning as memory update:** Fine-tuning can inject new knowledge into parametric memory, but it's expensive and suffers from *catastrophic forgetting* — adding new knowledge can degrade performance on existing knowledge. This makes parametric memory impractical for frequently updating information.

**Key insight:** Parametric memory is excellent for stable, general knowledge but completely inappropriate for dynamic, user-specific, or post-cutoff information. Don't try to fine-tune in facts that could just be put in the context or retrieved.

---

## 2. In-Context Memory

**What it is:** Any information present in the current context window — the prompt, conversation history, retrieved documents, and system instructions. The model can "remember" anything that's in its context because it attends to it directly.

**Characteristics:**
- **Capacity:** Limited by context window (4K–1M tokens depending on model)
- **Access latency:** Zero — it's already in the prompt
- **Update cost:** Free — just include it in the next request
- **Staleness:** Always current (you control what's in context)
- **Reliability:** High for explicit facts in context; model still sometimes ignores context and relies on parametric memory (known as *context-faithfulness failures*)

**Sub-types:**

**System prompt memory:** Instructions, persona, behavioral rules, and stable context written once and included on every request. Most effectively cached via [[concepts/prompt-caching|prompt caching]].

**Conversation history memory:** The record of what's been said in the current session. Grows with each turn; must be managed to stay within context limits. See [[concepts/context-window-management]].

**Injected document memory:** Retrieved or provided documents inserted into context for the current request. The core mechanism of RAG. Highly effective for specific factual queries.

**Working memory:** The "scratch pad" — intermediate reasoning steps, chain-of-thought, tool call results, partial outputs. Used and discarded within a single inference.

**The in-context learning effect:** Providing a few examples (few-shot prompting) temporarily "teaches" the model a new task or format. This is in-context learning — the model adapts its behavior based on examples in the prompt without any weight updates. It's a form of ephemeral memory.

---

## 3. External Memory

**What it is:** Information stored outside the model — in databases, vector stores, file systems — and retrieved on demand. The model doesn't "know" this information until it's retrieved and injected into context.

**Characteristics:**
- **Capacity:** Effectively unlimited (bounded by storage, not compute)
- **Access latency:** Moderate — requires embedding lookup, vector search, or DB query (typically 50–500ms)
- **Update cost:** Low — just write to the store; no model changes required
- **Staleness:** As fresh as the last write
- **Reliability:** High for exact facts; quality depends on retrieval relevance

**Sub-types:**

**Vector memory (semantic retrieval):** Documents are split into chunks, embedded, and stored in a vector database (Pinecone, Chroma, pgvector, Weaviate). Retrieval uses approximate nearest-neighbor search on query embeddings. Best for: large corpora where semantic similarity drives relevance.

**Structured/relational memory:** Entities, facts, and relationships stored in a SQL or graph database. Accessed via structured queries. Best for: well-defined schemas, exact lookups, relationship traversal.

**Key-value memory:** Simple storage of string keys to string values. Used for user profiles, preferences, session state. Best for: lookup by known identifier.

**Episodic memory:** Time-indexed records of events, interactions, and observations. Includes timestamps and sequence metadata. Used by frameworks like MemGPT and Letta to build persistent agent histories.

**Knowledge graph memory:** Entities and relationships represented as a graph. Multi-hop queries (find all projects related to person X who also worked with person Y) are natural. Best for: complex relationship reasoning that vectors handle poorly.

---

## Comparison Table

| Attribute | Parametric | In-Context | External |
|-----------|-----------|------------|----------|
| Capacity | High (fixed at training) | Limited (4K–1M tokens) | Unlimited |
| Access latency | 0 | 0 | 50–500ms |
| Update cost | Very high (retrain/finetune) | Free | Low (write to DB) |
| Staleness risk | High (training cutoff) | None | Low |
| Implementation complexity | Very high | Low | Medium–High |
| Cost per use | Amortized in inference | Charged per token | DB cost + embedding |
| Failure modes | Hallucination, outdated facts | Context overflow, context-faithfulness | Retrieval miss, stale data |

---

## The Interplay Between Types

Production systems rarely use just one type. Effective patterns combine them:

**RAG + Parametric:** Retrieve specific facts from external memory; rely on parametric memory for reasoning, language, and general knowledge. The model uses retrieved facts as ground truth, parametric knowledge as cognitive scaffolding.

**In-context + External (hybrid RAG):** For long documents too large to fit in context, inject a summary (in-context) and retrieve specific sections (external) when they're relevant to the query.

**Prompt cache + In-context:** Use prompt caching to cheaply maintain a large stable system prompt and base context; use in-context memory for dynamic session content.

**Episodic external + In-context summary:** Store raw interaction logs externally (episodic); at the start of each session, summarize the most relevant past episodes and inject them into context. MemGPT and similar systems work this way.

---

## The Cognitive Science Parallel

This taxonomy loosely maps to human memory systems:
- **Parametric → Long-term semantic memory** (general knowledge)
- **In-context → Working memory + episodic buffer** (current focus + recent events)
- **External → Exocortex / external tools** (notebooks, databases, the internet)

The parallel is useful but imperfect. LLM parametric memory doesn't consolidate the way human long-term memory does. And LLM "forgetting" is instantaneous (context cleared = forgotten) rather than gradual.

## See Also

- [[concepts/kv-cache]] — how in-context memory is processed at the hardware level
- [[concepts/context-window-management]] — managing in-context memory constraints
- [[concepts/prompt-caching]] — caching repeated in-context prefixes
- [[patterns/memory-architectures]] — combining all three types in production architectures
- [[wiki/agent-memory/concepts/types-of-memory|Agent Memory: Types]] — cognitive science framing (episodic, semantic, procedural, working)

## References

- Lewis et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.* NeurIPS. https://arxiv.org/abs/2005.11401
- Guu et al. (2020). *REALM: Retrieval-Augmented Language Model Pre-Training.* ICML. https://arxiv.org/abs/2002.08909
- Liang et al. (2023). *Holistic Evaluation of Language Models (HELM).* https://arxiv.org/abs/2211.09110
- Park et al. (2023). *Generative Agents: Interactive Simulacra of Human Behavior.* UIST. https://arxiv.org/abs/2304.03442
- Packer et al. (2024). *MemGPT: Towards LLMs as Operating Systems.* https://arxiv.org/abs/2310.08560
