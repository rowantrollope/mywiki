---
title: MemPalace — Structured Local-First AI Memory System
tags: [agent-memory, tools, mempalace, aaak, palace-structure, local, mcp, benchmarks]
created: 2026-04-07
last_updated: 2026-04-07
---

## What Is MemPalace?

MemPalace is a local-first, open-source AI memory system that claims the highest published score on the LongMemEval benchmark — 96.6% R@5 with zero API calls, 100% with an optional Haiku reranking pass. Its tagline: *"The highest-scoring AI memory system ever benchmarked. And it's free."*

GitHub: https://github.com/milla-jovovich/mempalace  
License: MIT  
Install: `pip install mempalace`

The core bet MemPalace makes is different from mem0, Zep, or Letta: rather than letting an LLM decide what's worth extracting, **store everything verbatim, then make it structurally findable**. It achieves that through two original inventions: the Palace structure and the AAAK compression dialect.

**Key stats at a glance:**

| Metric | Value |
|--------|-------|
| LongMemEval R@5 (raw, no API) | 96.6% |
| LongMemEval R@5 (Haiku rerank) | 100% (500/500) |
| Retrieval boost from palace structure | +34% R@10 vs flat vector search |
| Annual cost (MemPalace vs LLM summaries) | ~$10/yr vs ~$507/yr |
| MCP tools exposed | 19 |
| Cloud dependency | None |

---

## The Palace Structure

The name is not metaphor — it's a literal implementation of the ancient *method of loci* (memory palace technique), where Greek orators memorized speeches by mentally placing ideas in rooms of an imaginary building.

MemPalace organizes all stored memory into a six-level hierarchy:

```
WING (person or project)
 └── HALL (memory type: facts, events, discoveries, preferences, advice)
      └── ROOM (named idea: auth-migration, graphql-switch, ci-pipeline)
           └── CLOSET (compressed summary pointing to original content)
                └── DRAWER (verbatim original files — nothing is lost)

TUNNEL (cross-wing connection when the same room appears in multiple wings)
```

**Wings** map to people or projects — as many as you need. A single conversation about auth involving two people and one project would spread across three wings, but the shared room name creates a **tunnel** that cross-references them automatically.

**Halls** are fixed memory types, identical across all wings:

- `hall_facts` — decisions made, choices locked in
- `hall_events` — sessions, milestones, debugging runs
- `hall_discoveries` — breakthroughs, new insights
- `hall_preferences` — habits, likes, personal style
- `hall_advice` — recommendations and solutions offered

**Rooms** are named ideas. When `auth-migration` appears in wing_kai, wing_driftwood, and wing_priya, the system sees the same topic in three wings and automatically connects them via tunnel. Cross-project search becomes native rather than a feature bolted on.

**Closets and drawers** preserve the information at two levels: closets hold compressed summaries for fast AI reading; drawers hold the exact original text. Nothing is ever deleted or summarized away.

### Why Structure Improves Retrieval

The benchmark data makes the case:

| Search scope | R@10 |
|---|---|
| All closets (flat) | 60.9% |
| Within wing only | 73.1% (+12%) |
| Wing + hall | 84.8% (+24%) |
| Wing + room | 94.8% (+34%) |

The palace structure is not cosmetic. Knowing *where to look* before searching is worth 34 percentage points.

---

## AAAK Compression

AAAK is a lossless compression dialect designed specifically for AI agents to read — not humans. It is the second core invention and the one that makes long-term memory economically viable.

The problem it solves: six months of daily AI use generates roughly **19.5 million tokens**. You cannot paste that into a context window. You cannot even summarize it affordably with an LLM ($507/year). AAAK offers a third path.

**How it works:**

AAAK is structured English shorthand with a universal grammar. Names become codes, relationships become operators, ratings become symbols. Any LLM that reads text can parse it — no decoder, no fine-tuning, no cloud API required.

