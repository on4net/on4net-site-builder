---
name: on4net-site-builder
description: Build, redesign and edit a website on an ON4NET installation through the ON4NET MCP server - designs, theme layouts, elements, sections, pages, menus, media, blog posts with categories and tags, and data tables for forms. Use whenever the user asks to create or change a site, a page, a theme, a section, a blog or a contact form on ON4NET, or hands you an ON4NET agent key.
version: 2.5.0
---

# Building a site on ON4NET

You are connected to **one site** through an ON4NET MCP server, with a key its owner issued. Everything you can do is a tool
named below; everything you cannot do is refused with a `403`. This guide teaches the platform's structure and the order to build
in, so the site you produce is one the owner can keep editing in the panel - not a pile of one-off markup.

**This guide belongs to no domain.** ON4NET runs many sites, each on its own address - `example.com`, `shop.example.com`, any
domain or subdomain - and the connector may carry any name the owner gave it (it is the server that offers `get_site`,
`list_pages`, `upload_media` and the rest; use whichever it is called). Never assume an address, a language or a brand from this
file or from the connector's name: **the site is whatever `get_site` answers**, and the key you were given opens that site and no
other. If the owner has several sites, each has its own connector and its own key; work only in the one they point you at, and
never carry a theme, a code or a piece of content from one site into another unasked. Write links as paths (`/about`), not with a
domain, so the same markup survives a change of address; the primary address `get_site` lists is only for telling the owner where
to look.

Read this whole file once. Open a reference when you reach its subject:

| Topic (`get_guide`) | Read it when |
|---|---|
| `structure` | you need to know exactly how designs, themes, pages, sections and elements relate, per language |
| `shortcodes` | you write markup: the `[sk-…]` / `[el-…]` / `[gl-…]` syntax, parameters, loops |
| `builtins` | you need a built-in `gl-…` code: what each one does and its parameters (headings, buttons, forms, menus, icons, blog list, charts…) |
| `theme-and-style` | you build the theme layout, colours, dark mode, responsive and right-to-left behaviour |
| `content-and-data` | blog, categories, tags, menus, media, forms and data tables, SEO |
| `blueprint` | you build a whole site at once with one document |
| `google` | the owner asks about Analytics, Search Console, search keywords, visitor numbers or Google reviews: what is connected, what you may set, and what is public |
| `recipes` | you want a worked example: a business site, a bilingual site, a blog, a contact form, a redesign |

## 1. Start every session the same way

1. `get_site` - which site, its own verified **domain** (when it has one) and what is set up with Google, its **languages** (default first, with direction), and the key's **scopes** and permissions.
   If the user's request needs a permission the key lacks, say so now instead of failing later.
2. **Take inventory before you build.** `list_designs`, `list_theme_layouts`, `list_sections`, `list_elements`, `list_pages`,
   `list_menus`, and `list_global_shortcodes` (the built-ins you get for free). An empty site is the exception: most sites already
   have a theme, a few sections and pages, and the job is to extend them in their own style. Read one existing theme layout and one
   section (`get_theme_layout`, `get_section`) to learn the site's markup conventions and copy them.
3. Say back what you found and what you intend to build, in a few lines, and get a yes for anything large.

### Companion skills (check at session start, use when the task matches)

This guide teaches ON4NET's mechanics. Design and search work is delegated to companion skills. At the start of a session,
see which of them are available in your skill list, and **use them whenever the task falls in their area**:

| Area | Skill | Use it for |
|---|---|---|
| Design | `on4net-design-director` (bundled in `design-director/`) | brand discovery, visual direction, industry presets, animation, before building any theme or section |
| Design | `ui-ux-pro-max` | palettes, font pairings, UX and accessibility guidelines, style choice |
| SEO | `claude-seo` (`/seo …`) | technical SEO, content/E-E-A-T, schema, sitemap, hreflang, local SEO, page and site audits |
| GEO | `geo-seo-claude` (`/geo …`) | AI-search visibility, citability, AI-crawler access, `llms.txt`, brand mentions, GEO score |

Rules:

- **Design first.** For a new site, a redesign, a theme layout or any new section, apply `on4net-design-director` and
  `ui-ux-pro-max` before writing markup, then build it through ON4NET's sections, elements and theme layouts as this guide says.
- **SEO and GEO after content exists.** Use `claude-seo` for the SEO side and `geo-seo-claude` for the AI-search side. Where
  they overlap (schema, technical, content), run both and reconcile; do not duplicate findings.
- **Their advice is input, not authority.** Anything they suggest still has to be expressed with ON4NET tools and obey section 4
  (no third-party scripts, fonts or embeds without the owner's approval; fixes land in page SEO fields, sections, elements,
  `set_google_tags`, menus or media, never in files you cannot reach).
