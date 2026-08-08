# 06 — TDD in the Loop

The original draft treated verification as "run the checks." Real WordPress
projects that use a **conductor** workflow treat it as a formal **test-driven
development** discipline. This chapter folds that discipline into the loop.

## 6.1 The Red/Green/Refactor cycle

Every task follows a strict lifecycle:

1. **Write failing tests (Red).** Create a test file that defines the expected
   behavior. **Run it and confirm it fails as expected.** Do not proceed until you have
   failing tests.
2. **Implement to pass (Green).** Write the minimum code to make the tests pass. Run
   the suite and confirm all pass.
3. **Refactor (optional but recommended).** With passing tests as a safety net, clean
   up the code without changing behavior. Re-run tests.

> The "Red" phase is non-negotiable. If you never see the test fail, you have not
> proven the test tests anything.

## 6.2 The coverage gate

The conductor workflow targets **>80% code coverage** for new code. The agent must run
a coverage report and verify the target is met before marking a task complete.

## 6.3 The plan state machine

`plan.md` is the source of truth. Tasks move through three states:

- `[ ]` — todo
- `[~]` — in progress
- `[x]` — done

Each completed task records its commit SHA (first 7 chars). This gives an auditable
trail from plan to code.

## 6.4 Checkpoints & git notes

At phase boundaries, the conductor workflow creates a **checkpoint commit** and attaches
a **verification report** via `git notes`:

```bash
git log -1 --format="%H"          # get the commit hash
git notes add -m "<report>" <sha> # attach the report
```

The plan records the checkpoint SHA. This is how the loop *proves* work was done and
verified — not just claimed.

## 6.5 The autonomy knob (middleman vs end-of-loop)

This is the flexibility you asked for. When the loop is triggered, the user chooses a
mode:

- **`middleman` (pair-programming):** the agent pauses at each phase boundary and
  presents a **manual-verification plan**, then **awaits explicit "yes"** before
  continuing. The manual-verification plan is concrete:

  ```
  The automated tests have passed. For manual verification, please follow these steps:
  1. Start the dev server: npm run dev
  2. Open http://localhost:3000
  3. Confirm you see: the new profile page with the user's name and email.
  ```

  The agent asks: "**Does this meet your expectations? Please confirm with yes or
  provide feedback.**" and **pauses** until it gets an explicit answer.

- **`end-of-loop` (near-autonomous):** the agent runs the whole spec → PR cycle and
  pauses only once, right before opening the PR, for a final review and the
  manual-verification step.

The mode is chosen at trigger time — via a command argument or a spec header:

```markdown
# Spec: ...
autonomy: middleman   # or: end-of-loop
```

This gives a spectrum from **full autonomous** to **pair programming**, and you pick
per task.

## 6.6 The "two failed fix attempts then stop" rule

When automated tests fail during verification, the agent may propose a fix **a maximum
of two times**. If the tests still fail after the second proposed fix, the agent must
**stop**, report the persistent failure, and ask the user for guidance. This prevents
an agent from thrashing indefinitely.

## 6.7 Quality gates

Before marking any task complete, verify:

- [ ] All tests pass.
- [ ] Code coverage meets the target (>80%).
- [ ] Code follows the project's style guide.
- [ ] Public functions/methods are documented.
- [ ] Type safety is enforced.
- [ ] No linting or static-analysis errors.
- [ ] Works on mobile (if applicable).
- [ ] Documentation updated if needed.
- [ ] No security vulnerabilities introduced.

## 6.8 Mapping onto the loop

| Loop stage | TDD discipline |
|---|---|
| Gate 2 (plan) | Plan lists tasks; each has a "write failing test" step. |
| Gate 3 (implement) | Red → Green → Refactor per task; coverage gate; checkpoints. |
| Gate 3 (verify) | Run the full test checklist (Chapter 05); two-fix-stop rule. |
| Gate 4 (review) | Manual-verification plan presented to the human (middleman mode). |