English (≈1,000 tokens):
```
Priya manages the Driftwood team: Kai (backend, 3 years), Soren (frontend),
Maya (infrastructure), and Leo (junior, started last month). They're building
a SaaS analytics platform. Current sprint: auth migration to Clerk.
Kai recommended Clerk over Auth0 based on pricing and DX.
```

AAAK (≈120 tokens):
```
TEAM: PRI(lead) | KAI(backend,3yr) SOR(frontend) MAY(infra) LEO(junior,new)
PROJ: DRIFTWOOD(saas.analytics) | SPRINT: auth.migration→clerk
DECISION: KAI.rec:clerk>auth0(pricing+dx) | ★★★★
```

**Same information. 8x fewer tokens.** MemPalace claims 30x compression in general cases with zero information loss because the full originals remain in drawers.

### The Four-Layer Memory Stack

AAAK powers a layered wake-up architecture:

| Layer | Content | Size | When loaded |
|---|---|---|---|
| L0 | Identity — who is this AI? | ~50 tokens | Always |
| L1 | Critical facts — team, projects, preferences (AAAK) | ~120 tokens | Always |
| L2 | Room recall — recent sessions, current project | On demand | When topic comes up |
| L3 | Deep search — semantic query across all closets | On demand | When explicitly asked |

An AI agent wakes up with L0 + L1 (~170 tokens total) knowing your world. Searches only fire when the conversation requires them. Compare that to 650K tokens for LLM-generated summaries.

---

## Installation and Quick Start

**Requirements:** Python 3.9+, `chromadb>=0.4.0`, `pyyaml>=6.0`. No API key. No internet after install.

```bash
pip install mempalace

# Guided setup — generates AAAK bootstrap + wing config
mempalace init ~/projects/myapp

# Mine your data
mempalace mine ~/projects/myapp              # project files (code, docs, notes)
mempalace mine ~/chats/ --mode convos        # conversation exports (Claude, ChatGPT, Slack)
mempalace mine ~/chats/ --mode convos --extract general  # auto-classifies into decisions/milestones/problems

# Search
mempalace search "why did we switch to GraphQL"
mempalace search "auth decisions" --wing driftwood
mempalace search "rate limiting" --room auth-migration

# Wake-up context (for local models)
mempalace wake-up > context.txt
# Paste context.txt into your local model's system prompt

# Check status
mempalace status
```

**Mining modes:**
- `projects` — code files, docs, notes
- `convos` — chat exports (Claude, ChatGPT, Slack); normalizes 5 common export formats
- `general` — auto-classifies into decisions, milestones, problems, preferences, emotional context

If your chat exports concatenate multiple sessions into one file, split them first:

```bash
mempalace split ~/chats/               # split into per-session files
mempalace split ~/chats/ --dry-run     # preview what would happen
```

---

## MCP Integration

For Claude and other MCP-compatible hosts, MemPalace exposes 19 tools via an MCP server. Setup is one command:

```bash
claude mcp add mempalace -- python -m mempalace.mcp_server
```

The AI learns AAAK automatically from the `mempalace_status` response on first use. No manual configuration.

**Tools by category:**

*Palace (read):*
- `mempalace_status` — palace overview + AAAK spec + memory protocol
- `mempalace_list_wings` — wings with counts
- `mempalace_list_rooms` — rooms within a wing
- `mempalace_get_taxonomy` — full wing → room → count tree
- `mempalace_search` — semantic search with wing/room filters
- `mempalace_check_duplicate` — check before filing
- `mempalace_get_aaak_spec` — AAAK dialect reference

*Palace (write):*
- `mempalace_add_drawer` — file verbatim content
- `mempalace_delete_drawer` — remove by ID

*Knowledge Graph:*
- `mempalace_kg_query` — entity relationships with time filtering
- `mempalace_kg_add` — add facts
- `mempalace_kg_invalidate` — mark facts as ended
- `mempalace_kg_timeline` — chronological entity story
- `mempalace_kg_stats` — graph overview

