---
title: Anthropic Managed Memory API — Native Memory for Claude Agents
tags: [agent-memory, tools, anthropic, managed-agents, memory-stores, research-preview, api]
created: 2026-04-08
last_updated: 2026-04-08
---

## What Is It?

The Anthropic Managed Memory API is Anthropic's own first-party answer to agent memory — a workspace-scoped document store built directly into the Claude API. Rather than bolting on an external library (mem0, MemPalace, Zep), you provision a memory store through Anthropic's API and attach it to a Claude session. The agent then reads from it automatically before tasks and writes learnings back when done, with no prompt engineering required.

It's part of the **Managed Agents API**, currently in **Research Preview** as of early 2026. You opt in via a beta header:

```
anthropic-beta: managed-agents-2026-04-01
```

This is Anthropic-native, not a third-party integration. The tradeoffs flow from that: deep integration, automatic behavior, but also cloud dependency and the constraints of a hosted product.

---

## Architecture

Three objects make up the system:

```
Memory Store (memstore_...)
 └── Memory (path + content — like a file)
      └── Memory Version (immutable snapshot on every write)
```

### Memory Stores

A **memory store** is a named, workspace-scoped collection of text documents. "Workspace-scoped" means it persists across sessions and is shared by all API keys in the same Anthropic workspace.

```bash
POST /v1/memory_stores
{
  "name": "user-alice",
  "description": "Long-term memory for Alice's assistant sessions"
}
# Returns: { "id": "memstore_01abc...", "name": "user-alice", ... }
```

You can have as many stores as you want at the workspace level. The constraint is per-session (more on that below).

### Memories

Each **memory** is a text document with a path. Think of it like a filesystem: `/user/preferences`, `/project/auth/decisions`, `/team/conventions`. Paths are arbitrary strings — the convention you choose is up to you.

```bash
POST /v1/memory_stores/{store_id}/memories
{
  "path": "/user/preferences",
  "content": "Prefers TypeScript over JavaScript. Always wants error handling examples."
}
```

Individual memories are capped at **100KB / ~25K tokens**. The design guidance from Anthropic is explicit: **many small files, not few large ones**. This is not just a recommendation — it affects retrieval quality and token efficiency.

### Memory Versions

Every write creates an immutable **memory_version** — a timestamped snapshot of the document at that point. This provides:

- Full audit trail: who changed what and when
- Rollback to any prior version
- Diff history for debugging agent behavior

Memory versions are the closest thing to git commits in this system.

---

## How It Works in a Session

The key behavior: **automatic, zero-prompt memory I/O.**

When you attach a memory store to a session, Claude:

1. **Before the task** — reads the memory store automatically (scoped by the access configuration)
2. **After the task** — writes any learnings, corrections, or new facts back to the store

This is different from RAG, where you explicitly retrieve context before calling the model. It's also different from prompt-stuffing, where you manually include prior context. The agent decides what to read and write using a set of built-in memory tools — you don't orchestrate it.

### Attaching a Store to a Session

Memory stores are added to the `resources[]` array at session creation:

```json
POST /v1/sessions
{
  "model": "claude-opus-4",
  "resources": [
    {
      "type": "memory_store",
      "id": "memstore_01abc...",
      "access": "read_write",
      "prompt": "This store contains long-term memory for this user. Read relevant context before responding. After each session, write any new facts, preferences, or learnings you should remember for next time."
    }
  ]
}
```

**`access`** — controls what the agent can do:
- `read_write` (default) — agent can read and write
- `read_only` — agent reads but cannot modify (useful for shared reference stores)

**`prompt`** — optional, up to 4096 chars. Use this to tell the agent *how* to use this store: what kinds of things belong here, how to organize paths, when to write. Without a prompt, the agent uses judgment — with a prompt, you control the behavior.

**Session limit: 8 memory stores per session.** This is a hard cap. Plan your store architecture accordingly.

---

## Memory Tools

When a memory store is attached, Anthropic automatically adds six tools to the agent's toolset:

| Tool | What It Does |
|------|-------------|
| `memory_list` | List documents in the store; filter by path prefix |
| `memory_search` | Full-text search across all documents |
| `memory_read` | Read a specific document by path |
| `memory_write` | Create or overwrite a document |
| `memory_edit` | Modify an existing document (partial updates) |
| `memory_delete` | Remove a document |

