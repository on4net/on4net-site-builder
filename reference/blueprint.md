# The blueprint: a whole site as one document

Use a blueprint to **build** (a new site, a new section of a site, a rebuild). Use the granular tools to **edit**.

```
preview_blueprint {blueprint}   → what would be created and what is left alone; nothing is written
apply_blueprint   {blueprint}   → writes it (after the user has seen the preview)
```

## What applying does - and does not do

- It **adds**. A language, design, theme layout, page, section, element, menu or table that already exists (matched by code,
  slug, name or language) is **skipped and reported, never overwritten**. Nothing is deleted. So a second apply of the same document
  creates nothing new - and that means you cannot use a blueprint to *change* an existing piece: use `update_*` for that.
- It is **not one transaction**. The whole document is validated first (a bad index or a duplicate code is refused with a sentence
  naming it, before anything is written), but a failure part-way leaves the rows written before it. Preview first and fix what the
  preview says.
- Pieces refer to each other by **position in the document**, so the lists are ordered and the indices matter:
  a page's `designIndex` is a position in `designs`, `desktopThemeIndex` a position in `themes`, `parentIndex` a position in `pages`;
  a section's `pageIndex` is a position in `pages` (or `null` for a shared library section); a menu entry's `parentIndex` is a
  position in the same menu's `items`.
- What an **agent key** may carry is bounded by its scopes: designs, themes, sections, elements, menus and settings need `design`;
  tables need `data` and must not set a `settings` object; **languages must already exist** on the site (`get_site`).
  Anything else is refused with `403` naming the list.

## The shape

```
version      1
languages    [{ code, name, nativeName, dir: "ltr"|"rtl", enabled, isDefault, order }]          existing languages only
designs      [{ name, language, isDefault }]
themes       [{ name, language, isDefault, cssFramework: null|"tailwind"|"daisyui",
                headerHtml, htmlContent, footerHtml, cssContent, jsContent }]                  the layout's THREE parts (below)
pages        [{ slug, language, title, contentHtml, cssContent, published, isHome, queryParams: null,
                designIndex, desktopThemeIndex, tabletThemeIndex, mobileThemeIndex, parentIndex,
                seo: { title, description, keywords, indexable } | null }]
sections     [{ shortcode, name, type, htmlContent, cssContent, jsContent, jsonContent: null, params,
                order, language, isHome: false, isActive: true, pageIndex }]
elements     [{ shortcode, name, description, icon, htmlContent, cssContent, jsContent, params,
                componentType: null, isArchived: false }]
menus        [{ name, slug, language, location, isActive, items: [{ label, type, url, target, openInNewTab, cssClass, order, parentIndex }] }]
collections  [{ name, slug, description, fields, settings: {}, version: 1, isActive }]
settings     [{ key, value, language }]                                                        brand colours, headerHtml, footerHtml only
```

Differences from the granular tools that matter:

- A **theme** here is the layout's three parts, not one canvas: `headerHtml` (the `<header>`), `htmlContent` (the content wrapper that
  holds `[SYS-PAGE-CONTENT]`, e.g. `<main class="page-content-area">[SYS-PAGE-CONTENT]</main>`) and `footerHtml`. Wrap them in nothing
  else; the platform composes them.
- A **menu entry** names its target by **content**, not by id: `target` is `{ "kind": "page", "slug": "about", "language": "en" }`, or
  `{ "kind": "category"|"tag"|"blog", "slug": "…" }`, or `{ "kind": "custom", "url": "/blog" }` (then also set `url`). `type` is the
  matching `PAGE`, `CATEGORY`, `TAG`, `BLOG` or `CUSTOM`.
