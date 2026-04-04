---
title: MemGPT — Virtual Context Management
tags: [agent-memory, memgpt, letta, virtual-context, os-analogy, architecture]
created: 2026-04-04
last_updated: 2026-04-04
---

## What Is MemGPT?

MemGPT (Memory GPT) is a research system and framework introduced in the 2023 paper "MemGPT: Towards LLMs as Operating Systems" by Packer et al. from UC Berkeley. The core idea: treat the LLM context window as CPU registers and use external storage as a hierarchical memory system, analogous to how operating systems manage memory with virtual addressing.

The project has since evolved into **Letta** (formerly MemGPT), an open-source framework for building stateful agents with long-term memory. It's one of the most influential ideas in the agent memory space.

## The OS Analogy

Modern operating systems manage memory across a hierarchy:
- **Registers/Cache** — tiny, fast, immediately accessible
- **RAM** — larger, fast, volatile
- **Disk/Virtual Memory** — huge, slower, persistent

When a running process needs more memory than fits in RAM, the OS pages data to disk and swaps it in as needed — transparently to the application.

MemGPT applies the same principle to LLM context:
- **In-context storage** (registers) — what's currently in the context window, immediately accessible to the model
- **External storage** (disk) — archival memory outside the context window, retrieved on demand
- **Functions to move data** — the model itself controls what gets paged in and out

The key innovation: **the LLM is an active participant in its own memory management**, not just a passive consumer of pre-loaded context.

## Architecture

### Main Context (In-Context)
The context window is divided into structured regions:

```
┌─────────────────────────────────────┐
│ SYSTEM BLOCK                        │
│ Agent role, personality, core facts │
├─────────────────────────────────────┤
│ HUMAN BLOCK                         │
│ Key facts about the user            │
├─────────────────────────────────────┤
│ PERSONA BLOCK                       │
│ Agent's self-model                  │
├─────────────────────────────────────┤
│ WORKING MEMORY (conversation)       │
│ Recent turns                        │
├─────────────────────────────────────┤
│ [Tool Results / Retrieved Memory]   │
└─────────────────────────────────────┘
```

Each block has a defined size limit. When a block overflows, the agent is responsible for compressing it.

### Archival Storage (External)
Unlimited external storage — effectively a persistent database. The agent can search it and load content into the in-context working memory on demand.

### Recall Storage
A searchable history of past conversations and agent actions. More structured than archival storage; enables temporal and relational queries.

### Memory Functions
The agent is given tool-call access to memory management functions:

```python
# The agent can call these as tools
core_memory_append(name="human", content="Rowan is based in San Francisco")
core_memory_replace(name="human", old_content="...", new_content="...")
archival_memory_insert(content="Detailed notes on the Redis migration plan...")
archival_memory_search(query="Redis migration", page=0)
conversation_search(query="authentication service", start_date="2026-01-01")
```

The agent decides *when* to call these functions — when it detects that something should be remembered, that working memory is getting full, or that it needs to retrieve something from archival storage.

## The Key Innovations

### Agent-Driven Memory Management
Most RAG systems have the memory management happening outside the model — a system retrieves relevant context and injects it. MemGPT puts the model in control. The LLM decides what to remember, what to compress, and what to retrieve. This enables more context-aware memory management but requires a capable model.

### Structured Memory Blocks
Rather than unstructured context, MemGPT uses typed, bounded memory blocks. This makes the memory system inspectable, debuggable, and auditable — you can see exactly what the agent "knows" at any moment.

### Eviction and Compression
When the context fills, the agent doesn't just drop old content — it actively summarizes and compresses it into archival storage, preserving information in compressed form. This is analogous to the OS flushing dirty pages to disk rather than just dropping them.

## Letta: The Production Framework

MemGPT has been productionized as **Letta** (https://www.letta.com). Letta is a full agent framework with:
- Persistent agent state across restarts
- REST API for agent communication
- Multi-agent communication support
- Human-in-the-loop capabilities
- Cloud-hosted managed agents (Letta Cloud)

Letta represents the maturation of the MemGPT ideas into a deployable system.

## Comparison with RAG-Based Memory

| Dimension | RAG | MemGPT/Letta |
|-----------|-----|--------------|
| Who manages retrieval | External system | The agent itself |
| Context structure | Unstructured injection | Typed, bounded blocks |
| Memory eviction | External policy | Agent-driven with summarization |
| Transparency | Medium | High (inspectable blocks) |
| Model requirements | Any | Requires capable instruction-following model |
| Complexity | Lower | Higher |

## Limitations

**Model dependency:** MemGPT requires the model to reliably call memory management functions at the right times. Weaker models frequently forget to update memory, over-call memory functions, or corrupt their own memory state.

**Latency:** Each memory operation is a tool call — adds round-trip latency. High-frequency memory operations can significantly slow agent response time.

**Complexity:** The structured block architecture is more complex to set up and debug than a simple RAG pipeline. The gain in memory management quality may not justify the overhead for simple use cases.

## Key Takeaways

- MemGPT applies the OS virtual memory analogy to LLM context management
- The agent actively manages its own memory — deciding what to load, evict, and compress
- Structured memory blocks (system, human, persona, working) replace unstructured context stuffing
- Letta is the production evolution of MemGPT — a full stateful agent framework
- Powerful but requires capable models; overkill for simple retrieval tasks

## See Also

- [[rag]]
- [[mem0]]
- [[vector-stores]]
- [[concepts/working-memory]]
- [[concepts/long-term-memory]]
- [[research/key-papers]]