*Navigation:*
- `mempalace_traverse` — walk the graph from a room across wings
- `mempalace_find_tunnels` — find rooms bridging two wings
- `mempalace_graph_stats` — connectivity overview

*Agent Diary:*
- `mempalace_diary_write` — write AAAK diary entry
- `mempalace_diary_read` — read recent diary entries

### Claude Code Hooks

MemPalace ships two shell hooks for automatic memory capture during Claude Code sessions:

- **Save Hook** — fires every 15 messages; saves topics, decisions, quotes, code changes; regenerates L1
- **PreCompact Hook** — fires before context compression; emergency save before the window shrinks

```json
{
  "hooks": {
    "Stop": [{"matcher": "", "hooks": [{"type": "command", "command": "/path/to/mempalace/hooks/mempal_save_hook.sh"}]}],
    "PreCompact": [{"matcher": "", "hooks": [{"type": "command", "command": "/path/to/mempalace/hooks/mempal_precompact_hook.sh"}]}]
  }
}
```

---

## Temporal Knowledge Graph

MemPalace includes a built-in temporal entity-relationship graph on SQLite (local, free). It is functionally similar to Zep's Graphiti feature but without Neo4j or cloud:

```python
from mempalace.knowledge_graph import KnowledgeGraph

kg = KnowledgeGraph()
kg.add_triple("Maya", "assigned_to", "auth-migration", valid_from="2026-01-15")
kg.add_triple("Maya", "completed", "auth-migration", valid_from="2026-02-01")

# What was true in January?
kg.query_entity("Maya", as_of="2026-01-20")
# → [Maya → assigned_to → auth-migration (active)]

# When something stops being true:
kg.invalidate("Kai", "works_on", "Orion", ended="2026-03-01")
```

Facts have validity windows. Historical queries still return past states; current queries reflect invalidated entries. This handles the tenure/date checking that flat memory systems get wrong.

---

## Multi-Agent Support

MemPalace supports specialized agents, each with a dedicated wing and diary. Rather than growing CLAUDE.md, agents are defined in `~/.mempalace/agents/`:

```
~/.mempalace/agents/
 ├── reviewer.json    # code quality, bug patterns
 ├── architect.json   # design decisions, tradeoffs
 └── ops.json         # deploys, incidents, infra
```

Your CLAUDE.md only needs one line: `You have MemPalace agents. Run mempalace_list_agents to see them.`

Each agent builds expertise independently — the reviewer remembers every bug pattern it has seen; the architect remembers every design decision — using AAAK diary entries that persist across sessions.

---

## Benchmarks

Published results on standard academic benchmarks with reproducible runners in `benchmarks/`:

| Benchmark | Mode | Score | API Calls |
|---|---|---|---|
| LongMemEval R@5 | Raw (ChromaDB only) | **96.6%** | Zero |
| LongMemEval R@5 | Hybrid + Haiku rerank | **100%** (500/500) | ~500 |
| LoCoMo R@10 | Raw, session level | 60.3% | Zero |
| Personal palace R@10 | Heuristic bench | 85% | Zero |
| Palace structure impact | Wing+room vs flat | +34% R@10 | Zero |

Competitive position on LongMemEval:

| System | R@5 | API Required | Cost |
|---|---|---|---|
| MemPalace (hybrid) | 100% | Optional | Free |
| Supermemory ASMR | ~99% | Yes | — |
| **MemPalace (raw)** | **96.6%** | **None** | **Free** |
| Mastra | 94.87% | Yes (GPT) | API costs |
| mem0 | ~85% | Yes | $19–249/mo |
| Zep | ~85% | Yes | $25/mo+ |

---

## Honest Assessment

### Strengths

**Lossless by design.** Verbatim content in drawers means no retrieval failure from bad extraction. Other systems (mem0, etc.) lose information at the extraction step. MemPalace doesn't.

**Fully offline.** ChromaDB runs on-device. AAAK works with any text-reading model. No API key required after install. For privacy-sensitive or airgapped use cases, this is a genuine differentiator.

