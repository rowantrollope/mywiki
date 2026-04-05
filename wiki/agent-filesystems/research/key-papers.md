---
title: "Agent Filesystems: Key Papers and Sources"
tags: [agent-filesystems, research, papers, bibliography, arxiv]
created: 2026-04-05
last_updated: 2026-04-05
---

## Annotated Bibliography

Key papers and sources on agent filesystems, context engineering via files, workspace-as-context patterns, and file-based memory.

---

### arXiv:2510.21413 — "Context Engineering for AI Agents in Open-Source Software"

**Authors:** Anonymous (conference submission, 2026)
**Published:** October 2025 (arxiv v1)
**URL:** https://arxiv.org/html/2510.21413v1

**What it is:** The first empirical study of AI configuration files (AGENTS.md, CLAUDE.md, copilot-instructions.md) in real-world open-source software. Surveyed 466 GitHub repositories.

**Key findings:**
- Tens of thousands of GitHub repositories already contain AI configuration files as of late 2025
- No established structure exists yet for organizing content in AGENTS.md files
- High variation in how developers present context: descriptive, prescriptive, prohibitive, explanatory, conditional styles all appear
- Commits modifying AGENTS.md files reveal how projects continuously extend and evolve them over time
- AGENTS.md has emerged as a proposed open, tool-agnostic standard consolidating tool-specific formats

**Why it matters:** Establishes the scale of adoption and the lack of consensus on best practices — a signal that this is an active, unsettled design space with significant opportunity for improvement.

---

### arXiv:2602.11988 — "Reassessing the Value of AGENTS.md Files for AI Coding"

**Authors:** Gloaguen, Mündler, Müller, Raychev, Vechev (ETH Zurich)
**Published:** February 2026
**Covered in:** InfoQ, March 2026

**What it is:** Rigorous empirical study of AGENTS.md effectiveness across 138 real-world Python tasks in niche repositories. Tested Claude 3.5 Sonnet, GPT-5.2, GPT-5.1 mini, and Qwen Code.

**Key findings:**
- LLM-generated AGENTS.md files: −3% task success rate, +20% inference costs vs no file
- Human-written AGENTS.md files: +4% task success rate, +19% inference costs vs no file
- Architectural overview and repository structure descriptions did not reduce time agents spent locating files
- Agents follow AGENTS.md instructions reliably — but following wrong/redundant instructions wastes steps
- Recommendation: omit LLM-generated files entirely; limit human-written files to non-inferable specifics

**Why it matters:** The definitive counter-argument to "more context is better." Quality, specificity, and human authorship of context files matter more than comprehensiveness.

---

### arXiv:2512.05470 — "Everything is Context: Agentic File System Abstraction for Context Engineering"

**Authors:** Xiwei Xu et al.
**Published:** December 5, 2025
**URL:** https://arxiv.org/abs/2512.05470

**What it is:** Proposes a formal filesystem abstraction for context engineering, inspired by Unix's "everything is a file" philosophy. Implemented within the AIGNE open-source framework.

**Key findings:**
- Existing context practices (RAG, prompt engineering, tool integration) are fragmented and produce transient artifacts
- A file-system abstraction provides persistent, governed infrastructure for heterogeneous context artifacts through uniform mounting, metadata, and access control
- Architecture includes Context Constructor (assembles), Context Loader (delivers within token constraints), Context Evaluator (validates)
- Demonstrated via two exemplars: agent with memory and MCP-based GitHub assistant

**Why it matters:** The first formal theoretical treatment of filesystem-as-context-infrastructure, grounding the intuitive "files for agents" pattern in principled architecture.

---

### ReMe: Memory Management Kit for Agents (GitHub: agentscope-ai/ReMe)

**Published:** March 2026
**URL:** https://github.com/agentscope-ai/ReMe

**What it is:** Open-source memory management kit from the AgentScope team that formalizes the three-file memory pattern: MEMORY.md (long-term), daily YYYY-MM-DD.md logs, and tool_result/ cache directory.

**Key design decisions:**
- MEMORY.md holds persistent user preferences and key facts
- Daily journals auto-written after each conversation
- Raw dialog preserved as JSONL for compression/analysis
- Tool results written to UUIDs in tool_result/, auto-expired

**Why it matters:** Production-quality open-source implementation of the file-based memory architecture, with automatic maintenance built in.

---

### open-gitagent Framework (GitHub: open-gitagent/gitagent)

**Published:** March 2026
**URL:** https://github.com/open-gitagent/gitagent

**What it is:** Framework-agnostic, git-native standard for defining AI agents. Formalizes the GitOps-for-agents pattern with a `memory/runtime/` directory structure.

**Key design decisions:**
- `memory/runtime/` holds live agent knowledge: `dailylog.md`, `key-decisions.md`, `context.md`
- Git-native — every state change is a commit
- Framework-agnostic specification that any agent framework can implement

**Why it matters:** Establishes git as a first-class agent state management system, not just version control for code.

---

### Weaviate Blog: "Context Engineering - LLM Memory and Retrieval for AI Agents"

**Published:** December 9, 2025
**URL:** https://weaviate.io/blog/context-engineering

**What it is:** Comprehensive practitioner guide to context engineering from the Weaviate team. Defines context engineering vs prompt engineering, describes context window failure modes, and covers the full memory architecture spectrum.

**Key contributions:**
- Defines four context failure modes: poisoning, distraction, confusion, clash
- Clear articulation of context engineering vs prompt engineering distinction
- Practical memory architecture stack: short-term → working → long-term → external retrieval

**Why it matters:** Best practitioner-level treatment of context engineering as a discipline, grounding the "files for agents" pattern in a systematic theory.

---

### Reddit r/AI_Agents: "2 years building agent memory systems, ended up just using Git"

**Published:** August 2025
**URL:** https://www.reddit.com/r/AI_Agents/comments/1mw4jvp/2_years_building_agent_memory_systems_ended_up/

**What it is:** Practitioner retrospective on building agent memory systems. After trying vector databases, embeddings, and custom retrieval pipelines, the author converged on git + BM25 keyword search.

**Key quote:** "Search is just BM25 with an LLM generating the queries. Not fancy but completely debuggable. The entire memory for 2 years fits in a Git repo you could read with Notepad."

**Why it matters:** Field evidence that simple file-based approaches hold up over time and at real scale — the complexity ceiling of plain markdown + git is higher than most assume.

---

## See Also

- [[wiki/agent-filesystems/research/open-problems|Open Problems]] — What remains unsolved
- [[wiki/agent-filesystems/concepts/context-engineering|Context Engineering]] — Concepts grounded in this research
- [[wiki/agent-filesystems/concepts/workspace-as-context|Workspace as Context]] — The AGENTS.md pattern from these papers
- [[wiki/agent-memory/_index|Agent Memory]] — Related research on memory architectures
