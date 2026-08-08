# 17 — Glossary

- **Agent** — a configured runtime instance with a role, prompt, and permissions.
- **AGENTS.md** — the repo-level agent contract: how to work in this codebase. opencode
  reads it natively.
- **apiVersion** — the `block.json` API version. Modern blocks use `apiVersion: 3`.
- **Command** — a reusable prompt you invoke to trigger a workflow.
- **Hook** — code that runs on events; used to automate parts of the loop.
- **Skill** — procedural, reusable guidance ("how to do this kind of work").
- **Spec** — the macro-requirements document (the "what and why").
- **Plan** — the agent's task breakdown (the "how"), approved by the human.
- **PR** — the unit of review; the agent opens it, you review it.
- **Handoff** — any written document that passes state between stages.
- **Definition of done** — the checklist that must be true for a task to be complete.
- **Accelerator** — an optional tool (code-graph, memory) that makes the loop faster or
  smarter.
- **wp-env** — the official Docker-based local WordPress environment.
- **wp-scripts** — the CLI of `@wordpress/scripts` (build, lint, test).
- **wp-build** — the CLI of the new `@wordpress/build` package (esbuild-based,
  convention-driven; the planned future engine under `wp-scripts`).
- **PHP-Scoper** — a tool that prefixes third-party library namespaces to isolate them
  per plugin.
- **PSR-4** — the PHP autoloading standard used for modern plugin/theme namespaces.
- **Interactivity API (IAPI)** — the WordPress API for frontend interactivity via
  script modules and `data-wp-*` directives.
- **viewScriptModule** — the `block.json` field that enqueues an Interactivity API
  script module on the front end.
- **HTML API** — `WP_HTML_Tag_Processor` and related classes for safely reading and
  modifying HTML.
- **render_block filter** — a filter that modifies a block's rendered HTML server-side.
- **register_block_style** — the API to add a style variant to an existing block.
- **style_data** — block-style data attached via `register_block_style` (WP 6.6+).
- **TDD** — test-driven development: write failing tests first, then implement.
- **Red/Green/Refactor** — the TDD cycle: failing test → passing test → clean up.
- **Checkpoint** — a commit at a phase boundary, with a verification report attached.
- **git notes** — metadata attached to commits; used to store verification reports.
- **Autonomy knob** — the choice of `middleman` vs `end-of-loop` mode when triggering
  the loop.
- **QC** — a local review agent that runs the built-in `/review` before the PR.
- **PR Reviewer** — the human doing the GitHub review (the final quality gate).
- **Agent swarm** — two or more agents working together on a task.
- **ADR** — Architecture Decision Record: a document capturing a design decision.
- **data-wp-key** — a stable directive key used for client-side navigation stability.
- **client-side navigation** — Interactivity API feature where only the active region
  is replaced; stores and keys must survive across navigations.
- **Abilities API** — the WordPress API for capability-based permissions and REST
  authentication (`wp_register_ability`, `/wp-json/wp-abilities/v1/*`).
- **WP-CLI** — the command-line interface for WordPress operations (`wp`).
- **WordPress Playground** — a browser-based WordPress environment for previews,
  snapshots, and Blueprint-driven setup.
- **PHPStan** — a static analysis tool for PHP, used to catch type and logic errors.
- **PHPStan baseline** — a list of existing errors that are temporarily allowed while
  new code is held to a higher standard.
- **Project Triage** — deterministic inspection of a WordPress repo to detect type,
  tooling, and versions.
- **WP_Query** — the main WordPress query class for retrieving posts.
- **Taxonomy** — a way to group posts and custom post types (categories, tags, custom).
- **Multisite** — a WordPress installation that runs multiple sites from one codebase.
- **Rewrite rules** — the permalink-to-query mapping system (`class-wp-rewrite`).
- **Shortcode** — a WordPress macro that expands to output via `add_shortcode`.
- **Block Bindings** — the API for binding block attributes to data sources.
