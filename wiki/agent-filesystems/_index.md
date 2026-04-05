---
title: Agent Filesystems
tags: [agent-filesystems, index, context-engineering, memory]
created: 2026-04-05
last_updated: 2026-04-05
---

## Agent Filesystems

How AI agents use the filesystem as their workspace, memory, and context delivery mechanism — from the AGENTS.md pattern to full git-native memory architectures.

This topic sits at the intersection of practical agent engineering and system design: the choices you make about *how* information lives on disk determine how well your agent reasons, remembers, and recovers.

---

## Overview

→ **[[wiki/agent-filesystems/overview|Overview]]** — What an agent filesystem is, why it's a distinct design concern, and the core mental model

---

## Concepts

Foundational ideas behind agent-filesystem design:

- [[wiki/agent-filesystems/concepts/workspace-as-context|Workspace as Context]] — The AGENTS.md / CLAUDE.md pattern: how coding agents use files to understand projects
- [[wiki/agent-filesystems/concepts/file-based-memory|File-Based Memory]] — Using the filesystem as persistent long-term memory
- [[wiki/agent-filesystems/concepts/structured-context-files|Structured Context Files]] — YAML frontmatter, schemas, and markdown conventions for agent-readable files
- [[wiki/agent-filesystems/concepts/context-engineering|Context Engineering]] — The discipline of structuring information for LLM consumption
- [[wiki/agent-filesystems/concepts/file-vs-memory|Files vs Memory Stores]] — Tradeoff analysis: flat files, vector DBs, SQL, graphs

---

## Patterns

Recurring design patterns for agent file systems:

- [[wiki/agent-filesystems/patterns/workspace-instructions|Workspace Instructions]] — AGENTS.md, CLAUDE.md, .cursorrules — writing persistent instructions
- [[wiki/agent-filesystems/patterns/memory-files|Memory Files]] — MEMORY.md, daily logs, active-tasks — the agent continuity stack
- [[wiki/agent-filesystems/patterns/knowledge-files|Knowledge Files]] — Wiki-style knowledge bases as agent reference material
- [[wiki/agent-filesystems/patterns/tool-results-as-files|Tool Results as Files]] — Writing tool output to disk for future agent context
- [[wiki/agent-filesystems/patterns/gitops-for-agents|GitOps for Agents]] — Using git commits as versioned agent state snapshots

---

## Tools

How major tools and frameworks approach filesystem context:

- [[wiki/agent-filesystems/tools/claude-code|Claude Code]] — CLAUDE.md and auto-memory
- [[wiki/agent-filesystems/tools/openai-codex|OpenAI Codex]] — AGENTS.md-native approach
- [[wiki/agent-filesystems/tools/langchain-file-tools|LangChain File Tools]] — Filesystem tool integration
- [[wiki/agent-filesystems/tools/autogen-workspace|AutoGen Workspace]] — Conversation-driven file management

---

## Research

- [[wiki/agent-filesystems/research/key-papers|Key Papers]] — Annotated bibliography of agent filesystem research
- [[wiki/agent-filesystems/research/open-problems|Open Problems]] — Unsolved challenges: indexing, search, versioning, access control

---

## Related Topics

- [[wiki/agent-memory/_index|Agent Memory]] — The broader memory landscape (vector DBs, RAG, MemGPT, etc.)
