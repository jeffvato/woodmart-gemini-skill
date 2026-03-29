# WoodMart Technical Standards & Best Practices

## 1. Child Theme Architecture
- **Mandatory:** All PHP and CSS customizations must reside in `woodmart-child`.
- **Functions:** Use `functions.php` for enqueuing assets and adding custom hooks.
- **Overrides:** To override a parent template, copy it to the same relative path in the child theme (e.g., `woodmart-child/woocommerce/single-product.php`).

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
