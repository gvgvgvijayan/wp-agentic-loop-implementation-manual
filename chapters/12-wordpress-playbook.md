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

### `block.json` + `render.php`

Modern blocks register via `block.json` and render server-side via a `render.php`
callback (or a `render` field pointing to a PHP file). The build copies PHP with
`--webpack-copy-php`.

### Interactivity API & Data Views

- **Interactivity API** for frontend interactivity (script modules, `viewScriptModule`).
- **Data Views** for admin list tables (filtering, searching, bulk actions).

### `theme.json`

Block themes configure global styles, settings, and templates in `theme.json`. The
hospital themes use `wp-scripts build --webpack-copy-php --experimental-modules
--webpack-src-dir=assets --output-path=public`.

## 12.3 Common WordPress tasks and their verification

| Task | Where | Verify with |
|---|---|---|
| Register a block | `block.json` + `index.js` | `npm run build`, manual render |
| PHP function/class | `lib/` or plugin `includes/` | `composer run lint`, `npm run test:unit:php` |
| Block editor JS | `packages/block-editor/` or `src/` | `npm run test:unit`, `npm run lint:js` |
| REST API endpoint | `lib/` or plugin | `npm run test:unit:php`, manual `curl` |
| Theme (block theme) | `theme.json`, templates | `npm run build`, visual check |
| Extend a core block | `render_block` filter | `npm run test:unit:php`, manual render |

## 12.4 WordPress conventions the agent must respect

- **i18n:** use `__()`, `_e()`, `esc_html__()` — never hardcoded strings.
- **Escaping:** escape output (`esc_html`, `esc_attr`, `wp_kses`).
- **Nonces:** for any form/action.
- **Coding standards:** WordPress Coding Standards (PHPCS), ESLint for JS.
- **Accessibility:** keyboard nav, ARIA, focus management.
- **Back-compat:** don't break older WP versions unless the spec says so.

## 12.5 Where the code-graph tools help in WordPress

- **graphify / codegraph:** when the agent needs to find a function signature or callers
  across the huge Gutenberg tree, these return the answer in far fewer tokens than grep.
  Use them during the build/fix stages.
- They are *accelerators*, not the loop. The loop works without them.
