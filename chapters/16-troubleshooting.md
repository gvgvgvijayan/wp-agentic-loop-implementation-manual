# 16 — Troubleshooting & Common Pitfalls

## 16.1 The agent goes off-spec

**Fix:** The spec was too vague. Tighten the requirements and acceptance criteria. The
plan gate exists to catch this — approve only plans that match the spec.

## 16.2 The agent claims done but checks fail

**Fix:** Make "definition of done" explicit and have the build agent run checks itself.
Do not trust "it should work" — require the command output. Use the two-fix-stop rule
(Chapter 06).

## 16.3 The PR is too big to review

**Fix:** Break the spec into smaller PRs. Each PR should be reviewable in one sitting.
This is a spec/planning problem, not a tool problem.

## 16.4 The agent can't find the right code in Gutenberg

**Fix:** This is where codegraph/graphify help. Or point the agent at the exact file
path in the spec. Gutenberg is huge; give the agent a map.

## 16.5 Review comments are vague

**Fix:** You control this. Write specific, actionable comments. The fix agent can only
act on what you say.

## 16.6 The loop never terminates

**Fix:** Define a merge criterion. If you keep finding new issues, that's scope creep —
go back to the spec and decide whether to expand it or merge and iterate in a new PR.

## 16.7 Permission mistakes

**Fix:** Give the plan/review/QC agents `edit: deny`. Give the build/fix agents edit
permission. Never give an agent more permission than its role needs.

## 16.8 wp-env gotchas

| Symptom | Cause / fix |
|---|---|
| **403 Forbidden** on REST requests | Permissions not applied. Re-run `env:setup`. |
| **Empty dropdowns** | Meta missing or cache stale. Re-run `env:setup`. |
| **500 errors** | PHP fatal in `render.php`. Check container logs: `npx wp-env run cli tail -n 50 /var/www/html/wp-content/debug.log`. |
| **Playwright strict-mode failures** | Hidden steps still in the DOM. Use specific locators. |
| **`window is not defined`** | Node version too new. Pin to the LTS the scripts support. |

## 16.9 Host vs container

**Never run `vendor/bin/phpunit` directly on the host.** The host does not have
WordPress loaded. Run it inside the `wp-env` container (`npm run test:unit:php:base`).

## 16.10 The agent cites a fabricated "canonical pattern"

**Fix:** This is the swirl lesson (Chapter 13). When an agent claims a pattern is
canonical, verify it against the real core source before building on it. The QC/review
gate exists to catch this.

## 16.11 The agent thrashes on failing tests

**Fix:** Enforce the two-fix-stop rule. After two failed fix attempts, the agent must
stop and ask you for guidance.

## 16.12 The agent doesn't know the repo's commands

**Fix:** This is what `AGENTS.md` is for. If the agent runs the wrong commands, the
`AGENTS.md` quick-start and code-quality sections are incomplete. Update them.
