# 05 — wp-env & the Modern Test Stack

Every modern WordPress repo — Gutenberg, WooCommerce, wp-movies-demo, advanced-query-loop,
CoBlocks, and production plugin/theme projects — uses **`wp-env`** for local development
and testing. This
chapter is the practical guide the agent needs to run tests correctly.

## 5.1 What `wp-env` is

`wp-env` is the official local WordPress environment, Docker-based. It gives you a
real WordPress install with your plugin/theme mounted in, plus a CLI container for
running `wp` commands and PHPUnit.

```bash
npm run wp-env status   # Always check status first.
npm run wp-env start    # Only start if not already running.
npm run wp-env stop
```

## 5.2 `.wp-env.json`

The environment is configured in `.wp-env.json`. A realistic example for a plugin
that depends on another plugin and a set of mu-plugins:

```json
{
  "core": null,
  "phpVersion": "8.3",
  "plugins": [ ".", "../my-dependency-plugin" ],
  "mappings": {
    "wp-content/plugins/my-plugin": ".",
    "wp-content/plugins/my-dependency-plugin": "../my-dependency-plugin",
    "wp-content/mu-plugins": "../../mu-plugins"
  },
  "config": {
    "WP_DEBUG": true,
    "WP_DEBUG_LOG": true,
    "WP_DEBUG_DISPLAY": true,
    "SCRIPT_DEBUG": true
  }
}
```

Key fields:

- `core` — WordPress core version (`null` = latest).
- `phpVersion` — the PHP version to use.
- `plugins` / `themes` — what to mount.
- `mappings` — explicit path mappings (useful for mu-plugins and cross-plugin deps).
- `config` — `wp-config.php` constants.

## 5.3 The `env:setup` pattern (critical)

A bare `wp-env start` is often not enough. Real plugins with custom post types,
rewrite rules, or seed data need a setup script (commonly `bin/setup-test-env.sh`
or `tests/bin/env-setup.sh`) that:

1. Activates the required theme.
2. **Grants custom post-type or capability changes** to the relevant role.
3. Sets pretty permalinks and flushes rewrite rules.
4. **Creates linked mock data** that tests depend on.
5. Flushes caches/transients.

This is wired as `npm run env:setup` and documented as **CRITICAL** before running
tests. The agent must run it, or tests fail with confusing symptoms.

### The gotchas (from projects that use env:setup)

| Symptom | Cause / fix |
|---|---|
| **403 Forbidden** on REST requests | Permissions/capabilities not applied. Re-run `env:setup`. |
| **Empty dropdowns / missing related data** | Seed meta or terms missing, or cache stale. Re-run `env:setup`. |
| **500 errors** | PHP fatal in `render.php` or a plugin file. Check container logs: `npx wp-env run cli tail -n 50 /var/www/html/wp-content/debug.log`. |
| **Playwright strict-mode failures** | Hidden steps are still in the DOM. Use specific locators, e.g. `getByRole('heading', { name: '...' })`. |
| **`window is not defined`** | Node version too new. Pin to the LTS in `.nvmrc` / `engines`. |

## 5.4 PHPUnit — in the container, never on the host

The single most important rule: **run PHPUnit inside the `wp-env` container, never
directly on the host.** The host does not have WordPress loaded.

The exact npm script names differ by repo (check `AGENTS.md` and `package.json`), but
all follow the same two-step shape:

1. Start `wp-env` (once per session).
2. Run PHPUnit *inside* the test container.

A common wrapper:

```bash
npm run test:unit:php:setup   # e.g. `wp-env start`
npm run test:unit:php:base     # e.g. `wp-env run tests-cli --env-cwd=wp-content/plugins/my-plugin vendor/bin/phpunit`
```

Equivalent examples from real repos:

- Gutenberg: `npm run test:php`
- WooCommerce plugin: `pnpm run test:php:env -- --filter YourTestClass`
- wordpress-develop: `npm run test:php`

> **Invariant:** PHPUnit runs in a container that has WordPress and the test database
> loaded. Never run `vendor/bin/phpunit` directly on the host.

`phpunit.xml.dist` bootstraps the WordPress test environment:

```xml
<phpunit
    bootstrap="tests/phpunit/bootstrap.php"
    backupGlobals="false"
    colors="true"
    convertErrorsToExceptions="true"
    convertNoticesToExceptions="true"
    convertWarningsToExceptions="true">
    <testsuites>
        <testsuite name="default">
            <directory suffix=".php">tests/phpunit/tests</directory>
        </testsuite>
    </testsuites>
    <filter>
        <whitelist processUncoveredFilesFromWhitelist="true">
            <directory suffix=".php">./includes/</directory>
        </whitelist>
    </filter>
</phpunit>
```

## 5.5 Playwright (E2E)

Playwright runs through `wp-scripts test-playwright`. The config extends the
`@wordpress/scripts` defaults:

```ts
import { defineConfig, devices } from '@playwright/test';
const baseConfig = require( '@wordpress/scripts/config/playwright.config' );

export default defineConfig( {
    ...baseConfig,
    testDir: './tests/e2e/specs',
    use: { ...baseConfig.use, ignoreHTTPSErrors: true },
    projects: [ { name: 'chromium', use: { ...devices[ 'Desktop Chrome' ] } } ],
    webServer: undefined, // testing against an existing site
} );
```

Commands (use the script names defined in the repo's `package.json`; these are
representative):

```bash
npm run test:e2e          # run all specs (wp-scripts test-e2e)
npm run test:e2e:ui       # Playwright UI mode
npm run test:e2e:debug    # Playwright inspector
```

`wp-scripts test-e2e` is the canonical alias; `wp-scripts test-playwright` is an alias
that still works in most setups. Failed tests capture snapshots into `artifacts/`
(override with `WP_ARTIFACTS_PATH`).

## 5.6 Node version pinning

`@wordpress/scripts` requires a supported LTS Node. Pin it:

- `.nvmrc` with the version, and/or
- `engines` in `package.json`, and/or
- `wp-scripts check-engines`.

## 5.7 The agent's test checklist

Before claiming a task is done, the agent must run, in order:

1. `npm run wp-env status` (or start).
2. `npm run env:setup` (if the repo has one).
3. `npm run lint:js` and `npm run lint:css` (or the repo's lint scripts).
4. `composer run lint` (PHPCS) — or `npm run lint:php` in Gutenberg.
5. The repo's PHPUnit script, **inside the wp-env container**.
6. The repo's E2E script (Playwright, usually `test-e2e`) for user-facing flows.

Only when all pass is the task "done."
