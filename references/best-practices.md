# WoodMart Technical Standards & Best Practices

## 1. Customization Architecture: mu-plugins vs. Child Theme
- **In practice, a child theme is often not used at all** on a WoodMart site
  managed over SSH/WP-CLI. Site-specific logic (GEO/`llms.txt`, accessibility
  fixes, checkout-flow tweaks, fee-calculation overrides, cache purging,
  fingerprint hiding, etc.) is far more commonly shipped as small, individually
  named **mu-plugins** in `wp-content/mu-plugins/` (e.g. `btg-geo.php`,
  `btg-a11y.php`, `btg-gateway-fee-off.php`). Advantages over a child theme
  for this kind of work: no build step, survives theme/plugin updates
  untouched, can be dropped in or deleted over SSH in one command, and each
  file documents exactly one behavior.
- **If a child theme does exist**, all PHP/CSS customizations still belong
  there, not in the parent `woodmart` theme directory. Use `functions.php`
  for enqueuing assets and hooks; override a parent template by copying it to
  the same relative path (e.g. `woodmart-child/woocommerce/single-product.php`).
- **Either way:** back up the file you are about to replace or delete before
  touching it (a timestamped `.bak-<reason>-<date>` copy is enough), and note
  in the mu-plugin's own comment or your own log what it does and how to
  undo it — on a live store, "how do I turn this back off" matters as much
  as "how do I turn it on."
