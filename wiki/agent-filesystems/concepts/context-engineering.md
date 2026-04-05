---
title: "Context Engineering"
tags: [agent-filesystems, context-engineering, LLM, information-architecture]
created: 2026-04-05
last_updated: 2026-04-05
---

## What Context Engineering Is

Context engineering is the discipline of designing, structuring, and delivering the right information to an LLM at the right time. It's broader than prompt engineering — where prompt engineering focuses on *how you phrase* instructions, context engineering focuses on *what information* the model has access to when it reasons.

Weaviate's December 2025 primer defines it cleanly: "Context engineering is all about treating [the context window] as a scarce resource and designing everything around it — retrieval, memory systems, tool integrations, prompts, etc. — so that the model spends its limited attention budget only on high-signal tokens."

The filesystem is one of context engineering's primary delivery mechanisms.

## Why the Context Window Is the Core Constraint

Every LLM operates within a finite context window — the maximum tokens it can "see" at once. Everything the model knows about the current task must fit inside this window: instructions, conversation history, retrieved documents, tool outputs, and the current query.

This constraint creates hard tradeoffs:
- More context = more relevant information, but also more noise and higher cost
- Less context = faster and cheaper, but the model may miss critical information
- Wrong context = the model reasons from incorrect or stale premises

Context engineering is the practice of navigating these tradeoffs deliberately rather than accidentally.

## The Four Failure Modes of Poor Context

Weaviate identifies four ways context engineering breaks down in practice:

**Context Poisoning** — Incorrect or hallucinated information enters the context. Because agents reuse context across reasoning steps, early errors compound. A wrong assumption in step 1 corrupts steps 2 through 10.

**Context Distraction** — Too much past information buries the signal. The agent gets distracted by history and stops reasoning fresh about the current situation.

**Context Confusion** — Irrelevant tools, documents, or instructions crowd the context, causing the model to use the wrong resource or follow the wrong rule.

**Context Clash** — Contradictory information forces the model to guess which version to believe. Common when multiple context sources (CLAUDE.md + auto-memory + retrieved docs) give conflicting guidance.

Good filesystem design actively prevents all four failure modes.

## Context Engineering via Files

Files are one of the most powerful context engineering tools because they're:

- **Persistent** — survive across sessions without special infrastructure
- **Version-controlled** — change history is auditable via git
- **Human-readable** — engineers can review and refine what the agent sees
- **Composable** — multiple files can be combined at context assembly time
- **Selective** — agents can choose which files to load based on the current task

The AGENTS.md / CLAUDE.md pattern (see [[wiki/agent-filesystems/concepts/workspace-as-context|Workspace as Context]]) is context engineering applied to project instructions. The MEMORY.md pattern (see [[wiki/agent-filesystems/patterns/memory-files|Memory Files]]) is context engineering applied to agent state.

A formal treatment came in arXiv:2512.05470 ("Everything is Context"), which proposed a filesystem abstraction for context engineering inspired by Unix's "everything is a file." Their architecture includes three layers:
- **Context Constructor** — assembles relevant context artifacts
- **Context Loader** — delivers them within token constraints
- **Context Evaluator** — validates context quality before the agent reasons

## The Difference from Prompt Engineering

Prompt engineering and context engineering are complementary but distinct:

| | Prompt Engineering | Context Engineering |
|---|---|---|
| Focus | How to phrase instructions | What information to provide |
| Scope | Single interaction | Session and multi-session architecture |
| Artifact | The prompt itself | Files, retrieval systems, memory stores |
| Question | "How do I ask this?" | "What does the agent need to know?" |

In practice: prompt engineering is what happens in a single conversation; context engineering is what you design before the conversation starts.

## Practical Principles

**Signal density over completeness.** The goal is not to give the agent everything — it's to give it the right things. A 200-word CLAUDE.md with non-inferable specifics outperforms a 2,000-word one full of obvious information.

**Layer your context.** Different context has different scopes: organization-wide policies, project conventions, session-specific memory, task-specific retrieval. Design each layer separately and compose them intentionally.

**Make context auditable.** If your agent makes a wrong decision, you need to know what context it had. Files stored in git give you perfect auditability: check the commit at the time of the agent run.

**Prefer explicit over inferred.** The agent can infer many things from reading code, but explicit context is more reliable. State your conventions; don't hope the agent will discover them.

**Measure context quality.** The ETH Zurich research (arXiv:2602.11988) measured agent performance with and without context files. This approach — empirically testing what context actually helps — is underused in practice.

## Key Takeaways

- Context engineering is the discipline of treating the context window as a scarce resource and designing information delivery around it
- Files are one of the most powerful context engineering mechanisms: persistent, version-controlled, composable, auditable
- The four failure modes (poisoning, distraction, confusion, clash) are all prevented by intentional file design
- Prompt engineering asks "how?"; context engineering asks "what?" — both are required for reliable agents
- Signal density beats comprehensiveness: less, better-chosen context outperforms more, noisier context

## See Also

- [[wiki/agent-filesystems/concepts/workspace-as-context|Workspace as Context]] — Context engineering applied to project instructions
- [[wiki/agent-filesystems/patterns/memory-files|Memory Files]] — Context engineering applied to agent continuity
- [[wiki/agent-filesystems/concepts/file-vs-memory|Files vs Memory Stores]] — When files are the right context delivery mechanism
- [[wiki/agent-filesystems/research/key-papers|Key Papers]] — arXiv:2512.05470, arXiv:2510.21413
- [[wiki/agent-memory/_index|Agent Memory]] — The broader memory architecture landscape
