# The WordPress Agentic Loop — Implementation Manual

A practical, chapter-split book on building a **human-in-the-loop agent system** for
WordPress development with opencode. Written for a WordPress developer who works daily
in the Gutenberg codebase and ships plugins, themes, and blocks.

> **Start here.** This README is the landing page. Use [`toc.md`](./toc.md) as the
> indexer to navigate chapters, and [`checklist.md`](./checklist.md) as the
> implementation checklist an agent can follow to reproduce this loop in another system.

---

## What this is

A working agentic loop with a **human in the loop at the start and at the end**:

```
spec → plan → build → PR → review → fix → merge
```

Human owns the *edges* (requirements in, merge out). The agent owns the *middle*
(plan → implement → test → PR). Every handoff is a **written document**, not a chat.

This manual is deliberately **tool-agnostic at the core** — the loop works with plain
opencode, `gh`, and markdown files. The fancy tools (code-graph search, memory, skills)
are optional accelerators, not requirements.

## What changed from the original single-file draft

The original `deprecated/wordpress-agentic-loop.md` captured the timeless *concepts* (the four
gates, "every handoff is a written contract"). This manual keeps those concepts and
**rewrites the tooling** to match how modern WordPress development actually works,
verified against real codebases. See [`CHANGELOG.md`](./CHANGELOG.md) for the full
evolution.

## How to read it

1. Read [`toc.md`](./toc.md) for the chapter map and reading order.
2. Read chapters **00–02** for the concepts (the "why").
3. Read chapters **03–06** for the modern WordPress toolchain and TDD (the "how").
4. Read chapters **07–11** for the four gates and the feedback loop.
5. Read chapters **12–15** for the WordPress playbook, the agent pipeline, and a worked example.
6. Use chapters **16–18** for troubleshooting, glossary, and reference files.

## The agent ecosystem

The `opencode/` directory contains the **documented** agent ecosystem (agents,
commands, and a project `opencode.json`). It is intentionally **self-contained** — it
does not touch your global `~/.config/opencode/` config. To use it, copy the files you
want into your own project's `.opencode/` directory (see Chapter 18).

## Deprecated

The original single-file draft, [`deprecated/wordpress-agentic-loop.md`](./deprecated/wordpress-agentic-loop.md),
is retained for reference. It still captures the timeless *concepts* (the four gates,
"every handoff is a written contract") and remains usable, but it is **superseded** by
this chapter-split manual. New work should follow the chapters in [`toc.md`](./toc.md).

## License

GPL-2.0-or-later, matching the WordPress ecosystem this manual is written for.
