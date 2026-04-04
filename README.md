# Rowan's Personal Wiki

A structured knowledge base for topics that matter: AI agent architectures, system design, research, and everything worth remembering. Built for Obsidian. Maintained by Sancho.

## Structure

```
mywiki/
├── wiki/               ← Curated, structured articles
│   └── agent-memory/   ← Agent memory systems
│       ├── _index.md   ← Topic index
│       ├── overview.md
│       ├── concepts/   ← Foundational concepts
│       ├── architectures/ ← Implementation patterns
│       └── research/   ← Papers and open problems
├── raw/                ← Drop zone for unprocessed notes
│   └── agent-memory/
└── _global_index.md    ← Master cross-topic index
```

## Using This Wiki

### In Obsidian

1. Open this repo directory as an Obsidian vault
2. Install the **Obsidian Git** plugin for auto-sync
3. Start at `_global_index.md` or any `_index.md` file
4. Click wikilinks to navigate between articles

The `.obsidian/app.json` is pre-configured with:
- Preview mode as default view (rendered markdown)
- Readable line length enabled

### Obsidian Git Setup

```
1. Install: Settings → Community Plugins → Obsidian Git
2. Configure:
   - Auto pull interval: 5 minutes
   - Auto push interval: 5 minutes  
   - Vault backup interval: 10 minutes
   - Commit message: "vault backup: {{date}}"
3. Run "Obsidian Git: Initialize a new repo" if needed
   (repo is already initialized — just set remote)
```

### Adding Content via Sancho

The easiest way to add content is to ask Sancho directly:

**Adding a new article:**
> "Sancho, add a wiki article on [topic] under wiki/agent-memory/"

**Adding a new topic section:**
> "Sancho, create a new wiki section on [topic]"

**Updating an existing article:**
> "Sancho, update the rag.md article with information about [new thing]"

**Dumping raw notes for later:**
Drop anything into `raw/<topic>/` — notes, URLs, PDF text. Tell Sancho:
> "Sancho, I dropped some notes in raw/agent-memory/ — process them when you have time"

Sancho will extract the useful information, write/update articles, and commit the result.

### Article Format

Every wiki article follows this structure:

```markdown
---
title: Article Title
tags: [tag1, tag2]
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
---

## First Section

Content with [[wikilinks]] to related articles.

## Key Takeaways  (for concept articles)

- Bullet point summary of the most important points

## See Also

- [[related-article-1]]
- [[related-article-2]]
```

## Current Content

### Agent Memory (`wiki/agent-memory/`)

**Overview**
- `overview.md` — What agent memory is, why it matters, current state of the field

**Concepts**
- `concepts/types-of-memory.md` — Four memory types: episodic, semantic, procedural, working
- `concepts/episodic-vs-semantic.md` — The distinction, consolidation pipelines, implementations
- `concepts/working-memory.md` — Context window management, attention patterns, strategies
- `concepts/long-term-memory.md` — External stores, compression, retention policies
- `concepts/memory-retrieval.md` — Similarity search, recency, importance scoring

**Architectures**
- `architectures/rag.md` — Retrieval-Augmented Generation
- `architectures/vector-stores.md` — Embeddings, Pinecone, Chroma, pgvector
- `architectures/knowledge-graphs.md` — Graph memory, entities + relations
- `architectures/mem0.md` — mem0.ai managed memory layer
- `architectures/memgpt.md` — MemGPT / Letta: virtual context management

**Research**
- `research/key-papers.md` — Annotated bibliography
- `research/open-problems.md` — Unsolved problems in the field

## Contributing

This is a personal wiki. Content is curated and quality-controlled. Prefer substantive articles (300-800 words) over shallow stubs. When in doubt, ask Sancho to write it properly.

---

*Maintained by Sancho Navarro. Last seeded: April 2026.*
