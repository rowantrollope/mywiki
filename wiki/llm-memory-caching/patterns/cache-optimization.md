---
title: Cache Optimization — Structuring Prompts for Maximum Cache Hits
tags: [cache-optimization, prompt-caching, prompt-engineering, cost-optimization, patterns]
created: 2026-04-06
last_updated: 2026-04-06
---

## The Core Principle

Prompt caching works by matching a new request's prefix against cached prefixes. For a cache hit, **every byte of the prefix must match exactly**. Cache optimization therefore boils down to one rule:

> **Make the stable parts of your prompt as long as possible and put them first.**

Everything else is application of this rule.

## The Golden Prompt Structure

The optimal prompt structure for caching is:

```
1. [STATIC]  System prompt + persona + instructions (longest part)
2. [STATIC]  Documents, examples, reference material
3. [DYNAMIC] Conversation history (or recent history only)
4. [DYNAMIC] Current user message
```

Dynamic content always comes last. If dynamic content appears before static content, it breaks the cache for everything that follows — even if the static content is identical.

**Bad (breaks cache):**
```
System: You are a helpful assistant.
User: [DYNAMIC user message]
Context: [STATIC large document]  ← cache miss: different request ID
```

**Good:**
```
System: You are a helpful assistant. [STATIC document appended here]
User: [DYNAMIC user message]
```

## Anthropic (Claude) Cache Breakpoints

Anthropic requires explicit `cache_control` markers. You control exactly where cache boundaries are.

### Basic pattern (single cache breakpoint):
```json
{
  "system": [
    {
      "type": "text",
      "text": "Your full system prompt with all static content...",
      "cache_control": {"type": "ephemeral"}
    }
  ],
  "messages": [
    {"role": "user", "content": "What is in the document?"}
  ]
}
```

### Multi-breakpoint pattern (up to 4 breakpoints):
Use multiple breakpoints when you have distinct static blocks that may be updated independently:

```json
{
  "system": [
    {
      "type": "text",
      "text": "Core instructions...",
      "cache_control": {"type": "ephemeral"}   ← breakpoint 1
    },
    {
      "type": "text",
      "text": "Tool definitions (occasionally updated)...",
      "cache_control": {"type": "ephemeral"}   ← breakpoint 2
    }
  ],
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "Reference document...",
          "cache_control": {"type": "ephemeral"}   ← breakpoint 3
        },
        {"type": "text", "text": "What's in the document?"}
      ]
    }
  ]
}
```

**Key insight:** Each breakpoint creates a separate cache entry. Cache 1 caches up to breakpoint 1; cache 2 caches up to breakpoint 2 (and re-uses cache 1 as its prefix). If document changes but instructions don't, cache 1 is still hit — you only pay for breakpoint 2 re-computation.

### 5-minute vs 1-hour TTL
- Default ephemeral TTL is **5 minutes** — cheapest cache write (1.25× input price)
- You can request **1-hour TTL** — higher cache write cost (2× input price) but better for bursty traffic
- Use 1-hour TTL when request rate is lower but burst intervals exceed 5 minutes

### How to check cache hits:
```python
response = client.messages.create(...)
usage = response.usage
print(f"Cache write: {usage.cache_creation_input_tokens}")
print(f"Cache read: {usage.cache_read_input_tokens}")
print(f"Regular input: {usage.input_tokens}")
```

## OpenAI (GPT-4o, o1, o3) — Automatic Caching

OpenAI's caching is automatic — no markup required. Just structure your prompt with stable content first.

**Check cache hits:**
```python
response = client.chat.completions.create(...)
usage = response.usage
print(f"Cached tokens: {usage.prompt_tokens_details.cached_tokens}")
```

**Best practices for OpenAI:**
- Keep system prompt content stable and long (1024+ tokens)
- Don't insert dynamic timestamps or UUIDs in the system prompt
- Use consistent message structure across requests
- Tool definitions (functions) are also cacheable — keep them stable

**When OpenAI cache doesn't fire:**
- New conversation with fresh messages — the history won't match until several turns accumulate
- Using different model versions (e.g., switching between `gpt-4o` and `gpt-4o-2024-08-06`)
- Requests where the total prompt is under 1024 tokens

