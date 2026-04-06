---
title: State of the Art — LLM Memory and Caching (2025–2026)
tags: [research, sota, 2025, 2026, long-context, kv-compression, memory-augmented]
created: 2026-04-06
last_updated: 2026-04-06
---

## Where the Field Stands

As of early 2026, LLM memory and caching sits at an interesting inflection point. Context windows have expanded dramatically (Gemini 1.5 Pro: 1M tokens; Claude: 200K; GPT-4o: 128K), prompt caching is deployed and working, and RAG is well understood. But several hard problems remain unsolved, and the research frontier is moving fast.

This article surveys the current state of the art across three dimensions: **model-level capabilities**, **inference-level optimization**, and **application-level memory architectures**.

---

## Long-Context Models: Expanded but Imperfect

### What works now
Modern frontier models handle genuinely long contexts for retrieval-heavy tasks. Finding a specific passage in a 200K-token document, summarizing a book, or reasoning over an entire codebase is now routine with Gemini 1.5 Pro or Claude 3.5.

The "lost in the middle" problem (Liu et al., 2023) has improved but not disappeared. Later model generations show less pronounced degradation at the 50% position in long contexts, likely due to targeted training with long-context data.

### What doesn't work yet
**Multi-hop reasoning at long context:** Asking a model to combine facts scattered across a 100K-token document — where answering the question requires three or more inferential steps over disconnected passages — reliably degrades performance. The NoLiMa benchmark (ICML 2025) demonstrated that when keyword overlap between question and relevant passages is removed, even SOTA models struggle past 32K tokens.

**Reliable attention at 1M+ tokens:** Gemini 1.5 Pro's 1M-token window is impressive on synthetic benchmarks (needle-in-a-haystack retrieval) but degrades significantly on complex reasoning tasks at full context. The practical effective context for demanding tasks is closer to 100K–200K.

---

## KV Cache Compression: The Active Frontier

GPU memory is the primary bottleneck for serving long-context models at scale. The KV cache grows linearly with sequence length — at 128K tokens, the KV cache for a large model consumes tens of gigabytes per request. Three approaches are converging:

### Sparse Attention and Cache Eviction
**SnapKV** (NeurIPS 2024) uses prefill attention patterns to select which tokens' KV pairs to retain, pruning up to 90% of the cache with minimal quality loss. Follow-up work like **PyramidKV** (2024) applies different compression rates per layer (higher layers compress more) based on the observation that lower transformer layers need more context than higher layers.

**StreamingLLM** (Xiao et al., 2023) enables infinite-length generation by always keeping the initial "attention sink" tokens (positions 0–3) plus a recent sliding window, discarding everything in between. Works for streaming generation but loses non-recent context.

### KV Quantization
**KVQuant** (2024) showed 2-bit KV cache storage with <2% perplexity increase using non-uniform per-channel quantization. The key insight: K and V tensors have structured outliers that must be handled separately from the quantized bulk values.

**WKVQuant** (2024) extends this with window-aware quantization — different quantization schemes for tokens inside vs. outside the recent window.

### External Memory / Offloading
**InfLLM** (2024) and **EM-LLM** (2025) move KV cache for distant tokens to CPU RAM or SSD, paging them back when needed. EM-LLM integrates episodic memory principles — events are clustered into coherent episodes and retrieved by episode rather than individual token. Benchmarks in February 2025 showed EM-LLM surpassing RAG with state-of-the-art retrievers (NV-Embed-v2) on long-context tasks.

---

## Infini-Attention: A New Architectural Direction

Google's **Infini-Attention** (Munkhdalai et al., 2024) represents a qualitatively different approach: rather than evicting or offloading KV cache, it adds a compressed memory to each attention layer that accumulates all past context in a fixed-size matrix, updated incrementally using associative memory principles.

Results:
- 8B parameter Infini-Attention model achieves SOTA on book summarization over 500K tokens
- Fixed memory footprint regardless of sequence length (the compression is lossy but principled)
- Compatible with standard transformer training; can be added to existing architectures

The limitation: the compressive memory uses linear attention-like approximations that lose fine-grained token-level detail. Works well for thematic long-range dependencies; struggles for verbatim retrieval at extreme lengths.

**Related work:** Google's earlier **Memorizing Transformers** (Wu et al., 2022) added a k-NN memory over past KV pairs, demonstrating that non-differentiable external memory can be plugged into transformers effectively.

---

## Prompt Caching: Mature, Spreading

All three major providers now offer prompt caching (Anthropic since August 2024, OpenAI October 2024, Google May 2024). The technology is stable and the economics are clear: 50–90% cost reduction on stable prefixes.

The current frontier:
- **Persistent caching:** Google's model (explicit cache resources with configurable TTL) is spreading. The ability to cache content for hours or days rather than minutes enables new use cases (caching entire codebases, legal corpora, proprietary datasets).
- **Semantic caching:** Research into caching based on semantic similarity rather than exact prefix matching. If two prompts differ by minor paraphrase but not meaning, could the same KV cache be reused? Early work (e.g., GPTCache, 2023) shows promise but introduces quality risks.
- **Cross-user caching:** Sharing KV caches across users for public content (e.g., everyone querying the same Wikipedia article). Requires careful privacy considerations; experimentally demonstrated but not yet productized.

