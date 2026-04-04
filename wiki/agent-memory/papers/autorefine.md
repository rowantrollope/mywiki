---
title: "AutoRefine: From Trajectories to Reusable Expertise for Continual LLM Agent Refinement"
tags: [agent-memory, papers, arxiv, 2026, continual-learning, procedural-memory, expertise-extraction]
arxiv: "2601.22758"
created: 2026-04-04
last_updated: 2026-04-04
---

## Overview

Most LLM agents start fresh on every task. They may be given a few in-context examples, but they don't genuinely *accumulate* knowledge from experience the way a skilled human professional does. Recent work on experience replay has tried to address this by extracting "knowledge" from past trajectories and storing it for future use — but typically as flattened text: summaries, guidelines, plain prose.

AutoRefine argues that flattened text is structurally insufficient. Complex tasks involve *procedural* coordination — sequences of subtasks that depend on each other, require specialized sub-reasoning, and can't be captured in a single-layer text note. The paper introduces a dual-form experience pattern library and a continuous maintenance mechanism to prevent that library from degrading as it grows.

---

## Key Contributions

- **Dual-form Experience Patterns:** Distinguishes between two types of reusable knowledge extracted from execution histories — *subagent patterns* (procedural, for complex coordinated subtasks) and *skill patterns* (declarative, for static knowledge as guidelines or code snippets). Each form targets a different part of the problem.
- **Specialized subagents for procedural subtasks:** For complex subtasks that can't be captured as guidelines, AutoRefine extracts independent subagents with their own reasoning and memory — effectively crystallizing learned multi-step procedures into callable sub-policies.
- **Continuous maintenance mechanism:** As the experience repository grows, quality degrades — outdated patterns, duplicates, and low-quality entries accumulate. AutoRefine scores patterns, prunes low-quality ones, and merges redundant entries on an ongoing basis.
- **Outperforms manually designed systems on TravelPlanner:** Automatic extraction at 27.1% vs. manually designed systems at 12.1% — a striking result showing that learned procedural patterns can exceed hand-crafted expert systems.

---

## Architecture / Method

AutoRefine's pipeline has three phases:

### 1. Experience Extraction
After each task execution (whether successful or failed), AutoRefine analyzes the full trajectory:
- **For procedural subtasks** (those involving multi-step coordination with specialized logic): extract a subagent with its own system prompt, internal reasoning trace, and memory. The subagent becomes a callable module for future tasks.
- **For static knowledge** (facts, heuristics, common patterns): extract as skill patterns — structured guidelines or reusable code snippets.

### 2. Pattern Repository
All extracted patterns are stored in a typed repository:
- Subagent patterns: indexed by task type and subtask signature
- Skill patterns: indexed by domain and trigger conditions
New tasks retrieve relevant patterns (both types) to augment the main agent's context.

### 3. Continuous Maintenance
The repository runs periodic maintenance:
- **Scoring:** Each pattern is scored based on how often it's retrieved, whether tasks using it succeed, and a staleness penalty for age.
- **Pruning:** Patterns below a threshold score are removed.
- **Merging:** Redundant or near-duplicate patterns are merged into a single consolidated version.
This prevents the repository from becoming a graveyard of stale, low-quality "experience" that hurts more than it helps.

---

## Results

Evaluated on ALFWorld (household tasks), ScienceWorld (science experiment simulation), and TravelPlanner (complex travel itinerary planning):

| Benchmark | AutoRefine | Prior SOTA | Step Reduction |
|-----------|-----------|------------|----------------|
| ALFWorld | **98.4%** | ~95% | 20-30% |
| ScienceWorld | **70.4%** | ~65% | 30-40% |
| TravelPlanner | **27.1%** | 12.1% (manual) | 50-73% |

The TravelPlanner result is particularly notable: automatic extraction from trajectories substantially exceeds manually-designed expert systems, demonstrating that procedural coordination patterns are learnable and transferable without human curation.

Step reductions (20-73% across tasks) indicate that accumulated expertise genuinely shortens the agent's solution path — not just that it achieves better final outcomes.

---

## Key Takeaways

- **Flat text can't capture procedural knowledge:** For complex multi-step subtasks, extracting independent subagents with their own reasoning and memory is significantly better than summarizing the procedure as text guidelines. The representational form matters.
- **Repository maintenance is not optional:** Without pruning and merging, an experience repository becomes a liability. Stale and low-quality patterns degrade performance. Build maintenance into any continual learning system from day one.
- **Learned expertise can beat manual design:** The TravelPlanner result (27.1% vs 12.1%) is a strong empirical argument that agents should accumulate experience automatically rather than relying solely on human-authored expert rules. At sufficient scale and quality, learned patterns outperform hand-crafted ones.

---

## See Also

- [[papers/agemem]] — complementary work on end-to-end RL-trained memory management
- [[papers/reflexion]] — verbal self-reflection as a simpler form of experience-to-memory extraction
- [[concepts/memory-consolidation]] — the broader problem of converting episodic experience to reusable knowledge
- [[concepts/reflection-and-abstraction]] — abstraction mechanisms that AutoRefine's pattern extraction operationalizes
- [[research/state-of-the-art-2025]] — broader context of continual learning for LLM agents