The agent calls these tools directly during a session — you don't call them yourself. This is the auto-read/write behavior: the agent uses `memory_search` and `memory_read` to retrieve context before answering, and `memory_write` or `memory_edit` to save what it learned.

---

## Access Control and Multi-Store Patterns

The 8-store-per-session limit forces intentional architecture. Two patterns that work well:

### Pattern 1: Shared Reference + Per-Session Learnings

```
resources: [
  {
    type: "memory_store",
    id: "memstore_standards",      // read_only
    access: "read_only",
    prompt: "Team coding standards and conventions. Read-only reference."
  },
  {
    type: "memory_store",
    id: "memstore_session_alice",  // read_write
    access: "read_write",
    prompt: "Alice's personal preferences and session learnings."
  }
]
```

The shared store holds standards, conventions, documentation snippets — anything that shouldn't change session-to-session. The per-user store holds individual learnings. Multiple agents and users can share the same reference store without contaminating each other's memory.

### Pattern 2: Per-Project Isolation

One store per project/team. Each store scopes its context independently. Useful when agents handle multiple concurrent projects that shouldn't bleed into each other.

### Pattern 3: Lifecycle-Based Stores

- Long-lived store: user identity, stable preferences (rarely rewritten)
- Session-scoped store: task context for current session (cleared or archived when done)

Different stores can have different retention and access policies. You manage their lifecycle through the API — create, archive, or delete as appropriate.

---

## Versioning and Audit Trail

Every `memory_write` or `memory_edit` creates a new `memory_version`. This is automatic — you cannot disable it.

Practical implications:
- You can always roll back to a prior state if the agent writes something incorrect
- You have a full history of what the agent "learned" over time and when
- Debugging is tractable: compare memory versions to see what changed between sessions

This is one area where the Managed Memory API is meaningfully ahead of DIY approaches. Building immutable version history into your own memory layer isn't hard, but most teams don't bother. Here it's built in.

---

## API Walkthrough: End-to-End Example

```python
import anthropic

client = anthropic.Anthropic()

# 1. Create a memory store
store = client.beta.memory_stores.create(
    name="user-alice",
    description="Long-term memory for Alice's coding assistant",
    betas=["managed-agents-2026-04-01"]
)
store_id = store.id  # "memstore_01abc..."

# 2. Seed initial content
client.beta.memory_stores.memories.create(
    store_id,
    path="/user/profile",
    content="Alice is a senior TypeScript engineer. Works on a React/Node SaaS product. Prefers functional patterns, dislikes class-heavy OOP.",
    betas=["managed-agents-2026-04-01"]
)

client.beta.memory_stores.memories.create(
    store_id,
    path="/project/stack",
    content="Frontend: React 19, TypeScript, Tailwind. Backend: Node, Fastify, Prisma, Postgres.",
    betas=["managed-agents-2026-04-01"]
)

# 3. Attach to session
session = client.beta.sessions.create(
    model="claude-opus-4",
    resources=[{
        "type": "memory_store",
        "id": store_id,
        "access": "read_write",
        "prompt": "This is Alice's long-term coding context. Read relevant files before answering. After the session, write any new preferences or decisions Alice makes."
    }],
    betas=["managed-agents-2026-04-01"]
)

# 4. Agent now has memory_list, memory_search, memory_read, memory_write,
#    memory_edit, memory_delete tools available automatically.
#    It reads context before tasks and writes learnings after.
```

---

## Honest Assessment

### Strengths

**Zero integration overhead.** No vector database to provision, no embedding pipeline to build, no retrieval logic to write. If you're already using Claude via the API, adding managed memory is a few extra API calls. The agent handles the rest.

**Auto-read/write is a real DX win.** Most memory systems require you to orchestrate retrieval and injection explicitly. The Managed Memory API removes that layer — the agent reads what it needs and writes what it learns without you managing the loop.

**Versioning built in.** Immutable memory versions with full history and rollback is something you'd normally have to build yourself. Getting it for free is meaningful.

**Tight Claude integration.** Anthropic controls both the model and the memory API. They can optimize retrieval, tool calling behavior, and prompting strategies in ways a third-party library can't.

