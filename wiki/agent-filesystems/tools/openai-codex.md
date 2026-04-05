---
title: "OpenAI Codex: The AGENTS.md Standard"
tags: [agent-filesystems, openai, codex, AGENTS.md, coding-agents]
created: 2026-04-05
last_updated: 2026-04-05
---

## Overview

OpenAI Codex (the 2025 cloud-based coding agent, not the 2021 code completion model) natively supports AGENTS.md as its project context mechanism. By adopting AGENTS.md, Codex aligned with the emerging open, tool-agnostic standard for AI agent configuration files rather than creating a proprietary format.

This was a deliberate choice: AGENTS.md was introduced by the open agents.md community as a format any tool could support, and OpenAI's adoption gave it significant credibility. Tens of thousands of GitHub repositories now contain AGENTS.md files.

## How Codex Uses AGENTS.md

When Codex starts working in a repository, it searches for AGENTS.md files and adds their contents to the initial context. The lookup follows a hierarchical search:

1. Repository root `AGENTS.md`
2. Subdirectory `AGENTS.md` files relevant to the current task
3. User-level AGENTS.md in home directory (personal preferences)

Files from multiple levels are merged — the agent gets project-level context composed with personal preferences, similar to Claude Code's hierarchical CLAUDE.md approach.

## What AGENTS.md Covers (Per the Standard)

The AGENTS.md specification recommends content in several categories:

**Environment setup** — prerequisites, installation commands, service dependencies

**Build and test workflow** — the exact commands to build, test, lint, and validate changes in this repository

**Code style and architecture** — naming conventions, patterns to follow, patterns to avoid

**Repository-specific context** — non-obvious relationships between components, key design decisions, migration states

**Workflow guidance** — PR conventions, branch strategy, how to run integration tests, deployment process

The spec explicitly recommends keeping files concise and focusing on non-inferrable information — content the agent can't determine by reading the code.

## The AGENTS.md Research Finding

A 2025 ETH Zurich study (arXiv:2602.11988, "Reassessing the Value of AGENTS.md") tested AGENTS.md files across 138 real-world Python tasks. Results:

- LLM-generated AGENTS.md files: **−3% task success rate**, **+20% inference costs** vs no file
- Human-written AGENTS.md files: **+4% task success rate**, **+19% inference costs** vs no file

The counter-intuitive finding: more context isn't always better. Auto-generated AGENTS.md files (which are common — many tools offer "generate AGENTS.md for this repo") fill the file with redundant, inferrable information. The agent then dutifully follows those instructions, running more tests than needed and increasing cost without improving outcomes.

Human-written files that stick to non-inferrable specifics do provide marginal gains — but even these come with a cost overhead. The practical recommendation: write AGENTS.md selectively and manually. Do not auto-generate.

## Tool-Agnostic Standard vs Proprietary Files

The distinction between AGENTS.md (open standard) and CLAUDE.md / copilot-instructions.md (tool-specific) matters in practice:

| Format | Tool | Portability |
|---|---|---|
| `AGENTS.md` | OpenAI Codex, any AGENTS.md-compatible tool | High |
| `CLAUDE.md` | Claude Code only | Low |
| `.cursorrules` | Cursor only | Low |
| `copilot-instructions.md` | GitHub Copilot only | Low |

For teams using multiple agents, AGENTS.md as the primary file with tool-specific files for supplementary instructions is a reasonable approach. The open standard means the investment in writing a good AGENTS.md benefits any compatible tool the team adopts.

## Adoption Trajectory

A 2025 academic study surveyed 466 open-source GitHub repositories and found: no established structure exists yet for AGENTS.md content; there is significant variation in what information developers include and how they present it (descriptive, prescriptive, prohibitive, conditional). The community is still figuring out best practices.

This is an opportunity for teams that invest in intentional AGENTS.md design — the bar for "good" AGENTS.md is currently low enough that thoughtful implementation stands out.

## Key Takeaways

- OpenAI Codex adopted AGENTS.md as an open, tool-agnostic standard — a meaningful endorsement that drove widespread adoption
- AGENTS.md hierarchical search (root → subdirectory → user home) composes project and personal context
- LLM-generated AGENTS.md files degrade performance; human-written focused files provide modest gains
- AGENTS.md portability across multiple agent tools makes it the right choice for multi-tool teams
- The community hasn't yet converged on best-practice structure — intentional design still differentiates

## See Also

- [[wiki/agent-filesystems/concepts/workspace-as-context|Workspace as Context]] — The broader workspace context pattern
- [[wiki/agent-filesystems/patterns/workspace-instructions|Workspace Instructions]] — How to write effective instruction files
- [[wiki/agent-filesystems/tools/claude-code|Claude Code]] — Claude's complementary CLAUDE.md approach
- [[wiki/agent-filesystems/research/key-papers|Key Papers]] — The ETH Zurich research on AGENTS.md effectiveness
