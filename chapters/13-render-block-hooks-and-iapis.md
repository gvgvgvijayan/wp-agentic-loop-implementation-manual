# 13 — Render-Block Hooks, the HTML API & the Interactivity API

This chapter is distilled from the **swirl** case study in the vijayan-child theme
(`docs/issues/swirl/`). It teaches the canonical way to extend **core blocks** with
interactivity — a common, high-value WordPress task that the original draft did not
cover.

## 13.1 The problem: core blocks have frozen `save()` output

Core blocks have frozen `save()` output. You cannot type `data-wp-*` directives into
their markup, because the saved content is fixed. The only way to add interactivity to
core markup is **server-side directive injection** — a `render_block` filter that stamps
directives onto the rendered HTML.

## 13.2 The canonical pattern: `render_block` filter + `WP_HTML_Tag_Processor`

The `core/image` lightbox is the canonical example
(`gutenberg/packages/block-library/src/image/index.php`). It:

1. Instantiates a `WP_HTML_Tag_Processor` on the rendered block HTML.
2. Adds a class and sets `data-wp-*` attributes:

```php
$processor->add_class( 'wp-lightbox-container' );
$processor->set_attribute( 'data-wp-interactive', 'core/image' );
$processor->set_attribute( 'data-wp-context', wp_json_encode( [ 'imageId' => $unique_image_id ], JSON_HEX_TAG|JSON_HEX_APOS|JSON_HEX_QUOT|JSON_HEX_AMP ) );
$processor->set_attribute( 'data-wp-on--click', 'actions.showLightbox' );
```

3. Renders a **singleton overlay** into `wp_footer` with
   `data-wp-router-region='{ "attachTo": "body" }'` so it survives client-side
   navigation.

## 13.3 The swirl pattern (from the case study)

The swirl is an enhancement of `core/group`:

- `register_block_style( 'core/group', 'swirl' )` adds `is-style-swirl` to the wrapper
  for CSS targeting.
- A `render_block_core/group` filter stamps `data-wp-*` directives with
  `WP_HTML_Tag_Processor`.
- **Guard on `$block['attrs']['className']` before instantiating a processor** — only
  process the block when the style is present. This avoids touching every `core/group`
  on the page.

```php
add_filter( 'render_block_core/group', function ( $block_content, $block ) {
    if ( empty( $block['attrs']['className'] ) || false === strpos( $block['attrs']['className'], 'is-style-swirl' ) ) {
        return $block_content;
    }
    $processor = new WP_HTML_Tag_Processor( $block_content );
    // ... stamp directives ...
    return $processor->get_updated_html();
}, 10, 2 );
```

## 13.4 Key Interactivity API functions

- **`wp_interactivity_state( 'swirl', $data )`** — serialize server data for the
  client store.
- **`wp_interactivity_process_directives()`** — pre-resolve bindings server-side (e.g.
  `data-wp-bind--hidden` → `hidden`) to prevent a flash of all-details-visible on first
  paint.
- **`add_client_navigation_support_to_script_module( 'swirl/view' )`** — register a
  custom script module so its store survives client-side navigation (WP 6.9+). Without
  this, navigating away and back drops the store and the directives become inert.
- **Stable `data-wp-key`** — for client-side nav stability, use a stable key
  (`data-wp-key="swirl-card-{N}"`), not `uniqid()`. Core's lightbox uses `uniqid()`,
  but that breaks client-side nav.
- **Static set → no `data-wp-each`** — for a fixed set of items, use a PHP `foreach`
  with per-item `data-wp-context`, not `data-wp-each` (which is for dynamic lists).

## 13.5 Two-mechanism DOM mutation

The image lightbox shows two mechanisms:

- **`WP_HTML_Tag_Processor`** — for class and attribute mutation (`add_class`,
  `set_attribute`).
- **`preg_replace`** — for element interpolation (inserting sibling elements the HTML
  API can't easily add). Anchor the regex to a **stable, specific tag/attribute**, not
  a fragile "last `</div>`" lookahead.

## 13.6 A lesson in grounding agent claims

The swirl `review-analysis.md` documents a real failure mode: a plan cited a
"terminal-`</div>` lookahead regex" as the image-lightbox mechanism. **That pattern
appears nowhere in core.** What core actually does:

- **Image lightbox:** `preg_replace( '/<img[^>]+>/', $button, $body_content )` — targets
  the `<img>` **opening** tag.
- **Gallery:** `preg_replace_callback` over `'/<figure[^>]*\bwp-block-image\b[^>]*>.*?<\/figure>/s'`.
- **Media-text:** anchors a regex on a **unique `id`** it stamps first.

**The lesson:** when an agent claims a "canonical pattern," verify it against the real
core source before building on it. This is exactly why the loop has a QC/review gate —
and why `AGENTS.md` should point agents at the canonical reference files.

## 13.7 When to use this vs a custom block

- **Extend a core block** (frozen `save()`, no re-saving content): use a `render_block`
  filter + `WP_HTML_Tag_Processor`. This is the swirl approach.
- **New block with its own `save()`/attributes**: use a custom block with `block.json`
  + `render.php`.

The swirl ADR chose the filter approach because a custom block would require re-saving
all content and a custom `save()` path, risking SEO and layout shift.
