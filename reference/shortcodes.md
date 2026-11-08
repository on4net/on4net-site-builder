# Shortcodes: the markup language of a page

A page, a section, an element, a theme layout and a blog post are all written in HTML **plus shortcodes**: square-bracket
expressions the server replaces with markup when a visitor opens the page. This is how pieces are reused and how one piece says
different things in different places.

## The syntax

```
[sk-hero]                                   a section, without parameters
[el-cta-button label="Contact us" href="/contact"]
[gl-logo class="h-8" alt="Acme"]
```

- A code is lower case letters, digits, `-` and `_`. The first part is the family: **`sk-`** a section, **`el-`** an element,
  **`gl-`** a built-in.
- Values go in double quotes; a value may contain spaces and even `]`. Escape a quote or a backslash inside a value with `\`.
- Parameter names are the `key`s the piece declares. An unknown name is ignored. Every built-in and what it does is in the `builtins` topic.
- Expressions are expanded **inside** pages, sections, elements, theme layouts, the site frame and blog posts. A section's markup
  may use elements and built-ins, which may use others (up to 5 levels deep).
- An expression the server cannot resolve (no such code) stays in the page as literal text. That is a bug on the live page:
  `render_page` lists them as `unresolvedShortcodes`.

## Section or element?

Sections and elements are not interchangeable:

- A **section** is one band of one page layout (a hero, a features grid, a pricing band). A page lists sections.
- An **element** is a piece that may be used in more than one place: inside several sections, on a page, in a blog post, in a theme
  layout, or twice on the same page (a button, a card, a badge, a testimonial, a price box, a call-to-action strip).
- **If a piece may be used somewhere else, it must be an element.** The test: would you ever copy its markup, or paste the code in a
  second place? Then extract it into an element (`create_element`) and let the section use `[el-…]`. A section that repeats the same
  card three times, or that another section also needs, is two pieces wearing one name: the card is an element, the band is a section.
- Changing an element changes every place that uses it, which is the point; changing a section changes every page that lists it.

## Writing a piece that takes parameters

**Parameters belong to elements.** Declare parameters once on an element (`params`), use `{{key}}` in the element's html, css and
js, and give every parameter a default. The section then supplies values when it calls the element:

```
create_element {
  code: "feature-card", name: "Feature card",
  params: [
    { key: "title", label: "Title", type: "text", default: "Feature" },
    { key: "text", label: "Description", type: "textarea", default: "" },
    { key: "icon", label: "Icon", type: "text", default: "" }
  ],
  html: "<article><h3>{{title}}</h3><p>{{text}}</p></article>"
}

