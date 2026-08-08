# 04 — The Toolchain: wp-scripts & wp-build

This chapter covers the modern WordPress build toolchain. It is the foundation for
everything in the loop that touches code — because the agent must run the *right*
commands, and the commands changed.

## 4.1 `@wordpress/scripts` (`wp-scripts`)

`@wordpress/scripts` is a collection of reusable scripts tailored for WordPress
development. You install one npm module and get integrated, recommended configuration
for build, lint, format, and test. It is the standard for plugins and themes.

Install:

```bash
npm install @wordpress/scripts --save-dev
```

It requires a Node.js version with LTS status. Pin it (`.nvmrc` or `engines`).

### The core commands

| Command | What it does |
|---|---|
| `wp-scripts build` | Production build (webpack). Exits after one build. |
| `wp-scripts start` | Development build with watch. |
| `wp-scripts format` | Prettier formatting (JS, JSON, TS, YAML). |
| `wp-scripts lint-js` | ESLint (WordPress recommended rules). |
| `wp-scripts lint-style` | Stylelint (WordPress CSS standards). |
| `wp-scripts lint-pkg-json` | Lint `package.json`. |
| `wp-scripts lint-md-docs` | Markdownlint for docs. |
| `wp-scripts test-unit-js` | Jest unit tests (alias `test-unit-jest`). |
| `wp-scripts test-e2e` | Playwright E2E tests (alias `test-playwright`). |
| `wp-scripts plugin-zip` | Create a distributable plugin zip. |
| `wp-scripts build-blocks-manifest` | Generate a PHP block-metadata manifest. |
| `wp-scripts check-engines` | Check node/npm versions. |
| `wp-scripts check-licenses` | Validate dependency licenses. |
| `wp-scripts packages-update` | Update `@wordpress/*` packages. |

### Flags that matter for modern blocks

- `--webpack-copy-php` — copies PHP files from `src` to the output directory. By
  default only the PHP files listed in the `render` and `variations` fields of detected
  `block.json` files are copied. Modern plugin/theme projects pass this always.
- `--experimental-modules` — enables the `viewScriptModule` field (Interactivity API
  script modules). Modern plugin/theme projects pass this always.
- `--source-path` / `--output-path` — customize source and output directories. For
  example, x3p0-ideas uses `--webpack-src-dir=resources --output-path=public`.
- `--blocks-manifest` — generate a PHP file with block metadata from all `block.json`
  files (for `wp_register_block_metadata_collection()`).

### Automatic `block.json` detection

`wp-scripts build` scans `src/` (and subdirectories) for `block.json` files and treats
the JS files listed in their metadata as entry points. This means one command builds
all your blocks:

```json
{
  "editorScript": "file:index.js",
  "script": "file:script.js",
  "viewScript": "file:view.js"
}
```

The fallback entry point is `src/index.js` if no `block.json` is found.

### A realistic `package.json` (plugin/theme example)

```json
{
  "scripts": {
    "build": "wp-scripts build --webpack-copy-php --experimental-modules",
    "start": "wp-scripts start --webpack-copy-php --experimental-modules",
    "format": "wp-scripts format",
    "lint:js": "wp-scripts lint-js",
    "lint:css": "wp-scripts lint-style",
    "plugin-zip": "wp-scripts plugin-zip",
    "test:unit:php": "npm run test:unit:php:setup && npm run test:unit:php:base",
    "test:unit:php:base": "wp-env run tests-cli --env-cwd=wp-content/plugins/my-plugin vendor/bin/phpunit",
    "test:unit:php:setup": "wp-env start",
    "test:e2e": "wp-scripts test-playwright --config playwright.config.ts"
  }
}
```

## 4.2 The new `wp-build` (`@wordpress/build`)

`wp-build` is the CLI binary of the new **`@wordpress/build`** package. It is the
"next generation" of WordPress plugin build tooling. Key facts (verified against the
official docs and the Gutenberg repo):

- **Engine:** replaces the webpack + Babel pipeline with **esbuild** (a Go-based
  bundler). Much faster — Gutenberg builds its 100+ packages in seconds instead of
  minutes.
- **Convention over configuration:** instead of a webpack config, it uses fixed folder
  conventions — `packages/`, `routes/`, `blocks/` — and reads configuration from
  `package.json` fields (`wpPlugin`, `wpScript`, `wpScriptModuleExports`, etc.).
- **Auto-generates PHP registration:** it produces `build/build.php` (plus
  `scripts.php`, `modules.php`, `styles.php`) that you load with
  `require_once plugin_dir_path( __FILE__ ) . 'build/build.php';`. No hand-written
  registration.
- **Commands:** `wp-build` (production) and `wp-build --watch` (incremental dev).

### Status and guidance

- **Experimental / pre-1.0.** The API is described as "malleable" and "still being
  shaped." The package uses `0.x` versioning (e.g. v0.19.0).
- **Not ready for every use case.** In particular, a plugin registering blocks still
  has gaps that require manual workarounds.
- **Convergence roadmap:** the long-term plan is for `@wordpress/build` to become the
  engine *underneath* `@wordpress/scripts`, so `wp-scripts build` keeps working but
  uses the faster engine internally. Eventually webpack and Babel are deprecated from
  `@wordpress/scripts`.

> **Bottom line for the loop:** for most plugin/theme developers today, **stay on
> `@wordpress/scripts`**. Watch `wp-build` as it matures. The agent should know both
> exist and which one the repo uses.

### Sources

- `@wordpress/scripts` docs: developer.wordpress.org/block-editor/reference-guides/packages/packages-scripts/
- `@wordpress/build` docs: developer.wordpress.org/block-editor/reference-guides/packages/packages-wp-build/
- Announcement: developer.wordpress.org/news/2026/04/wordpress-build-the-next-generation-of-wordpress-plugin-build-tooling/
- Vision issue: github.com/WordPress/gutenberg/issues/72032
