# 10 — Gate 4: The PR Review in GitHub

## 10.1 The PR body as a contract

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

## 10.2 Your review

You review in GitHub as you normally would. You can:

- Approve and merge (loop ends).
- Request changes with comments (loop continues).

## 10.3 The review comment as a handoff

Your comments are the input to the next stage. They should be specific and actionable:

- Good: "The `get_post_meta` call on line 42 needs a default value."
- Bad: "This doesn't work."

## 10.4 QC vs PR Reviewer

In the full pipeline (Chapter 14), there are **two** review layers:

- **QC** — a local review agent that runs the **built-in `/review`** on the working tree
  *before* the PR
  is opened. It checks bugs, style, tests, and unintended changes. It has `edit: deny`.
- **PR Reviewer** — **you**, the human, doing the review in GitHub. This is the final
  quality gate and the merge decision.

The QC catches what the builder missed; the PR Reviewer is the human-in-the-loop at the
end.
