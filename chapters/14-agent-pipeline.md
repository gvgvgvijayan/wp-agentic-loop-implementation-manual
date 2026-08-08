# 14 — The Agent Pipeline

This chapter turns the loop into a **pipeline of cooperating agents**. It is the
"many agents that interact and integrate with each other" design you asked for. The
pipeline is a handoff chain — which is exactly the book's core principle: every handoff
is a written document.

## 14.1 The pipeline at a glance

```
Requirement Asker → WP Lingo Translator → Requirement Asker (round 2)
→ Requirement Generator → Task Generator → Tracer Bullet → Issue Creator
→ Implementor → QC → Code Reviewer → PR Reviewer (human)
```

Each stage produces a written artifact the next stage consumes.

## 14.2 The pipeline agents

| Agent | Role | Produces | Permissions |
|---|---|---|---|
| **requirement-asker** | Interviews you via the `question` tool to cover all needed details. | structured interview notes | read, question |
| **wp-lingo-translator** | Converts layman requirements into WordPress terms and reports open questions back to `wp`. | translated requirement notes | read |
| **requirement-generator** | Turns interview notes into a spec. | `docs/specs/<name>.md` | edit (docs) |
| **task-generator** | Turns the spec into a phased plan with TDD steps. | `docs/plans/<name>.md` | edit (docs) |
| **tracer-bullet** | Builds a vertical slice / tracer bullet to validate the approach. | working slice | edit |
| **issue-creator** | Opens a GitHub issue from the spec/plan. | GitHub issue | bash (gh) |
| **implementor** | Implements the plan task by task. | code + tests | edit |
| **qc** | Local review of the working tree before the PR. | review report | edit: deny |
| **code-reviewer** | Reviews the diff in a fresh context before the PR. | review findings | edit: deny |
| **PR-reviewer** | **You** — the human doing the GitHub review. | review comments | — |

## 14.3 The WP Lingo Translator

This agent converts layman requirements into WordPress-based terms, then **reports the
open WordPress-specific questions back to the `wp` primary agent**, which re-invokes
the requirement-asker to interview the user about the specifics.

Example: the user says *"I need an alternative style."*

The translator recognizes this maps to **`register_block_style()`** and reports back to
`wp` with the open question: how the style will be stored — inline, `style_data`, or
other options — per the `register_block_style` documentation. `wp` re-invokes the asker
to interview the user.

This is the key pattern: **the translator does not guess; it asks.** It converts the
language, then hands the open questions back to `wp` so the asker can pin down the
WordPress-specific decision.

The translator runs **before** the requirement-generator: the asker's first round
gathers the layman requirements, the translator maps them to WordPress terms and
surfaces the WordPress-specific decisions, the asker interviews the user a second time
on those decisions, and only then does the generator write the spec.

## 14.4 Supporting agents

### ADR agent

An agent responsible for building **ADRs** (Architecture Decision Records) and other
engineering/architectural design documents. It writes to `docs/adr/` and follows the
ADR format (Context, Decision, Consequences). See the swirl ADR as a model.

### Research agent

Like the built-in `explore` agent, but **self-evolving**: it writes its learnings back
to a memory/skills location so future sessions benefit. This is the "self-evolve" idea.
It is optional but is how the loop improves over time.

### Domain-specific agents

- **WP sub-agents** — each an expert in one area of WordPress. The core set:
  Interactivity API, HTML API, Block Markup Designer, Security.
- **Extended WP sub-agents** (mapped from the official
  [WordPress/agent-skills](https://github.com/WordPress/agent-skills) repo): REST API,
  Block Themes, Plugin Development, Performance, PHPStan, WP-CLI & Ops, Playground,
  Project Triage, Abilities API.
- **Core-subsystem sub-agents** (mapped from the WordPress core codebase,
  `wordpress-develop`): Query, Taxonomy, Media, Cron, Multisite, Rewrite, HTTP,
  Option, Formatting, Customizer, Widgets, Feed, Shortcode, Nav Menu, Locale,
  Sitemaps, Block Bindings, AI Client.
- **Real-world domain experts** — not necessarily WordPress: Hospital Manager, Bank
  Customer, Service Subscriber, or any real-world problem-domain expert.

These are **sub-agents**. The primary `wp` agent delegates to them. When a task spans
multiple domains, the primary agent can have two or more domain experts **discuss** the
problem (an agent swarm).

> **Note on skills:** the `WordPress/agent-skills` repo ships these as *skills* (procedural
> knowledge). In this manual they are modeled as *sub-agents* (roles). You can use both:
> install the skills for deep procedural reference, and delegate to the sub-agents for
> role-based orchestration. See Chapter 03 for the agent-vs-skill distinction.

## 14.5 Agent swarms

A swarm is two or more agents working together. For example, a Hospital Manager and a
Bank Customer domain expert discussing a payment flow. The primary agent orchestrates
the discussion and synthesizes the result.

## 14.6 The primary agent

The `wp` agent is the **primary** orchestrator. It:

- Owns the loop (spec → plan → build → PR → review → fix).
- Delegates to sub-agents (pipeline + domain).
- Decides which domain experts to involve.
- Reports back to you.

## 14.7 Commands that trigger the pipeline

- `/start-loop` — triggers the pipeline from a spec (or starts the requirement-asker).
- `/review` — the **built-in** opencode command; runs the local QC review.
- `/address-review` — feeds GitHub comments back to the fix agent.

## 14.8 Permissions model

- Plan/review/QC agents: `edit: deny`.
- Implement/fix/tracer-bullet agents: `edit: allow`.
- No agent has more permission than its role needs.

## 14.9 Where the files live

The `opencode/` directory in this manual contains the **documented** agent files. They
are self-contained and do not touch your global config. To use them, copy the files you
want into your project's `.opencode/` directory. See Chapter 18 and the `opencode/`
reference files.
