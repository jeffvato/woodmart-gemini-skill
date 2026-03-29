# WoodMart SEO & LLM Readiness Strategy

To achieve top-tier visibility in Google search results and AI discovery (LLMs), follow these guidelines within the WoodMart environment.

## 1. Semantic Structure (The "HTML" Layer)
- **HTML5 Tags:** Use WoodMart's layout builder to ensure proper use of `<article>`, `<section>`, and `<aside>`.
- **Heading Hierarchy:** One `<h1>` per page (usually the product or page title). Use `<h2>` for major sections and `<h3>` for feature descriptions.
- **Alt Text:** Every image MUST have descriptive alt text that explains the image's content and its relevance to the page.

## 2. Structured Data (The "Schema" Layer)
- **JSON-LD Schema:** Ensure WoodMart's default Schema.org markup is active. Supplement it with custom JSON-LD for:
  - **Product & Offer:** Including SKU, price, availability, and brand.
  - **FAQ:** For high-traffic product pages or manufacturer landings.
  - **Organization:** For the brand's main homepage.
- **Microdata:** Avoid mixing Microdata with JSON-LD; JSON-LD is preferred by Google and LLMs.

## 3. LLM Discovery & Recommendation
- **Natural Language Descriptions:** Write product descriptions that sound like an expert explaining the product to a human. This helps LLMs understand context and intent.
- **Detailed Specifications:** Use table-based specs for technical details (Caliber, Action, Finish) as these are easily parsed by AI models.
- **E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness):**
  - Use WoodMart's blog and custom page templates to build detailed "About" and "Expert Guides."
  - Highlight real user reviews and expert endorsements.

## 4. Performance Metrics (Core Web Vitals)
- **LCP (Largest Contentful Paint):** Optimize the hero image or main product image.
- **CLS (Cumulative Layout Shift):** Avoid late-loading elements (like banners or ads) that shift content.
- **FID (First Input Delay):** Minimize main-thread JS execution.
