# Changelog

All notable changes to this manual from the original single-file draft are documented
here. The original `deprecated/wordpress-agentic-loop.md` is retained for reference but
superseded by this chapter-split manual.

## [Unreleased]

### Added
- New **Part II — Toolchain** chapters (03–06):
  - `AGENTS.md` & skills.
  - `@wordpress/scripts` and `@wordpress/build`.
  - `wp-env` and the modern test stack.
  - TDD in the loop.
- New **Part IV** chapters:
  - Render-block hooks, the HTML API, and the Interactivity API.
  - The full agent pipeline.
  - A complete worked example.
- Documented agent ecosystem under `opencode/` (agents and commands), self-contained so
  it does not touch global `~/.config/opencode/`.
- `checklist.md` as an implementation checklist an agent can follow to reproduce the
  loop in another system.

### Changed
- Replaced the old custom `.opencode/agent/*.md`-only pattern with the modern
  `AGENTS.md` + skills + opencode agents/commands layered approach.
- Rewrote build/test commands to match real repos (Gutenberg, WooCommerce,
  wordpress-develop, wp-movies-demo, x3p0-ideas, advanced-query-loop, CoBlocks) and private
  plugin/theme projects cross-checked for this manual.
- Updated block patterns for `apiVersion: 3`, `viewScriptModule`, `render.php`, and the
  Interactivity API.
- Generalized advice away from private/hospital repo specifics.

### Removed
- `[NEW]` tags from the table of contents; version control tracks additions.
