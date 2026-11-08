# Recipes

Worked examples of whole jobs, as the sequence of calls. Adapt the words, keep the order.

## 1. A new brochure site for a small business (one language)

Brief: "Acme Roofing - home, services, about, contact. Orange, friendly, trustworthy. Family firm in Leeds."

1. `get_site` → languages: `en` (default, ltr). Scopes include `design`, `content`, `data`.
2. Inventory: `list_designs`, `list_theme_layouts`, `list_sections`, `list_pages`, `list_menus`. Empty site: nothing to reuse.
3. **Plan, in chat, before building** - get a yes:
   - Pages: Home, Services (children: Flat roofs, Pitched roofs, Repairs), About, Contact.
   - Home: `sk-hero-split`, `sk-services-grid`, `sk-proof-stats`, `sk-testimonials`, `sk-cta-band`.
   - Services: `sk-page-header`, `sk-services-detail`, `sk-cta-band`. About: `sk-page-header`, `sk-story`, `sk-team`, `sk-cta-band`.
   - Contact: `sk-page-header`, `sk-contact-form`. Table `contact`. Menu `main` (header) and `footer`.
4. Foundation: `create_design {name: "Main (en)", language: "en"}`, then `create_theme_layout` (canvas with `[gl-logo]`,
   `[gl-menu-main]`, `[SYS-PAGE-CONTENT]`, footer with `[gl-menu-footer]`; css with the spacing scale and brand variables).
   Then `set_site_frame` with the same header and footer.
5. Sections, one `create_section` each with **scoped** CSS. Put parameters and defaults on reusable elements; have each section pass
   the values it needs to its `[el-…]` calls. Create the shared elements and sections first (`sk-page-header`, `sk-cta-band`) because
   several pages use them.
6. Images: `upload_media` for the logo-free hero photo and team photos (the user's), `set_media_alt` for each.
7. Pages: `create_page` for Home, About, Contact, Services, then its children with `parentId`. Each with `html`
   (a list of `[sk-…]`; values vary inside `[el-…]` calls), `desktopThemeId`, and `seo`.
   Then `update_page {id: home, isHome: true}`.
8. `create_collection {name: "Contact", slug: "contact", fields: […]}`; the `sk-contact-form` section's `<form data-collection="contact">`.
9. Menus: `create_menu` main + footer; `set_menu_items` with ids from `list_pages`.
10. Verify: `render_page` for **every** page. Fix `unresolvedShortcodes`, missing images, text that overflows.
11. Report: pages and addresses (all drafts), placeholder copy you wrote (mark exactly which), questions (real phone, hours, service
    area, guarantee wording). **Ask** to publish.
12. After a yes: `publish_page` for each page (home last, so the menu never points at a draft).

Using a blueprint instead of steps 4-9: build one document (see `blueprint`), `preview_blueprint`, show the user, `apply_blueprint`,
then do steps 10-12.

## 2. Add a page to an existing site, in its style

1. `list_pages`, then `get_page` of a similar page: copy its section pattern and the `desktopThemeId` it uses.
2. `list_sections`: reuse (`[sk-page-header title="Pricing"]`, `[sk-cta-band]`). Only if no section fits: `get_section` of the closest one,
   and `create_section` a sibling with a new code, copying its markup conventions and class prefix.
3. `create_page` (draft) → `render_page` → report → ask → `publish_page` → offer to add it to the menu
   (`get_menu`, then `set_menu_items` with **the existing entries plus the new one**).

## 3. Redesign an existing site

1. Read before touching: `get_theme_layout`, `list_sections` + `get_section` for the main ones, `get_page` for the home page.
   Say what the current structure is.
2. Work **additively**: create new sections (`sk-hero-v2`) and a new theme layout (`Main v2`) beside the old ones.
3. Create the new pages as drafts (or a draft copy of the home page using the new pieces under a temporary slug `home-v2`).
4. `render_page` and show the user. Only on approval switch the live pages: `update_page` their `html` / `desktopThemeId`,
   one at a time, rendering after each. Keep the old pieces until the user says delete (`archive_element` for elements).
5. Never edit a shared section in place to redesign it while the site is live, unless the user asked for the change to go live
   everywhere at once.

## 4. A bilingual site (English + Persian)

1. `get_site` must list both languages (you cannot add one; if `fa` is missing, tell the user to add it in Settings > Languages).
2. A design, a theme layout, a menu and pages **per language**. The Persian theme mirrors the header; CSS uses logical properties.
3. The structure can be shared, but translated words should be passed to reusable element parameters. Create separate `sk-…`
   sections when two languages or pages need materially different values or layout (image side, ornament).
4. Same slugs for pages that correspond (`about` in both) so the language switcher can link them; Persian copy written natively.
5. `[gl-lang-switcher]` in each theme header. `render_page` each page in each language (`locale`).
6. Persian blog: its own categories and tags with Persian names; slugs are lower-case Latin transliterations or short English words.

## 5. A blog with a few starter posts

1. `create_category` (3-5 broad), `create_tag` (the specific ones you will really use).
2. `create_post` ×N as drafts, each with category, 2-4 tags, seo title/description, one real image with alt text.
3. Add a blog entry to the main menu (`CUSTOM`, `/blog`) and a `[gl-blog-list limit="3"]` strip to the home page (check its parameters with
   `list_global_shortcodes {filter: "blog"}`).
4. Make the blog's header and footer match the theme: `set_site_frame`.
5. Read each post back (`get_post`), then ask before `publish_post`.

## 6. A contact form that works

1. `create_collection` with `name`, `email`, `message` (+ `phone`, `topic` if wanted), required flags set.
2. A section `sk-contact-form` whose markup is the form in `content-and-data` (labels, `data-collection`, `data-form-message`),
   styled with `:focus-visible` and error states using `form[data-state="error"]`.
3. A page that uses it, plus a line saying what happens with the data (ask the user for their privacy policy wording).
4. Tell the user: entries appear in the panel (Data); notifications by email or webhook are theirs to set there.

## 7. Fixing what `render_page` shows

| Symptom | Likely cause | Fix |
|---|---|---|
| `[sk-something]` visible in the page | the section code is wrong or missing | `list_sections`; correct the page's `html` or create the section |
| a section appears but unstyled | its CSS targets a class the markup lacks, or relies on a utility class the site's stylesheet does not ship | write plain scoped CSS in the section |
| the whole site's headings changed | a section's CSS has an unscoped selector | prefix every selector with the section's class |
| the header is missing on a page | no `desktopThemeId` on the page | `update_page` with a theme layout |
| the blog looks different from the pages | the site frame is the platform default | `set_site_frame` with the theme's header/footer |
| a parameter shows empty | it is used as `{{key}}` in an element but not declared, or has no default | `update_element` with the declaration |
| a menu is empty | `[gl-menu-<slug>]` slug differs from the menu's, or the menu has no items | `list_menus`; fix the slug or `set_menu_items` |
| text runs off the screen on a phone | fixed widths, or no `min-width: 0` on a grid/flex child | fluid CSS (`minmax`, `clamp`, `flex-wrap`) |
