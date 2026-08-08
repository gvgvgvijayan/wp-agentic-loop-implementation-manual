# 12 — The WordPress Playbook

This chapter maps the generic loop onto modern WordPress realities, verified against the
reference repos.

## 12.1 The Gutenberg monorepo

Gutenberg is a monorepo: `packages/` (JS/TS blocks and packages), `lib/` (PHP),
`phpunit/` (PHP tests), `test/` (JS tests). When the agent works here:

- Point it at the exact package path.
- Use the repo's own lint/test scripts (`npm run lint:js`, `npm run lint:php`,
  `npm run test:unit`, `npm run test:php`).
- Be aware of the build step (`npm run build`) before testing.
- Respect the architectural decisions in `AGENTS.md` (package layering, data layer,
  styles system, etc.).

## 12.2 Modern plugin/theme architecture

The hospital plugins and themes demonstrate the modern patterns:

### PSR-4 autoloading

```json
{
  "autoload": {
    "psr-4": {
      "VG\\Hospital_Appointment_Booking\\": "includes/"
    }
  }
}
```

### PHP-Scoper dependency prefixing

WordPress has no central dependency manager. If two plugins bundle the same library at
different versions, the second plugin's class redeclaration causes a fatal error.
PHP-Scoper renames every namespace in a third-party library to your own prefix
(`VG\Hospital_Appointment_Payment\ThirdParty\Stripe` instead of `Stripe`), giving you a
fully isolated copy.

- Configured in `scoper.inc.php`.
- Output goes to `third-party/`.
- `composer prefix-deps` runs `php-scoper add-prefix --force`.
- `composer build` runs `prefix-deps` then `composer install --no-dev --optimize-autoloader`.
- **Your source code must always import the prefixed namespace**, never the original.

### `block.json` + `render.php` (plugin/theme convention)

For plugins and themes, modern blocks register via `block.json` and render server-side
via a `render` field pointing to `render.php`. The build copies PHP with
`--webpack-copy-php`. This is the pattern used in wp-movies-demo and advanced-query-loop.

> In Gutenberg core, dynamic blocks still live in `packages/block-library/src/<block>/index.php`
> and use `register_block_type_from_metadata(..., ['render_callback' => '...'])`.
> Do not copy the `render.php` convention into a Gutenberg core patch unless the repo
> has moved to it.

### Interactivity API & Data Views

- **Interactivity API** for frontend interactivity (script modules, `viewScriptModule`).
  - Modern blocks use `apiVersion: 3` in `block.json`.
  - Use `wp_interactivity_state()` for global state, `wp_interactivity_data_wp_context()`
    for local context, and `supports.interactivity: true` so directives are processed on
    the server.
  - `data-wp-ignore` is deprecated in WordPress 6.9.
- **Data Views** for admin list tables (filtering, searching, bulk actions).

### `theme.json`

Block themes configure global styles, settings, and templates in `theme.json`. Modern
themes use `theme.json` version 3 with newer top-level keys such as
`settings.background.backgroundImage`, `settings.typography.fluid`, and per-element
styles. x3p0-ideas, for example, uses `theme.json` v3 and builds with
`wp-scripts build --webpack-src-dir=resources --output-path=public --experimental-modules`.

## 12.3 Common WordPress tasks and their verification

| Task | Where | Verify with |
|---|---|---|
| Register a block | `block.json` (`apiVersion: 3`) + `index.js` | `npm run build`, manual render |
| PHP function/class | `lib/` or plugin `includes/` | `composer run lint`, containerized PHPUnit |
| Block editor JS | `packages/block-editor/` or `src/` | `npm run test:unit`, `npm run lint:js` |
| REST API endpoint | `lib/` or plugin | containerized PHPUnit, manual `curl` |
| Theme (block theme) | `theme.json` (version 3), templates | `npm run build`, visual check |
| Extend a core block | `render_block` filter | containerized PHPUnit, manual render |

## 12.4 WordPress conventions the agent must respect

- **i18n:** use `__()`, `_e()`, `esc_html__()` — never hardcoded strings.
- **Escaping:** escape output (`esc_html`, `esc_attr`, `wp_kses`).
- **Nonces:** for any form/action.
- **Coding standards:** WordPress Coding Standards (PHPCS), ESLint for JS.
- **Accessibility:** keyboard nav, ARIA, focus management.
- **Back-compat:** don't break older WP versions unless the spec says so.

## 12.6 PHPStan baselines as migration tools

PHPStan is increasingly common in WordPress projects. Many repos use baseline files to
avoid being overwhelmed by legacy issues:

- **wordpress-develop** ships ~80 baseline files under `tests/phpstan/baselines/` and a
  `phpstan.neon.dist` at level 5.
- **WooCommerce** uses a single `phpstan-baseline.neon`.

Treat the baseline as a **migration boundary**, not an excuse. New code should not add
to the baseline. If an existing baseline error blocks your change, fix the underlying
issue or add a narrowly-scoped inline exception with a comment. The goal is to shrink
the baseline over time.

## 12.5 Where the code-graph tools help in WordPress

- **graphify / codegraph:** when the agent needs to find a function signature or callers
  across the huge Gutenberg tree, these return the answer in far fewer tokens than grep.
  Use them during the build/fix stages.
- They are *accelerators*, not the loop. The loop works without them.
