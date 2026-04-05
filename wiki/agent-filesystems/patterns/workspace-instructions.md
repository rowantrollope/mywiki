---
title: "Workspace Instructions: Writing Effective AGENTS.md and CLAUDE.md Files"
tags: [agent-filesystems, workspace-instructions, AGENTS.md, CLAUDE.md, patterns]
created: 2026-04-05
last_updated: 2026-04-05
---

## What These Files Are

Workspace instruction files are the primary mechanism for giving an AI coding agent persistent, project-specific context. They're the difference between an agent that asks "what testing framework do you use?" every session and one that already knows.

The files go by different names depending on the tool — CLAUDE.md (Claude Code), AGENTS.md (OpenAI Codex / open standard), .cursorrules (Cursor), copilot-instructions.md (GitHub Copilot), GEMINI.md (Gemini CLI) — but the design principles are the same across all of them.

See [[wiki/agent-filesystems/concepts/workspace-as-context|Workspace as Context]] for a conceptual overview. This article focuses on practical authoring guidance.

## What to Include

**Build and test commands.** The most immediately valuable content. Every project has idiosyncratic commands that differ from the defaults. Don't make the agent guess.

```markdown
## Commands
- Install: `npm ci`
- Test (all): `npm run test -- --forceExit`  
- Test (single file): `npm run test -- path/to/file.test.ts --forceExit`
- Lint: `npm run lint`
- Build: `npm run build`
- DO NOT run `npm install` — use `npm ci` to respect lockfile
```

**Project structure.** Key directories and what lives where. Only the non-obvious stuff — the agent can read a README for the rest.

```markdown
## Structure
- `src/api/` — REST endpoint handlers
- `src/services/` — Business logic (no direct DB calls here)
- `src/db/` — All database access (use repository pattern)
- `tests/integration/` — Integration tests (require running Docker)
```

**Coding conventions.** Import paths, naming patterns, error handling approaches, anything the agent will get wrong if it just follows common practice.

**Forbidden patterns.** Explicit prohibitions are highly effective — agents follow "never do X" instructions reliably. Document anti-patterns that have caused bugs.

**Architecture decisions.** ADRs (Architecture Decision Records) that are too detailed for a README but too important to lose. "We use optimistic locking because X" prevents the agent from suggesting a naive solution.

## What to Exclude

The ETH Zurich research (arXiv:2602.11988) found that **inferrable information in AGENTS.md actively degrades performance** — it wastes tokens and sometimes confuses the agent. Skip:

- Information visible in the code itself (what imports are used, how functions are structured)
- Generic best practices the agent already knows (write tests, handle errors, use async/await)
- Descriptions of public APIs the agent can look up
- Architecture that's clearly visible from the directory structure

Rule of thumb: if the agent could discover it by spending 10 seconds reading the code, don't include it.

## Tone and Style

**Prescriptive beats descriptive.** "Always use `UserRepository.findById()`" is more effective than "we have a user repository that provides database access."

**Explicit beats implicit.** "NEVER call the database directly from service layer" works better than "we try to keep services separate from data access."

**Concise beats comprehensive.** A 200-word AGENTS.md that hits the key non-inferrable facts outperforms a 2,000-word one that pads with obvious information.

**Lists over prose.** Bullet lists with clear headers are faster for an agent to scan than paragraphs. The agent often needs to "find the testing command" not "read the full context."

## Hierarchical Scoping

Claude Code's implementation shows how scoping works in practice. Create CLAUDE.md files at different levels for different scopes:

- `/etc/claude-code/CLAUDE.md` — Organization-wide standards (managed by DevOps)
- `~/CLAUDE.md` — Your personal development preferences
- `~/project/CLAUDE.md` — Project-specific rules (checked into git, shared with team)
- `~/project/src/auth/CLAUDE.md` — Subdirectory rules for a complex subsystem

More specific files take precedence over broader ones. This lets a team share project conventions while each developer maintains personal workflow preferences.

## Lifecycle: Keeping the File Current

A AGENTS.md file that's never updated becomes misleading — worse than useless. Treat it like a living document:

- Update it when you change a build command or testing pattern
- Add a "forbidden" entry when you encounter a recurring mistake the agent makes
- Review it monthly and remove anything that's become obvious or stale
- Ask the agent itself: "What was confusing or missing in this session?" and update based on the answer

Claude Code's approach of auto-updating MEMORY.md (the agent writes its own learnings) provides a complementary layer to human-written CLAUDE.md — the machine accumulates what it learned, the human maintains the authoritative project rules.

## Example: A Minimal but Effective AGENTS.md

```markdown
# AGENTS.md

## Commands
- Test: `pytest tests/ -x --tb=short`
- Lint: `ruff check . && mypy src/`
- Single test: `pytest tests/test_auth.py::test_login -v`

## Conventions
- All DB access via `src/repositories/` — never query DB directly in services
- Use `Result[T, E]` return types for error-prone operations (see `src/types.py`)
- Import from `src.core.config` for settings — never read env vars directly

## Never Do
- Never call `settings.reload()` in tests — breaks parallel test runs
- Never commit to `main` directly — all changes via PR
- Never hardcode URLs — use `settings.api_base_url`
```

That's ~150 words. It covers non-inferrable specifics, explicit prohibitions, and project-specific patterns. It will save the agent (and the developer reviewing its output) significant time every session.

## Key Takeaways

- Include non-inferrable specifics: build commands, custom conventions, forbidden patterns, key architecture decisions
- Exclude anything the agent can discover by reading the code for 10 seconds
- Prescriptive, explicit, list-based beats descriptive, implicit, prose
- Use hierarchical scoping for organization-wide → project → personal → subdirectory rules
- Treat the file as a living document — update it when the project changes or when you see the agent make recurring mistakes

## See Also

- [[wiki/agent-filesystems/concepts/workspace-as-context|Workspace as Context]] — Conceptual foundation
- [[wiki/agent-filesystems/tools/claude-code|Claude Code]] — Claude's CLAUDE.md implementation
- [[wiki/agent-filesystems/tools/openai-codex|OpenAI Codex]] — Codex's AGENTS.md implementation
- [[wiki/agent-filesystems/research/key-papers|Key Papers]] — arXiv:2602.11988 on AGENTS.md effectiveness
