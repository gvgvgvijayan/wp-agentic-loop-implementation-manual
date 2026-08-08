# Table of Contents — The WordPress Agentic Loop

This is the **indexer** for the manual. Each chapter has a one-line summary, its
prerequisites, and its place in the reading order.

## Reading order

- **Part I — Concepts** (00–02): read first, in order.
- **Part II — Toolchain** (03–06): read before any implementation.
- **Part III — The Loop** (07–11): the four gates + feedback.
- **Part IV — WordPress in Practice** (12–15): playbook, pipeline, worked example.
- **Part V — Reference** (16–18): troubleshooting, glossary, appendix.

---

## Part I — Concepts

| # | Chapter | Summary | Prereqs |
|---|---------|---------|---------|
| 00 | [Preface](./chapters/00-preface.md) | What this book is, who it's for, what you'll be able to do. | — |
| 01 | [The Big Picture](./chapters/01-big-picture.md) | The loop in one diagram; why human-in-the-loop at the edges; the three layers. | 00 |
| 02 | [Core Concepts](./chapters/02-core-concepts.md) | Shared vocabulary: agent, command, hook, spec, plan, PR, handoff, definition of done. | 01 |

## Part II — Toolchain

| # | Chapter | Summary | Prereqs |
|---|---------|---------|---------|
| 03 | [AGENTS.md & Skills](./chapters/03-agents-and-skills.md) | The real agent contract: how Gutenberg/WooCommerce onboard agents, and how opencode consumes it. | 02 |
| 04 | [wp-scripts & wp-build](./chapters/04-toolchain-wp-scripts-wp-build.md) | The modern build toolchain: `@wordpress/scripts` commands and the new `@wordpress/build` (`wp-build`). | 02 |
| 05 | [wp-env & the Test Stack](./chapters/05-wp-env-and-test-stack.md) | Local env, `.wp-env.json`, the `env:setup` pattern, PHPUnit-in-container, Playwright. | 04 |
| 06 | [TDD in the Loop](./chapters/06-tdd-in-the-loop.md) | Red/Green/Refactor, coverage gate, checkpoints, git notes, and the autonomy knob. | 05 |

## Part III — The Loop

| # | Chapter | Summary | Prereqs |
|---|---------|---------|---------|
| 07 | [Gate 1 — The Spec](./chapters/07-gate-1-spec.md) | Macro requirements: the "what and why" the agent plans against. | 02, 03 |
| 08 | [Gate 2 — The Plan](./chapters/08-gate-2-plan.md) | The agent's task breakdown, approved by the human before any code. | 07 |
| 09 | [Gate 3 — Implement & Verify](./chapters/09-gate-3-implement-verify.md) | Build, verify, self-review, open the PR. Correct commands for Gutenberg/plugin/theme. | 08, 04–06 |
| 10 | [Gate 4 — The PR Review](./chapters/10-gate-4-pr-review.md) | The PR body as a contract; your review; the review comment as a handoff. | 09 |
| 11 | [The Feedback Loop](./chapters/11-feedback-loop.md) | The fix agent; manual vs automated feedback; the loop until merge. | 10 |

## Part IV — WordPress in Practice

| # | Chapter | Summary | Prereqs |
|---|---------|---------|---------|
| 12 | [WordPress Playbook](./chapters/12-wordpress-playbook.md) | Modern architecture: PSR-4, PHP-Scoper, `block.json`+`render.php`, Interactivity API, Data Views, `theme.json`. | 04–06 |
| 13 | [Render-Block Hooks, HTML API & IAPI](./chapters/13-render-block-hooks-and-iapis.md) | Extending core blocks: `render_block` filters, `WP_HTML_Tag_Processor`, Interactivity directives. From the swirl case study. | 12 |
| 14 | [The Agent Pipeline](./chapters/14-agent-pipeline.md) | The full handoff chain: requirement-asker → … → PR-reviewer, plus ADR/research/domain agents. | 03, 07–11 |
| 15 | [A Complete Worked Example](./chapters/15-worked-example.md) | A block built through the whole loop, with a conductor-style phased plan. | 07–14 |

## Part V — Reference

| # | Chapter | Summary | Prereqs |
|---|---------|---------|---------|
| 16 | [Troubleshooting & Pitfalls](./chapters/16-troubleshooting.md) | wp-env gotchas, host-vs-container, Node pin, two-fix-stop rule, and more. | all |
| 17 | [Glossary](./chapters/17-glossary.md) | Every term used in the manual. | — |
| 18 | [Appendix: Reference Files](./chapters/18-appendix-reference.md) | `AGENTS.md` template, skills, `wp-env`/`env:setup`, agent & command files. | all |

---

## Companion files

| File | Purpose |
|------|---------|
| [`checklist.md`](./checklist.md) | 3-level implementation checklist an agent can follow to reproduce the loop in another system. |
| [`opencode/`](./opencode/) | Documented agent ecosystem (agents, commands, project `opencode.json`). Self-contained; does not touch global config. |

