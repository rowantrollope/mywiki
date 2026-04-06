---
title: KV Cache — Transformer Attention Caching Mechanics
tags: [kv-cache, transformer, attention, inference, optimization]
created: 2026-04-06
last_updated: 2026-04-06
---

## What Is the KV Cache?

During transformer inference, the attention mechanism computes three vectors for each token: **Query (Q)**, **Key (K)**, and **Value (V)**. Attention is computed as:

```
Attention(Q, K, V) = softmax(QK^T / √d_k) · V
```

When generating token `t`, the model needs the Q of token `t` and the K and V of *every previous token*. Without caching, this would require recomputing K and V for all previous tokens at every generation step — O(n²) work in total.

The **KV cache** solves this by storing the K and V tensors for every processed token. When generating the next token, only the new token's K and V need to be computed. All prior K/V pairs are read from the cache. This reduces per-step computation from O(n) matrix operations to O(1) — a critical optimization that makes autoregressive generation practical at scale.

## How It Works in Practice

Consider generating a 100-token response to a 1000-token prompt:

**Without KV cache:**
- Step 1: Process 1000 tokens → compute K,V for all 1000
- Step 2: Process 1001 tokens → recompute K,V for all 1001
- Step 3: Process 1002 tokens → recompute K,V for all 1002
- Total: ~100,100 K/V computations

**With KV cache:**
- Prefill step: Process 1000 tokens → compute and store K,V for all 1000
- Step 2: Compute K,V for token 1001 only; read 1000 from cache
- Step 3: Compute K,V for token 1002 only; read 1001 from cache
- Total: 1000 + 100 = 1,100 K/V computations

The KV cache converts the generation phase from compute-bound to memory-bandwidth-bound — which is why GPU memory capacity is such a critical constraint for LLM serving.

## Memory Requirements

KV cache memory scales with:
- **Number of layers** — each transformer layer has its own K and V tensors
- **Number of attention heads** — multi-head attention stores per-head K/V
- **Sequence length** — one K and one V vector per token per layer per head
- **Hidden dimension** — determines vector size
- **Batch size** — each concurrent request has its own KV cache

For a typical large model (e.g., 70B parameters, 80 layers, 64 heads, 128 head dim):

```
KV cache per token = 2 × layers × heads × head_dim × dtype_bytes
                   = 2 × 80 × 64 × 128 × 2 (fp16)
                   = ~2.6 MB per token
```

At a 4096-token context: ~10 GB of KV cache per single request. This is why batch sizes are small and GPU memory quickly becomes the bottleneck in production deployments.

## Prefill vs. Decode

LLM inference has two distinct phases with different KV cache behavior:

**Prefill:** The entire prompt is processed in parallel (like a standard forward pass). K and V tensors are computed for all input tokens and stored in the cache. This is compute-bound — the GPU is busy doing matrix multiplications.

**Decode:** Tokens are generated one at a time (autoregressive). Each step reads the full KV cache and appends one new K/V pair. This is memory-bandwidth-bound — the GPU spends most of its time moving data, not computing.

This distinction matters for optimization. Prefill benefits from large batch sizes and compute-efficient hardware. Decode benefits from high memory bandwidth and cache-efficient data layouts (e.g., PagedAttention).

## PagedAttention

Managing KV caches naively wastes memory because you must allocate the maximum possible sequence length upfront (you don't know how long generation will be). **PagedAttention**, introduced by the vLLM project (Kwon et al., 2023), borrows the virtual memory paging concept from OS design.

Instead of allocating contiguous KV cache blocks, PagedAttention divides KV cache into fixed-size blocks (pages) and allocates them on demand. Multiple sequences can share pages (for prefix sharing). This eliminates internal fragmentation and increases effective GPU utilization by 2–4× in multi-user serving scenarios.

PagedAttention is now implemented in vLLM, TensorRT-LLM, and most production inference frameworks.

## Prefix Caching (Cross-Request Reuse)

Standard KV caches are per-request and discarded when the request completes. **Prefix caching** (also called prompt caching or KV cache reuse) extends this to share KV cache across requests that share a common prefix.

If 100 requests all start with the same 4000-token system prompt, prefix caching computes the KV cache for that prefix once and reuses it for all 100. This is the engine that powers API-level [[concepts/prompt-caching|prompt caching]] from Anthropic, OpenAI, and Google.

Prefix caching requires:
1. Content-addressable storage (hash the prefix to identify cache entries)
2. Exact prefix matching (even one changed token invalidates the cache for all subsequent tokens)
3. A memory management policy (LRU eviction, TTL, etc.)

## Research: KV Cache Compression

As context windows grow to 128K–1M tokens, KV cache memory becomes prohibitive. Active research areas include:

**Eviction-based methods:** Not all tokens' K/V pairs are equally important. H2O (Heavy-Hitter Oracle, Zhang et al., 2023) identifies and evicts K/V pairs for tokens that receive low attention scores, keeping only the "heavy hitters." SnapKV (Li et al., 2024) uses observation of attention patterns during prefill to prune the cache before decode.

**Quantization:** KV caches are stored in lower precision (INT8, INT4). KVQuant (Hooper et al., 2024) achieves 2-bit KV cache with minimal quality loss by applying non-uniform quantization schemes that account for outlier values.

**Offloading:** KV cache is offloaded from GPU VRAM to CPU RAM or NVMe SSD when not actively needed, then paged back in. vLLM's CPU offload and InfLLM use this to extend effective context.

**Architectural changes:** Some newer architectures reduce KV cache requirements by design. Multi-Query Attention (MQA, Shazeer, 2019) and Grouped-Query Attention (GQA, Ainslie et al., 2023) reduce the number of K and V heads while keeping query heads unchanged. GQA is now standard in models like Llama 3, Mistral, and Gemma.

## Limitations and Gotchas

- **Greedy decoding only** — KV cache assumes tokens generated previously won't change. Beam search and sampling with backtracking require careful cache management.
- **Speculative decoding compatibility** — draft tokens need their own K/V; cache management becomes complex when draft tokens are rejected.
- **Multi-modal models** — image and audio tokens can have very large K/V representations, multiplying memory requirements.
- **System prompt changes break prefix cache** — even minor edits invalidate the cache for the entire subsequent context.

## See Also

- [[concepts/prompt-caching]] — API-level caching that builds on prefix KV cache reuse
- [[patterns/cache-optimization]] — practical structuring for maximum cache hits
- [[research/state-of-the-art-2026]] — current research on KV cache compression and alternatives
- [[overview]] — the full memory and caching stack

## References

- Vaswani et al. (2017). *Attention Is All You Need.* NeurIPS. https://arxiv.org/abs/1706.03762
- Kwon et al. (2023). *Efficient Memory Management for Large Language Model Serving with PagedAttention.* SOSP. https://arxiv.org/abs/2309.06180
- Zhang et al. (2023). *H₂O: Heavy-Hitter Oracle for Efficient Generative Inference of Large Language Models.* NeurIPS. https://arxiv.org/abs/2306.14048
- Ainslie et al. (2023). *GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints.* EMNLP. https://arxiv.org/abs/2305.13245
- Hooper et al. (2024). *KVQuant: Towards 10 Million Context Length LLM Inference with KV Cache Quantization.* https://arxiv.org/abs/2401.18079
- Li et al. (2024). *SnapKV: LLM Knows What You are Looking for Before Generation.* https://arxiv.org/abs/2404.14469
