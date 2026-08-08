# The WordPress Agentic Loop

## A Practical Book on Building a Human-in-the-Loop Agent System with opencode

*Version 1.0 — written for a WordPress developer working daily in the Gutenberg codebase*

> **Deprecated.** This single-file draft is retained for reference. The concepts
> (four gates, "every handoff is a written contract") are still valid, but the tooling
> and examples are superseded by the chapter-split manual. New work should follow
> [`toc.md`](../toc.md) and [`checklist.md`](../checklist.md).

---

## Table of Contents

1. [Preface: What This Book Is](#1-preface-what-this-book-is)
2. [The Big Picture](#2-the-big-picture)
3. [Core Concepts](#3-core-concepts)
4. [The Four Gates](#4-the-four-gates)
5. [Gate 1 — Macro Requirements (The Spec)](#5-gate-1--macro-requirements-the-spec)
6. [Gate 2 — The Plan](#6-gate-2--the-plan)
7. [Gate 3 — Implementation & Verification](#7-gate-3--implementation--verification)
8. [Gate 4 — The PR Review in GitHub](#8-gate-4--the-pr-review-in-github)
9. [The Feedback Loop](#9-the-feedback-loop)
10. [WordPress-Specific Playbook](#10-wordpress-specific-playbook)
11. [The Tools You Researched, Decoded](#11-the-tools-you-researched-decoded)
12. [A Complete Worked Example](#12-a-complete-worked-example)
13. [Troubleshooting & Common Pitfalls](#13-troubleshooting--common-pitfalls)
14. [Glossary](#14-glossary)
15. [Appendix: Reference Files](#15-appendix-reference-files)

---

## 1. Preface: What This Book Is

You are a WordPress developer. Most of your daily work happens in the Gutenberg codebase — a large, mature, heavily-tested monorepo. You asked a question that started this whole journey:

> *"How do I reduce tokens when opencode wants to identify a function signature?"*

That question led you through graphify, codegraph, TencentDB-Agent-Memory, and ECC. But along the way you realized something more important: **your real goal is not cheaper search. It is a working agentic loop with a human in the loop at the start and at the end.**

This book teaches you how to build that loop. It is written like a textbook: each chapter builds on the last, with concepts, examples, and WordPress-specific guidance. It is deliberately **tool-agnostic at the core** — the loop works with plain opencode, `gh`, and markdown files. The fancy tools (graphify, codegraph, ECC) are presented as *optional accelerators*, not requirements.

### Who this is for
- A WordPress developer who uses opencode (or wants to).
- Someone who wants the agent to do the *middle* of the work (plan → implement → test → PR) while the human owns the *edges* (requirements in, merge out).
- Someone who wants to understand *why* the loop works, not just copy-paste config.

### What you will be able to do after reading
- Write a macro-requirements spec that an agent can actually plan against.
- Define 3–4 opencode agents with clear roles and permissions.
- Have the agent open a GitHub PR with a structured body.
- Feed your GitHub review comments back into the agent for iteration.
- Do all of this with WordPress-specific conventions (PHPCS, PHPUnit, block registration, etc.).

### What this book is NOT
- Not a full opencode manual. (See the official docs for that.)
- Not a WordPress development tutorial. (You already know WordPress.)
- Not a promise that any tool "solves" everything. The loop is the product; tools are ingredients.

---

## 2. The Big Picture

### 2.1 The loop in one diagram

```
        ┌────────────────────────────────────────────────────────────┐
        │                                                            │
        ▼                                                            │
[HUMAN] Macro Requirements ──► [AGENT] Plan ──► [AGENT] Implement ──► [AGENT] Test/Verify
   (spec file)                    (no edits)      (edits + tests)          │
                                                                          │
[HUMAN] PR Review in GitHub ◄── [AGENT] Open PR ◄──────────────────────────┘
   (you review)                    (gh pr create)
        │
        └──► [AGENT] Address feedback ──► Update PR ──► back to review ──► merge
```

### 2.2 The principle that makes it work

> **Every handoff is a written contract, not a chat.**

The agent's "memory" is the files on disk, not the conversation. When you hand the agent requirements, you write a spec. When the agent plans, it writes a plan. When it opens a PR, it writes a structured body. When you review, you write comments. Each of these is a *document* that the next stage consumes.

This is the same principle behind ECC's tagline — *"Optimize the context window. Persist everything else."* — and behind TencentDB's "memory assets." You do not need their infrastructure to benefit from the idea. You need discipline: **write things down.**

### 2.3 Why human-in-the-loop at the edges

- **At the start:** The agent cannot read your mind. "Macro requirements" means the *what and why*, not the *how*. You supply intent; the agent supplies execution.
- **At the end:** The agent cannot judge quality the way you can. A PR review is your quality gate. You are the final reviewer, the one who merges.

The middle — planning, coding, testing, iterating — is where the agent earns its keep.

### 2.4 The three layers of any agentic system

1. **The Runtime** — the thing that runs the agent (opencode).
2. **The Workflow** — the loop itself (spec → plan → build → PR → review → fix).
3. **The Accelerators** — optional tools that make the middle faster (code-graph search, memory, skills).

Most people start by installing accelerators. This book starts with the workflow, because the workflow is what actually delivers value.

---

## 3. Core Concepts

Before we build, you need a shared vocabulary.

### 3.1 Agent
An agent is a configured instance of the runtime with a **role**, a **prompt**, and **permissions**. In opencode, agents are defined in `.opencode/agent/<name>.md` files (or in `opencode.json`).

A good agent has:
- A clear **description** (when to use it).
- A **mode** (`primary` for the main agent, `subagent` for helpers).
- A **permission** policy (what it may edit, what it may run).
- A **prompt** (its instructions and personality).

### 3.2 Command
A command is a reusable prompt you can invoke. In opencode, commands live in `.opencode/command/<name>.md`. A command can take arguments (`$ARGUMENTS`, `$1`, `$2`). Commands are how you *trigger* the loop.

### 3.3 Hook
A hook is code that runs on events (before/after a tool, on session start/end, etc.). Hooks are how you *automate* parts of the loop. For a local setup, you may not need hooks at all — commands are enough.

### 3.4 Spec
A spec is a markdown document describing *what* to build and *why*. It is the contract for the whole loop. See Chapter 5.

### 3.5 Plan
A plan is the agent's breakdown of the spec into concrete tasks, with acceptance criteria. It is written *before* any code. See Chapter 6.

### 3.6 PR (Pull Request)
A PR is the unit of review. In the loop, the agent opens a PR; you review it; the agent updates it. See Chapter 8.

### 3.7 Handoff
A handoff is any written document that passes state from one stage to the next. Specs, plans, PR bodies, and review comments are all handoffs.

### 3.8 The "definition of done"
A checklist that must be true before the agent considers a task complete. It is part of the spec and the plan.

---

## 4. The Four Gates

The loop has four gates. Each gate is a *decision point* where control passes between human and agent.

| Gate | Who decides | What is produced | Where it lives |
|---|---|---|---|
| **1. Requirements** | Human | Spec file | `docs/specs/<name>.md` |
| **2. Plan** | Human approves | Plan doc | `docs/plans/<name>.md` |
| **3. Implementation** | Agent | Code + tests + PR | working tree + GitHub |
| **4. Review** | Human | Review comments | GitHub PR |

### 4.1 Why four gates and not two?
You said "human at the start and at the end." That is true at a high level. But between "start" and "end" there is a natural checkpoint: **the plan**. Having the human approve the plan *before* the agent writes code is the single highest-leverage thing you can do. It catches wrong assumptions when they are cheap to fix (a doc edit) instead of expensive (a rewrite).

So the loop is really: **human (spec) → human (plan approval) → agent (build) → human (review)**. Two of the four gates are human; the middle two are agent.

### 4.2 The agent's job at each gate
- **Gate 1:** Read the spec. Ask clarifying questions if needed. Do not code.
- **Gate 2:** Produce a plan. Stop. Wait for approval.
- **Gate 3:** Implement, test, iterate, open a PR.
- **Gate 4:** Read review comments, fix, update the PR.

---

## 5. Gate 1 — Macro Requirements (The Spec)

### 5.1 What "macro requirements" means
Macro requirements are the *what* and *why*, stated at a level the agent can plan against but not so detailed that you are doing the design. You are the product owner; the agent is the engineer.

### 5.2 The spec template

```markdown
# Spec: <Short Name>

## Goal
One or two sentences: what are we building and why does it matter?

## Context
Background the agent needs. Link relevant files, docs, or existing code.
For WordPress: link the relevant block, plugin, or core file.

## Requirements
A numbered list of concrete, testable requirements.
Each should be verifiable ("the block renders X when Y").

## Non-Goals
What we are explicitly NOT doing. This prevents scope creep.

## Constraints
- Tech stack / conventions (e.g. "must pass PHPCS", "must use block.json")
- Compatibility (e.g. "must work with WP 6.4+")
- Performance / accessibility / i18n requirements

## Acceptance Criteria
A checklist that must ALL be true for the work to be "done."
This becomes the plan's definition of done.

## Definition of Done
- [ ] Code passes lint (PHPCS / ESLint)
- [ ] Tests pass (PHPUnit / Jest)
- [ ] No regressions in related areas
- [ ] PR opened with summary
```

### 5.3 Writing requirements that agents can act on
Bad: "Make the block better."
Good: "The block must render a heading with the post title, and must be translatable via `__()`."

The difference: the good requirement is **testable**. The agent can verify it.

### 5.4 WordPress-specific spec tips
- Reference the exact file paths (e.g. `packages/block-library/src/heading/`).
- State the WordPress version target.
- State whether this is a block, plugin, theme, or core change.
- If it touches the block editor, mention the relevant APIs (e.g. `useBlockProps`, `registerBlockType`).
- If it touches PHP, mention PHPCS/PHPUnit expectations.

### 5.5 Where specs live
Keep them in the repo under `docs/specs/`. They are versioned, reviewable, and become the PR's reference. This is the "persist everything" principle in action.

---

## 6. Gate 2 — The Plan

### 6.1 What a plan is
A plan is the agent's translation of the spec into concrete work. It should include:
- A task breakdown (ordered steps).
- For each task: what file(s) change, what the change does, how it is verified.
- Risks and open questions.
- The definition of done (from the spec).

### 6.2 The plan template

```markdown
# Plan: <Short Name>

## Summary
One paragraph restating the goal in the agent's own words.

## Tasks
1. **Task name** — description. Files: `path/to/file`. Verify: how.
2. ...

## Risks
- What could go wrong, and the mitigation.

## Open Questions
- Things to confirm with the human before proceeding.

## Definition of Done
- (copy from spec)
```

### 6.3 Why the plan stops the agent
The plan agent has `edit: deny` permission. It cannot change code. It can only read and write the plan. This forces it to *think before acting* — and it gives you a cheap checkpoint.

### 6.4 The approval handoff
You read the plan. If it is wrong, you edit the spec or the plan and re-run. If it is right, you tell the agent "approved, proceed." That single word is the handoff.

---

## 7. Gate 3 — Implementation & Verification

### 7.1 The build agent
The build agent takes the approved plan and implements it. It has edit permission. It works task by task, verifying each before moving on.

### 7.2 Verification is not optional
The build agent must run the project's checks. For WordPress/Gutenberg:
- **PHP:** `composer run phpcs` (or the repo's lint script), `composer run phpunit`.
- **JS/TS:** `npm run lint`, `npm run test-unit` (Jest).
- **TypeScript:** `npm run build` or `tsc`.

The agent should not claim a task is done until the checks pass. This is the "definition of done" enforced.

### 7.3 Self-review before PR
Before opening a PR, a **review agent** (fresh context) reads the diff and looks for:
- Bugs, edge cases, missing tests.
- Style violations.
- Unintended changes.
- Missing documentation.

This catches what the builder missed. It is cheap insurance.

### 7.4 Opening the PR
The agent uses `gh`:

```bash
gh pr create --title "Fix: <summary>" --body "$(cat pr-body.md)"
```

The PR body is a handoff document (see Chapter 8). It tells you what changed, why, and what to review.

---

## 8. Gate 4 — The PR Review in GitHub

### 8.1 The PR body as a contract
The PR body should answer three questions for you, the reviewer:
1. **What** changed?
2. **Why** did it change?
3. **What do you want me to specifically review?**

A good PR body template:

```markdown
## Summary
<what and why>

## Changes
- <file>: <what changed and why>

## Tests
- <what was run and the result>

## Review Focus
- <the risky or important parts you want the human to check>

## Related
- Spec: docs/specs/<name>.md
- Plan: docs/plans/<name>.md
```

### 8.2 Your review
You review in GitHub as you normally would. You can:
- Approve and merge (loop ends).
- Request changes with comments (loop continues).

### 8.3 The review comment as a handoff
Your comments are the input to the next stage. They should be specific and actionable:
- Good: "The `get_post_meta` call on line 42 needs a default value."
- Bad: "This doesn't work."

---

## 9. The Feedback Loop

### 9.1 The fix agent
The fix agent reads your review comments and addresses them. It is a fresh agent with edit permission, pointed at the PR.

### 9.2 Two ways to feed comments back

**Manual (recommended to start):**
1. You review in GitHub.
2. You copy your comments into opencode: "Address these review comments: <paste>".
3. The fix agent edits, re-runs checks, and updates the PR (`gh pr` / push).

**Automated (graduate later):**
1. A GitHub webhook or Actions workflow fires on `pull_request_review` events.
2. It hands the comments to the agent.
3. The agent responds and updates the PR.

The manual path keeps you in control and needs no infrastructure. Start there.

### 9.3 The loop continues until merge
After the fix agent updates the PR, you review again. This repeats until you approve. Each cycle is: **your comment → agent fix → re-review**. The loop terminates when you merge.

### 9.4 Closing the loop: what the agent learns
After merge, the agent can write a short "what I learned" note (a skill or a memory entry). This is optional but is how the loop *improves over time* — the "self-evolving" part you were curious about. It is not required to start.

---

## 10. WordPress-Specific Playbook

This chapter maps the generic loop onto WordPress realities.

### 10.1 The Gutenberg monorepo
Gutenberg is a monorepo: `packages/` (JS/TS blocks and packages), `lib/` (PHP), `phpunit/` (PHP tests), `test/` (JS tests). When the agent works here:
- Point it at the exact package path.
- Use the repo's own lint/test scripts.
- Be aware of the build step (`npm run build`) before testing.

### 10.2 Common WordPress tasks and their verification

| Task | Where | Verify with |
|---|---|---|
| Register a block | `block.json` + `index.js` | `npm run build`, manual render |
| PHP function/class | `lib/` or plugin | `composer run phpcs`, `phpunit` |
| Block editor JS | `packages/block-editor/` | `npm run test-unit`, `npm run lint` |
| REST API endpoint | `lib/` or plugin | `phpunit`, manual `curl` |
| Theme (block theme) | `theme.json`, templates | `npm run build`, visual check |

### 10.3 WordPress conventions the agent must respect
- **i18n:** use `__()`, `_e()`, `esc_html__()` — never hardcoded strings.
- **Escaping:** escape output (`esc_html`, `esc_attr`, `wp_kses`).
- **Nonces:** for any form/action.
- **Coding standards:** WordPress Coding Standards (PHPCS), ESLint for JS.
- **Accessibility:** keyboard nav, ARIA, focus management.
- **Back-compat:** don't break older WP versions unless the spec says so.

### 10.4 The spec for a WordPress task — example
See Chapter 12 for a full worked example (a custom block).

### 10.5 Where the code-graph tools help in WordPress
- **graphify / codegraph:** when the agent needs to find a function signature or callers across the huge Gutenberg tree, these return the answer in far fewer tokens than grep. Use them during the build/fix stages.
- They are *accelerators*, not the loop. The loop works without them.

---

## 11. The Tools You Researched, Decoded

You spent time on graphify, codegraph, TencentDB-Agent-Memory, and ECC. Here is the honest, consolidated view.

### 11.1 The categories

| Tool | Category | What it does | Role in your loop |
|---|---|---|---|
| **graphify** | Code-graph index | AST → knowledge graph; cheap symbol/relationship lookup | Optional accelerator for search |
| **codegraph** (colbymchenry) | Code-graph index | Rust AST index; verbatim source + blast radius; auto-sync | Optional accelerator for search |
| **TencentDB-Agent-Memory** | Memory platform | Chat memory, skills, wiki, code-graph; team governance | Overkill for solo local loop |
| **ECC** | Workflow/skills/hooks layer | 67 agents, 284 skills, memory, hooks | Optional workflow/memory layer |

### 11.2 The key insight
- **graphify / codegraph** solve *search* (the middle of the loop). They do not build the loop.
- **ECC** and **TencentDB** solve *memory and workflow* (making the loop smarter over time). They do not build the loop either.
- **opencode** is the *runtime* that runs the loop. It is the core.

### 11.3 What you actually need to start
1. opencode (you have it).
2. `gh` (GitHub CLI) for PRs.
3. Markdown files for specs, plans, PR bodies.
4. 3–4 hand-written agents.

That's it. Everything else is optional polish.

### 11.4 When to add the accelerators
- Add **codegraph/graphify** when the middle of the loop (search) is slow or token-heavy. For Gutenberg, this is a real win.
- Add **ECC** when you want the loop to "learn" (instincts, memory) and you accept a large install footprint.
- Add **TencentDB** when you have a team and need governance/sharing.

---

## 12. A Complete Worked Example

Let's walk a real WordPress task through the whole loop: **"Add a 'Latest Posts by Category' block."**

### 12.1 Gate 1 — The spec

```markdown
# Spec: Latest Posts by Category Block

## Goal
Add a block that lists the latest posts from a chosen category, with a
configurable number of posts and a "View all" link.

## Context
This is a new block in the block-library. Reference existing blocks:
- packages/block-library/src/latest-posts/ (similar pattern)
- packages/block-library/src/query/ (for category filtering)

## Requirements
1. The block renders a list of the latest N posts from a selected category.
2. N is configurable in the inspector (1–20, default 5).
3. The category is selectable in the inspector.
4. Each item links to the post.
5. A "View all" link points to the category archive.
6. All strings are translatable.

## Non-Goals
- No pagination.
- No custom styling beyond a minimal list.

## Constraints
- Must use block.json registration.
- Must pass PHPCS and ESLint.
- Must work with WP 6.4+.
- Must be accessible (semantic list, keyboard nav).

## Acceptance Criteria
- [ ] Block appears in the inserter.
- [ ] Selecting a category shows that category's posts.
- [ ] Changing N changes the count.
- [ ] "View all" links to the category archive.
- [ ] All strings use __().

## Definition of Done
- [ ] PHPCS passes
- [ ] ESLint passes
- [ ] PHPUnit tests for the render callback
- [ ] Jest tests for the editor
- [ ] PR opened with summary
```

### 12.2 Gate 2 — The plan

```markdown
# Plan: Latest Posts by Category Block

## Summary
Create a new block-library block that renders recent posts from a category,
with inspector controls for count and category.

## Tasks
1. **Scaffold block** — create `packages/block-library/src/latest-posts-by-category/`
   with `block.json`, `index.js`, `edit.js`, `save.js`, `render.php`.
   Verify: block appears in inserter after build.
2. **Render callback** — PHP `render.php` queries posts by category via
   `WP_Query`, outputs a list. Verify: PHPUnit test.
3. **Editor controls** — `edit.js` adds InspectorControls for category
   (SelectControl) and count (RangeControl). Verify: Jest test.
4. **i18n** — wrap all strings in `__()`. Verify: lint.
5. **Accessibility** — semantic `<ul>`/`<li>`, focusable links. Verify: manual.

## Risks
- Category query performance on large sites — mitigate with `posts_per_page` cap.

## Open Questions
- Should the block support multiple categories? (Spec says single; confirm.)

## Definition of Done
- (copy from spec)
```

### 12.3 Gate 3 — Implementation
The build agent implements task by task, running:
- `npm run lint`
- `npm run test-unit`
- `composer run phpcs`
- `composer run phpunit`

Then the review agent reads the diff, then the agent opens the PR.

### 12.4 Gate 4 — The PR

```markdown
## Summary
Adds a "Latest Posts by Category" block to the block-library.

## Changes
- packages/block-library/src/latest-posts-by-category/block.json: registration
- .../render.php: PHP render callback with WP_Query
- .../edit.js: inspector controls for category and count
- .../index.js: block entry

## Tests
- PHPUnit: render callback tests pass
- Jest: editor tests pass
- PHPCS / ESLint: clean

## Review Focus
- The WP_Query in render.php (performance + escaping)
- The category SelectControl (data flow)

## Related
- Spec: docs/specs/latest-posts-by-category.md
- Plan: docs/plans/latest-posts-by-category.md
```

### 12.5 The feedback loop
You review. You comment: "The `WP_Query` on line 30 needs `'no_found_rows' => true` for performance." The fix agent adds it, re-runs checks, updates the PR. You re-review, approve, merge.

---

## 13. Troubleshooting & Common Pitfalls

### 13.1 The agent goes off-spec
**Fix:** The spec was too vague. Tighten the requirements and acceptance criteria. The plan gate exists to catch this — approve only plans that match the spec.

### 13.2 The agent claims done but checks fail
**Fix:** Make "definition of done" explicit and have the build agent run checks itself. Do not trust "it should work" — require the command output.

### 13.3 The PR is too big to review
**Fix:** Break the spec into smaller PRs. Each PR should be reviewable in one sitting. This is a spec/planning problem, not a tool problem.

### 13.4 The agent can't find the right code in Gutenberg
**Fix:** This is where codegraph/graphify help. Or point the agent at the exact file path in the spec. Gutenberg is huge; give the agent a map.

### 13.5 Review comments are vague
**Fix:** You control this. Write specific, actionable comments. The fix agent can only act on what you say.

### 13.6 The loop never terminates
**Fix:** Define a merge criterion. If you keep finding new issues, that's scope creep — go back to the spec and decide whether to expand it or merge and iterate in a new PR.

### 13.7 Permission mistakes
**Fix:** Give the plan agent `edit: deny`. Give the build/fix agents edit permission. Never give an agent more permission than its role needs.

---

## 14. Glossary

- **Agent** — a configured runtime instance with a role, prompt, and permissions.
- **Command** — a reusable prompt you invoke to trigger a workflow.
- **Hook** — code that runs on events; used to automate parts of the loop.
- **Spec** — the macro-requirements document (the "what and why").
- **Plan** — the agent's task breakdown (the "how"), approved by the human.
- **PR** — the unit of review; the agent opens it, you review it.
- **Handoff** — any written document that passes state between stages.
- **Definition of done** — the checklist that must be true for a task to be complete.
- **Accelerator** — an optional tool (code-graph, memory) that makes the loop faster or smarter.

---

## 15. Appendix: Reference Files

### 15.1 opencode agent file (`.opencode/agent/plan.md`)

```markdown
---
description: Reads a spec and produces a plan. No edits.
mode: subagent
permission:
  edit: deny
---

You are a planning agent. Read the spec at the given path. Produce a plan
with tasks, risks, open questions, and a definition of done. Do NOT edit
any code. Stop and wait for approval.
```

### 15.2 opencode agent file (`.opencode/agent/build.md`)

```markdown
---
description: Implements an approved plan. Edits code and runs checks.
mode: subagent
permission:
  edit: allow
---

You are a build agent. Implement the approved plan task by task. After each
task, run the project's lint and test commands. Do not claim a task is done
until its checks pass. When all tasks are done, open a PR with a structured
body.
```

### 15.3 opencode agent file (`.opencode/agent/review.md`)

```markdown
---
description: Reviews a diff in a fresh context before PR.
mode: subagent
permission:
  edit: deny
---

You are a review agent. Read the current diff. Look for bugs, edge cases,
missing tests, style violations, and unintended changes. Report findings.
Do not edit code.
```

### 15.4 opencode agent file (`.opencode/agent/fix.md`)

```markdown
---
description: Addresses GitHub review comments on a PR.
mode: subagent
permission:
  edit: allow
---

You are a fix agent. Read the review comments provided. Address each one,
re-run checks, and update the PR. Report what you changed and why.
```

### 15.5 opencode command (`.opencode/command/start-loop.md`)

```markdown
---
description: Start the agentic loop from a spec.
---

Read the spec at $1. Produce a plan and stop for approval. Do not edit code.
```

### 15.6 opencode command (`.opencode/command/address-review.md`)

```markdown
---
description: Address GitHub review comments.
---

Address the following review comments on the current PR: $ARGUMENTS
Re-run checks and update the PR.
```

---

## Closing

You now have the complete picture. The loop is:

**spec → plan → build → PR → review → fix → merge**

Human at the edges, agent in the middle, every handoff a written document. Start small — one spec, three agents, one PR. Add the accelerators (codegraph, ECC) only when the middle gets slow or you want the loop to learn.

The tools you researched are ingredients. The loop is the recipe. Now go cook.