**Structured retrieval is a real advantage.** The +34% retrieval improvement from palace structure is grounded in benchmark data and intuitive — knowing *where to search* before searching matters.

**Cost economics are compelling.** $10/year vs $507/year for equivalent recall. The AAAK wake-up context approach is genuinely efficient.

**Active development, MIT licensed.** Open source, reproducible benchmarks, plugin architecture for custom wings and agents.

### Caveats

**Self-reported benchmarks.** All results are published by the MemPalace team on datasets they chose. LongMemEval is a real academic benchmark but independent replication would strengthen the claim significantly. The 100% result in particular warrants independent validation.

**AAAK is unvalidated externally.** The claim that AAAK is "lossless" rests on the assumption that any LLM will correctly parse a custom shorthand dialect. In practice, performance likely varies by model and degrades on edge cases not represented in the training corpus. The "zero information loss" claim has not been independently verified.

**Setup complexity.** Unlike managed systems (mem0, Zep), MemPalace requires understanding wings, halls, rooms, mining modes, and configuration files. The onboarding tool helps, but there is genuine complexity.

**Newer project.** The repo appears relatively new (v3.0.0 as of early 2026). Production battle-testing is limited compared to mem0 or Letta.

**LoCoMo scores are moderate.** The 60.3% LoCoMo R@10 raw score is competitive but not dominant — significantly below the LongMemEval headline number. LoCoMo tests multi-session conversational memory in more naturalistic settings and may be the more relevant benchmark for typical agent use.

---

## Comparison to Other Memory Systems

| System | Approach | Storage | Cost | Local? | Lossless? |
|---|---|---|---|---|---|
| **MemPalace** | Hierarchical structure + verbatim | ChromaDB + SQLite | Free | Yes | Yes |
| **mem0** | LLM extraction + deduplication | Qdrant/Neo4j | $0 OSS / $19–249/mo cloud | Configurable | No (extraction loses detail) |
| **Zep (Graphiti)** | Temporal KG + vector | Neo4j | $25/mo+ | Enterprise only | No |
| **Letta (MemGPT)** | OS-inspired virtual context | Postgres | $20–200/mo | Self-host available | No |
| **RAG + vector store** | Embed and retrieve | Chroma/Pinecone/pgvector | Infra cost | Yes | Depends on chunking |

**The core philosophical difference:** mem0, Zep, and Letta all make LLM-driven extraction decisions — they decide what to keep. MemPalace keeps everything and uses structure to make it findable. Neither approach is universally better; the tradeoff is between storage cost (MemPalace uses more disk) and information fidelity (MemPalace loses nothing).

For local-only, offline, and privacy-first deployments, MemPalace has no real competitor. For managed, cloud-integrated, team-scale memory, mem0 and Zep are more mature options.

---

## Key Takeaways

- MemPalace = store everything + make it structurally findable, vs. extract-and-summarize approaches
- Palace structure (wings/halls/rooms/tunnels) provides a +34% retrieval improvement over flat vector search
- AAAK compression enables ~170-token wake-up context for months of conversation history
- Fully local: ChromaDB, SQLite, no API keys, works with any LLM that reads text
- Benchmark claims are impressive and reproducible, but self-reported — independent validation pending
- Best fit: local-first, offline, privacy-sensitive, or cost-constrained AI memory deployments

---

## See Also

- [[architectures/mem0]] — LLM-extraction approach; most direct philosophical counterpart
- [[architectures/memgpt]] — Virtual context management, the Letta approach
- [[architectures/knowledge-graphs]] — Temporal graphs; compare to MemPalace's built-in KG
- [[architectures/vector-stores]] — ChromaDB (MemPalace's underlying vector store)
- [[concepts/memory-retrieval]] — Retrieval strategies; palace structure as a structured retrieval layer
- [[concepts/long-term-memory]] — Layered compression; L0–L3 stack context
- [[research/state-of-the-art-2025]] — SOTA landscape and where MemPalace fits
- [[_index]] — Agent Memory topic index
