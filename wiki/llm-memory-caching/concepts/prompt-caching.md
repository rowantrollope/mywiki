---
title: Prompt Caching — API-Level Prefix Caching
tags: [prompt-caching, anthropic, openai, google, api, cost-optimization]
created: 2026-04-06
last_updated: 2026-04-06
---

## What Is Prompt Caching?

Prompt caching (also called context caching or prefix caching) is an API-level feature that lets inference providers reuse the computed [[concepts/kv-cache|KV cache]] for a repeated prompt prefix across multiple API calls. Instead of reprocessing your system prompt and documents on every request, the provider stores the intermediate attention computations and reads them back on subsequent calls.

The result: dramatically lower costs and latency for applications with stable, long prefixes.

## Why Prompts Are Expensive to Repeat

A typical production LLM application sends something like this on every API call:

```
[System prompt: 2000 tokens of instructions and persona]
[Retrieved documents: 5000 tokens of context]
[Few-shot examples: 3000 tokens]
[Conversation history: varies]
[User message: 100 tokens]
```

The first three sections are often identical across every request. Without caching, they're reprocessed — and charged — every time. For an application making 10,000 daily requests with a 10,000-token stable prefix at $3/M input tokens, that's $300/day on the prefix alone. With 90% prompt caching discount: $30/day.

## Provider Comparison (2025–2026)

### Anthropic (Claude)

Anthropic launched prompt caching in August 2024. It uses explicit **cache breakpoint markers** in the API — you annotate which parts of your prompt to cache using `"cache_control": {"type": "ephemeral"}`.

**Pricing:**
- Cache write: 1.25× the base input price (5-minute TTL) or 2× (1-hour TTL)
- Cache read: 0.1× the base input price (~90% savings)
- Regular input: 1× base price

**TTLs:**
- Default (5-minute): `"type": "ephemeral"` — cheapest cache write, shorter window
- Extended (1-hour): Available on supported models — higher write cost, but amortizes better across high-volume burst traffic

**Minimum cacheable prefix:** 1,024 tokens (Claude 3 Haiku: 2,048 tokens)

**How to use:**
```json
{
  "system": [
    {
      "type": "text",
      "text": "Your long system prompt here...",
      "cache_control": {"type": "ephemeral"}
    }
  ]
}
```

You can add up to 4 cache breakpoints per request. The cache is keyed on the exact prefix content — any change invalidates it.

### OpenAI (GPT-4o, o1, o3)

OpenAI launched automatic prompt caching in October 2024. Unlike Anthropic, it's **fully automatic** — no markup needed. OpenAI detects repeated prefixes automatically and applies caching.

**Pricing:**
- Cache read: 0.5× the base input price (50% savings)
- Cache write: included in normal input price (no extra charge)
- TTL: 1 hour (matches their extended tier)

**Minimum cacheable prefix:** 1,024 tokens

**Behavior:**
- No explicit control — OpenAI decides when to cache
- Works with text, images, and tool definitions
- Cache status visible in API response (`usage.cached_tokens`)

The tradeoff: less savings than Anthropic (50% vs 90%) but zero implementation work. For simple use cases, this is preferable.

### Google (Gemini / Vertex AI)

Google launched context caching for Gemini 1.5 in May 2024. It takes a different approach: you create an explicit **cache resource** that persists independently of any single request.

**Pricing:**
- Cache storage: charged per token per hour (e.g., $0.0004375/M tokens/hour for Gemini 1.5 Flash)
- Cache read: 0.25× the base input price (75% savings)
- Minimum TTL: 1 hour; Maximum: flexible (hours to weeks)

**Minimum cacheable content:** 32,768 tokens (much higher than Anthropic/OpenAI)

**How to use:**
```python
import google.generativeai as genai

cache = genai.caching.CachedContent.create(
    model="models/gemini-1.5-pro",
    contents=[large_document],
    ttl=datetime.timedelta(hours=24)
)
model = genai.GenerativeModel.from_cached_content(cache)
```

The explicit resource model is more powerful for very long-lived content (e.g., caching an entire codebase for hours), but requires more infrastructure to manage cache lifecycles.

## Provider Comparison Table

| Feature | Anthropic | OpenAI | Google |
|---------|-----------|--------|--------|
| Mechanism | Explicit markers | Automatic | Explicit resource |
| Cost reduction | ~90% | ~50% | ~75% |
| Cache write cost | 1.25–2× input | 1× (included) | Storage fee |
| TTL | 5 min or 1 hour | 1 hour | 1 hour – weeks |
| Min tokens | 1,024 | 1,024 | 32,768 |
| Max breakpoints | 4 per request | Automatic | Unlimited (per resource) |
| Complexity | Medium | Low | High |

## What Gets Cached

Cacheable content includes:
- System prompts and instructions
- Static retrieved documents
- Few-shot examples
- Tool/function definitions
- Conversation history (if treated as a stable prefix)

**What doesn't benefit:**
- User messages (change every request)
- Dynamic content injected mid-prompt
- Content after the last cache breakpoint (Anthropic)

## Cache Hit vs. Miss

A cache hit occurs when the server finds an existing valid KV cache for the exact prefix. Common reasons for misses:

1. **First request** — cache hasn't been populated yet
2. **TTL expiration** — cache entry has aged out
3. **Content change** — even a single character difference invalidates the cache for that token onward
4. **Concurrent requests** — race condition during cache population
5. **Cold node** — request routed to a server that doesn't have the cache

For Anthropic, the response includes `cache_creation_input_tokens` and `cache_read_input_tokens` in the usage object so you can monitor hit rate.

## Latency Impact

In addition to cost savings, prompt caching dramatically reduces time-to-first-token (TTFT) for cached prefixes. The prefill step (which is compute-bound) is skipped; only the new tokens need to be processed. This can cut TTFT by 50–80% for long-prefix requests, which is especially impactful for real-time applications.

## See Also

- [[concepts/kv-cache]] — the underlying mechanism that makes prompt caching work
- [[patterns/cache-optimization]] — practical guide to structuring prompts for maximum cache hits
- [[concepts/context-window-management]] — strategies for staying within context limits
- [[overview]] — the full memory and caching stack

## References

- Anthropic. (2024). *Prompt Caching.* https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching
- OpenAI. (2024). *Prompt Caching.* https://platform.openai.com/docs/guides/prompt-caching
- Google. (2024). *Context Caching.* https://ai.google.dev/gemini-api/docs/caching
- Gim et al. (2023). *Prompt Cache: Modular Attention Reuse for Low-Latency Inference.* https://arxiv.org/abs/2311.04934
