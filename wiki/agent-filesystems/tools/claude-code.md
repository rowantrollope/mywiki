---
title: "Claude Code: Workspace Files and Auto-Memory"
tags: [agent-filesystems, claude-code, CLAUDE.md, auto-memory, anthropic]
created: 2026-04-05
last_updated: 2026-04-05
---

## Overview

Claude Code is Anthropic's agentic coding tool — a terminal-based coding agent that reads codebases, edits files, runs commands, and integrates with development tools. Released in February 2025, it introduced one of the most mature implementations of file-based agent context.

Its memory architecture uses two complementary systems loaded at the start of every session: **CLAUDE.md files** (instructions you write) and **auto-memory** (notes Claude writes itself).

## CLAUDE.md Files

CLAUDE.md files are Markdown files that give Claude persistent, session-spanning instructions. You write them; Claude reads them every time it starts in that project.

**Hierarchical scoping** — CLAUDE.md files can exist at multiple levels, with more specific files taking precedence:

| Scope | Location | Who manages |
|---|---|---|
| Organization policy | `/etc/claude-code/CLAUDE.md` (Linux) | IT/DevOps |
| Personal workflow | `~/.claude/CLAUDE.md` | You |
| Project instructions | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team (in git) |
| Subdirectory rules | `./src/auth/CLAUDE.md` | Team (in git) |

This allows an organization to enforce security policies (managed policy level), teams to share project conventions (project level), and individual developers to maintain personal preferences (user level) — all composing cleanly.

**.claude/rules/ for targeted scoping** — Claude Code supports a `.claude/rules/` directory where individual rule files can be scoped to specific file types using front matter. A rule for TypeScript files doesn't get loaded when Claude is working on Python.

## Auto-Memory

Auto-memory is Claude Code's mechanism for the agent to write its own persistent notes. Unlike CLAUDE.md (which you write), auto-memory is maintained by Claude based on what it learns from your corrections and preferences.

Auto-memory lives at `.claude/memory/MEMORY.md` (per working tree). It's loaded into every session — up to the first 200 lines or 25KB, whichever comes first. When the file exceeds this limit, Claude is instructed to curate it, removing stale entries and compressing older learnings.

The division of labor:
- **CLAUDE.md** → your explicit rules and project knowledge
- **MEMORY.md** → Claude's accumulated learnings and observed preferences

Subagents in Claude Code can also maintain their own auto-memory, with the subagent system prompt automatically including the first 200 lines of their MEMORY.md.

## How Context Is Loaded at Session Start

Every Claude Code session starts by loading:
1. Managed policy CLAUDE.md (if present)
2. User-level CLAUDE.md (if present)  
3. Project CLAUDE.md files (working directory and parents, up to git root)
4. Applicable `.claude/rules/` files
5. MEMORY.md (first 200 lines / 25KB)

This creates a layered initial context that carries all persistent knowledge into the session before the first user message.

## What to Put in CLAUDE.md

Anthropic's best practice is to put CLAUDE.md in version control so all team members benefit from consistent agent behavior. Recommended content:

- Build, test, and lint commands specific to your project
- Code style and naming conventions
- Architecture constraints and key design patterns
- Anti-patterns to avoid ("never call DB directly from service layer")
- PR creation workflow, branch naming, review process

The research finding holds: keep it concise and specific. Inferrable information wastes tokens. Human-written content beats LLM-generated content.

## The /memory Command

Claude Code exposes a `/memory` command for directly editing auto-memory content from within a session. This lets you inject specific facts or preferences without waiting for Claude to infer them. Combined with the `#` key (which adds content to MEMORY.md mid-conversation), it creates an interactive memory management system.

## Key Takeaways

- Claude Code implements a two-tier memory system: human-written CLAUDE.md + agent-written MEMORY.md
- CLAUDE.md supports hierarchical scoping from organization to project to subdirectory
- Auto-memory (MEMORY.md) loads the first 200 lines / 25KB every session; Claude curates it when it exceeds this limit
- Version-control CLAUDE.md so your team shares consistent agent behavior
- The `/memory` command and `#` key let you update auto-memory interactively during sessions

## See Also

- [[wiki/agent-filesystems/concepts/workspace-as-context|Workspace as Context]] — The broader pattern
- [[wiki/agent-filesystems/patterns/workspace-instructions|Workspace Instructions]] — How to write effective CLAUDE.md files
- [[wiki/agent-filesystems/patterns/memory-files|Memory Files]] — Memory architecture patterns
- [[wiki/agent-filesystems/tools/openai-codex|OpenAI Codex]] — Codex's AGENTS.md approach
