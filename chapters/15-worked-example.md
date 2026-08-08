# 15 — A Complete Worked Example

Let's walk a real WordPress task through the whole loop: **"Add a 'Latest Posts by
Category' block."** This uses the modern patterns from Chapters 04–06 and 12.

## 15.1 Gate 1 — The spec

```markdown
# Spec: Latest Posts by Category Block

## Goal
Add a block that lists the latest posts from a chosen category, with a
configurable number of posts and a "View all" link.

## Context
This is a new block in the block-library. Reference existing blocks:
- packages/block-library/src/latest-posts/ (similar pattern)
- packages/block-library/src/query/ (for category filtering)

## Requirements
1. The block renders a list of the latest N posts from a selected category.
2. N is configurable in the inspector (1–20, default 5).
3. The category is selectable in the inspector.
4. Each item links to the post.
5. A "View all" link points to the category archive.
6. All strings are translatable.

## Non-Goals
- No pagination.
- No custom styling beyond a minimal list.

## Constraints
- Must use `block.json` registration with `apiVersion: 3` (introduced in WordPress 6.3).
- Must pass PHPCS and ESLint.
- Must work with WP 6.3+ (the minimum for `apiVersion: 3`).
- Must be accessible (semantic list, keyboard nav).

## Acceptance Criteria
- [ ] Block appears in the inserter.
- [ ] Selecting a category shows that category's posts.
- [ ] Changing N changes the count.
- [ ] "View all" links to the category archive.
- [ ] All strings use __().

## Definition of Done
- [ ] PHPCS passes
- [ ] ESLint passes
- [ ] PHPUnit tests for the render callback
- [ ] Jest tests for the editor
- [ ] PR opened with summary

autonomy: middleman
```

## 15.2 Gate 2 — The plan (conductor-style)

```markdown
# Plan: Latest Posts by Category Block

## Summary
Create a new block-library block that renders recent posts from a category,
with inspector controls for count and category.

## Phase 1: Scaffold [checkpoint: ]
- [ ] Task: Scaffold block — create
  `packages/block-library/src/latest-posts-by-category/` with `block.json`,
  `index.js`, `edit.js`, `render.php`. Verify: block appears in inserter after build.

## Phase 2: Backend (TDD) [checkpoint: ]
- [ ] Task: Render callback — write failing PHPUnit test for `render.php`
  (queries posts by category via `WP_Query`, outputs a list). Then implement.
  Verify: `npm run test:php`.

## Phase 3: Editor (TDD) [checkpoint: ]
- [ ] Task: Editor controls — write failing Jest test for `edit.js`
  (InspectorControls: SelectControl for category, RangeControl for count).
  Then implement. Verify: `npm run test:unit`.

## Phase 4: Polish [checkpoint: ]
- [ ] Task: i18n — wrap all strings in `__()`. Verify: lint.
- [ ] Task: Accessibility — semantic `<ul>`/`<li>`, focusable links. Verify: manual.

## Risks
- Category query performance on large sites — mitigate with `posts_per_page` cap
  and `no_found_rows => true`.

## Open Questions
- Should the block support multiple categories? (Spec says single; confirm.)

## Definition of Done
- (copy from spec)
```

## 15.3 Gate 3 — Implementation

The implementor works task by task, running (in `middleman` mode, pausing after each
phase):

- `npm run lint:js`
- `npm run test:unit`
- `npm run lint:php` (or `composer run lint`)
- `npm run test:php`

Then the QC agent runs the built-in `/review`, the code-reviewer reads the diff, and the agent opens
the PR.

## 15.4 Gate 4 — The PR

```markdown
## Summary
Adds a "Latest Posts by Category" block to the block-library.

## Changes
- packages/block-library/src/latest-posts-by-category/block.json: registration
- .../render.php: PHP render callback with WP_Query
- .../edit.js: inspector controls for category and count
- .../index.js: block entry

## Tests
- PHPUnit: render callback tests pass
- Jest: editor tests pass
- PHPCS / ESLint: clean

## Review Focus
- The WP_Query in render.php (performance + escaping)
- The category SelectControl (data flow)

## Related
- Spec: docs/specs/latest-posts-by-category.md
- Plan: docs/plans/latest-posts-by-category.md
```

## 15.5 The feedback loop

You review. You comment: "The `WP_Query` on line 30 needs `'no_found_rows' => true` for
performance." The fix agent adds it, re-runs checks, updates the PR. You re-review,
approve, merge.

## 15.6 A second example: extending a core block

For a task like the swirl (Chapter 13), the spec would instead say:

- Use `register_block_style( 'core/group', 'swirl' )`.
- Use a `render_block_core/group` filter + `WP_HTML_Tag_Processor` to stamp
  `data-wp-*` directives.
- Render a singleton overlay into `wp_footer` with `data-wp-router-region`.

The plan would have phases for the style registration, the render filter, the
Interactivity store, and the overlay — each with a TDD step where feasible.