- **If a companion is missing, do not skip it silently.** Tell the user which one is missing and give the install command from
  `README.md`. Never run an install script, `curl | bash` or a marketplace add without the user's explicit yes. Until they decide,
  continue with this guide alone and say which part of the work had no companion support.
- Companion audits take a public URL. Use the primary address `get_site` lists, and only after the pages are published.

## 2. The model in one minute

```
Site
 └─ Language (en, fa, …)                       every page, post, design and layout belongs to ONE language
     └─ Design                                 the container of that language's pages (the tree of addresses)
         └─ Page  /about, /services/web        a DRAFT until published
             content = [sk-hero][sk-features][sk-cta]        ← a list of SECTION codes, plus a little markup
             theme   = a THEME LAYOUT per viewport           ← the header + footer + the slot the page goes in

Section  [sk-code]   one band of a page (hero, features, pricing, testimonials, CTA…). html + css + js.
Element  [el-code]   a small reusable piece (button, card, badge, testimonial, price box…) used inside sections, on pages, in
                     posts and in theme layouts. Same anatomy. ANYTHING THAT MAY BE USED IN MORE THAN ONE PLACE IS AN ELEMENT.
Built-in [gl-…]      pieces that exist on every site with no creating: logo, menus, language switcher, blog list,
                     FAQ list and ask-a-question form, the owner’s YouTube videos and playlists and Instagram posts, the site’s
                     location as a map, data tables, charts, icons, shapes, headings, buttons.
Parameters           {{title}} in the markup  ⇄  [sk-hero title="Hello"] on the page. Every parameter has a default.
Menu                 a small tree of links, shown with [gl-menu-<slug>].
Media                files in the library; a page points at their url.
Blog                 posts (HTML) + categories + tags, per language. Drafts until published.
Data table           a named list of fields; a `<form data-collection="slug">` fills it; owners read the entries.
FAQ                  questions the owner answered in the panel, drawn by [gl-faq]; [gl-faq-ask] is the visitor's form.
Social feeds         a YouTube channel and/or an Instagram account the owner connected in the panel and synced; drawn by
                     [gl-youtube] (videos, filter by playlist), [gl-youtube-playlists] and [gl-instagram]. Ask get_social_status first.
Panel page           a page with audience: "dashboard" - signed-in customers read it at /dashboard/<slug>; visitors never.
```

Four consequences you must keep in mind:

- **A page is mostly a list of section codes.** Do not paste a whole page of HTML into a page. Make each band a **section**, then
  `create_page` with `html: "[sk-hero title=\"…\"][sk-features][sk-cta]"`. That is what lets the owner re-order, edit and reuse
  bands in the panel.
- **Codes are the links between things.** A page that mentions `[sk-pricing]` breaks if no section has that code. Create the
  pieces a page uses **before** the page, and `render_page` afterwards to prove nothing is left unresolved.
- **Languages are separate.** A Persian page and its English twin are two pages in two designs. Create each language's design,
  theme layout, menu and pages; do not assume one serves both. Right-to-left languages need `dir="rtl"` thinking (see
  `theme-and-style`).
- **Two kinds of page.** `audience: "dashboard"` puts a page in the signed-in customer panel (`/dashboard/...`) instead of the
  site: visitors cannot reach it. A menu entry for one needs the `/dashboard/...` address written out, because a `PAGE` entry
  resolves to the public address - give the entry a `url`.

## 3. The build order

Follow it unless the site already has the earlier layers. Each step lists its tools.

| # | Step | Tools |
|---|---|---|
| 0 | **Plan.** Pages and their sections on paper first: the list of pages, the section list per page, the elements those need, the menu, the languages. Show it to the user. | - |
| 1 | **Foundation: the theme layout.** One canvas with header, the `[SYS-PAGE-CONTENT]` slot and footer; the logo, menu and language switcher as built-ins; the site's CSS. Then give the **site frame** (blog, search, contact) the same header and footer. | `create_theme_layout` / `update_theme_layout`, `set_site_frame` |
| 2 | **Elements** the sections will share (button, card, heading style). Skip if the built-ins (`gl-el-button`, `gl-el-heading`) are enough. | `create_element` |
| 3 | **Sections**, one per band. A reusable element supplies parameters and defaults; each section makes element calls with its own values. A piece used elsewhere is an element (step 2), not a section. | `create_section` |
| 4 | **Media.** Upload images first, set alt text, use the returned urls; a picture far larger than it is drawn gets `resize_media` and the copy's url. | `upload_media`, `create_upload_link`, `resize_media`, `set_media_alt` |
| 5 | **Menus.** Create the menu, then fill it with ids of pages that exist (so create pages first, then `set_menu_items`). | `create_menu`, `set_menu_items` |
| 6 | **Pages** as drafts, each with its content, theme, address and SEO. A second set with `audience: "dashboard"` is the customer panel. | `create_page` |
| 7 | **Blog**: categories and tags first, then posts as drafts. | `create_category`, `create_tag`, `create_post` |
| 8 | **Forms and data.** A table with its fields, then the form markup in a section. | `create_collection` |
| 9 | **Verify** every page. Look for unresolved codes, empty images, broken links. | `render_page` |
| 10 | **Report** what you built, with each page's address, and **ask** before publishing. | - |
| 11 | **Publish** only after a yes. | `publish_page`, `publish_post` |

