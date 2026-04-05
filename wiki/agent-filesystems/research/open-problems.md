---
title: "Open Problems in Agent Filesystems"
tags: [agent-filesystems, research, open-problems, future-work]
created: 2026-04-05
last_updated: 2026-04-05
---

## Unsolved Challenges

Agent filesystems are a young design space. The patterns documented in this wiki represent what works *today*, but significant problems remain unsolved — or solved only partially. These are the frontier issues that will shape how agent filesystems evolve.

---

## Indexing and Search at Scale

**The problem:** File-based memory works well up to a few thousand files. Beyond that, an agent can't find what it needs by listing directories and reading files sequentially. It needs search — and the search problem for agent filesystems is unsolved in a satisfying way.

**Current approaches:**
- BM25 keyword search (simple, works surprisingly well, but misses semantic similarity)
- Vector embeddings over file contents (requires infrastructure, stale when files change)
- LLM-generated search queries over file indexes (creative but unreliable)

**What's missing:** A lightweight, zero-infrastructure semantic search that stays in sync with the filesystem. Something like "SQLite for agent file search" — embedded, fast, and always consistent with the files on disk. The memsearch library (Zilliz, 2026) is an early attempt at this, but the space needs more work.

---

## Context Selection: What to Load When

**The problem:** An agent's context window can hold maybe 100-200KB of text. The agent's file corpus might be 10MB. Which files should the agent load for a given task?

**Current approaches:**
- Load everything at fixed paths (MEMORY.md, CLAUDE.md) every session — simple but inflexible
- Agent decides dynamically which files to read — works but burns tokens on navigation
- Pre-computed relevance scores based on task type — requires task classification

**What's missing:** Adaptive context selection that learns from past sessions which files were useful for which tasks. Something like: "when the agent is doing code review, these 5 files are almost always loaded; when doing research, these other 3 files matter." This is implicit in how humans navigate file systems but not yet formalized for agents.

---

## Concurrent Multi-Agent Access

**The problem:** When multiple agent sessions operate on the same workspace simultaneously — which is increasingly common with orchestrator + subagent architectures — file conflicts arise. Two agents writing to MEMORY.md at the same time can corrupt state.

**Current approaches:**
- Sequential execution (only one agent writes at a time) — safe but slow
- Agent-specific files (each agent writes to its own namespace) — avoids conflicts but fragments state
- Git branches per agent with merge — principled but heavyweight

**What's missing:** A lightweight concurrent-safe file layer for agent workspaces. Something between "one writer at a time" and "full database ACID." File locking semantics designed for LLM agent access patterns — short writes, infrequent, append-mostly.

---

## Memory Summarization and Compression

**The problem:** Memory files grow. Daily logs accumulate. After six months, there might be 180 daily log files totaling several megabytes. MEMORY.md tends toward bloat without active curation. How do you compress old memory without losing important context?

**Current approaches:**
- Manual curation (human reviews and prunes periodically) — high quality but doesn't scale
- Agent-driven summarization (agent reads old logs, writes summaries) — scales but quality degrades
- Time-based archival (archive logs older than N days) — simple but lossy

**What's missing:** Reliable automated memory distillation. An agent (or specialized pipeline) that reads a month of daily logs and produces a summary that retains all critical facts while compressing by 90%. Current LLMs can do this reasonably well for small corpora but struggle with consistency over large time spans.

---

## Access Control and Information Boundaries

**The problem:** Not all files should be accessible to all agents or all sessions. A personal assistant agent in a group chat shouldn't load MEMORY.md (which contains private context). A subagent spawned for a narrow task shouldn't have access to the full workspace.

**Current approaches:**
- Convention-based ("only load MEMORY.md in main session") — fragile, depends on agent compliance
- Directory-scoped tooling (LangChain's `root_dir`) — works but coarse-grained
- No access control at all — the most common reality

**What's missing:** A lightweight permission model for agent file access. Not full OS-level ACLs, but something like: "this agent session has read access to `knowledge/` and read-write access to `memory/today.md`, and no access to `secrets/`." The arXiv:2512.05470 paper proposes access control as part of their filesystem abstraction, but practical implementations are sparse.

---

## File Format Standardization

**The problem:** Every agent framework has slightly different conventions for memory files, config files, and knowledge bases. CLAUDE.md vs AGENTS.md vs .cursorrules. MEMORY.md vs memory.json vs memory.db. YAML frontmatter vs JSON headers vs no metadata at all.

**Current approaches:**
- De facto standards emerging from popular tools (CLAUDE.md from Claude Code, AGENTS.md from Codex)
- The agents.md community proposing AGENTS.md as a cross-tool standard
- No standardization for memory file formats

**What's missing:** A specification — even a loose one — for agent memory file formats that multiple tools and frameworks can interoperate on. The ReMe library adopts the MEMORY.md + daily logs convention, and memsearch (Zilliz) explicitly follows the "OpenClaw-style daily markdown" pattern, suggesting convergence is happening bottom-up.

---

## Versioning and Rollback in Practice

**The problem:** Git provides versioning and rollback for agent files, but agents don't actually use git for recovery today. If MEMORY.md gets corrupted, the agent doesn't know to `git revert` — it just reads the corrupted file and proceeds.

**Current approaches:**
- Agents commit regularly (good for human rollback, not agent-driven)
- Manual recovery (human notices the problem and reverts)
- No rollback at all (most common)

**What's missing:** Agent-driven integrity checking and self-healing. At session start, the agent could verify that memory files pass basic consistency checks (valid YAML frontmatter, expected sections present, no obvious corruption). On failure, it could automatically revert to the last known-good commit. This is straightforward in principle but not implemented in practice.

## Key Takeaways

- Indexing and search at scale is the most pressing technical gap — keyword search works but semantic search over changing files remains unsolved
- Context selection (which files to load for this task) is an optimization problem with no automated solution yet
- Concurrent multi-agent access needs lightweight coordination primitives — not full databases, but more than file locks
- Memory compression, access control, format standardization, and agent-driven rollback are all areas where current practice relies on conventions that break under pressure

## See Also

- [[wiki/agent-filesystems/research/key-papers|Key Papers]] — Research addressing these problems
- [[wiki/agent-filesystems/concepts/file-vs-memory|Files vs Memory Stores]] — When to graduate beyond files
- [[wiki/agent-filesystems/patterns/gitops-for-agents|GitOps for Agents]] — Versioning as a partial solution
- [[wiki/agent-filesystems/concepts/context-engineering|Context Engineering]] — The discipline these problems exist within
