---
title: Global Wiki Index
tags: [index, navigation]
created: 2026-04-04
last_updated: 2026-04-04
---

## Rowan's Personal Wiki

This is the master index across all knowledge domains in the wiki. Each topic is maintained in `wiki/<topic>/` with its own `_index.md`.

---

## Topics

### 🧠 Agent Memory
Everything about AI agent memory systems — concepts, architectures, research.

**[[wiki/agent-memory/_index|→ Agent Memory Index]]**

Key articles:
- [[wiki/agent-memory/overview|Overview]] — What agent memory is and why it matters
- [[wiki/agent-memory/architectures/rag|RAG]] — Retrieval-augmented generation
- [[wiki/agent-memory/architectures/memgpt|MemGPT]] — Virtual context management
- [[wiki/agent-memory/architectures/mem0|mem0]] — Managed memory layer
- [[wiki/agent-memory/research/key-papers|Key Papers]] — Annotated bibliography

---

### 📁 Agent Filesystems
How agents use filesystems for memory, context, and state — workspace patterns, file-based memory, GitOps for agents.

**[[wiki/agent-filesystems/_index|→ Agent Filesystems Index]]**

Key articles:
- [[wiki/agent-filesystems/overview|Overview]] — Why filesystems are a distinct design concern for agents
- [[wiki/agent-filesystems/concepts/workspace-as-context|Workspace as Context]] — AGENTS.md, CLAUDE.md patterns
- [[wiki/agent-filesystems/concepts/file-vs-memory|File vs Memory]] — Files vs vector DBs vs SQL vs graphs
- [[wiki/agent-filesystems/patterns/gitops-for-agents|GitOps for Agents]] — Git as agent state management
- [[wiki/agent-filesystems/patterns/memory-files|Memory Files]] — MEMORY.md, daily logs, active-tasks patterns

Also add to raw ingest queue: `raw/agent-filesystems/`

---

## Adding New Topics

When adding a new knowledge domain:

1. Create `wiki/<topic>/` directory
2. Create `wiki/<topic>/_index.md` following the agent-memory index format
3. Add a section in this file linking to it
4. Ask Sancho: "Add a new wiki section on <topic>" and describe what to cover

---

## Raw Ingest Queue

The `raw/<topic>/` directories are drop zones for unprocessed notes, links, PDFs, and scratchpad content. Sancho processes these on request and promotes them to wiki articles.

Current queues:
- `raw/agent-memory/` — drop agent memory notes/links here

---

## Wiki Philosophy

- **Substantive over shallow** — 300-800 words of real content per article, not bullet-point summaries
- **Wikilinked** — articles reference each other via `[[wikilinks]]`; navigation should feel natural
- **Living document** — articles are updated as knowledge evolves, not replaced
- **Opinionated** — where there's a clear better answer, say so (e.g., "pgvector is underrated")
- **Source-linked** — link to papers, repos, and primary sources

---

## Obsidian Setup

This wiki is designed for use with Obsidian. Open the `mywiki/` directory as an Obsidian vault.

Key settings (in `.obsidian/app.json`):
- Default view: Preview mode (rendered markdown)
- Readable line length: Enabled
- Fold indent: Enabled

For sync: use the Obsidian Git plugin pointed at this repository.