## Google Gemini — Explicit Cache Resources

Google's approach requires managing cache resources explicitly, which adds complexity but gives maximum control.

```python
import google.generativeai as genai
import datetime

# Create a persistent cache resource
cache = genai.caching.CachedContent.create(
    model="models/gemini-1.5-pro-001",
    system_instruction="You are an expert code reviewer...",
    contents=[{"role": "user", "parts": [large_document_text]}],
    ttl=datetime.timedelta(hours=4)
)

# Use the cache in requests
model = genai.GenerativeModel.from_cached_content(cached_content=cache)
response = model.generate_content("Review this code")
```

**Best practices for Google:**
- Cache large, stable documents (minimum 32K tokens — plan for this)
- Manage TTLs explicitly; extend before expiration for long-lived content
- Track cache names to reuse across services/instances
- Monitor cache cost vs. savings: the storage fee accumulates even with no reads

## Universal Best Practices

### 1. Never put volatile data in static positions
Common mistakes:
- Including current date/time in system prompt: `"Today is {{date}}"` — breaks cache every day
- User-specific personalization in the shared cache prefix: `"You are assisting {{user_name}}"` — breaks cache per user
- Request IDs or trace IDs in context: always breaks cache

**Fix:** Put volatile data in the final user message, not in cached prefixes.

### 2. Consolidate static content
Instead of splitting into many small message turns, consolidate static content:

```
# Worse: many small static chunks scattered around
system: "You are helpful."
user: "Here is background: [100 tokens]"
assistant: "Understood."
user: "Here are the rules: [100 tokens]"

# Better: one large cacheable prefix
system: "You are helpful. Background: [100 tokens]. Rules: [100 tokens]."
user: "What is your question?"
```

### 3. Version your system prompts
When you update the system prompt, all cache entries for that prefix are invalidated. Plan prompt changes thoughtfully:
- Batch multiple small changes into fewer, larger changes
- Avoid changing prompts more than once per TTL window (no benefit to updates that outpace TTL)
- Use A/B test infrastructure to compare cached vs. non-cached variants

### 4. Measure, don't assume
Instrument your application to track:
- Cache hit rate (`cache_read_tokens / (cache_read_tokens + cache_miss_tokens)`)
- Cost per request with and without caching
- Time-to-first-token for cached vs. uncached requests

Target hit rate: >80% for stable prefixes in production traffic.

### 5. Warmup for cold starts
After deployment or cache expiry:
- The first request after a cold start always misses
- Send a warmup request immediately after deployment to populate the cache
- For Anthropic 5-minute TTL in very low-traffic applications, consider a warmup keepalive request every 4 minutes

## Cost Estimation Example

Application: Customer support bot, 10,000 requests/day, 8,000-token system prompt + docs, 500-token user messages.

**Without caching (Anthropic Claude Sonnet, $3/M input):**
- 10,000 × 8,500 tokens × $3/M = **$255/day**

**With prompt caching (90% reduction on 8,000 tokens):**
- Cache writes: 1 × 8,000 × $3.75/M = $0.03 (negligible)
- Cache reads: 10,000 × 8,000 × $0.30/M = **$24/day**
- User message tokens: 10,000 × 500 × $3/M = $15/day
- **Total: ~$39/day** — savings of $216/day (~85%)

## See Also

- [[concepts/prompt-caching]] — how prompt caching works across providers
- [[concepts/kv-cache]] — underlying mechanics
- [[concepts/context-window-management]] — managing what goes in context
- [[patterns/memory-architectures]] — broader memory system design

## References

- Anthropic. (2024). *Prompt Caching Documentation.* https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching
- Gim et al. (2023). *Prompt Cache: Modular Attention Reuse for Low-Latency Inference.* https://arxiv.org/abs/2311.04934
- PromptHub. (2025). *Prompt Caching with OpenAI, Anthropic, and Google Models.* https://www.prompthub.us/blog/prompt-caching-with-openai-anthropic-and-google-models
