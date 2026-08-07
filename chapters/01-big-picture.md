# 01 — The Big Picture

## 1.1 The loop in one diagram

```
        ┌────────────────────────────────────────────────────────────┐
        │                                                            │
        ▼                                                            │
[HUMAN] Macro Requirements ──► [AGENT] Plan ──► [AGENT] Implement ──► [AGENT] Test/Verify
   (spec file)                    (no edits)      (edits + tests)          │
                                                                           │
[HUMAN] PR Review in GitHub ◄── [AGENT] Open PR ◄──────────────────────────┘
   (you review)                    (gh pr create)
        │
        └──► [AGENT] Address feedback ──► Update PR ──► back to review ──► merge
```

## 1.2 The principle that makes it work

> **Every handoff is a written contract, not a chat.**

The agent's "memory" is the files on disk, not the conversation. When you hand the
agent requirements, you write a spec. When the agent plans, it writes a plan. When it
opens a PR, it writes a structured body. When you review, you write comments. Each of
these is a *document* that the next stage consumes.

This is the same principle behind ECC's tagline — *"Optimize the context window.
Persist everything else."* — and behind TencentDB's "memory assets." You do not need
their infrastructure to benefit from the idea. You need discipline: **write things
down.**

## 1.3 Why human-in-the-loop at the edges

- **At the start:** The agent cannot read your mind. "Macro requirements" means the
  *what and why*, not the *how*. You supply intent; the agent supplies execution.
- **At the end:** The agent cannot judge quality the way you can. A PR review is your
  quality gate. You are the final reviewer, the one who merges.

The middle — planning, coding, testing, iterating — is where the agent earns its keep.

## 1.4 The three layers of any agentic system

1. **The Runtime** — the thing that runs the agent (opencode).
2. **The Workflow** — the loop itself (spec → plan → build → PR → review → fix).
3. **The Accelerators** — optional tools that make the middle faster (code-graph search,
   memory, skills).

Most people start by installing accelerators. This book starts with the workflow,
because the workflow is what actually delivers value.

## 1.5 The autonomy spectrum

The loop is not binary. Between "full autonomous" and "pair programming" there is a
spectrum, and you choose where you sit **each time you trigger the loop**:

- **`end-of-loop` (near-autonomous):** the agent runs spec → PR and pauses only once,
  right before opening the PR, for a final review and a manual-verification step.
- **`middleman` (pair-programming):** the agent pauses at each phase boundary (after
  red, after green, after refactor) and presents a manual-verification plan, awaiting
  your explicit "yes" before continuing.

This is covered in depth in [Chapter 06](./06-tdd-in-the-loop.md) and
[Chapter 09](./09-gate-3-implement-verify.md). For now, hold the idea: **the loop is a
knob, not a switch.**
