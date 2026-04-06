---
title: Key Papers — LLM Memory and Caching
tags: [research, papers, bibliography, kv-cache, rag, memory, prompt-caching]
created: 2026-04-06
last_updated: 2026-04-06
---

## Reading Order

**Start here if you're new to the area:**
1. Vaswani et al. (2017) — the attention mechanism that everything builds on
2. Lewis et al. (2020) — RAG, the foundational retrieval pattern
3. Kwon et al. (2023) — PagedAttention; how modern serving works
4. Packer et al. (2024) — MemGPT; the dominant agent memory architecture

**For inference optimization:** Vaswani → Kwon → Zhang (H2O) → Hooper (KVQuant)

**For memory architectures:** Lewis → Park (Generative Agents) → Packer (MemGPT) → Hu (SurveyMemory)

---

## Foundational Papers

### Attention Is All You Need (2017)
**Authors:** Vaswani, Shazeer, Parmar et al.  
**Venue:** NeurIPS 2017  
**Link:** https://arxiv.org/abs/1706.03762

The paper that introduced the transformer architecture and the self-attention mechanism. Every subsequent work on KV caching, context extension, and memory-augmented LLMs builds directly on the attention formulation here. Required reading.

**Key contribution:** The Q, K, V attention computation; the multi-head attention mechanism; positional encodings. The KV cache exists precisely because computing K and V for all previous tokens is redundant in autoregressive decoding.

---

### Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks (2020)
**Authors:** Lewis, Perez, Piktus et al. (Meta AI)  
**Venue:** NeurIPS 2020  
**Link:** https://arxiv.org/abs/2005.11401

The paper that named and formalized RAG. Demonstrates that combining a parametric memory (the LM) with a non-parametric memory (a dense retrieval system over Wikipedia) outperforms both alone on knowledge-intensive tasks like open-domain QA.

**Key contribution:** The RAG-Sequence and RAG-Token formulations; the demonstration that external retrieval dramatically improves factual accuracy; the FAISS-based dense retrieval pipeline that became standard.

**Legacy:** RAG is now deployed in essentially every enterprise LLM application. The paper's basic architecture (embed query → retrieve → generate) remains unchanged five years later.

---

### REALM: Retrieval-Augmented Language Model Pre-Training (2020)
**Authors:** Guu, Lee, Tung et al. (Google Research)  
**Venue:** ICML 2020  
**Link:** https://arxiv.org/abs/2002.08909

Extends RAG by incorporating retrieval into pre-training itself, not just inference. The model learns to retrieve useful documents as part of learning to generate. Demonstrated that retrieval can be end-to-end differentiable.

**Key contribution:** Knowledge retriever trained jointly with the language model. Showed that models can learn *when* to retrieve, not just *how* to use retrieved content.

---

## KV Cache and Inference Optimization

### Efficient Memory Management for Large Language Model Serving with PagedAttention (2023)
**Authors:** Kwon, Li, Zhuang et al. (UC Berkeley)  
**Venue:** SOSP 2023  
**Link:** https://arxiv.org/abs/2309.06180

The paper behind vLLM. Introduces PagedAttention, which manages KV cache memory like OS virtual memory — non-contiguous pages allocated on demand instead of pre-allocated contiguous buffers.

**Key contribution:** Eliminated internal fragmentation in KV cache allocation; enabled prefix sharing across requests; achieved 2–4× throughput improvement over HuggingFace Transformers baselines. PagedAttention is now the standard in production serving.

---

### H₂O: Heavy-Hitter Oracle for Efficient Generative Inference (2023)
**Authors:** Zhang, Sheng, Zhou et al.  
**Venue:** NeurIPS 2023  
**Link:** https://arxiv.org/abs/2306.14048

Observes that attention scores are highly skewed — a small subset of tokens ("heavy hitters") consistently receive most attention mass. Proposes evicting KV cache entries for non-heavy-hitter tokens during decode.

**Key contribution:** Up to 20× KV cache memory reduction with under 5% performance degradation on most tasks. Established eviction-based KV compression as a viable approach.

---

### GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints (2023)
**Authors:** Ainslie, Lee-Thorp, de Jong et al. (Google)  
**Venue:** EMNLP 2023  
**Link:** https://arxiv.org/abs/2305.13245

Grouped-Query Attention (GQA) reduces the number of K and V heads relative to Q heads. With 8 query heads per KV head (a common ratio), KV cache size is reduced 8×. GQA achieves near-MHA quality while dramatically cutting memory.

**Key contribution:** Practical architecture improvement now standard in Llama 3, Mistral, Gemma, and most modern open models. Directly reduces the memory cost per token in the KV cache.

---

### KVQuant: Towards 10 Million Context Length LLM Inference with KV Cache Quantization (2024)
**Authors:** Hooper, Kim, Mohammadzadeh et al. (UC Berkeley)  
**Venue:** NeurIPS 2024  
**Link:** https://arxiv.org/abs/2401.18079

