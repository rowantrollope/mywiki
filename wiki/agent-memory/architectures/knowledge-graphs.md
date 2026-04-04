---
title: Knowledge Graphs for Agent Memory
tags: [agent-memory, knowledge-graph, graph-db, entities, relations, neo4j]
created: 2026-04-04
last_updated: 2026-04-04
---

## What Is a Knowledge Graph?

A knowledge graph represents information as a network of entities and typed relationships: nodes (entities) connected by edges (relations). Where a vector store asks "what is semantically similar to this query?", a knowledge graph asks "what is structurally connected to this entity?"

The canonical representation is the RDF triple: `(subject, predicate, object)` — for example:
- `(Rowan, WORKS_AT, Redis)`
- `(Redis, IS_A, Database_Company)`
- `(Rowan, PREFERS, Python)`
- `(Rowan, OWNS_PROJECT, AuthRefactor)`
- `(AuthRefactor, USES, Redis)`

This structure enables queries that vectors handle poorly: "What projects does Rowan own that use the same technology stack as the authentication service he's refactoring?"

## Why Graphs Beat Vectors for Certain Problems

Vector retrieval excels at: "find me things semantically similar to X."

Graphs excel at: "find me things *related* to X via specific chains of relationships."

**Multi-hop reasoning** is where graphs shine. A vector search for "Rowan's database projects" might miss the authentication refactor unless the document explicitly mentions databases. But a graph traversal from `Rowan → OWNS → AuthRefactor → USES → Redis → IS_A → Database` finds it definitively.

**Exact relationships** matter in many agent contexts. Who approved this decision? Which team members have worked on this component? What tasks depend on this task? These require structural traversal, not semantic similarity.

**Contradiction detection** is natural in graphs. If you add `(Rowan, PREFERS, JavaScript)` and `(Rowan, PREFERS, Python)` already exists, a graph database makes the contradiction explicit in a way a vector store cannot.

## Knowledge Graph Structure for Agent Memory

A practical agent memory knowledge graph has:

### Entity Types
- **Person** — users, team members, stakeholders
- **Project** — codebases, initiatives, features
- **Organization** — companies, teams, departments
- **Technology** — tools, languages, databases, frameworks
- **Decision** — key choices made with rationale
- **Task** — action items with status and ownership
- **Event** — meetings, milestones, incidents
- **Concept** — abstract ideas and preferences

### Relationship Types
- `WORKS_AT`, `MANAGES`, `REPORTS_TO`
- `OWNS`, `CONTRIBUTES_TO`, `REVIEWS`
- `USES`, `DEPENDS_ON`, `INTEGRATES_WITH`
- `MADE_DECISION`, `APPROVED`, `REJECTED`
- `MENTIONED_IN`, `REFERENCED_BY`
- `PREFERS`, `DISLIKES`, `EXPERT_IN`

### Properties
Entities and edges carry properties: `created_at`, `confidence`, `source_session`, `last_verified`.

## Graph Databases for Agent Memory

### Neo4j
The dominant graph database. Mature, feature-rich, excellent tooling. Uses Cypher query language.

```cypher
// Find all projects Rowan is involved in that use Python
MATCH (r:Person {name: "Rowan"})-[:OWNS|CONTRIBUTES_TO]->(p:Project)-[:USES]->(t:Technology {name: "Python"})
RETURN p.name, p.description

// Find colleagues who've worked on Redis-related projects
MATCH (r:Person {name: "Rowan"})<-[:COLLABORATES_WITH]-(colleague)-[:CONTRIBUTES_TO]->(proj)-[:USES]->(:Technology {name: "Redis"})
RETURN colleague.name, proj.name
```

**Best for:** Production deployments with complex queries and large graphs. Has vector index support for hybrid graph+vector retrieval.

### Kuzu
Embeddable, in-process graph DB (like SQLite for graphs). Fast, no server required. Excellent for agent memory use cases where you want low-latency local graph access.

### MemGraph
PostgreSQL-compatible graph DB with strong streaming support. Good for real-time agent memory updates.

### LlamaIndex Property Graph
LlamaIndex's graph memory abstraction — builds and queries a property graph over agent interactions. Higher-level API than raw graph DB access.

## Hybrid Graph + Vector Architectures

The most powerful agent memory architectures combine graphs and vectors:

1. **Graph for structure**: entities, relationships, decisions, facts
2. **Vectors for content**: semantic search over free-text notes, conversation summaries, documents

Retrieval proceeds in two passes:
- **Pass 1**: Vector search to find relevant entities/episodes by semantic similarity
- **Pass 2**: Graph traversal from identified entities to expand context along relationship edges

Example: Query "what did we discuss about the database migration?" → vector search identifies the `DBMigration` project entity → graph traversal expands: team members, decisions made, related tasks, dependent systems.

This pattern is implemented in systems like [[mem0]] (which uses both a vector store and a graph layer) and some advanced LangChain/LlamaIndex workflows.

## Graph Memory Extraction

Building a knowledge graph from unstructured conversations requires extraction — parsing natural language into entities and relationships.

Common approach:
```
Conversation text
  → LLM extraction prompt: "Extract all entities and relationships from this text"
  → Structured triples: [(entity1, relation, entity2), ...]
  → Deduplication + merge with existing graph
  → Graph DB upsert
```

Extraction quality is the bottleneck. LLMs are reasonably good at this for common entity types but hallucinate relationships and miss implicit connections. Human validation or confidence thresholds help.

## When NOT to Use Graphs

- **Simple factual retrieval** — "what's Rowan's preferred language?" doesn't need a graph. A KV store or vector search is faster and simpler.
- **Small memory stores** — the overhead of graph extraction and maintenance isn't worth it for <1000 facts.
- **Purely semantic queries** — when you're searching by content similarity, not structure, vectors are better.
- **Team without graph DB expertise** — graphs are harder to operationalize than vector stores.

## Key Takeaways

- Knowledge graphs store entities + typed relationships, enabling structural traversal that vectors can't do
- Use graphs for multi-hop reasoning, exact relationship queries, and contradiction detection
- Hybrid graph + vector architectures are the state of the art for agent memory
- Neo4j for production; Kuzu for lightweight embedded use cases
- Graph extraction from conversation is the hard part — quality is the bottleneck

## See Also

- [[rag]]
- [[vector-stores]]
- [[mem0]]
- [[concepts/episodic-vs-semantic]]
- [[concepts/long-term-memory]]
- [[research/open-problems]]
