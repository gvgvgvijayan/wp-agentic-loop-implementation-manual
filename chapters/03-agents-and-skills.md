# 03 — AGENTS.md & Skills: The Real Agent Contract

The original draft of this book defined the loop around custom `.opencode/agent/*.md`
files. That is still a valid opencode mechanism, but the **dominant pattern in the real
WordPress ecosystem is `AGENTS.md`** — a repo-level contract that any agent reads on
entry. This chapter shows you how the big repos do it and how it pairs with opencode.

## 3.1 What `AGENTS.md` is

`AGENTS.md` is a markdown file at the repo root that onboards an agent into the
codebase. It is the single source of truth for "how to work here." opencode reads it
natively and injects it into the agent's context.

Gutenberg's `AGENTS.md` is a great model. It contains:

1. **Dev environment tips** — exact setup and run commands.
2. **Key directories** — where things live.
3. **Progressive discovery** — read only what the task needs, when it needs it.
4. **Code quality** — lint/format/test commands.
5. **Architectural decisions** — constraints the agent must respect.
6. **Common pitfalls** — things to avoid.
7. **PR instructions** — what must be true before a PR.

## 3.2 A template

```markdown
# AGENTS.md

## Dev environment tips

```bash
nvm use                    # Use the required node version
npm install && composer install
npm run wp-env status      # Always check status first.
npm run wp-env start       # Only start if not already running.

npm start     # Development with watch
npm run build # Production build
```

## Key directories

- `/packages/` — JavaScript packages
- `/lib/` — PHP code
- `/phpunit/` — PHP tests
- `/docs/` — documentation

## Progressive discovery

Read only what your task needs, when it needs it. Some directories carry their own
`AGENTS.md` — read it before changing files there.

## Code quality

```bash
npm run format            # Fix JS formatting
npm run lint:js           # Check JS linting
npm run lint:php          # Check PHP standards (via wp-env)
npm run test:php          # PHPUnit (in-container)
npm run test:unit         # Jest unit tests
```

## Architectural decisions

- (list the constraints that matter for this repo)

## Common pitfalls

- (list the things agents get wrong here)

## PR instructions

- Ensure build passes
- Fix all formatting/linting issues; these are enforced through CI
```

## 3.3 Skills: procedural knowledge

`AGENTS.md` answers "how do I work here?" **Skills** answer "how do I do *this specific
kind of work*?" WooCommerce organizes its procedural knowledge into `.ai/skills/`;
Gutenberg keeps skills under `.agents/skills/` and copies them to `.claude/skills/` via
`npm run agents:setup`:

- `woocommerce-dev-cycle` — testing and linting workflows.
- `woocommerce-local-env` — local environment setup, wp-env commands.
- `woocommerce-code-review` — code review standards and critical violations.
- `woocommerce-git-commit` — commit with conventional messages.
- `woocommerce-git-draft-pr` — create draft PRs with the proper template.

A skill is a `SKILL.md` file in its own folder. Its frontmatter `description` must
front-load the trigger keywords so the model knows when to load it:

```markdown
---
name: wp-dev-cycle
description: Use when writing or running WordPress tests and linting (PHPUnit, Playwright, PHPCS, ESLint). Covers wp-env setup and the test commands.
---

# WP Dev Cycle

(procedural instructions...)
```

In Gutenberg, run `npm run agents:setup` after changing skills so they are copied to the
Claude-compatible `.claude/skills/` directory.

## 3.4 How this pairs with opencode

- **`AGENTS.md`** is the repo contract — opencode reads it automatically.
- **Skills** are the procedural knowledge — opencode loads them on demand.
- **`.opencode/agent/*.md`** are the *roles* — the pipeline agents (asker, implementor,
  reviewer, etc.) that orchestrate the loop. See [Chapter 14](./14-agent-pipeline.md).
- **`.opencode/command/*.md`** are the *triggers* — `/start-loop`, `/address-review`.
  The local review uses opencode's **built-in `/review`** command (no custom file needed).

The relationship:

| Artifact | Answers | Where it lives |
|---|---|---|
| `AGENTS.md` | "How do I work in this repo?" | repo root |
| Skill | "How do I do this kind of work?" | `.ai/skills/` or `.opencode/skills/` |
| Agent | "What is my role?" | `.opencode/agent/` |
| Command | "What do I trigger?" | `.opencode/command/` |

## 3.5 Progressive discovery

Gutenberg's `AGENTS.md` emphasizes **progressive discovery**: read only what the task
needs, when it needs it. This is a token-saving discipline that matters in a huge
monorepo. The agent should:

- Read the contributor guide for the *kind* of work before starting.
- Read a directory's own `AGENTS.md`/`README.md` before changing files there.
- Not load the whole codebase into context up front.

This is the same principle as the code-graph accelerators: give the agent a map, not
the whole territory.