Whole-site shortcut: for a new site or a large rebuild, build ONE blueprint document (see `blueprint`), `preview_blueprint`,
show the user the diff, then `apply_blueprint`. Use the granular tools for everything after that.

## 4. Rules that are not optional

1. **Never publish on your own.** New pages and posts are drafts for a reason. Show the rendered result and get a clear yes
   before `publish_page` / `publish_post`. Never unpublish or delete a published page without asking.
2. **Read before you change.** `get_*` the thing, edit what you read, send the change. `update_theme_layout`, `update_section` and
   `update_element` change every page that uses them. Never rewrite from memory.
3. **Reuse before creating.** Check the lists first. A second `sk-hero` beside an existing one is clutter; extend or add a
   parameter instead. **Section or element?** A section is one band of one page layout. The moment a piece may be used somewhere
   else - in another section, on another page, in a post, in a theme layout, or twice on the same page (a button, a card, a badge,
   a testimonial, a price box, a call-to-action strip) - it must be an **element** (`create_element`) and the section uses
   `[el-…]`. If you are about to copy a section's markup, or to paste the same markup into a second place, stop and extract it into an
   element first.

   **A page is made of sections.** A section is the page's own composition: one band that combines several elements
   (`[el-…]`, `[gl-…]` built-ins) with plain HTML around them - a heading, a grid, a wrapper, some text. That is what you place on
   a page, and most of a page is sections. An element is placed on a page by itself **only** when that one element does the whole
   job (a single button, a lone banner); otherwise wrap it in a section. So: build elements for the reusable pieces, then compose
   them into sections, then put the sections on pages.
4. **Parameters belong to elements, not sections.** Create a reusable element with parameters and default values, then pass its
   different values from the section, for example `[el-product-card title="Monthly Plan" price="$20"]`. Do not declare parameters
   on the section itself or use `{{key}}` in section markup for repeated page-specific values. If a section needs materially
   different values, create another section and use the appropriate element calls inside it instead of changing the element's
   shared definition.
5. **Use the built-ins; never invent a code.** `list_global_shortcodes` is the list of `gl-…` codes that exist. An unknown code
   renders as literal text on the live page.
6. **Every element parameter has a default**, so `[el-code]` still renders. Passing values from a section or page overrides the
   element default; do not hard-code copy that belongs to one page inside a shared element.
7. **Unique, readable codes.** Lower case, hyphens: `sk-hero-split`, `el-cta-button`. Codes are per site.
8. **Scope your CSS.** Prefix every rule with a class unique to the piece (`.sk-hero-split …`). Unscoped CSS in one section
   restyles the whole site.
9. **No secrets, no tracking you were not asked for.** Do not put API keys, tokens or third-party scripts into markup. Do not copy
   visitor data (form entries) anywhere.
9. **Stay inside your scopes.** A `403` means the key does not cover the act. Do not retry, do not look for a way around it:
   tell the user which scope is missing.
10. **Be honest about what you could not verify.** If a render shows something you did not intend, say so; do not call it done.

## 5. Quality bar (every page, every section)

- **Responsive.** Mobile first; test the markup at phone width mentally: no fixed widths, images `max-width:100%`, tap targets
  of at least 44px, grids that collapse to one column. Write layout in plain CSS inside the piece's own `css` (flex, grid,
  media queries): the platform ships a **fixed** stylesheet, so a utility class only works if that sheet already contains it.
  Copy classes from the site's existing theme and sections, or check them in a `render_page` of an existing page.
- **Accessible.** One `<h1>` per page (the page title); headings in order; real `<button>`/`<a>` elements; every image has alt
  text (`set_media_alt`, or `alt=""` for decoration); text contrast of at least 4.5:1; visible focus; form inputs have `<label>`.
