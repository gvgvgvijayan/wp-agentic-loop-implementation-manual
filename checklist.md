# Implementation Checklist — The WordPress Agentic Loop

This checklist is the **definition of done** for reproducing the agentic loop in another
system. It is written so an agent (or a human) can follow it top-to-bottom and end up
with a working, self-contained loop.

It has **three levels of hierarchy**:

- **Level 1 — Phase**: a major stage of the implementation.
- **Level 2 — Deliverable**: a concrete artifact produced in that phase.
- **Level 3 — Action**: a verifiable step that must be true before the deliverable is done.

Check each Level-3 box. A phase is complete when all its Level-3 boxes are checked.

---

## Phase 1 — Foundation

### 1.1 Repo contract (`AGENTS.md`)
- [ ] `AGENTS.md` exists at the repo root.
- [ ] It has a **Quick start** section with exact setup commands (`nvm use`, `npm install`, `composer install`, `wp-env status/start`).
- [ ] It lists **key directories** and what lives in each.
- [ ] It documents **code quality** commands (lint, format, test) verbatim.
- [ ] It records **architectural decisions** that constrain the agent.
- [ ] It lists **common pitfalls** the agent must avoid.
- [ ] It has a **PR instructions** section (build passes, lint clean).

### 1.2 Skills (optional but recommended)
- [ ] Procedural knowledge is split into skills (e.g. `dev-cycle`, `local-env`, `code-review`).
- [ ] Each skill has a `SKILL.md` with a `description` that front-loads trigger keywords.
- [ ] Skills are referenced from agent prompts or `AGENTS.md`.

### 1.3 Toolchain
- [ ] `package.json` scripts exist for build, start, lint, format, test (via `@wordpress/scripts`).
- [ ] `composer.json` scripts exist for PHP lint/format/test.
- [ ] `wp-env` is available and `.wp-env.json` is configured (core, phpVersion, plugins, mappings, config).
- [ ] Node version is pinned (`.nvmrc` or `engines`) to an LTS that `@wordpress/scripts` supports.

---

## Phase 2 — Test Infrastructure

### 2.1 PHPUnit (in-container)
- [ ] `phpunit.xml.dist` exists with a bootstrap and a testsuite.
- [ ] `tests/phpunit/bootstrap.php` loads the environment via `wp-env`.
- [ ] `package.json` has `test:unit:php:setup` (starts wp-env) and `test:unit:php:base` (runs PHPUnit inside the container).
- [ ] PHPUnit is **never** run directly on the host (documented in `AGENTS.md`).

### 2.2 Environment setup (`env:setup`)
- [ ] A setup script grants custom post-type capabilities.
- [ ] It sets pretty permalinks and flushes rewrite rules.
- [ ] It creates linked mock data needed by tests.
- [ ] It is wired as an npm script (e.g. `env:setup`) and documented as **critical** before tests.

### 2.3 Playwright (E2E)
- [ ] `playwright.config.ts` extends `@wordpress/scripts/config/playwright.config`.
- [ ] `test:e2e`, `test:e2e:ui`, `test:e2e:debug` scripts exist.
- [ ] Specs live in a `tests/e2e/specs` (or `specs`) directory.
- [ ] Failed-test artifacts are captured (default `artifacts/`).

### 2.4 Coverage gate
- [ ] A coverage target is defined (e.g. >80% for `includes/`).
- [ ] The coverage command is documented and runnable.

---

## Phase 3 — The Loop

### 3.1 Spec (Gate 1)
- [ ] A spec template exists (Goal, Context, Requirements, Non-Goals, Constraints, Acceptance Criteria, Definition of Done).
- [ ] Specs are stored under `docs/specs/` (versioned, reviewable).
- [ ] Requirements are **testable** ("the block renders X when Y"), not vague.

### 3.2 Plan (Gate 2)
- [ ] A plan template exists (Summary, Tasks, Risks, Open Questions, Definition of Done).
- [ ] The plan agent has `edit: deny` — it cannot change code.
- [ ] The human approves the plan before any implementation.

### 3.3 Implementation (Gate 3)
- [ ] The build agent implements task by task.
- [ ] It runs the project's checks after each task (lint, format, unit tests).
- [ ] A review agent (fresh context) reads the diff before the PR.
- [ ] The agent opens a PR with a structured body (Summary, Changes, Tests, Review Focus, Related).

### 3.4 Review (Gate 4)
- [ ] The PR body answers: what changed, why, and what to review.
- [ ] The human reviews in GitHub.
- [ ] Review comments are specific and actionable.

