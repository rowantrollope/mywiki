---
title: "Structured Context Files: YAML Frontmatter, Schemas, and Conventions"
tags: [agent-filesystems, structured-data, YAML, frontmatter, markdown, schemas]
created: 2026-04-05
last_updated: 2026-04-05
---

## Why Structure Matters in Agent Files

A plain text file can hold any information, but an LLM has to work harder to extract structured data from unstructured prose. A file that says "updated: April 5, 2026" somewhere in its body requires the agent to parse natural language to find the date. A file with `last_updated: 2026-04-05` in its YAML frontmatter can be parsed programmatically and included in index queries without reading the full content.

Structure in agent-readable files serves two audiences simultaneously: the LLM (which reads the natural language body) and any tooling around it (which can parse frontmatter metadata without loading full content). Good structure makes files useful to both.

## YAML Frontmatter as Metadata Layer

The Markdown + YAML frontmatter convention (popularized by static site generators like Jekyll and Hugo, and knowledge tools like Obsidian) has become the dominant pattern for structured agent context files. The convention is simple:

```yaml
---
title: "Project Architecture Notes"
tags: [architecture, backend, auth]
created: 2026-01-15
last_updated: 2026-04-05
status: active
related: [auth-service, user-service]
---

## Overview

The backend is a microservices architecture...
```

The frontmatter block gives any file a machine-parseable identity without sacrificing human readability. For agent systems, frontmatter enables:

- **Fast indexing** — scan thousands of files for relevant metadata without loading full content
- **Filtering** — find all articles tagged `active` or updated in the last 30 days
- **Linking** — `related` fields can drive automatic cross-reference generation
- **Lifecycle management** — `status: archived` signals files that shouldn't be loaded into context

## Common Frontmatter Fields for Agent Files

Different file types need different metadata. A practical set:

**All files:**
- `title` — human-readable name (also used in wikilinks)
- `tags` — array of semantic categories for search and filtering
- `created`, `last_updated` — ISO 8601 dates for freshness assessment
- `status` — `active`, `archived`, `draft`, `deprecated`

**Memory and note files:**
- `session` — which agent session created this
- `confidence` — `high`, `medium`, `low` (for facts the agent is uncertain about)
- `source` — URL, conversation ID, or document the fact came from

**Task files:**
- `priority` — `p0`, `p1`, `p2`
- `blocked_by` — list of task IDs
- `due` — deadline date
- `assigned_to` — which agent or human owns it

**Reference/knowledge files:**
- `domain` — subject area
- `version` — document version for tracking changes
- `related` — cross-references to other files

## Wikilinks and Cross-Reference Conventions

Obsidian-style `[[wikilinks]]` have become the standard cross-reference format for agent knowledge bases. They're simple, human-readable, and easy for LLMs to follow. A mention of `[[auth-service]]` in any document creates an implicit graph edge that agents and tools can traverse.

The key convention: use `[[topic/subtopic]]` path-style wikilinks (not just `[[title]]`) to avoid ambiguity in large knowledge bases. This matches how Obsidian's "unique note" model works, and makes links robust even when titles change.

For agent systems with wiki-style knowledge bases (like this one), the combination of wikilinks + YAML frontmatter creates a lightweight graph: nodes are files, edges are wikilinks, and the frontmatter provides node metadata.

## Naming Conventions

Consistent naming conventions let agents navigate the filesystem predictably:

- `kebab-case` for file names (lowercase, hyphens): `auth-service.md`, not `AuthService.md`
- `YYYY-MM-DD` for date-based files: `memory/2026-04-05.md`
- `ALL_CAPS.md` for sentinel/protocol files that agents always check: `MEMORY.md`, `AGENTS.md`, `ACTIVE-TASKS.md`
- Plural directories, singular files: `concepts/` contains `context-engineering.md`

## The AGENTS.md Convention vs Free-Form Files

There's a spectrum between fully structured files (frontmatter + schema-validated body) and completely free-form files (whatever the agent decided to write). The right point on that spectrum depends on use case:

**More structured** for files that tooling needs to parse: indexes, task lists, entity records, reference material that gets embedded or indexed.

**More free-form** for files that only agents will read: daily logs, reasoning traces, scratch notes during a task. Structure here adds friction without benefit.

The discipline is not to impose schema for its own sake — it's to add structure *where it creates leverage* for tooling, search, or future retrieval.

## Key Takeaways

- YAML frontmatter creates a machine-parseable metadata layer on human-readable files without sacrificing readability
- Standard frontmatter fields (title, tags, created, status) enable fast indexing and filtering across large file corpora
- Wikilinks (`[[topic/subtopic]]`) create lightweight graph relationships between files
- Consistent naming conventions (kebab-case, YYYY-MM-DD dates, ALL_CAPS for sentinel files) let agents navigate predictably
- Add structure where tooling or retrieval needs it; leave reasoning traces and logs free-form

## See Also

- [[wiki/agent-filesystems/concepts/workspace-as-context|Workspace as Context]] — Structure in AGENTS.md / CLAUDE.md files
- [[wiki/agent-filesystems/patterns/knowledge-files|Knowledge Files]] — Wiki-style knowledge bases with frontmatter
- [[wiki/agent-filesystems/patterns/memory-files|Memory Files]] — How to structure MEMORY.md and daily logs
- [[wiki/agent-filesystems/concepts/context-engineering|Context Engineering]] — Why structure improves context quality