Quantizes KV cache entries to 2–4 bits instead of 16-bit floats. Uses non-uniform quantization that accounts for outlier values in attention keys and values.

**Key contribution:** Enables 10M+ context length inference on single-GPU setups through aggressive KV compression. Demonstrated that KV quantization can be done with negligible quality loss when outliers are handled properly.

---

### SnapKV: LLM Knows What You are Looking for Before Generation (2024)
**Authors:** Li, Zhang, Shan et al.  
**Venue:** NeurIPS 2024  
**Link:** https://arxiv.org/abs/2404.14469

Observes that attention patterns during prefill reveal which tokens will be important during decoding. Uses prefill attention to select which K/V pairs to retain, pruning the cache before generation starts.

**Key contribution:** Combines automatic importance detection with KV cache compression; outperforms H2O on most benchmarks. Particularly effective for long-context question answering.

---

## Agent Memory and Long-Context

### Lost in the Middle: How Language Models Use Long Contexts (2023)
**Authors:** Liu, Lin, Hewitt et al. (Stanford)  
**Venue:** TACL 2023  
**Link:** https://arxiv.org/abs/2307.03172

Demonstrates that LLM performance on multi-document QA degrades when relevant information is in the middle of long contexts. Performance is best when relevant info is at the beginning or end ("primacy and recency effects").

**Key contribution:** Quantified the "lost in the middle" phenomenon across models. Fundamentally changed how practitioners structure long-context prompts.

---

### Generative Agents: Interactive Simulacra of Human Behavior (2023)
**Authors:** Park, O'Brien, Cai et al. (Stanford/Google)  
**Venue:** UIST 2023  
**Link:** https://arxiv.org/abs/2304.03442

Simulates 25 AI agents in a social environment, each with episodic memory, daily planning, and reflection mechanisms. Agents exhibit emergent social behaviors including gossip, coordination, and relationship formation.

**Key contribution:** The "memory stream" architecture — a time-indexed log of observations — combined with importance scoring and reflection to synthesize higher-level insights from raw memories. Became the reference architecture for episodic agent memory.

---

### MemGPT: Towards LLMs as Operating Systems (2023/2024)
**Authors:** Packer, Fang, Patil et al. (UC Berkeley)  
**Venue:** NeurIPS 2024  
**Link:** https://arxiv.org/abs/2310.08560

Treats LLMs like an operating system kernel managing a memory hierarchy. The context window is "physical memory" (limited, fast); external storage is "disk" (unlimited, slow). The LLM itself decides when to page information in and out using special function calls.

**Key contribution:** Virtual context management via self-directed memory operations. Demonstrated that agents can manage their own memory effectively, enabling arbitrarily long conversations without context overflow. Open-sourced as Letta.

---

### Augmenting Language Models with Long-Term Memory (NeurIPS 2023)
**Authors:** Wang, Dong, Zhu et al.  
**Venue:** NeurIPS 2023  
**Link:** https://proceedings.neurips.cc/paper_files/paper/2023/file/ebd82705f44793b6f9ade5a669d0f0bf-Paper-Conference.pdf

Proposes LONGMEM, a system that caches KV pairs from previous segments of long documents in an external memory bank and retrieves them via side-network attention.

**Key contribution:** Demonstrated that KV-level external memory (not just document retrieval) enables efficient long-document reasoning beyond context limits.

---

### Prompt Cache: Modular Attention Reuse for Low-Latency Inference (2023)
**Authors:** Gim, Chen, Lee et al.  
**Venue:** MLSys 2024  
**Link:** https://arxiv.org/abs/2311.04934

Formalizes prompt caching as "modular attention reuse" — defines cacheable schema that identifies which prompt segments can be precomputed and reused across requests. The academic foundation for commercial prompt caching APIs.

**Key contribution:** Formal framework for prefix caching; demonstrated 8× latency reduction for long static prompts with negligible quality impact.

---

### Leave No Context Behind: Efficient Infinite Context Transformers with Infini-Attention (2024)
**Authors:** Munkhdalai, Peng, Iyer (Google)  
**Link:** https://arxiv.org/abs/2404.07143

Proposes Infini-Attention, which adds a compressed memory to standard attention. Each attention layer maintains a fixed-size compressive memory that accumulates context beyond the window, updated incrementally during processing.

**Key contribution:** 8B parameter model achieves SOTA on book summarization over 500K+ tokens. Enables effectively infinite context without quadratic attention cost. Memory is fixed-size regardless of sequence length.

---

## See Also

- [[research/state-of-the-art-2026]] — current SOTA and open problems
- [[concepts/kv-cache]] — explains the mechanics behind most of these papers
- [[concepts/memory-types]] — taxonomic context for memory architectures
- [[wiki/agent-memory/research/key-papers|Agent Memory: Key Papers]] — overlapping bibliography with agent focus