create_section {
  code: "features-simple", name: "Features: simple",
  html: "<section class=\"sk-features-simple\">
           [el-feature-card title=\"Fast setup\" text=\"Start in minutes.\"]
           [el-feature-card title=\"Easy edits\" text=\"Change one shared element.\"]
         </section>"
}
```

Then on a page: `[sk-features-simple]`.

Do not put page-specific parameter definitions or `{{key}}` placeholders in a section for values that belong to repeated elements. If
the same band layout needs a different set of values on another page, create another section (such as `sk-features-enterprise`) and
make its element calls use that section's values. Each section is a separate band; each element remains reusable with one shared
parameter definition.

### Parameter types

| `type` | Value | Rendered |
|---|---|---|
| `text` (default) | one line | HTML-escaped |
| `textarea` | several lines | HTML-escaped |
| `html` | markup | a **hardened** subset: `<script>`, `<style>`, `<iframe>`, `<object>`, `<embed>`, `<link>`, `<meta>`, `<base>`, `<form>` tags, `on…=` handlers and `style=` attributes are removed |
| `number` | digits | numeric only |
| `boolean` | `true` or `false` | the text as given - use it in a class name or attribute (`class="is-{{dark}}"`), not as a condition |
| `color` | `#f97316` or a colour name | validated |
| `image`, `video` | a media-library `url` | the library's address |
| `url` | a link | sanitised (no `javascript:`) |
| `select` | one of `options` | the choice |
| `list` | rows (see below) | used with `{{#each}}` |
| `rows` | a data **source** name | the rows that source imported |

Values placed in **css** or **js** are sanitised for that context (no `{`, `}`, `<`, `>`), so a parameter can carry a colour or a
size, never code. Put anything dynamic in a parameter; never concatenate user text into a `js` string yourself.

### Lists and loops

A `list` repeats a block once per row. Declare the row's fields in `itemFields`, loop with `{{#each key}}…{{/each}}`, and pass the
rows as JSON in the attribute (quotes escaped with `\`):

```
params: [{ key: "items", type: "list", itemFields: [
          { key: "name", type: "text" }, { key: "role", type: "text" }, { key: "photo", type: "image" } ] }]
html:   "<ul class=\"sk-team\">{{#each items}}<li><img src=\"{{photo}}\" alt=\"{{name}}\"><b>{{name}}</b> {{role}}</li>{{/each}}</ul>"

[sk-team items="[{\"name\":\"Ana\",\"role\":\"CEO\",\"photo\":\"/uploads/ana.jpg\"},{\"name\":\"Bo\",\"role\":\"CTO\",\"photo\":\"/uploads/bo.jpg\"}]"]
```

Inside a loop `{{name}}` is the row's field, `{{@index}}` is 0, 1, 2…, `{{@number}}` is 1, 2, 3…, and `{{this}}` is the value of a plain
string list (`items="[\"a\",\"b\"]"`). A top-level parameter is still available inside the loop. Loops nest up to 4 deep. A list
that is not valid JSON renders as empty rather than breaking the page.

## The built-ins (`gl-…`)

They exist on every site; never create one with that prefix, and **do not guess their names**: `list_global_shortcodes` is the
truth, with each one's parameters. The families:

| Family | Codes | Use |
|---|---|---|
| Brand | `[gl-logo]` `[gl-logo-dark]` `[gl-logo-icon]` | the site's logo (switches with the theme), the dark-mode file, the favicon. Take `class` and `alt`. Never hard-code the logo. |
| Menus | `[gl-menu-<slug>]` | a menu made with `create_menu` / `set_menu_items` - slug `main` is `[gl-menu-main]` |
| Language | `[gl-lang-switcher]` `[gl-locale]` | the language switcher (empty on a one-language site); the current language's link prefix |
| Blog | `[gl-blog-list]` `[gl-blog-list layout="cards" limit="3" columns="title,excerpt,date" category="news"]` | the site's published posts in the page's language (drafts never appear), as a table or as linked cards - every parameter is in **The blog** below |
| Data | `[gl-data-table table="slug" limit="5" columns="a,b" order="oldest" title="…"]` `[gl-feed source="…" field="…" limit="5"]` | a data table's entries, or an imported source's rows |
| Charts | `[gl-chart-bar]` `[gl-chart-line]` `[gl-chart-donut]` `[gl-chart-stat]` with `table=` `field=` `bucket=` `metric=` | draw a table's own records |
| Icons | `[gl-icon-<name>]` with `size="2rem"` `class="…"` | the platform's icons: `list_icons` (search by word) answers every name that exists, including the ones the operator added to the repository; do not invent one. The ~30 that ship: qr, leaf, chart, brain, chat, messenger, live, publish, editor, bot, zap, shield, arrow, check, globe, award, trend, device, pen, cpu, megaphone, phone, mail, share, linkedin, instagram, x, facebook, sun, moon. Colour comes from the surrounding text colour. |
| Shapes | `[gl-shape-rect]` `-circle` `-triangle` `-diamond` `-hexagon` `-star` `-arrow` `-line` `-ring` `-pentagon` `-octagon` `-plus` `-svg src="…"` | decorative drawing shapes with `width`, `height`, `fill`… |
| Elements | `[gl-el-heading text="…" size="2rem"]` `[gl-el-button label="…" href="/contact"]` and paragraphs, lists, quotes, images, dividers, spacers | plain building blocks with parameters |
| Chrome | `[gl-cursor]` `[gl-network]` `[gl-social]` `[gl-theme-toggle]` `[gl-anim]` | cursor effect, particle background, social panel, dark/light switch, entrance animations - place **once**, in the theme layout, and only if the user wants the effect |
| Commerce | `[gl-store-products]` `[gl-pay]` | only when the owner has set up the store / a payment account; ask before using |
| Plans | `[gl-plans highlight="pro" cta="Start" columns="3" only="free,pro"]` `[gl-signup plan="pro" label="Start with Pro"]` | the installation's public plans as cards, and a button to sign-up. **Only on the installation's own site** - on a tenant's site they draw nothing and are not in `list_global_shortcodes`, so a tenant agent never sees them. Prices and limits come from the plan rows; never retype them. |

Prefer a built-in over writing the same markup: it follows the site's branding and works on the designer's canvas.

## The blog: `[gl-blog-list]`

One code covers the blog: written anywhere a shortcode may go, it lists this site's posts. The read happens on the server while
the page is rendered, so the titles are in the HTML a crawler receives, and no attribute can make it read another site's posts.

| Parameter | Default | What it does |
|---|---|---|
| `layout` | `table` | `table` draws a grid of the fields the author chose; `cards` draws each post as a card that links to it, with the first picture of the post's body as the cover. |
| `columns` | `title,date` | The fields a **table** shows, comma-separated: `title`, `excerpt`, `date`, `category`, `tags`. A key outside this list is dropped, never shown as an empty column. A `cards` list ignores it and draws its own card (cover picture, category, title, excerpt, date). |
| `limit` | `10` | How many posts, at most `50`. A draft is not counted, because it is never shown. |
| `order` | `newest` | `newest` or `oldest`, by the day the post was created. |
| `category` | *(empty)* | One category's **slug** (the filter's vocabulary; the list shows the category's name). Empty shows every category. |
| `tag` | *(empty)* | One tag's slug, the same way. Empty shows every tag. |
| `title` | *(empty)* | A heading above the list. |
| `empty` | `No data yet.` | What the reader is told while this site has published nothing in the page's language. It is never an empty box. |
| `class` | *(empty)* | Extra CSS classes on the frame. |

**What it never shows.** Drafts, posts of another language, and another site's rows. The language is the page's own, so one
language's index never mixes the other's posts. A read that fails leaves a sentence (`The posts could not be read.`) where the
rows belong, rather than breaking the page.

**Class hooks.** `cards` draws the same markup as the built-in blog screens, so one theme styles both: `.blog-list` ·
`.post-card` with `.post-card__image`, `.post-card__category`, `.post-card__title`, `.post-card__excerpt`, `.post-card__date`.
`table` draws the data family's grid (`.on4-data-grid`).

**As a template page.** A page can be the design of a screen (see `theme-and-style`):

- slug `blog` - the blog's front page: `[gl-blog-list limit="12" columns="title,excerpt,date,category"]`.
- slug `category` - the template of every category's page: declare `queryParams: [{key: "category"}]` and write
  `[gl-blog-list category="{{category}}"]`; `/category/news` fills `news` in.
- slug `tag` - the same for tags: `queryParams: [{key: "tag"}]` and `[gl-blog-list tag="{{tag}}"]`.

The **indexes** `/category` and `/tag` are still drawn by the platform: no shortcode lists the terms themselves yet, so a page
cannot replace those two screens. A single post (`/blog/<slug>`) is always drawn by the platform.

## Where shortcodes may and may not appear

- **Page content, sections, elements, theme layout, site frame, blog posts:** yes.
- **A page's SEO title/description and a menu's labels:** no - plain text.
- **Inside a parameter value:** no - the value is data; put the shortcode in the markup around it.
- **`{{…}}` placeholders** only resolve in an element that **declares** that parameter (or inside a loop over it). A `{{key}}` on a
  page itself resolves only if the page declares that key as a query parameter; never declare page-specific placeholders in a section.

## Mistakes that cost an afternoon

1. A page uses `[sk-pricing-table]` but the section was created as `sk-pricing` → literal text on the live page. Copy codes from
   `list_sections`.
2. An element parameter used as `{{subtitle}}` but never declared → shows empty. Declare every element key you use, with a default.
3. Unscoped CSS (`h2 { … }` in a section) restyles every h2 on the site. Prefix with the piece's own class.
4. Passing a list without escaping quotes (`items="[{"a":1}]"`) → the attribute ends early. Escape: `\"`.
5. A `<form>` inside an `html`-typed element parameter is stripped. Put forms in the section's own markup or in a dedicated element.
6. A section should not carry a long list of parameters. Put reusable values on elements and keep each section element call readable;
   if the set of values differs substantially, create another section.
