# 18 — Appendix: Reference Files

This appendix collects the reference files the loop needs. The `opencode/` directory in
this manual contains the **documented** agent ecosystem; the templates below are the
starting points.

## 18.1 `AGENTS.md` template

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

## 18.2 Skill file template

```markdown
---
name: wp-dev-cycle
description: Use when writing or running WordPress tests and linting (PHPUnit, Playwright, PHPCS, ESLint). Covers wp-env setup and the test commands.
---

# WP Dev Cycle

(procedural instructions...)
```

## 18.3 `wp-env` / `env:setup` reference

`.wp-env.json`:

```json
{
  "core": null,
  "phpVersion": "8.3",
  "plugins": [ "." ],
  "mappings": {
    "wp-content/plugins/my-plugin": "."
  },
  "config": {
    "WP_DEBUG": true,
    "WP_DEBUG_LOG": true,
    "SCRIPT_DEBUG": true
  }
}
```

`package.json` test scripts:

```json
{
  "scripts": {
    "test:unit:php": "npm run test:unit:php:setup && npm run test:unit:php:base",
    "test:unit:php:base": "wp-env run tests-cli --env-cwd=wp-content/plugins/my-plugin vendor/bin/phpunit",
    "test:unit:php:setup": "wp-env start",
    "test:e2e": "wp-scripts test-e2e --config playwright.config.ts"
  }
}
```

## 18.4 opencode agent files

The `opencode/agent/` directory in this manual contains the documented pipeline agents.
A representative example:

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

## 18.5 opencode command files

The `opencode/command/` directory contains the documented custom commands
(`start-loop`, `address-review`). The local review uses opencode's **built-in
`/review`** command — no custom file is needed. A representative custom command:

```markdown
---
description: Start the agentic loop from a spec.
---

Read the spec at $1. Produce a plan and stop for approval. Do not edit code.
```

## 18.6 How to use the `opencode/` directory

The `opencode/` directory is **self-contained** — it does not touch your global
`~/.config/opencode/` config. To use the agents and commands in a project:

1. Copy the agent files you want into the project's `.opencode/agent/`.
2. Copy the command files into the project's `.opencode/command/`.
3. Copy `opencode/opencode.json` into the project root (or merge its `agent`/`command`
   sections into the project's existing `opencode.json`).
4. Restart opencode for the changes to take effect.

## 18.7 The implementation checklist

For a step-by-step, verifiable path to reproducing the loop in another system, follow
[`checklist.md`](../checklist.md) top-to-bottom. It is the "definition of done" for the
whole implementation.
