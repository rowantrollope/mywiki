---
title: "Workspace as Context: AGENTS.md, CLAUDE.md, and the Config File Pattern"
tags: [agent-filesystems, workspace, AGENTS.md, CLAUDE.md, context-engineering, coding-agents]
created: 2026-04-05
last_updated: 2026-04-05
---

## The Core Idea

When an AI coding agent opens your project for the first time, it knows nothing about your conventions, your architecture, or why certain decisions were made. It will infer what it can from code, but it will miss things — the non-obvious tooling, the team's naming preferences, the forbidden patterns, the test commands that actually work.

The *workspace-as-context* pattern solves this by encoding project knowledge into a file the agent reads automatically at the start of every session. That file is variously named `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `.cursorrules`, `copilot-instructions.md`, or `AGENTS.md` — the name depends on the tool, but the idea is the same.

## How It Works

The file lives in the project root (or a designated config directory). When the agent starts, it reads the file and injects its contents into the initial context window. Every subsequent conversation in that project benefits from those instructions without the developer having to re-explain anything.

A well-structured CLAUDE.md might contain:
- **Project structure** — key directories and what lives where
- **Build and test commands** — the exact commands that work in this repo
- **Coding conventions** — naming styles, import patterns, error handling approach
- **Architecture notes** — key design decisions and the reasoning behind them
- **Forbidden patterns** — things the agent should never do in this codebase
- **Workflow guidance** — how to create PRs, what branches to use, review process

Claude Code's official documentation puts it succinctly: CLAUDE.md files "give Claude persistent context for a project, your personal workflow, or your entire organization." The files can be scoped at multiple levels — organization-wide (managed policy), project-specific (repo root), and personal (user home directory).

## The Convergence Across Tools

By early 2026, every major AI coding tool had adopted some variant of this pattern:

| Tool | File Name |
|------|-----------|
| Claude Code (Anthropic) | `CLAUDE.md` |
| OpenAI Codex | `AGENTS.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Cursor | `.cursorrules` |
| Windsurf | `.windsurfrules` |
| Gemini CLI | `GEMINI.md` |

`AGENTS.md` has emerged as a proposed open, tool-agnostic standard that consolidates these formats. A 2025 research paper (arXiv:2510.21413) surveyed 466 open-source projects and found tens of thousands of GitHub repositories containing AI configuration files — with adoption growing rapidly throughout 2025.

## What Goes in a Good Config File

The same ETH Zurich research found nuanced results: LLM-generated context files *degraded* agent performance (−3% task success, +20% inference costs), while carefully human-written files showed modest gains (+4% task success). The implication: quality matters more than quantity. Bloated auto-generated files with redundant or obvious information actively harm the agent.

Best practices:
- **Non-inferable only** — don't explain things the agent can figure out from reading the code
- **Be specific** — "run `npm run test:unit -- --watch=false`" beats "run the tests"
- **Use prescriptive language** — "Always import from `@company/utils-v2`" is clearer than "try to use the v2 utils"
- **Keep it concise** — context window real estate is finite; every token should earn its place
- **Version control it** — the file evolves with the project; commit history shows how conventions changed

## Hierarchical Scoping

Claude Code's implementation shows the sophistication this pattern can reach. CLAUDE.md files can exist at multiple directory levels, with more specific files taking precedence over broader ones:

```
/etc/claude-code/CLAUDE.md          # Organization policy (managed by IT)
~/CLAUDE.md                         # Personal preferences
~/myproject/CLAUDE.md               # Project-level rules
~/myproject/src/CLAUDE.md           # Subdirectory-specific rules
```

This enables a team to share project conventions while each developer maintains personal preferences — and an organization to enforce security policies across all projects.

## The Auto-Memory Extension

Claude Code adds an interesting extension: *auto memory*, where Claude itself writes notes to a MEMORY.md file as it learns your preferences through corrections. This is distinct from CLAUDE.md (which you write) — it's the agent accumulating its own context over time. The first 200 lines or 25KB are loaded into every session.

This creates a two-tier system: human-written instructions plus machine-accumulated learnings, both delivered as files at session start.

## Key Takeaways

- Workspace config files (AGENTS.md, CLAUDE.md, etc.) solve the "blank slate" problem by encoding project context the agent reads every session
- Every major coding agent tool has converged on this pattern as of 2025-2026; AGENTS.md is emerging as a tool-agnostic standard
- Human-written files help; LLM-generated files often hurt — quality and specificity matter more than comprehensiveness
- Files should contain non-inferable information only: custom tooling, conventions, forbidden patterns, workflow specifics
- Hierarchical scoping (org → project → directory → personal) enables shared team standards with individual customization

## See Also

- [[wiki/agent-filesystems/patterns/workspace-instructions|Workspace Instructions Pattern]] — Practical guide to writing good config files
- [[wiki/agent-filesystems/tools/claude-code|Claude Code]] — Claude's implementation in depth
- [[wiki/agent-filesystems/tools/openai-codex|OpenAI Codex]] — Codex's AGENTS.md approach
- [[wiki/agent-filesystems/concepts/context-engineering|Context Engineering]] — The broader discipline
- [[wiki/agent-filesystems/research/key-papers|Key Papers]] — Research on config file effectiveness
