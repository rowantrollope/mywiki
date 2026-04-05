---
title: "Memory Files: MEMORY.md, Daily Logs, and Active-Tasks Patterns"
tags: [agent-filesystems, memory, MEMORY.md, daily-logs, active-tasks, continuity]
created: 2026-04-05
last_updated: 2026-04-05
---

## The Problem: Agent Amnesia

An agent that wakes up every session with no memory of previous work is frustrating and expensive. It re-asks questions already answered, re-researches topics already covered, and loses the thread of multi-day projects. This is the default state for LLM agents — and memory files are the cure.

Memory files are a set of plain text documents that an agent writes during sessions and reads at the start of future sessions. Together they form the agent's continuity stack: enough context to pick up where the last session left off without human re-briefing.

## The Stack: Three Layers

A production memory file system has three layers, each serving a distinct purpose:

### Layer 1: MEMORY.md — The Long-Term Index

The master memory file. Contains curated, distilled facts the agent should always know about the person or project it serves. Think of it as a compressed representation of everything learned over many sessions.

What belongs here:
- User identity, preferences, and recurring patterns ("Rowan prefers no red-eye flights")
- Project status summaries ("Auth service migration is in progress, ~60% complete")
- Hard-won insights ("The staging environment has flaky tests on Mondays — always run twice")
- Key relationships and contacts

What doesn't belong here:
- Raw session logs — those live in daily files
- Temporary task state — that lives in active-tasks
- Detailed reference material — that lives in knowledge files

**Size discipline is critical.** Claude Code loads up to the first 200 lines or 25KB of MEMORY.md at every session start. If MEMORY.md grows unbounded, it consumes context window before the agent even starts working. Target: under 2KB. Maximum: 25KB if you have no choice.

The file is a *living document* — regularly distilled, not just appended. Every few sessions, stale entries should be removed and key recent learnings promoted from daily logs.

```markdown
# Memory

## About Rowan
- Timezone: America/Los_Angeles. Home base: San Francisco.
- Prefers no red-eye flights. Prefers direct where possible.
- Responds quickly via Telegram; email is slower.

## Current Projects
- **Auth migration**: In progress. API is done; working on UI. ETA: next sprint.
- **Wiki**: Building out agent-filesystems topic as of 2026-04-05.

## Lessons Learned
- The gog-cache is much faster than calling Google APIs directly; always use it.
- The canvas server sometimes needs manual restart after gateway restarts.
```

### Layer 2: Daily Logs — The Session Journal

A new Markdown file for each day: `memory/2026-04-05.md`. The agent appends notes, decisions, completed tasks, and observations throughout the session. These are raw and conversational — not curated.

Daily logs answer the question: "What happened recently?" They're read on-demand when the agent needs recent history — not loaded into every session.

Useful things to capture in daily logs:
- What tasks were completed and their outcomes
- Decisions made and the reasoning behind them
- Problems encountered and how they were resolved
- New facts learned about users, projects, or tools
- Anything that *might* belong in MEMORY.md someday

Over time (weekly or monthly), review daily logs and promote significant items to MEMORY.md. Archive logs older than 30-60 days to keep the working directory manageable.

### Layer 3: Active-Tasks — The Crash Recovery File

The active-tasks file (`memory/active-tasks.md`) is a live tracker of what's in-flight right now. Its primary job is crash recovery: if the agent session dies in the middle of a task, the next session reads this file and resumes without asking the human "what were we doing?"

Standard structure:

```markdown
## 🔥 In Progress
- **Auth UI migration** — Working on LoginForm component; started 2026-04-05 14:00 UTC

## ⏳ Waiting
- **Deploy staging** — Waiting for CI to pass on PR #342

## 🚫 Blocked
- **Email campaign** — Waiting for design team to deliver final assets

## ✅ Recently Completed (keep last 5)
- **AGENTS.md update** — Updated with new git conventions — 2026-04-05
```

The agent updates this file:
- **Before starting** a new task (add to In Progress)
- **When spawning** a subagent (note the session key)
- **When switching** tasks (move from In Progress to Waiting or Blocked)
- **When completing** a task (move to Recently Completed)

## Writing Discipline: What to Write vs What to Skip

Not everything needs to be written down. The overhead of constant memory writes can slow an agent down. A useful heuristic:

**Write it when:**
- A decision was made that will affect future sessions
- A fact was learned that isn't easily re-discoverable
- A task state changed (started, blocked, completed)
- Something went wrong and there's a lesson

**Skip it when:**
- It's retrievable in seconds (web search, file read, API call)
- It's session-specific trivia that won't matter in 24 hours
- It's already covered in a reference document

## The Memory Maintenance Cycle

Memory files rot if not maintained. Build a review cycle into the agent's workflow:

1. **End of session** — update active-tasks, append key observations to daily log
2. **Weekly** — read last 7 daily logs, promote 3-5 key items to MEMORY.md, archive the logs
3. **Monthly** — review MEMORY.md for stale entries, prune anything no longer relevant

Claude Code's auto-memory feature does a simplified version of this automatically — it writes to MEMORY.md during sessions and trims when the file exceeds 25KB. Manual curation still produces better results.

## Key Takeaways

- The memory file stack has three layers: MEMORY.md (curated long-term), daily logs (raw session journal), active-tasks (crash recovery and task state)
- MEMORY.md must be kept small enough to load into context every session — target under 2KB, hard cap at 25KB
- Active-tasks is the most critical file for multi-session reliability; without it, task continuity depends on the human re-briefing the agent
- Write discipline matters: record decisions and lessons, skip easily-retrievable trivia
- Maintain a regular distillation cycle: daily logs → MEMORY.md, with stale entries pruned

## See Also

- [[wiki/agent-filesystems/concepts/file-based-memory|File-Based Memory]] — The conceptual foundation
- [[wiki/agent-filesystems/patterns/gitops-for-agents|GitOps for Agents]] — Using git to version-control memory files
- [[wiki/agent-filesystems/concepts/file-vs-memory|Files vs Memory Stores]] — When to upgrade beyond flat files
- [[wiki/agent-filesystems/tools/claude-code|Claude Code]] — Auto-memory implementation
