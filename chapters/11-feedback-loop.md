# 11 — The Feedback Loop

## 11.1 The fix agent

The fix agent reads your review comments and addresses them. It is a fresh agent with
edit permission, pointed at the PR.

## 11.2 Two ways to feed comments back

**Manual (recommended to start):**

1. You review in GitHub.
2. You copy your comments into opencode: "Address these review comments: <paste>".
3. The fix agent edits, re-runs checks, and updates the PR (`gh pr` / push).

**Automated (graduate later):**

1. A GitHub webhook or Actions workflow fires on `pull_request_review` events.
2. It hands the comments to the agent.
3. The agent responds and updates the PR.

The manual path keeps you in control and needs no infrastructure. Start there.

## 11.3 The loop continues until merge

After the fix agent updates the PR, you review again. This repeats until you approve.
Each cycle is: **your comment → agent fix → re-review**. The loop terminates when you
merge.

## 11.4 Closing the loop: what the agent learns

After merge, the agent can write a short "what I learned" note (a skill or a memory
entry). This is optional but is how the loop *improves over time* — the "self-evolving"
part. It is not required to start.

## 11.5 The `/address-review` command

In the pipeline, a command wraps this:

```markdown
---
description: Address GitHub review comments.
agent: wp
---

Address the following review comments on the current PR: $ARGUMENTS
Re-run checks and update the PR.
```

See Chapter 14 and the `opencode/command/` reference files.
