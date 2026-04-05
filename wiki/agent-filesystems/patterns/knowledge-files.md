---
title: "Knowledge Files: Wiki-Style Knowledge Bases for Agents"
tags: [agent-filesystems, knowledge-base, wiki, documentation, reference]
created: 2026-04-05
last_updated: 2026-04-05
---

## What Knowledge Files Are

Knowledge files are the reference layer of an agent's filesystem — the articles, documentation, notes, and structured information that an agent consults when it needs domain knowledge rather than session-specific context. They're written primarily for the agent to read, though well-maintained knowledge files are also useful to humans.

The key distinction from memory files: knowledge files contain *reference material* (how things work, what things mean, documented facts about the world) rather than *episodic memory* (what happened in this session, what tasks are in progress). They change slowly and deliberately; memory files change constantly.

This wiki is itself an example of an agent-facing knowledge base. The articles here are written so that an agent loading them gets substantive, actionable information with enough context to reason well.

## Why Wikis Make Excellent Agent Context

Wiki-style knowledge bases — interconnected Markdown articles with [[wikilinks]] and consistent structure — have several properties that make them ideal agent context:

**Dense signal.** A well-written 500-word article on a topic provides more useful context than a sprawling 5,000-word document. The wiki format encourages focused, substantive writing.

**Navigable graph structure.** Wikilinks create a traversable knowledge graph. An agent reading about "context engineering" can follow links to "AGENTS.md", "file-vs-memory", and "RAG" — expanding its context as needed without loading everything upfront.

**Incremental loading.** Because articles are separate files, an agent can load only what's relevant to the current task. A task about memory architecture doesn't need to load the article on LangChain file tools.

**LLM-native format.** Markdown is the language LLMs are most fluent in. Reading and reasoning over Markdown articles is something models do exceptionally well — no parsing overhead, no schema navigation.

## Designing Knowledge Files for Agent Consumption

Writing for an agent audience changes how you structure articles. Key principles:

**Lead with the core answer.** Agents often need a specific fact, not a complete survey. Put the most important information first — the "what" and "when to use it" before the "how it works under the hood."

**Use concrete examples.** Abstract descriptions require the agent to do more reasoning; concrete examples provide pattern-matchable anchors. Show a real AGENTS.md, not just "AGENTS.md files contain project information."

**Include decision criteria.** Agents make choices. "When should I use X vs Y?" should be explicitly answered in the file, not left as an exercise. This is the purpose of the [[wiki/agent-filesystems/concepts/file-vs-memory|Files vs Memory Stores]] article's decision framework.

**Maintain Key Takeaways.** A bulleted summary at the end of each article lets an agent quickly assess what the article says without reading every word. This is especially valuable when the agent is scanning multiple files to answer a question.

**Cross-link aggressively.** Every mention of a related concept that has its own article should be a wikilink. Agents follow links; orphaned articles get missed.

## The Personal Knowledge Base as Agent Interface

The pattern of building a personal knowledge base specifically for an AI assistant is gaining traction. The files in `~/knowledge/` or a dedicated wiki repo serve as a durable, queryable interface between human expertise and agent sessions.

Practical domains to capture in a personal agent knowledge base:

- **Tooling notes** — how to use specific tools, CLIs, APIs; idiosyncrasies and gotchas
- **Domain knowledge** — facts about your industry, customers, technology stack
- **Decision history** — why past technical choices were made (invaluable for consistent future decisions)
- **People and context** — relevant professional contacts, project stakeholders, organizational context
- **Recurring procedures** — step-by-step playbooks for operations the agent performs regularly

The goal: an agent with access to this knowledge base reasons like someone with years of context on your situation, not like a generic assistant seeing your setup for the first time.

## Maintenance Discipline

Knowledge files rot. Information becomes stale; facts change; tools update. A knowledge base that's 80% accurate is dangerous — the agent can't tell which 20% is wrong.

Practical maintenance:

- **Date everything.** YAML frontmatter `last_updated` field makes stale articles visible at a glance.
- **Mark uncertainty.** A `status: needs-review` tag on articles you're not sure are current prevents overconfident agent reasoning.
- **Update on correction.** When an agent makes a mistake because it had outdated knowledge, update the relevant article immediately.
- **Prune aggressively.** An article that's wrong is worse than no article. Delete or archive rather than letting stale content accumulate.

## Key Takeaways

- Knowledge files are the reference layer — stable domain knowledge distinct from session-specific memory
- Wiki-style articles with wikilinks, frontmatter, and consistent structure are ideal for agent consumption
- Write for agents differently than for humans: lead with answers, include decision criteria, use concrete examples
- A personal knowledge base lets an agent reason with years of accumulated context on your specific domain
- Maintenance matters: dated, tagged, regularly pruned knowledge is far more valuable than a large but stale corpus

## See Also

- [[wiki/agent-filesystems/concepts/structured-context-files|Structured Context Files]] — YAML frontmatter and file conventions
- [[wiki/agent-filesystems/patterns/memory-files|Memory Files]] — The episodic memory layer
- [[wiki/agent-filesystems/concepts/context-engineering|Context Engineering]] — Why signal density beats comprehensiveness
- [[wiki/agent-filesystems/patterns/tool-results-as-files|Tool Results as Files]] — Capturing tool output as durable knowledge