- **WP-CLI access**, when the theme runs in a Docker container on a VPS:
  `sudo docker exec -u www-data -w /var/www/html php wp --no-color <command>`.
  Expect harmless `Deprecated` notices on newer PHP; filter them rather than
  treating them as failures. `wp theme update woodmart` updates the premium
  parent theme by hand (it isn't on WordPress.org, so core/plugin auto-update
  doesn't cover it) — back up theme + child theme + the `woodmart-core`
  plugin first, and re-check the plugin's required-core-version afterward.

## 2. CSS & Design System
- **Variables:** Leverage WoodMart's CSS variable system for consistency:
  - `--wd-primary-color`: Main brand color.
  - `--wd-entities-title-color`: Color for product/post titles.
  - `--wd-brd-radius`: Global border radius.
- **Theme Settings:** Prefer **Theme Settings > Custom CSS** for small tweaks, but use `style.css` for complex structural changes.

## 3. Performance & Asset Loading
- **Conditional Enqueuing:** Use `woodmart_enqueue_inline_style()` for theme-specific styles to ensure they load only when needed.
- **Lazy Loading:** Ensure WoodMart's native lazy loading is active under **Theme Settings > Performance**.
- **Selectors:** Avoid broad selectors; target WoodMart-specific classes like `.wd-product`, `.whb-header`, etc.

## 4. Hook Patterns
- **WooCommerce:** Standard hooks work, but watch for WoodMart overrides.
- **WoodMart Specifics:**
  - `woodmart_before_shop_page`
  - `woodmart_after_shop_page`
  - `woodmart_product_action_buttons`

## 5. Builder storage mechanisms — verify per-builder, don't assume Elementor

Confirmed against a live Woodmart 8.5.7 install (2026-08-25/26) — WoodMart
uses **at least four different, unrelated storage mechanisms** across its
builders. Do not assume any single one applies to a feature you haven't
checked, and do not assume "no posts of type X exist" means "feature X
hasn't been built" without confirming X is the right post type first — that
exact mistake cost several planning iterations here.

**Path correction:** the modules referenced below live in the **parent
`woodmart` theme**, not the `woodmart-core` plugin —
`wp-content/themes/woodmart/inc/modules/<feature>/`. The `woodmart-core`
plugin's own `inc/` only holds `class-auth.php`, `widgets.php`,
`shortcodes.php`, `meta-boxes.php` — no `modules/` directory there at all.
An earlier version of this note had the wrong plugin; grepping the wrong
path silently returns nothing rather than erroring, so this is easy to get
wrong without noticing.

- **Header Builder** persists as `wp_options`, not posts:
  `whb_<id>` (one option per saved header, holding its full row/element
  structure), indexed by `whb_saved_headers`, with `whb_main_header`
  pointing at the currently-active one. Source:
  `wp-content/themes/woodmart/inc/modules/header-builder/
  class-manager.php` and `class-headers-list.php`. No `wp_insert_post` or
  `WP_Post` anywhere in that module. The Header Builder admin screen is at
  `wp-admin/admin.php?page=xts_header_builder` — editing a specific header
  opens `#/builder/<header_id>` (e.g. `#/builder/default_header`).
- **Mega Menu** attaches to standard WordPress nav menus: per-item mega-menu
  config lives in `_menu_item_block` postmeta on `nav_menu_item` posts.
  Source: `inc/modules/mega-menu-walker/class-mega-menu-walker.php`. If
  `wp menu list` returns zero menus, there is no mega menu yet — building
  one means creating a nav menu from scratch, not editing an existing post.
- **Footer** has no dedicated builder at all — it's plain WordPress widget
  areas. Theme Settings > Footer > Footer content lets you pick a column
  layout (1-5 columns); saving that registers standard sidebars
  `footer-1` through `footer-N` (`wp sidebar list` to confirm), populated
  with ordinary core "block" widgets (`wp:html` content, etc.) via
  `wp widget add` or by writing directly to the `widget_block` /
  `sidebars_widgets` options. No XTemos-specific mechanism, no
  `woodmart_layout` post.
- **General Theme Settings** (colors, typography, layout, and per-tab
  things like footer background/text-color scheme) live in ONE big option,
  `xts-woodmart-options` — a PHP-serialized array (NOT a JSON string;
  `get_option('xts-woodmart-options')` returns a native PHP array
  directly — a `json_decode()` on it will fatal). Find a specific setting
  with `wp eval 'foreach(get_option("xts-woodmart-options",array()) as
  $k=>$v){if(stripos($k,"footer")!==false)echo $k."=".$v.PHP_EOL;}'`, and
  update one key while preserving the rest with
  `update_option('xts-woodmart-options', array_merge(get_option(...),
  array('key'=>'value')))`. Related but separate: `xts-<preset>-css-data`
  (e.g. `xts-default_header-css-data`, `xts-theme_settings_default-css-data`)
  holds the *compiled* CSS output — useful to grep to confirm a Theme
  Settings change actually took effect (e.g. `.wd-footer {
  background-color: rgb(23,26,24); }`), but it's a cache, not the source
  of truth — don't edit it directly.
- **`woodmart_layout`** (a real, registered CPT) is unrelated to all of the
  above — it's used by the Shop/Product/Checkout/My-Account/Popup
  **Theme Builder** templates (`inc/modules/layouts/class-*.php`). An empty
  `woodmart_layout` post count tells you nothing about whether a header,
  mega menu, or footer has been built.

**Method that actually works:** grep the parent theme's PHP source on the
live site for `wp_insert_post` / `update_option` / `get_option` inside the
specific builder's module directory (`wp-content/themes/woodmart/inc/
modules/<feature>/` — NOT `wp-content/plugins/woodmart-core/`), rather than
inferring storage from WP-CLI post-type/term counts alone. Cross-check by
building one throwaway item through the actual wp-admin UI and diffing
`wp option list` / `wp post list` / `wp menu list` / `wp sidebar list`
before and after.

## 6. Browser-automating the Header Builder UI (Playwright/CDP)

The Header Builder is a React SPA — plain CSS-selector clicks are
unreliable. What actually works, learned the hard way over many failed
attempts:

- **Row-level edit buttons (`button.whb-edit-btn`, the larger ~30×40px
  ones) are hidden by CSS `:hover` reveal**, not just visually collapsed.
  A single `element.click()` sometimes fails silently because the browser
  never actually hovered first. Do a real two-step hover — `mouse.move()`
  to a neutral point on the row, wait ~400ms, `mouse.move()` to the button,
  wait ~400ms, then `mouse.down()` + `mouse.up()` — rather than one
  `.click()` call.
- **Never guess pixel coordinates from a screenshot for a second action on
  the same page.** Layout shifts slightly after any save/modal-close.
  Instead, query the DOM directly (`document.querySelectorAll` +
  `getBoundingClientRect()`) for the element by its distinguishing class
  or nearby label text, every time, and click those live coordinates.
- **Modal `Save` buttons collide with the page's own `Save header` button**
  when using `text=Save` — `.xts-popup-holder.xts-opened >> text=Save`
  scopes correctly. But **don't wait on
  `.xts-popup-holder.xts-opened` for visibility** — multiple elements can
  carry that class (an open modal AND a hidden tooltip container), and
  Playwright's `wait_for_selector` can end up waiting on the wrong one
  forever even though the real modal is visibly open. Wait on unique
  visible text instead (e.g. `text=Configure Top bar settings`).
- **Color pickers**: the Header Builder's own row/element style pickers use
  a custom hex/RGBA widget — find the hex input by locating the text node
  whose content is the (lowercase-rendered) label near it, then reading
  its parent's `<input>` `getBoundingClientRect()`, not by guessing which
  of several nearby inputs holds the hex value (R/G/B/A/Hex are 5 separate
  inputs in a tight cluster). **Theme Settings' color fields are a
  different, standard WordPress `wp-color-picker` (Iris)** —
  `button.wp-color-result` opens it, and the real text input is
  `input.wp-color-picker` (NOT the swatch button itself, and NOT the
  nearby "Upload image" button, which sits close enough to misfire a
  coordinate-based click into an unrelated media-library modal).

## 7. Caching & Verifying a Change Actually Shipped

On a WoodMart store sitting behind a server-side page cache and a CDN, "I
changed it but it still looks old" is usually a caching problem, not a
reverted change:

- **Page cache and CDN cache are independent.** A reverse-proxy page cache
  (e.g. an nginx cache with its own purge queue) and an edge CDN (e.g.
  Cloudflare) each need their own purge step; clearing one does not clear
  the other.
- **Query-string cache-busting (`?x=1`) can silently not work** if the site
  301-redirects away query strings before the cache sees them. Verify by
  curling the origin directly (bypassing the CDN, e.g. with a `--resolve`
  override to the origin IP) rather than trusting a browser reload.
- **`purge_everything`-style CDN purges and page-cache purge queues are not
  instant** — allow roughly a minute or more before re-checking, and purge
  the specific path(s) rather than only the bare domain when the CDN purge
  API is scoped by URL (purging a prefix does not always purge the home
  page itself).
- **Compiled-CSS/JS caches** (see `xts-<preset>-css-data` in §4) and the
  page/CDN cache are three separate layers — a Theme Settings change can be
  saved, compiled, and still not visible until the page cache and CDN are
  both purged.
- After any production change, re-verify the actual rendered response
  (status code, and for visual/behavioral changes, the real markup or a
  screenshot) rather than relying on the admin screen reporting success.
