# wordpress-woodmart-skill

An agent skill for working on WordPress sites built with the **WoodMart**
WooCommerce theme — development, design, performance, accessibility, and
technical SEO/GEO (generative-engine optimization, i.e. visibility to AI
assistants). It's written for an AI coding agent (Gemini CLI extension or a
Claude Code / Claude Agent SDK skill) to load as reference material, not for
humans to follow step by step.

The content comes from hands-on maintenance of a live WoodMart/WooCommerce
store over SSH and WP-CLI — real storage locations, real caching gotchas,
and real bugs found and fixed — rather than from the theme's marketing
copy. WoodMart spreads its settings across several unrelated storage
mechanisms (`wp_options`, postmeta, widgets, a custom post type) and ships
admin screens that are React single-page apps, so generic
"WordPress/Elementor" knowledge is often wrong for it; this skill exists to
avoid relearning those lessons from scratch each time.

## Contents

- **[SKILL.md](SKILL.md)** — the skill definition: when to trigger it, and
  a condensed workflow for Header Builder/Mega Menu/Theme Builder layouts,
  WP-CLI access patterns, and where to look for deeper detail.
- **[references/best-practices.md](references/best-practices.md)** —
  mu-plugin vs. child-theme customization, WoodMart's CSS variable system,
  the four different builder storage mechanisms and how to tell them apart,
  browser-automating the Header Builder's React UI with Playwright/CDP, and
  caching/deployment gotchas on a cached, CDN-fronted site.
- **[references/seo-llm-readiness.md](references/seo-llm-readiness.md)** —
  structured data, `llms.txt` and AI-crawler policy, and concrete
  performance/accessibility findings (contrast, lazy-loaded hero images,
  Lighthouse-in-headless-browser pitfalls, select2 accessible names, SVG
  maps) from auditing a live install.

## Using it

**As a Gemini CLI extension** (`gemini-extension.json` is the manifest):
```
gemini extensions install https://github.com/jeffvato/wordpress-woodmart-skill
```

**As a Claude Code / Claude Agent SDK skill:** copy this repository into
your skills directory (e.g. `~/.claude/skills/wordpress-woodmart-skill/`) so
`SKILL.md` is discovered, or reference its path directly if your setup loads
skills from an arbitrary location.

## Scope

This skill assumes a self-hosted WoodMart install administered over
SSH/WP-CLI, sitting behind a reverse-proxy page cache and/or CDN — the
common setup for a production WooCommerce store. Pure wp-admin-only
workflows with no server access will still find the builder/storage and
SEO/GEO sections useful; the WP-CLI and caching-layer sections assume
server access.

## License

[MIT](LICENSE)