---

## Memory-Augmented LLMs: Research Progress

The space between RAG and fully parametric models is getting more crowded:

**MemoRAG** (2025) combines a lightweight "global memory model" (a cheap LLM that reads the full document) with retrieval. The global model generates clues that guide retrieval, enabling multi-hop reasoning that standard RAG misses. Published results show improvement on tasks where RAG fails due to requiring cross-document reasoning.

**LONGMEM** (NeurIPS 2023) caches KV pairs from prior document segments and retrieves them via side network attention. Extends beyond context limits without the latency of full retrieval.

**Memory in the Age of AI Agents** (Survey, Shichun-Liu et al., 2025) — comprehensive survey covering in-context, external, parametric memory in agentic settings. Identifies the key open problems: selective forgetting, memory coherence, multi-agent memory sharing, and privacy-preserving memory.

**MemoryAgentBench** (Hu et al., 2025) — first comprehensive benchmark grounded in cognitive science, probing four competencies: accurate retrieval, test-time learning, long-range understanding, and selective forgetting. Most current LLMs perform poorly on selective forgetting (knowing what to not remember) — a capability humans use constantly.

---

## Open Problems

### 1. Memory Coherence Over Time
As agents accumulate memories across thousands of sessions, contradictions proliferate. User preferences change; facts become stale; earlier statements are superseded. No current system handles this gracefully at scale. The best approaches (mem0, Letta) do explicit conflict detection and resolution, but rely on LLMs to judge which memory is "current" — which itself hallucination-prone.

### 2. Selective Forgetting
Knowing what *not* to remember is as important as knowing what to store. Humans forget trivial details and retain meaningful patterns. LLM systems either remember everything (expensive, noisy) or apply fixed TTLs (blunt). Importance scoring (as in Generative Agents) is a start, but scoring accuracy degrades at scale and is expensive.

### 3. Catastrophic Forgetting in Fine-Tuning
Fine-tuning a model to learn new domain knowledge consistently degrades performance on prior tasks. Despite years of work (EWC, LoRA, continual learning research), catastrophic forgetting remains largely unsolved for general-purpose LLMs. Recent work (January 2026, mechanistic analysis of catastrophic forgetting) shows it's linked to specific attention head behaviors — suggesting targeted interventions may be possible.

### 4. Privacy in Memory Systems
External memory contains sensitive user information. Single-user agents can handle this with access control. Multi-user and multi-agent systems face harder problems: preventing cross-user memory leakage, honoring deletion requests, and complying with regulations like GDPR's right to erasure. Most current frameworks have no principled solution.

### 5. Efficient Multi-Agent Memory
When multiple agents work on the same task, shared memory introduces coordination problems: race conditions, conflicting updates, and the challenge of deciding which agent's observation is authoritative. This is an active research area with no dominant solution.

### 6. The Retrieval–Context Tradeoff
The choice between "put it in context" and "retrieve it on demand" depends on factors that change dynamically: current context budget, retrieval latency, how likely the information is needed. Optimal dynamic routing between retrieval and in-context usage remains an unsolved planning problem.

---

## What to Watch in 2026

- **KV cache compression standardization:** Techniques like GQA and SnapKV are converging into standard production stacks (vLLM, TensorRT-LLM). Expect 4-bit KV quant to become default.
- **Persistent cross-session prompt caching:** Provider APIs extending cache TTLs to days/weeks for enterprise use cases.
- **Infini-Attention and variants:** Will a compressive memory approach displace RAG for long-document tasks? The 2026 benchmark season will be decisive.
- **Memory benchmarking:** MemoryAgentBench and similar evaluations will pressure model providers to improve selective forgetting and long-range reasoning.
- **Agentic memory frameworks maturing:** Letta (MemGPT), mem0, and Zep are moving toward production-grade reliability. Expect consolidation and cloud-hosted managed memory services.

---

## See Also

- [[research/key-papers]] — foundational papers behind these developments
- [[concepts/kv-cache]] — the mechanical substrate most of this research builds on
- [[concepts/prompt-caching]] — the API-level embodiment of prefix caching
- [[patterns/memory-architectures]] — how SOTA techniques combine in production systems

## References

- Munkhdalai et al. (2024). *Leave No Context Behind: Efficient Infinite Context Transformers with Infini-Attention.* https://arxiv.org/abs/2404.07143
- Hu et al. (2025). *MemoryAgentBench.* (Cited in Memory for Autonomous LLM Agents survey, arxiv.org/html/2603.07670)
- Xiao et al. (2023). *Efficient Streaming Language Models with Attention Sinks.* https://arxiv.org/abs/2309.17453
- Tang et al. (2024). *InfLLM: Training-Free Long-Context Extrapolation for LLMs with an Efficient Context Memory.* https://arxiv.org/abs/2402.04617
- Liu et al. (2023). *Lost in the Middle: How Language Models Use Long Contexts.* https://arxiv.org/abs/2307.03172
- Liu et al. (2024). *Shichun-Liu/Agent-Memory-Paper-List.* https://github.com/Shichun-Liu/Agent-Memory-Paper-List
- Zhang et al. (2024). *PyramidKV: Dynamic KV Cache Compression based on Pyramidal Information Funneling.* https://arxiv.org/abs/2406.02069