### Caveats

**Research Preview — production risk.** This is explicitly experimental. APIs may change, rate limits are unknown, and there's no SLA. Don't build production-critical workflows on it yet without a fallback plan.

**Cloud-only.** Memory stores live in Anthropic's infrastructure. Not suitable for airgapped deployments, strict data-residency requirements, or use cases where you need full data control.

**8 memory stores per session is a real constraint.** For complex multi-agent or multi-project setups, you'll need to design carefully. You can't attach 20 stores per session.

**100KB per memory is limiting for dense content.** Codebases, long documents, and rich conversation histories can exceed this quickly. You need to chunk thoughtfully.

**No cross-provider portability.** Memory stores live in Anthropic's API. If you switch models or providers, you can't natively port the stores. Unlike a local ChromaDB instance, your data is in Anthropic's ecosystem.

**Opaque retrieval.** You don't control how the agent decides what to read. The agent uses `memory_search` and `memory_read` based on its own judgment. That's convenient but can be unpredictable for memory-sensitive applications.

---

## Comparison to Other Approaches

| System | Approach | Hosting | Auto I/O | Versioning | Local? | Status |
|--------|----------|---------|----------|------------|--------|--------|
| **Anthropic Memory API** | Managed doc store | Anthropic cloud | Yes (native) | Yes (built-in) | No | Research Preview |
| **MemPalace** | Hierarchical local store | Self-hosted | Via hooks | Manual | Yes | MIT OSS |
| **mem0** | LLM extraction + dedup | Cloud or self-host | Semi (SDK) | No | Configurable | GA |
| **Zep / Graphiti** | Temporal KG + vector | Cloud or self-host | No | No | Enterprise only | GA |
| **RAG + vector store** | Embed + retrieve | Self-hosted or cloud | No | No | Yes | DIY |

**Anthropic Memory API vs MemPalace:** MemPalace is local-first, free, open-source, lossless by design, and benchmarks at 96.6% R@5 on LongMemEval. It requires more setup and is a third-party dependency. Anthropic's Memory API is simpler to start with, cloud-hosted, and natively integrated — but opaque and in preview.

**Anthropic Memory API vs mem0:** mem0 uses LLM-driven extraction to decide what to remember (lossy by design) and is provider-agnostic. The Anthropic API stores full documents and is Claude-only. For Claude-native projects, Anthropic's approach avoids the extraction quality problem; for multi-provider setups, mem0 is more flexible.

**Anthropic Memory API vs RAG:** RAG with your own vector store gives you full control — chunking strategy, retrieval logic, embedding model, hosting. The Anthropic Memory API trades that control for zero-setup convenience. RAG is the right choice when you have specific retrieval requirements or need portability; the Managed Memory API is the right choice when you want to move fast and trust Anthropic's defaults.

**The honest summary:** The Managed Memory API is the fastest path from zero to working agent memory on Claude, and the versioning story is legitimately better than most DIY approaches. But it's cloud-locked, in preview, and opaque. Use it for prototypes and internal tools today; for production systems, pair it with an export strategy or wait for GA.

---

## Key Takeaways

- Anthropic-native managed memory: workspace-scoped doc stores attached to Claude sessions
- Agent auto-reads context before tasks, auto-writes learnings after — no orchestration code needed
- Six memory tools injected automatically: list, search, read, write, edit, delete
- Every write creates an immutable `memory_version` — full audit trail + rollback
- Access control: `read_write` (default) or `read_only` per store per session
- Hard limit: 8 memory stores per session; 100KB per individual memory
- Multi-store patterns: shared reference (read-only) + per-user learnings (read-write)
- **Research Preview** — not production-ready without a fallback plan
- Best fit: Claude-native projects that want fast, zero-infra memory with versioning

---

## See Also

- [[mempalace]] — Local-first alternative; highest published LongMemEval score; fully offline
- [[architectures/mem0]] — Extraction-based memory; provider-agnostic; most direct comparison
- [[architectures/rag]] — DIY retrieval; full control; best for custom retrieval requirements
- [[architectures/knowledge-graphs]] — Graph-based memory for relational queries across entities
- [[concepts/long-term-memory]] — How long-term memory works conceptually in agents
- [[_index]] — Agent Memory topic index
