# 02 — Core Concepts

Before we build, you need a shared vocabulary.

## 2.1 Agent

An agent is a configured instance of the runtime with a **role**, a **prompt**, and
**permissions**. In opencode, agents are defined in `.opencode/agent/<name>.md` files
(or inline in `opencode.json`).

A good agent has:

- A clear **description** (when to use it).
- A **mode** (`primary` for the main agent, `subagent` for helpers).
- A **permission** policy (what it may edit, what it may run).
- A **prompt** (its instructions and personality).

## 2.2 Command

A command is a reusable prompt you can invoke. In opencode, commands live in
`.opencode/command/<name>.md`. A command can take arguments (`$ARGUMENTS`, `$1`, `$2`).
Commands are how you *trigger* the loop.

## 2.3 Hook

A hook is code that runs on events (before/after a tool, on session start/end, etc.).
Hooks are how you *automate* parts of the loop. For a local setup, you may not need
hooks at all — commands are enough.

## 2.4 AGENTS.md

`AGENTS.md` is the **repo-level agent contract**. It is a markdown file at the repo
root that tells any agent how to work in this codebase: setup commands, key
directories, code-quality commands, architectural decisions, common pitfalls, and PR
instructions. opencode reads it natively. This is the modern pattern — Gutenberg,
WooCommerce, and many production WordPress plugins/themes all ship one. See [Chapter 03](./03-agents-and-skills.md).

## 2.5 Skill

A skill is **procedural, reusable guidance** — a "how-to" for a specific kind of work.
WooCommerce organizes its procedural knowledge into `.ai/skills/` (dev-cycle,
local-env, code-review, git-commit, git-draft-pr). A skill is distinct from an agent
(a role) and a command (a trigger): a skill is *knowledge* an agent can load. See
[Chapter 03](./03-agents-and-skills.md).

## 2.6 Spec

A spec is a markdown document describing *what* to build and *why*. It is the contract
for the whole loop. See [Chapter 07](./07-gate-1-spec.md).

## 2.7 Plan

A plan is the agent's breakdown of the spec into concrete tasks, with acceptance
criteria. It is written *before* any code. See [Chapter 08](./08-gate-2-plan.md).

## 2.8 PR (Pull Request)

A PR is the unit of review. In the loop, the agent opens a PR; you review it; the agent
updates it. See [Chapter 10](./10-gate-4-pr-review.md).

## 2.9 Handoff

A handoff is any written document that passes state from one stage to the next. Specs,
plans, PR bodies, and review comments are all handoffs.

## 2.10 The "definition of done"

A checklist that must be true before the agent considers a task complete. It is part
of the spec and the plan.

## 2.11 wp-env

`wp-env` is the official local WordPress environment (Docker-based). It is the
standard way to run WordPress, PHPUnit, and Playwright locally. See
[Chapter 05](./05-wp-env-and-test-stack.md).

## 2.12 wp-scripts / wp-build

`wp-scripts` is the CLI of `@wordpress/scripts` (build, lint, test). `wp-build` is the
CLI of the new `@wordpress/build` package — an esbuild-based, convention-driven build
tool that is the planned future engine *under* `wp-scripts`. See
[Chapter 04](./04-toolchain-wp-scripts-wp-build.md).
