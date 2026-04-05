---
title: "File-Based Memory for AI Agents"
tags: [agent-filesystems, memory, markdown, persistence, MEMORY.md]
created: 2026-04-05
last_updated: 2026-04-05
---

## The Statelessness Problem

Every LLM-based agent session starts fresh. The model has no recollection of previous conversations, no awareness of decisions made last week, no knowledge that it already solved a problem yesterday. This is the fundamental statelessness of LLM agents — and it's why memory architecture matters.

File-based memory is one answer to this problem: the agent writes observations, decisions, and learned facts to disk during a session; future sessions read those files to reconstruct context. The files become the agent's exocortex — an external memory that persists where the model's weights cannot.

## Why Files Are a Natural Memory Medium

LLMs are unusually well-suited to file-based memory because of their training data. Markdown, plain text, YAML — the formats used for file-based memory — are formats LLMs have consumed billions of examples of. Reading and writing structured Markdown is not just possible for modern LLMs; it's something they're genuinely good at.

This matters. An agent that needs to read a vector database query result needs middleware; an agent that reads a `.md` file is just reading a file. The simplicity compounds: simpler systems have fewer failure modes and are easier to debug.

## The Three-File Pattern

Several independent agent systems have converged on a similar minimal pattern for file-based memory:

**A long-term memory index** (`MEMORY.md` or similar) — a curated, human-readable summary of what the agent knows persistently about the user, the project, and the world. Kept intentionally short (under 2KB ideally) to fit comfortably in the context window. The agent reads this every session.

**Daily logs** (`memory/YYYY-MM-DD.md`) — raw per-session notes, observations, and task outputs appended as the agent works. Not loaded by default; consulted when recent history is needed. Over time, significant items are promoted to the long-term index.

**Active state** (`memory/active-tasks.md` or similar) — a live tracker of in-progress work: what's running, what's waiting, what's blocked. Critical for crash recovery and multi-session tasks.

This pattern appears in Claude Code's auto-memory system (MEMORY.md, 200-line limit), in the ReMe memory framework from AgentScope (identical directory structure), in the mem-agent work from Hugging Face (Obsidian-style linked markdown), and in the Manus agent (task_plan.md + notes.md + output file).

## The ReMe Architecture

The ReMe framework (open-sourced by AgentScope) formalizes this structure explicitly:

```
working_dir/
├── MEMORY.md                    # Long-term: persistent user preferences
├── memory/
│   └── YYYY-MM-DD.md           # Daily journal: auto-written each conversation
├── dialog/                      # Raw conversation records
│   └── YYYY-MM-DD.jsonl
└── tool_result/                 # Cache for long tool outputs (auto-managed)
    └── <uuid>.txt
```

The `tool_result/` directory is an important detail: long tool outputs (web searches, API responses, code execution results) are written to files rather than kept in the context window. The agent receives a file path and loads the content only when needed. This is a form of [[wiki/agent-filesystems/patterns/tool-results-as-files|tool results as files]] pattern.

## Memory Hierarchy in Practice

A production file-based memory system typically operates at four levels:

1. **Session context** — what's in the context window right now (ephemeral)
2. **Session log** — what happened in this session, written to today's daily file (persistent, raw)
3. **Curated memory** — distilled insights promoted to MEMORY.md (persistent, refined)
4. **Project knowledge** — reference material, documentation, wiki articles (persistent, structured)

The agent moves information *up* this hierarchy deliberately: raw observations → session log → curated memory. Moving down (reading from higher levels into context) happens at session start.

## Limits and When to Graduate

File-based memory works well up to roughly:
- A few thousand facts in total
- MEMORY.md under ~25KB (to fit in context with room to spare)
- Daily logs spanning months to a year
- A single agent or tightly coupled pair of agents

Beyond these limits, you need augmentation:
- Semantic search over the file corpus (a lightweight vector index or BM25)
- Summarization pipelines to compress old logs without losing key facts
- A structured layer (SQL) for facts that need query/filter/aggregate operations

See [[wiki/agent-filesystems/concepts/file-vs-memory|Files vs Memory Stores]] for the full tradeoff analysis.

## Key Takeaways

- File-based memory solves agent statelessness by persisting observations and decisions to disk between sessions
- LLMs are natively suited to reading and writing Markdown — no middleware required
- The three-file pattern (MEMORY.md index + daily logs + active state) has emerged independently across multiple agent frameworks
- Information flows up a hierarchy: raw session log → curated memory → long-term index
- Files scale comfortably to thousands of facts; augment with vector search or SQL when you hit limits

## See Also

- [[wiki/agent-filesystems/patterns/memory-files|Memory Files Pattern]] — How to structure and maintain your memory files in practice
- [[wiki/agent-filesystems/concepts/file-vs-memory|Files vs Memory Stores]] — When to graduate to vector DBs or SQL
- [[wiki/agent-filesystems/patterns/tool-results-as-files|Tool Results as Files]] — Handling large tool outputs
- [[wiki/agent-memory/_index|Agent Memory]] — The broader memory architecture landscape
