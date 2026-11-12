# WoodMart SEO, GEO & Accessibility

Guidance for visibility in Google search, AI assistants (GEO —
Generative Engine Optimization), and WCAG compliance on a WoodMart store,
with the concrete issues and fixes found auditing a live install rather
than generic advice.

## 1. Semantic Structure (The "HTML" Layer)
- One `<h1>` per page (usually the product or page title); `<h2>` for major
  sections, `<h3>` for feature descriptions.
- Every image needs descriptive alt text — WoodMart's defaults are
  generally fine for product images but custom banners/HTML Blocks are
  easy to ship without any.
- Prefer real `<article>`/`<section>`/`<aside>` over generic `<div>` soup
  when hand-building HTML Blocks.

## 2. Structured Data (The "Schema" Layer)
- WoodMart's built-in Product/Offer schema is a reasonable baseline;
  supplement with custom JSON-LD rather than Microdata (JSON-LD is what
  Google and LLM crawlers actually parse reliably) for:
  - **LocalBusiness** on the homepage (and any location/transfer pages),
    sharing one `@id` across all instances so crawlers resolve them as the
    same entity rather than duplicates.
  - **Service + Offers** on service-description pages (e.g. a transfers/
    how-it-works page).
  - **WebPage / AboutPage / ContactPage + BreadcrumbList** on static pages.
- Ship schema additions as a single mu-plugin with the facts centralized in
  one function (e.g. a `*_facts()` helper) rather than scattered inline —
  when a fee, hour, or policy changes, there should be exactly one place to
  edit, and it should match what's printed on the actual page.

## 3. `llms.txt` and LLM Discovery (GEO)
- A rich `/llms.txt` (facts, pricing/requirements, shipping/returns policy,
  a page index, and recent content) plus `<link rel="alternate">` pointing
  to it from every page is a concrete, shippable GEO win — more effective
  than vague "write for LLMs" advice.
- **Check for conflicting generators.** An SEO plugin (e.g. Yoast) may have
  its own `llms.txt` feature that writes a *different*, thinner static file
  the web server serves directly — which silently wins over a custom one if
  both are active. Turn the plugin's version off and confirm the custom
  mu-plugin's output is actually what's being served, not just that it ran.
- **AI crawler policy is a decision to make explicitly, not a default.**
  "Welcome AI answer/search bots, block AI training crawlers" needs two
  independent layers in practice: a CDN/WAF rule keyed on the bot category
  (verified-bot categories typically distinguish "AI Search"/"AI Assistant"
  from "AI Crawler"/training), *and* a `robots.txt` `Disallow` block for the
  named training-bot user agents (GPTBot, ClaudeBot, Google-Extended,
  Bytespider, etc.), because not every crawler respects the CDN layer and
  not every CDN plan distinguishes bot categories finely. Keep both layers
  in sync and don't "fix" one by loosening the other.
- Natural-language product descriptions (write like an expert explaining
  the product to a person) and table-based technical specs are what
  actually help both classic SEO and LLM answer quality — this part of the
  original generic advice holds up.

## 4. Performance and Accessibility Findings

These are concrete issues an audit turned up on a live WoodMart install —
useful as a checklist of what to actually look for, rather than generic
Core Web Vitals advice:

**Performance (Lighthouse):**
- Article/content-template pages were far slower than the homepage: the
  biggest single cause was an oversized header logo image (multi-MB PNG
  shown at a few dozen pixels tall) and `loading="lazy"` on the actual
  above-the-fold hero/content image. Fix: serve a properly sized WebP logo,
  and make the real LCP image eager + high priority + preloaded — lazy
  loading the *first* paint-critical image is a common self-inflicted
  WoodMart mistake.
- Shop/category/product pages are the theme's weak spot: a large number of
  scripts (many unused on a given page) and a sizeable render-blocking
  stylesheet are a known, not-yet-solved cost of the theme's AJAX/builder
  features. Don't expect a quick fix here; it's a real trade-off against
  the theme's feature set.
- If measuring Lighthouse yourself in a headless/WSL environment: a
  **hidden browser window gets throttled to ~1 fps**, which rounds every
  paint timestamp up to the next second and makes simulated LCP look far
  worse than real-user LCP. Pass flags to disable backgrounding/throttling
  (`--disable-backgrounding-occluded-windows`,
  `--disable-renderer-backgrounding`, `--disable-background-timer-throttling`,
  `--run-all-compositor-stages-before-draw`), and still expect run-to-run
  variance of 10+ points — compare several runs, not one.
- A layout-shift source worth checking: anything revealed by JavaScript
  *after* first paint (a "reveal the rest of the page" script, a banner
  that reserves its space only once its own JS runs) causes CLS that a
  reserved-height CSS rule or an early inline script fixes cheaply.

**Accessibility (axe-core):**
- WoodMart's own markup (landmarks, form labels, skip link, focus outlines,
  single `<h1>`) is typically sound; problems tend to be in theme *options*
  and custom content, not the theme's base templates:
  - Product-card price / secondary-text color presets can fail 4.5:1
    contrast against the theme's dark backgrounds — check the actual
    computed colors, don't assume a "dark mode" preset was contrast-checked.
  - Text-body links that rely on color alone (no underline) for
    link-in-text-block.
  - The shop/category manufacturer or sort filter (often a `select2`
    widget) can ship with no accessible name.
  - A decorative SVG map with embedded links and `role="img"` traps
    keyboard/screen-reader users — it needs `role="group"` (or similar) so
    its links are reachable, not `role="img"`.
  - The age-gate overlay (if present) should be checked for keyboard
    operability specifically — it's often built with `<a href="#">`
    buttons that work by accident, not by design.
- An automated axe pass must dismiss any age-gate/consent overlay first —
  these commonly hide the whole page from the scanner, which can make
  "zero violations found" a false negative rather than a clean result.
- After fixing, re-scan every page type (home, shop, category, product,
  cart, my-account, any custom page templates) — fixes for one page's
  components don't always propagate to every instance of that component.
- Checkout, screen-reader (NVDA/VoiceOver), and 200%-zoom passes need a
  manual check; axe-core alone does not cover them.

## 5. Caching Caveat
Schema, `llms.txt`, and accessibility-fix mu-plugins all suffer from the
same trap as any other production change on a cached WoodMart site: see
[best-practices.md §7](best-practices.md) before concluding a fix "didn't
work" — it is very often a cache that hasn't been purged yet.
