# 09 — Gate 3: Implementation & Verification

## 9.1 The build agent

The build agent takes the approved plan and implements it. It has edit permission. It
works task by task, verifying each before moving on, following the TDD discipline from
Chapter 06 (Red → Green → Refactor).

## 9.2 Verification is not optional — and the commands are correct

The build agent must run the project's checks. **The commands below are representative**
for modern WordPress repos (verified against Gutenberg, WooCommerce, wp-movies-demo,
x3p0-ideas, advanced-query-loop, and CoBlocks). The exact script names come from the
target repo's `AGENTS.md` and `package.json`. The old draft's `composer run phpcs` /
`composer run phpunit` are wrong for most repos.

### Gutenberg (monorepo)

```bash
npm run lint:js          # ESLint
npm run lint:php         # PHPCS, via wp-env
npm run test:unit        # Jest unit tests
npm run test:php         # PHPUnit, via wp-env
npm run build            # production build
```

### Plugin / theme (using @wordpress/scripts)

```bash
npm run lint:js          # ESLint
npm run lint:css         # Stylelint
composer run lint        # PHPCS (or npm run lint:php)
npm run test:unit:php    # PHPUnit in-container (verify exact name in package.json)
npm run test:e2e         # Playwright (wp-scripts test-e2e)
npm run build            # wp-scripts build --webpack-copy-php --experimental-modules
```

> Always confirm the exact script names in the target repo's `AGENTS.md` and
> `package.json`. The invariant is containerized PHPUnit and the project's own
> lint/test commands, not a specific npm script name.

### The verification order

1. `npm run wp-env status` (or start).
2. `npm run env:setup` (if the repo has one) — critical.
3. Lint (JS, CSS, PHP).
4. Unit tests (PHPUnit in-container, Jest).
5. E2E (Playwright) for user-facing flows.

The agent should not claim a task is done until the checks pass. This is the
"definition of done" enforced.

## 9.3 The autonomy knob in action

- **`middleman` mode:** the agent pauses after each phase, presents a manual-verification
  plan, and awaits your explicit "yes."
- **`end-of-loop` mode:** the agent runs the whole cycle and pauses once, right before
  opening the PR.

## 9.4 Self-review before PR

Before opening a PR, a **review agent** (fresh context) reads the diff and looks for:

- Bugs, edge cases, missing tests.
- Style violations.
- Unintended changes.
- Missing documentation.

This catches what the builder missed. It is cheap insurance.

## 9.5 Opening the PR

The agent uses `gh`:

```bash
gh pr create --title "Fix: <summary>" --body "$(cat pr-body.md)"
```

The PR body is a handoff document (see Chapter 10). It tells you what changed, why, and
what to review.

## 9.6 The two-fix-stop rule

If automated tests fail during verification, the agent may propose a fix **twice**. If
tests still fail after the second attempt, the agent must **stop** and ask you for
guidance. See Chapter 06.