- **Fast.** Optimise images (`optimize: true` on upload), no huge inline SVG or base64, no render-blocking external fonts or
  scripts; keep `js` minimal and inside the piece.
- **SEO.** Each page: a unique `seo.title` (50-60 chars) and `seo.description` (150-160 chars), one `<h1>`, descriptive link text,
  a slug a person can read. **Link wherever you can**: every page and every blog post links to the pages it relates to, with a real
  `<a href>` (or `[gl-el-link]`) and words that say what the target is, as a path (`/pricing`), after checking the target exists. A page
  nothing links to is a page search engines will not value. `indexable: false` only for pages that must stay out of search.
- **Consistent.** Same spacing scale, same colours (use the brand variables, see `theme-and-style`), same button styles across
  sections. A site is a system, not a set of pages.
- **Copy.** Write in the page's language, in the voice the site already has. No lorem ipsum in anything you hand over; if you do not
  have the copy, say what is placeholder and where.

## 6. Tool map

**Orient:** `get_site`, `get_guide`, `list_global_shortcodes`
**Google:** `get_google_status` (the site's own verified `domain`, and whether the Analytics tag, the Search Console tag and the Business Profile are set) · `set_google_tags` (the GA4 `G-…` id and the Search Console verification value; refused until the site has its own verified domain, and needs the Google scope) · `setup_search_console` (verification, property and sitemap in one call) · `search_console_queries` (the keywords, pages, countries or devices the site is found by) · `setup_analytics` (finds or creates the GA4 property and stream and sets the tag) · `analytics_report` (users, sessions and views by page, source, country, device or day) · `business_reviews` (the Business Profile's newest reviews) · `reply_to_review` and `create_business_post` (both **public**: show the owner the text and get their agreement first). All the Google tools need a key with the **Google** scope

**Social feeds:** `get_social_status` (is YouTube / Instagram connected and synced, how many items, and the YouTube `playlists` to filter by; call it before using `[gl-youtube]`, `[gl-youtube-playlists]` or `[gl-instagram]`. Connecting and syncing need the owner in a browser, so tell them - you cannot do it)
**Structure:** `list_designs` `get_design` `create_design` `update_design` `delete_design` · `set_site_frame` ·
`list_theme_layouts` `get_theme_layout` `create_theme_layout` `update_theme_layout` `delete_theme_layout` `audit_theme_colors` · `list_icons` · `list_fonts` (the hosted web fonts: use these, not Google Fonts; see `theme-and-style`)
**Pieces:** `list_elements` `get_element` `create_element` `update_element` `archive_element` `delete_element` ·
`list_sections` `get_section` `create_section` `update_section` `delete_section` `reorder_sections`
**Pages:** `list_pages` `get_page` `render_page` `create_page` `update_page` `publish_page` `delete_page`
**Navigation:** `list_menus` `get_menu` `create_menu` `update_menu` `set_menu_items` `delete_menu`
**Media:** `list_media` `upload_media` `create_upload_link` `resize_media` `set_media_alt`
**Blog:** `list_posts` `get_post` `create_post` `update_post` `publish_post` · `list_categories` `create_category` `update_category` · `list_tags` `create_tag` `update_tag`
**Data:** `list_collections` `get_collection` `create_collection` `update_collection`
**Whole site:** `preview_blueprint` `apply_blueprint`

What you **cannot** do, on purpose: change settings, the team, billing or languages; add a language; send a table's records to a
webhook or an address; read the records visitors sent through a form; run the platform's own AI agents; touch any other
site. If the user asks for one of these, tell them where in the panel to do it.

## 7. When something goes wrong

| You see | It means | Do |
|---|---|---|
| `403` / "may not" | the key's scopes do not cover it | stop; tell the user the scope (`get_site` shows what the key holds) |
| `401` | key wrong, revoked or expired | stop; ask for a new key (Settings > AI agents) |
| `422` with `fields` | the platform refused a value | read the field messages, fix exactly those, retry once |
| `404` | wrong id, or it belongs to another language/site | re-list and use the id you get |
| `429` | too many calls | wait; batch with a blueprint or fewer calls |
| a code shown as text in `render_page` (`unresolvedShortcodes`) | the section/element/built-in does not exist | create it or correct the code |
| a section looks unstyled | its CSS is unscoped or the theme framework differs | scope it; check the theme's `framework` |

## 8. Definition of done

- Every page you made renders with **no unresolved shortcodes**.
- Every image has alt text; every page has an SEO title and description; the menu links to pages that exist.
- Nothing is published that the user did not approve.
- You have told the user: what you created (names, codes, addresses), what is a draft, what is placeholder copy, and the one or two
  decisions you would like them to make next.
