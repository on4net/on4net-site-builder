# Structure: how the pieces relate

Read this when you need the exact rules behind "a page is a list of sections", or you are about to create something and want to
know what it belongs to, what it affects and what deleting it does.

## The entities

| Entity | What it is | Belongs to | Addressed by | Key fields |
|---|---|---|---|---|
| **Site** | the one site your key works on | an installation | (your key) | name, addresses, languages |
| **Language** | a language the site is written in | the site | code (`en`, `fa`) | name, direction (`ltr`/`rtl`), default |
| **Design** | the container of one language's pages | site + language | id | name, language, isDefault |
| **Theme layout** | the chrome around a page: header, page slot, footer, plus css/js | site + language | id | name, html (canvas), css, js, framework |
| **Page** | what a visitor opens | a design (and so a language) | address `slug`, under its parent's | title, slug, content html, css, theme per viewport, SEO, published, isHome |
| **Section** | one band of a page | site (library) or owned by one page | code `sk-…` | name, type, html, css, js, parameters, order |
| **Element** | a reusable piece | site | code `el-…` | name, html, css, js, parameters, archived |
| **Menu** | navigation | site + language (or `all`) | slug → `[gl-menu-<slug>]` | items (tree, one drop-down level) |
| **Media file** | an uploaded file | site | its `url` | alt text, description |
| **Post** | a blog article | site + language | slug under the blog | title, slug, content html, category, tags, SEO, published |
| **Category / Tag** | how posts are grouped | site + language | id, slug | name, slug |
| **Data table** (collection) | a list of fields | site | slug | name, fields, entries |

## One language, one world

Designs, theme layouts, pages, posts, categories and tags each belong to **one language**. A bilingual site is two parallel
sets:

```
en: design "Main (en)"  → pages: home, about, services     theme layout "Main (en)"   menu "main" (language en)
fa: design "Main (fa)"  → pages: home, about, services     theme layout "Main (fa)"   menu "main" (language fa)
```

- The page **slug** is unique among its siblings inside a design and language. `about` in `en` and `about` in `fa` do not clash.
- A menu with language `all` is one menu shared by every language (its labels are then the same in all of them - fine for a
  logo-only or icon menu, wrong for text links).
- Use only languages `get_site` lists. You cannot add one; the owner does that in Settings.
- Sections and elements are written in a language too (`create_section` takes `language`), but a code is unique per **site**, so
  a Persian hero and an English hero need different codes (`sk-hero-en`, `sk-hero-fa`) unless one section is built with
  parameters for the words and used by both.

## How a page is put together

```
 visitor opens /about
        │
        ▼
 Page "About"  ──theme layout──►  header   ← theme layout: [gl-logo] [gl-menu-main] [gl-lang-switcher]
        │                         ┌──────────────────────────┐
        │  contentHtml ────────►  │  [SYS-PAGE-CONTENT]        │  ← replaced by the page's content
        │   [sk-page-header …]    │   = [sk-page-header …]     │
        │   [sk-team]             │     [sk-team]              │
        │   [sk-cta]              │     [sk-cta]               │
        │                         └──────────────────────────┘
        ▼                         footer
 each [sk-…] expands to that section's html (+ its css and js), with the parameters the page passed
 each section may contain [el-…] and [gl-…], which expand the same way (nesting up to 5 levels)
```

- The page's content is stored **as written** (`[sk-hero title="…"]`); the server expands it when a visitor opens the page.
  `get_page` shows the stored form, `render_page` shows what a visitor receives.
- The **theme layout** is chosen per page and per viewport: `desktopThemeId`, and `tabletThemeId` / `mobileThemeId` when
  `sameInAllViews` is false. Most pages use one layout everywhere: set `desktopThemeId` only.
- A page with **no theme layout** is drawn in the platform's **site frame** (see below), not in your design. Always give pages a
  theme layout - copy the one the site's existing pages use.
- A **section can own a page** (`pageId`) or sit in the shared library (no `pageId`). Library sections are the norm: a page uses
  them by code. Page-owned sections are for a one-off band; they still work by code.
- `isHome` marks the design's home page (the address `/`). Making a page home through `update_page` demotes the previous one;
  creating a page with `isHome` does not, so do it in two steps.
- `parentId` nests a page under another page of the **same design** (`/services/web`). One level of nesting is the norm.

## The site frame: the blog, categories, tags, search and contact

A page that names a theme layout is drawn in **that layout**. Everything else a visitor can reach - the blog list and its
posts, category and tag pages, search, the contact page, and any page with no theme layout - is drawn in the **site frame**: a
header and a footer stored per language as two settings. They hold markup with shortcodes (`[gl-logo]`, `[gl-menu-main]`) just
like a theme layout does.

So a site with a beautiful theme on its pages and the default frame on its blog looks like two sites. After you build the theme
layout, give the frame the **same header and footer** with `set_site_frame` (same markup, same shortcodes). The frame's header
and footer are replaced as written; to keep part of what is there, you must send it again.

## Draft and published

- New pages and posts are **drafts**. A draft is invisible to visitors; with your key you can still `get_page` it and
  `render_page` it. Only `publish_page` / `publish_post` make it public. `published: false` takes it back.
- Sections, elements, theme layouts and menus have no draft state: changing one **changes every published page that uses it, at
  once**. Treat edits to shared pieces as live edits - tell the user what is affected before you change a section that many
  pages use (`get_page` of a few of them shows which codes they contain).

## What each delete does

| Delete | Result |
|---|---|
| `delete_page` | removes the page and the sections it owns; its address answers `404` unless you passed `redirectTo` |
| `delete_section` / `delete_element` | gone for good; any page still using the code shows `[sk-…]` as literal text |
| `delete_theme_layout` | pages that wore it fall back to the site frame until you assign another |
| `delete_design` | only if it is empty and not the default |
| `delete_menu` | `[gl-menu-<slug>]` renders nothing |
| `archive_element` | hides it from the library only; pages using it keep working (prefer this to deleting) |

## Addresses (slugs)

- Lower case letters, digits and hyphens; no spaces or leading slash: `about-us`. The platform normalises what you send.
- A page's full address is its parent chain: parent `services`, child `web` → `/services/web`. `render_page` takes that path
  without the leading slash.
- Changing a published page's slug breaks the links people already have. Do it only on request, and then create a redirect by
  telling the user (you cannot create redirects yourself).
- The blog lives under `/blog/<slug>`; the post `slug` is the last part.

## Codes (shortcodes)

- `el-…` elements, `sk-…` sections: yours, stored per site, lower case `[a-z0-9_-]`.
- `gl-…`: built into the platform, identical on every site; never create one. Names taking a variable part: `gl-menu-<slug>`,
  `gl-icon-<name>`, `gl-shape-<shape>`, `gl-el-<element>`.
- A code the platform cannot resolve is left in the page as text. `render_page` returns those as `unresolvedShortcodes`.
