# 08 — Gate 2: The Plan

## 8.1 What a plan is

A plan is the agent's translation of the spec into concrete work. It should include:

- A task breakdown (ordered steps).
- For each task: what file(s) change, what the change does, how it is verified.
- Risks and open questions.
- The definition of done (from the spec).

## 8.2 The plan template

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

## 8.3 The conductor-style phased plan

For larger work, use the conductor pattern: group tasks into **phases**, each with a
checkpoint. Each task records its commit SHA when done:

```markdown
# Plan: Establish Testing Infrastructure

## Phase 1: Backend (PHPUnit) [checkpoint: dc8e29a]
- [x] Task: Scaffold PHPUnit & wp-env Environment 59c4775
    - [x] Configure `.wp-env.json`
    - [x] Create `phpunit.xml.dist`
    - [x] Create `tests/phpunit/bootstrap.php`
- [x] Task: Test Database & Lifecycle (TDD) fb2f9b5
    - [x] Write failing test for `DB\Installer`
    - [x] Write failing tests for `DB\Appointments` CRUD
    - [x] Implement to pass tests

## Phase 2: Frontend (Playwright) [checkpoint: fcf677e]
- [x] Task: Scaffold Playwright & Gutenberg E2E Utilities 77a4337
    ...
```

The `[checkpoint: <sha>]` markers and per-task SHAs make the plan an auditable record.

## 8.4 Why the plan stops the agent

The plan agent has `edit: deny` permission. It cannot change code. It can only read and
write the plan. This forces it to *think before acting* — and it gives you a cheap
checkpoint.

## 8.5 The approval handoff

You read the plan. If it is wrong, you edit the spec or the plan and re-run. If it is
right, you tell the agent "approved, proceed." That single word is the handoff.

## 8.6 The task-generator

In the full pipeline (Chapter 14), a **task-generator** agent turns the spec into this
phased plan, applying the TDD discipline from Chapter 06 (each task has a
"write failing test" step).
