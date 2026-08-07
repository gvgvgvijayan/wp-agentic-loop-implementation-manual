# 07 — Gate 1: The Spec (Macro Requirements)

## 7.1 What "macro requirements" means

Macro requirements are the *what* and *why*, stated at a level the agent can plan
against but not so detailed that you are doing the design. You are the product owner;
the agent is the engineer.

## 7.2 The spec template

```markdown
# Spec: <Short Name>

## Goal
One or two sentences: what are we building and why does it matter?

## Context
Background the agent needs. Link relevant files, docs, or existing code.
For WordPress: link the relevant block, plugin, or core file.

## Requirements
A numbered list of concrete, testable requirements.
Each should be verifiable ("the block renders X when Y").

## Non-Goals
What we are explicitly NOT doing. This prevents scope creep.

## Constraints
- Tech stack / conventions (e.g. "must pass PHPCS", "must use block.json")
- Compatibility (e.g. "must work with WP 6.4+")
- Performance / accessibility / i18n requirements

## Acceptance Criteria
A checklist that must ALL be true for the work to be "done."
This becomes the plan's definition of done.

## Definition of Done
- [ ] Code passes lint (PHPCS / ESLint)
- [ ] Tests pass (PHPUnit / Jest)
- [ ] No regressions in related areas
- [ ] PR opened with summary
```

## 7.3 Writing requirements that agents can act on

Bad: "Make the block better."
Good: "The block must render a heading with the post title, and must be translatable
via `__()`."

The difference: the good requirement is **testable**. The agent can verify it.

## 7.4 WordPress-specific spec tips

- Reference the exact file paths (e.g. `packages/block-library/src/heading/`).
- State the WordPress version target.
- State whether this is a block, plugin, theme, or core change.
- If it touches the block editor, mention the relevant APIs (e.g. `useBlockProps`,
  `registerBlockType`).
- If it touches PHP, mention PHPCS/PHPUnit expectations.
- If it extends a core block, mention the mechanism (`render_block` filter,
  `register_block_style`, `WP_HTML_Tag_Processor`). See Chapter 13.

## 7.5 The autonomy header

Add an optional header so the loop knows how to run:

```markdown
autonomy: middleman   # or: end-of-loop
```

See Chapter 06.

## 7.6 Where specs live

Keep them in the repo under `docs/specs/`. They are versioned, reviewable, and become
the PR's reference. This is the "persist everything" principle in action.

## 7.7 The requirement-asker

In the full agent pipeline (Chapter 14), the spec is not written cold. A
**requirement-asker** agent interviews you via the `question` tool to cover all the
needed details, then a **requirement-generator** turns the interview notes into the
spec. This is especially valuable when the requirement is vague or touches multiple
domains.
