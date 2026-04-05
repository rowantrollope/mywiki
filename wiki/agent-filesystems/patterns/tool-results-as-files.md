---
title: "Tool Results as Files: Writing Tool Output to Disk for Agent Context"
tags: [agent-filesystems, tool-results, caching, context-management, patterns]
created: 2026-04-05
last_updated: 2026-04-05
---

## The Problem: Large Tool Outputs Clog the Context Window

When an agent calls a tool — executes a web search, runs a terminal command, calls an API, reads a large file — the output often needs to be incorporated into the agent's reasoning. The naive approach is to dump the entire output into the context window. For small outputs, this is fine. For large ones, it's a disaster.

A web scrape of a documentation page might return 50KB of text. An API call might return a 10,000-line JSON response. A code analysis tool might produce pages of output. Stuffing all of this directly into the context window leaves little room for the actual reasoning the agent needs to do — and costs proportionally more in token usage.

The *tool results as files* pattern is the solution: write large tool outputs to disk, give the agent a file path, and let the agent load only what it needs.

## The Pattern

Instead of:
```
[Tool: web_search] → 8000 tokens of search results injected into context
```

The pattern is:
```
[Tool: web_search] → write results to /tmp/search_results_abc123.md
                  → return: "Results written to /tmp/search_results_abc123.md (8,432 tokens)"
[Agent reads selectively] → read first 50 lines of file, extract relevant section
```

This converts a mandatory context window cost into an on-demand file read. The agent pays the token cost only for the portions it actually uses.

## Where This Pattern Appears in the Wild

**ReMe (AgentScope)** explicitly includes a `tool_result/` directory in its memory architecture:

```
working_dir/
└── tool_result/        # Cache for long tool outputs
    └── <uuid>.txt      # Auto-managed, expired entries auto-cleaned
```

Long tool outputs are written here and the agent receives a UUID reference. The directory is automatically cleaned when entries expire, preventing unbounded disk growth.

**Claude Code** uses a similar pattern when processing large file reads — rather than injecting entire files into context, it chunks and references.

**Manus agent** (the $2B acquisition) used a three-file structure where tool research results were written to `notes.md` and the agent accumulated findings there rather than keeping everything in the live context.

## What Qualifies for File Storage

Apply the pattern when tool output exceeds roughly 1,000 tokens (~750 words). Specific cases:

- **Web fetches and scrapes** — documentation pages, articles, search results
- **API responses** — large JSON responses, paginated data, analytics exports
- **Code execution output** — test results, build logs, analysis reports
- **File system listings** — large directory trees, code search results
- **Research synthesis** — multi-source research compiled into a working document

Small, crisp tool outputs (a weather reading, a single database record, a short API response) can stay in context — the overhead of writing to disk isn't worth it for a 100-token result.

## File Organization and Naming

For temporary tool results (useful only during current session):
```
/tmp/agent-{session-id}/
├── search_results_{timestamp}.md
├── api_response_{uuid}.json
└── analysis_output_{timestamp}.txt
```

For tool results worth keeping across sessions (research, analysis, reference data):
```
~/workspace/tool-results/
├── research/
│   └── 2026-04-05-langchain-docs.md
├── api-snapshots/
│   └── 2026-04-05-weather-api-response.json
└── analysis/
    └── 2026-04-05-codebase-audit.md
```

The session-scoped `/tmp/` location is appropriate for ephemeral work; the workspace location is for results worth referencing in future sessions.

## The Knowledge Promotion Pattern

Tool results can be promoted from raw output to curated knowledge. The lifecycle:

1. **Raw capture** — tool runs, output written to file as-is
2. **Agent annotation** — agent reads the file, adds a summary or key findings at the top
3. **Selective extraction** — agent pulls the relevant facts into MEMORY.md or a knowledge file
4. **Archive or delete** — raw output archived after key facts are extracted; deleted if not useful

This prevents tool result files from becoming a graveyard of unread output. The goal is to extract signal into durable knowledge, not to accumulate raw data indefinitely.

## Handling Sensitive Tool Output

Tool results sometimes contain sensitive information — API responses with tokens, search results with personal data, code analysis with security findings. Apply the same discipline as any sensitive file:

- Exclude from git with `.gitignore` patterns for `/tmp/` and sensitive directories
- Use session-scoped `/tmp/` paths for results that shouldn't persist
- Explicitly sanitize before promoting to long-lived knowledge files
- Don't write credentials or personal data to the workspace knowledge base

## Key Takeaways

- Large tool outputs written to files instead of injected into context window prevent token waste and context clog
- The threshold for file storage is roughly 1,000 tokens — small results can stay in context, large ones go to disk
- Session-scoped `/tmp/` for ephemeral results; workspace directory for results worth keeping across sessions
- The knowledge promotion pattern converts raw tool output into durable knowledge: capture → annotate → extract → archive
- Apply the same security discipline to tool result files as to any file containing sensitive data

## See Also

- [[wiki/agent-filesystems/concepts/file-based-memory|File-Based Memory]] — The broader memory file system
- [[wiki/agent-filesystems/patterns/memory-files|Memory Files]] — Where promoted facts land
- [[wiki/agent-filesystems/concepts/context-engineering|Context Engineering]] — Why context window budgeting matters
- [[wiki/agent-filesystems/patterns/knowledge-files|Knowledge Files]] — Promoting tool results to durable knowledge
