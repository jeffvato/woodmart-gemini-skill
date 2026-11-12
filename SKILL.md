---
name: woodmart
description: Expert development, design, SEO/GEO, performance and accessibility work on the WoodMart WooCommerce theme. Use when customizing a live WoodMart site over WP-CLI/SSH, building headers/mega-menus/theme-builder layouts, diagnosing performance or accessibility issues, or implementing technical SEO and LLM-discovery (GEO) features.
---

# WoodMart Development & Design Skill

Guidance for working with the WoodMart WordPress/WooCommerce theme, drawn from
hands-on work maintaining a live WoodMart store over SSH/WP-CLI rather than
from the theme's marketing docs. WoodMart has several independent storage
mechanisms and admin-only builders (Header Builder, Mega Menu, Theme Builder),
so guessing from Elementor/generic-WordPress experience leads to wrong
answers — verify per feature.

## 1. Core Principles
- **Don't assume a child theme is in play.** Many WoodMart sites are run
  without ever touching `woodmart-child` — day-to-day customizations live in
  **mu-plugins** instead (see §2), because they survive theme updates, need
  no theme-editor access, and can be dropped in or removed over SSH without
  a build step. Check `wp-content/mu-plugins/` and `wp-content/themes/woodmart-child/`
  (if present) before assuming where a behavior comes from.
- **Verify storage before you build.** WoodMart spreads settings across at
  least four unrelated mechanisms (options, postmeta, widgets, a CPT) — see
  §3. An empty query on the wrong one ("no posts of this type exist") does
  not mean the feature hasn't been configured.
- **Everything is live production.** There is normally no staging copy.
  Back up before any WP-CLI write or option change, verify key pages after
  (home/shop/category/product/cart/checkout/my-account return 200, no PHP
  fatals in the error log), and purge caches (§5) before declaring a fix done.

## 2. Working Over WP-CLI / SSH
Typical access pattern when the theme runs in a Docker container on a VPS:
```
sudo docker exec -u www-data -w /var/www/html php wp --no-color <command>
```
- `wp --no-color core verify-checksums` "fails" harmlessly if `readme.html`
  was deliberately removed for fingerprint-hiding — don't treat that as a
  real integrity problem without checking first.
- `wp theme update woodmart` updates the premium parent theme by hand
  (it's not on WordPress.org, so core auto-update doesn't cover it); back up
  the theme + child theme + `woodmart-core` plugin directories first and
  confirm the plugin's required-core-version is still satisfied after.
- Prefer a small, named mu-plugin per behavior (`btg-<feature>.php` style)
  over editing theme files — it's self-documenting, independently
  deletable, and immune to theme updates. Keep a backup copy of anything you
  replace or delete before you delete it.
- A `curl` from the server itself to a public page often returns 403 from
  Cloudflare/WAF unless you send a real browser `User-Agent`; don't take
  that as evidence the page is broken.

## 3. Storage Mechanisms — Verify Per Builder, Don't Assume Elementor
WoodMart uses several different, unrelated storage mechanisms. Confirm which
one applies to the feature you're touching rather than assuming any single
one covers it — see [Technical Standards & Best Practices](references/best-practices.md)
for the full breakdown (Header Builder as `wp_options`, Mega Menu as
`nav_menu_item` postmeta, Footer as plain widget areas, Theme Settings as one
big serialized option `xts-woodmart-options`, and the `woodmart_layout` CPT
used only by the Shop/Product/Checkout/My-Account/Popup Theme Builder).

## 4. Workflows

### Layout & Header Building
1. **Header Builder:** `wp-admin/admin.php?page=xts_header_builder` — a React
   SPA; see [best-practices.md](references/best-practices.md) before
   automating it with Playwright/CDP.
2. **HTML Blocks:** reusable content (footers, mega menus, banners) under
   **HTML Blocks > All Items**.
3. **WooCommerce Layouts:** Shop/Product/Cart/Checkout templates via the
   Theme Builder (`woodmart_layout` CPT) — no deep coding required.

### Theme Styling
1. Typography, colors and button styles live in **Theme Settings**, stored
   in the single `xts-woodmart-options` serialized array — never
   `json_decode()` it.
2. Use WoodMart's CSS variables (`--wd-primary-color`, `--wd-brd-radius`,
   `--wd-entities-title-color`, …) for consistency with the rest of the theme.
3. A setting's *compiled* CSS lands in `xts-<preset>-css-data` — useful to
   grep to confirm a change took effect, but it's a cache, never edit it directly.

### E-commerce Performance & Accessibility
1. AJAX filters/search/add-to-cart are built in; don't reinvent them.
2. Audit real pages with axe-core and Lighthouse rather than trusting the
   theme's "optimized" reputation — see [SEO & LLM Readiness](references/seo-llm-readiness.md)
   for concrete issues found on a live install (contrast, lazy-loaded hero
   images, select2 accessible names, SVG maps) and how they were fixed.

### SEO & LLM Discovery (GEO)
See [SEO & LLM Readiness Strategy](references/seo-llm-readiness.md) for
schema, `llms.txt`, and generative-engine-optimization patterns that were
actually shipped, plus the caching gotchas that make "I changed it but the
site didn't change" a non-issue to investigate before assuming the fix failed.

## 5. Caching & Deployment Gotchas
- Query-string cache-busting (`?x=1`) often doesn't work if query strings
  are 301-redirected away at the edge — test the origin directly instead.
- A CDN/edge cache (e.g. Cloudflare) and a server-side page cache (e.g. an
  nginx purge queue file) are usually independent — clearing one does not
  clear the other. Purge-everything calls can take several minutes to
  propagate; don't conclude a fix failed from an early recheck.
- Changing certain security/perf mu-plugins or Theme Settings "builder"
  state can require a WP cron tick or nginx reload to take visible effect —
  watch the actual rendered output/response headers, not just the saved
  admin state.

## 6. When to Trigger This Skill
- Designing or modifying WordPress pages/layouts using the WoodMart theme
  (Header Builder, Mega Menu, Theme Builder templates).
- Diagnosing or fixing WoodMart-specific performance, accessibility, or SEO/GEO issues.
- Automating the Header Builder or other React-SPA admin screens.
- Working with WoodMart's theme-settings storage (`xts-woodmart-options`) or
  compiled-CSS cache directly, instead of through the UI.
- Troubleshooting AJAX filters, mu-plugin conflicts, or cache-purge behavior
  on a live WoodMart/WooCommerce store.
