# 00 — Preface

You are a WordPress developer. Most of your daily work happens in the Gutenberg
codebase — a large, mature, heavily-tested monorepo. You asked a question that started
this whole journey:

> *"How do I reduce tokens when opencode wants to identify a function signature?"*

That question led you through graphify, codegraph, TencentDB-Agent-Memory, and ECC.
But along the way you realized something more important: **your real goal is not cheaper
search. It is a working agentic loop with a human in the loop at the start and at the
end.**

This book teaches you how to build that loop. It is written like a textbook: each
chapter builds on the last, with concepts, examples, and WordPress-specific guidance.
It is deliberately **tool-agnostic at the core** — the loop works with plain opencode,
`gh`, and markdown files. The fancy tools (graphify, codegraph, ECC) are presented as
*optional accelerators*, not requirements.

## Who this is for

- A WordPress developer who uses opencode (or wants to).
- Someone who wants the agent to do the *middle* of the work (plan → implement → test →
  PR) while the human owns the *edges* (requirements in, merge out).
- Someone who wants to understand *why* the loop works, not just copy-paste config.

## What you will be able to do after reading

- Write a macro-requirements spec that an agent can actually plan against.
- Author an `AGENTS.md` that onboards an agent into your repo (the modern pattern).
- Define a pipeline of opencode agents with clear roles and permissions.
- Use `wp-env`, `@wordpress/scripts`, and `wp-build` correctly for build and test.
- Run a conductor-style TDD loop (red/green/refactor, checkpoints, git notes).
- Have the agent open a GitHub PR with a structured body.
- Feed your GitHub review comments back into the agent for iteration.
- Extend core blocks with `render_block` hooks, the HTML API, and the Interactivity API.

## What this book is NOT

- Not a full opencode manual. (See the official docs for that.)
- Not a WordPress development tutorial. (You already know WordPress.)
- Not a promise that any tool "solves" everything. The loop is the product; tools are
  ingredients.

## How this manual is organized

Use [`toc.md`](../toc.md) as the indexer. The chapters are split into five parts:

1. **Concepts** (00–02) — the "why".
2. **Toolchain** (03–06) — the modern WordPress build/test stack.
3. **The Loop** (07–11) — the four gates and the feedback loop.
4. **WordPress in Practice** (12–15) — playbook, agent pipeline, worked example.
5. **Reference** (16–18) — troubleshooting, glossary, appendix.

If you are implementing this in another system, follow
[`checklist.md`](../checklist.md) top-to-bottom.

## A note on the rewrite

The original single-file draft captured the timeless concepts. This manual keeps those
concepts and **rewrites the tooling** to match how modern WordPress development actually
works — verified against the Gutenberg, WooCommerce, wordpress-develop, wp-movies-demo,
x3p0-ideas, advanced-query-loop, and coblocks codebases, plus the hospital plugins and
themes and the swirl case study. Where the old draft said `composer run phpcs`, this
manual says `npm run lint:php` (via `wp-env`). Where it defined custom `.opencode/agent`
files as the only pattern, this manual leads with `AGENTS.md` and skills.