### 3.5 Feedback loop
- [ ] A fix agent addresses review comments, re-runs checks, updates the PR.
- [ ] The loop repeats until the human approves and merges.

---

## Phase 4 — TDD Discipline (conductor-style)

### 4.1 Red/Green/Refactor
- [ ] Tests are written **before** implementation (Red phase).
- [ ] Tests are confirmed to fail as expected before proceeding.
- [ ] Minimum code is written to pass (Green phase).
- [ ] Refactor happens only with passing tests as a safety net.

### 4.2 Plan state machine
- [ ] `plan.md` tracks tasks as `[ ]` (todo) → `[~]` (in progress) → `[x]` (done).
- [ ] Each completed task records its commit SHA (first 7 chars).

### 4.3 Checkpoints & audit
- [ ] Phase boundaries create checkpoint commits.
- [ ] A verification report is attached to the checkpoint via `git notes`.
- [ ] The plan records the checkpoint SHA.

### 4.4 Autonomy knob
- [ ] The loop supports at least two modes: `middleman` (pause at each phase) and `end-of-loop` (near-autonomous).
- [ ] The mode is chosen when the loop is triggered (command arg or spec header).
- [ ] In `middleman` mode, the agent pauses and presents a manual-verification plan, awaiting explicit "yes".
- [ ] The "two failed fix attempts then stop and ask" rule is documented.

---

## Phase 5 — Agent Ecosystem

### 5.1 Pipeline agents
- [ ] `requirement-asker` interviews the user via the `question` tool.
- [ ] `requirement-generator` turns interview notes into a spec.
- [ ] `wp-lingo-translator` converts layman terms to WordPress terms and reports open questions back to `wp`, which re-invokes the requirement-asker.
- [ ] `task-generator` turns the spec into a phased plan.
- [ ] `tracer-bullet` builds a vertical slice.
- [ ] `issue-creator` opens a GitHub issue from the spec/plan.
- [ ] `implementor` implements the plan.
- [ ] `qc` does a local review of the working tree before the PR.
- [ ] `code-reviewer` reviews the diff before the PR.
- [ ] `PR-reviewer` is the **human** doing the GitHub review.

### 5.2 Supporting agents
- [ ] `adr-agent` produces ADRs and architectural docs.
- [ ] `research-agent` explores and writes back learnings (self-evolving).
- [ ] Domain agents (`interactivity-api`, `html-api`, `block-markup-designer`, `security-agent`) exist as sub-agents.
- [ ] Extended domain agents (`rest-api`, `block-themes`, `plugin-development`, `performance`, `phpstan`, `wpcli-ops`, `playground`, `project-triage`, `abilities-api`) exist as sub-agents.
- [ ] Core-subsystem agents (`query`, `taxonomy`, `media`, `cron`, `multisite`, `rewrite`, `http`, `option`, `formatting`, `customize`, `widgets`, `feed`, `shortcode`, `nav-menu`, `locale`, `sitemaps`, `block-bindings`, `ai-client`) exist as sub-agents.
- [ ] Real-world domain experts (e.g. `hospital-manager` as a sample stub) exist as sub-agents, delegated by the primary agent. Their knowledge is backed by markdown/graph references, not hardcoded business rules.

### 5.3 Commands
- [ ] `/start-loop` triggers the pipeline.
- [ ] `/review` (built-in) runs the local QC review.
- [ ] `/address-review` feeds GitHub comments back to the fix agent.

### 5.4 Permissions
- [ ] Plan/review/QC agents have `edit: deny`.
- [ ] Implement/fix agents have `edit: allow`.
- [ ] No agent has more permission than its role needs.

---

## Phase 6 — Verification & Handoff

### 6.1 Self-check
- [ ] All lint and format commands pass.
- [ ] All unit tests pass (in-container).
- [ ] E2E tests pass for critical flows.
- [ ] Coverage meets the target.
- [ ] No regressions in related areas.

### 6.2 Documentation
- [ ] `AGENTS.md` is up to date with any new commands.
- [ ] The checklist itself is updated if the loop changes.

### 6.3 Handoff
- [ ] The PR body links the spec and plan.
- [ ] The human has reviewed and merged.
- [ ] A "what I learned" note is written (optional, for self-evolution).

---

## Done

When every box above is checked, the loop is reproducible in the target system. The
human owns the edges; the agent owns the middle; every handoff is a written document.