- Section **codes** carry their prefix (`sk-…`), element codes `el-…`, and every code is unique inside the document and the site.
- Set `published: false` on every page, and `isHome: true` on **exactly one page per design**.
- A page's content lists **section codes**: `"[sk-hero-split][sk-cta]"` - every code must exist in `sections` (or already on the site).
- `type` of a section is one of `HERO WELCOME SERVICES INTRO STATS CTA TESTIMONIALS BOLD_STATEMENT SPLIT_FEATURE CUSTOM`.
- `params` is the same list of parameter definitions `create_section` takes (`key`, `label`, `type`, `default`, `options`…).
- `settings` accepts only `brand_color_primary`, `brand_color_secondary`, `brand_color_light`, `brand_color_dark`, `headerHtml` and
  `footerHtml` (`footerHtml`/`headerHtml` are the **site frame**: the blog's header and footer). `language` is a language code, or
  `all` for the colours.

## A complete, valid example

A one-language site with a home page, a contact page with a form, a menu, a table and the shared header and footer.

<!-- blueprint-example -->
```json
{
  "version": 1,
  "languages": [
    { "code": "en", "name": "English", "nativeName": "English", "dir": "ltr", "enabled": true, "isDefault": true, "order": 0 }
  ],
  "designs": [
    { "name": "Main (en)", "language": "en", "isDefault": true }
  ],
  "themes": [
    {
      "name": "Main (en)",
      "language": "en",
      "isDefault": true,
      "cssFramework": null,
      "headerHtml": "<header class=\"theme-header\"><a class=\"brand\" href=\"/\">[gl-logo alt=\"Acme\"]</a><nav aria-label=\"Main\">[gl-menu-main]</nav></header>",
      "htmlContent": "<main id=\"main\" class=\"page-content-area flex-1\">[SYS-PAGE-CONTENT]</main>",
      "footerHtml": "<footer class=\"theme-footer\"><p>&copy; Acme</p></footer>",
      "cssContent": ".theme-header{display:flex;align-items:center;justify-content:space-between;padding:1rem 1.5rem}.theme-footer{padding:2rem 1.5rem;text-align:center}",
      "jsContent": null
    }
  ],
  "pages": [
    {
      "slug": "home", "language": "en", "title": "Home",
      "contentHtml": "[sk-hero-simple title=\"Roofs that last\" text=\"Installed by people who live here.\" cta=\"Get a quote\" href=\"/contact\"]",
      "cssContent": null, "published": false, "isHome": true, "queryParams": null,
      "designIndex": 0, "desktopThemeIndex": 0, "tabletThemeIndex": null, "mobileThemeIndex": null, "parentIndex": null,
      "seo": { "title": "Acme Roofing - flat and pitched roofs", "description": "Family-run roofing, from repairs to full replacements, with a ten-year workmanship guarantee.", "keywords": null, "indexable": true }
    },
    {
      "slug": "contact", "language": "en", "title": "Contact",
      "contentHtml": "[sk-contact-form]",
      "cssContent": null, "published": false, "isHome": false, "queryParams": null,
      "designIndex": 0, "desktopThemeIndex": 0, "tabletThemeIndex": null, "mobileThemeIndex": null, "parentIndex": null,
      "seo": { "title": "Contact Acme Roofing", "description": "Tell us about your roof and we will reply within one working day.", "keywords": null, "indexable": true }
    }
  ],
  "sections": [
    {
      "shortcode": "sk-hero-simple", "name": "Hero: simple", "type": "HERO",
      "htmlContent": "<section class=\"sk-hero-simple\"><h1>{{title}}</h1><p>{{text}}</p><a class=\"sk-hero-simple__cta\" href=\"{{href}}\">{{cta}}</a></section>",
      "cssContent": ".sk-hero-simple{padding:clamp(3rem,10vw,7rem) 1.5rem;text-align:center}.sk-hero-simple__cta{display:inline-block;padding:.75rem 1.5rem;border-radius:.5rem;background:var(--brand-primary);color:#fff}",
      "jsContent": null, "jsonContent": null,
      "params": [
        { "key": "title", "label": "Headline", "type": "text", "default": "Welcome" },
        { "key": "text", "label": "Intro", "type": "textarea", "default": "" },
        { "key": "cta", "label": "Button text", "type": "text", "default": "Contact us" },
        { "key": "href", "label": "Button link", "type": "url", "default": "/contact" }
      ],
      "order": 0, "language": "en", "isHome": false, "isActive": true, "pageIndex": null
    },
    {
      "shortcode": "sk-contact-form", "name": "Contact form", "type": "CUSTOM",
      "htmlContent": "<section class=\"sk-contact\"><h1>Contact</h1><form class=\"sk-contact__form\" data-collection=\"contact\" novalidate><label>Your name <input name=\"name\" autocomplete=\"name\" required></label><label>Email <input name=\"email\" type=\"email\" autocomplete=\"email\" required></label><label>Message <textarea name=\"message\" rows=\"5\" required></textarea></label><button type=\"submit\">Send</button><p data-form-message role=\"status\" aria-live=\"polite\"></p></form></section>",
      "cssContent": ".sk-contact{max-width:36rem;margin:0 auto;padding:3rem 1.5rem}.sk-contact__form{display:grid;gap:1rem}.sk-contact__form input,.sk-contact__form textarea{width:100%;padding:.6rem;border:1px solid #cbd5e1;border-radius:.4rem}",
      "jsContent": null, "jsonContent": null, "params": [],
      "order": 1, "language": "en", "isHome": false, "isActive": true, "pageIndex": null
    }
  ],
  "elements": [],
  "menus": [
    {
      "name": "Main", "slug": "main", "language": "en", "location": "header", "isActive": true,
      "items": [
        { "label": "Home", "type": "PAGE", "url": null, "target": { "kind": "page", "slug": "home", "language": "en" }, "openInNewTab": false, "cssClass": null, "order": 0, "parentIndex": null },
        { "label": "Contact", "type": "PAGE", "url": null, "target": { "kind": "page", "slug": "contact", "language": "en" }, "openInNewTab": false, "cssClass": null, "order": 1, "parentIndex": null }
      ]
    }
  ],
  "collections": [
    {
      "name": "Contact", "slug": "contact", "description": "Messages from the contact form", "settings": {}, "version": 1, "isActive": true,
      "fields": [
        { "key": "name", "label": "Your name", "type": "text", "required": true },
        { "key": "email", "label": "Email", "type": "email", "required": true },
        { "key": "message", "label": "Message", "type": "textarea", "required": true }
      ]
    }
  ],
  "settings": [
    { "key": "headerHtml", "value": "<a href=\"/\">[gl-logo alt=\"Acme\"]</a><nav aria-label=\"Main\">[gl-menu-main]</nav>", "language": "en" },
    { "key": "footerHtml", "value": "<p>&copy; Acme</p>", "language": "en" }
  ]
}
```

## Procedure

1. `get_site` (languages), `list_*` (what exists). Build the document from what is **missing** - an existing code in the document is
   skipped, which is a silent no-op.
2. Write every page's content as section codes; write every section the pages use into `sections`.
3. `preview_blueprint`. Read the summary: counts to be created, items skipped, any refusal sentence. Fix and preview again until clean.
4. Show the user the plan (pages, sections, menu, table) in a few lines. Get a yes.
5. `apply_blueprint`. Then `render_page` for each page and fix what is wrong with `update_*`.
6. Ask before `publish_page`.

Large documents are fine (a whole site is a few hundred lines), but keep each section's markup tight; one document per site or per
language is easier to review than one giant one.
