# The built-in shortcodes (`gl-…`): what each one does

These exist on every site. Never create a code with the `gl-` prefix and never guess a name: `list_global_shortcodes` answers the list
for the site you are connected to (it also shows the site's own menus and any icons the operator added), and this file says what each
one is **for**. Where the two differ, the tool is right.

Syntax: `[code key="value" other="value"]`. Every parameter has a default, so `[code]` alone renders. An unknown parameter is ignored.
Every code that draws something also takes `class` (extra CSS classes); it is not repeated below.

## Two rules that apply to everything on this page

### Anything that may be used in more than one place is an element

- A **section** (`[sk-...]`) is one band of one page layout. An **element** is a reusable piece that can be used anywhere and owns its
  parameters. A section supplies those parameter values when it calls `[el-...]`.
  inside several sections, on a page, in a blog post, in a theme layout, or twice on the same page. The built-in `[gl-el-...]` codes below
  are elements, and so are the `[el-...]` ones you create.
- If a piece may be used somewhere else (a button, a card, a badge, a testimonial, a price box, a call-to-action strip), **make it an
  element** and let the section use it: `[el-cta-button label="Contact us" href="/contact"]`. The test: would you copy its markup or paste
  the code in a second place? Then it is an element.
- Reach for a built-in element first. Create a new element only when none of the codes below does the job.

### Link to other pages wherever you can (SEO)

Every page and every blog post links to the pages it is related to, with a real `<a href>` and words that say what the target is about.

- Link the first mention of a topic that has a page of its own: `<a href="/pricing">our pricing</a>`, never "click here".
- Use `[gl-el-link]` or `[gl-el-button]` for a link made from a shortcode, and a menu (`[gl-menu-<slug>]`) for site navigation so the same
  links appear everywhere.
- A post links to its category and tag pages and to the product or service page it supports; a service page links back to the posts that
  explain it.
- Write the link as a path (`/about`, `/blog/my-post`), no domain, no `#` placeholder. Check that the target exists (`list_pages`,
  `list_posts`) before you link to it. Do not link a page to itself, and do not leave a page that nothing links to.

## Text and layout elements

| Code | What it does | Parameters worth knowing |
|---|---|---|
| `[gl-el-heading]` | A heading. Use one `<h1>` per page, and the rest in order. | `text`, `size` (`1.25rem` … `3rem`), `align` (`start`/`center`/`end`), `color` |
| `[gl-el-text]` | A paragraph. | `text`, `size`, `align`, `color` |
| `[gl-el-list]` | A bulleted list from a JSON array. | `items="[\"One\",\"Two\"]"` |
| `[gl-el-quote]` | A pull-quote with an accent bar. | `text`, `accent`, `color` |
| `[gl-el-link]` | A text link. The way to link to another page from a shortcode. | `label`, `href`, `color` |
| `[gl-el-button]` | A button that is a link: the call to action. | `label`, `href`, `fill`, `color`, `radius` |
| `[gl-el-image]` | An image. Always give `alt`. | `src`, `alt`, `width`, `radius` |
| `[gl-el-caption]` | A small line under a picture. | `text`, `align`, `color` |
| `[gl-el-divider]` | A horizontal rule. | `color`, `thickness`, `width` |
| `[gl-el-spacer]` | Empty vertical space. Prefer padding in CSS; use this on a page. | `height` |
| `[gl-el-note]` | A highlighted aside with an accent bar. | `text`, `accent`, `background`, `color` |
| `[gl-el-alert]` | A notice box in four tones. | `text`, `tone` (`info`/`success`/`warning`/`danger`), `accent`, `background`, `border` |
| `[gl-el-badge]` | A small label ("New"). | `text`, `background`, `color` |
| `[gl-el-details]` | A fold-out question and answer (accessible, no script). Good for FAQs. | `title`, `text`, `border`, `color` |
| `[gl-el-columns]` | Two columns of text side by side, stacking on a phone. | `left`, `right`, `gap`, `color` |
| `[gl-el-card]` | A card with an optional picture and link. | `title`, `body`, `image`, `link`, `width`, `border`, `background` |
| `[gl-el-gallery]` | A grid of pictures from a JSON array. | `images="[{\"src\":\"/uploads/a.jpg\",\"alt\":\"…\"}]"`, `columns` (2-4), `height`, `gap` |
| `[gl-el-tabs]` | Tabs from a JSON array. | `tabs="[{\"title\":\"…\",\"body\":\"…\"}]"`, `accent` |
| `[gl-el-modal]` | A button that opens a dialog. | `label`, `title`, `body`, `close`, `width` |
| `[gl-el-table]` | A static table from JSON. For a table of **site data** use `gl-data-table`. | `head="[…]"`, `rows="[{…}]"`, `border`, `headBackground` |
| `[gl-el-code]` | A code block. | `text`, `background`, `color` |
| `[gl-el-video]` | A video player. | `src`, `poster`, `width`, `radius` |
| `[gl-el-embed]` | An embedded page (map, player). Only from a source the owner trusts. | `src`, `title`, `width`, `height` |
| `[gl-el-svg]` | An SVG file drawn as a mask, so it takes the text colour. | `src`, `size` |
| `[gl-el-icon]` | An icon from a font set (Lucide, Font Awesome, Bootstrap, Material). Prefer `gl-icon-<name>`. | `set`, `name`, `size`, `color` |

## Form elements

Compose a form from these. A form's data goes to a data table (`create_collection`); put the form in a section or page, never inside a
parameter value (a `<form>` there is stripped).

| Code | What it does | Parameters worth knowing |
|---|---|---|
| `[gl-el-form]` | The `<form>` wrapper. | `action`, `gap`, `width` |
| `[gl-el-input]` | A one-line field with its `<label>`. | `label`, `name`, `kind` (`text`/`email`/`tel`/`number`/`date`/`password`/`search`), `placeholder`, `required` |
| `[gl-el-textarea]` | A multi-line field. | `label`, `name`, `rows`, `required` |
| `[gl-el-select]` | A dropdown. | `label`, `name`, `options="[\"A\",\"B\"]"` |
| `[gl-el-checkbox]` | A checkbox. | `label`, `name`, `checked` |
| `[gl-el-submit]` | The submit button. | `label`, `fill`, `color`, `radius` |

## Brand, language, menus, theme

| Code | What it does |
|---|---|
| `[gl-logo]` | The site's logo; shows the light or dark file by itself when both exist. Takes `alt`. Never hard-code the logo. |
| `[gl-logo-dark]` | The dark-mode logo only. |
| `[gl-logo-icon]` | The favicon, as an image. |
| `[gl-lang-switcher]` | The language switcher. Draws nothing on a one-language site. |
| `[gl-locale]` | The current language's link prefix (`fa` on `/fa/…`, empty on the default language). Build links that stay in the visitor's language. |
| `[gl-menu-<slug>]` | A menu made with `create_menu` / `set_menu_items`: slug `main` is `[gl-menu-main]`. Its layout is stored **on the menu** (`orientation`: `horizontal` row with fly-out sub-menus or `vertical` tree; `mobileMode`: `collapse` = a three-bar toggle with the sub-menus as a cascade, `stack`, `keep`) - set it with `create_menu` / `update_menu`, and the shortcode follows it. A placement can still override with `orientation`, `mobile` and `label` (the nav's accessible name). The place for site-wide links. |
| `[gl-social]` | The social-links panel built from the owner's settings (a link with no value is left out). |
| `[gl-theme-toggle]` | The dark/light switch. Put it **once**, in the theme layout. |
| `[gl-map]` | The site’s own location (panel: Settings ▸ Google ▸ Location) as an interactive map, with the address and a directions link under it. Draws nothing until the owner sets a location. | `title`, `height`, `zoom`, `button`, `class` |
| `[gl-map-pin]` | The same map as a **picture with the site’s logo as the pin**. Needs a Maps API key in the same panel; without one it draws the interactive map instead. | `title`, `height`, `zoom`, `button`, `class` |

## Pictures

| Code | What it does |
|---|---|
| `[gl-icon-<name> size="2rem" class="text-orange-500"]` | One of the platform's icons, drawn as a mask in the surrounding text colour. `list_icons` (search by word) answers every name that exists, including the ones the operator added; do not invent one. About thirty ship: qr, leaf, chart, brain, chat, messenger, live, publish, editor, bot, zap, shield, arrow, check, globe, award, trend, device, pen, cpu, megaphone, phone, mail, share, linkedin, instagram, x, facebook, sun, moon. |
| `[gl-shape-rect]` `-circle` `-triangle` `-diamond` `-hexagon` `-star` `-arrow` `-line` `-ring` `-pentagon` `-octagon` `-plus` | A decorative drawn shape: `width`, `height`, `fill`, `rotate`, plus `radius` (rect) and `thickness` (ring). |
| `[gl-shape-svg src="…"]` | Your own SVG file as a shape. |

## Data, blog and charts

These draw what the site itself holds. They show a frame and are filled when the page is rendered, so a page that uses one is dynamic
and `render_page` is how you check it.

| Code | What it does | Parameters worth knowing |
|---|---|---|
| `[gl-blog-list]` | The published posts in the page's language (drafts never appear). The way to make a blog index, and to put "related posts" on a page. | `layout` (`table`, or `cards` for linked cards with each post's first picture), `columns` (`title,excerpt,date,category,tags`; default `title,date`), `limit` (1-50, default 10), `order` (`newest`/`oldest`), `category` and `tag` (one slug each), `title`, `empty`, `class`. Full detail in the `shortcodes` topic. |
| `[gl-data-table]` | The entries of one of the site's data tables, or of an imported source. | `table` or `source`, `columns`, `limit`, `order`, `title`, `empty` |
| `[gl-feed]` | A short list of one field from a table or source, optionally linked. | `source`, `field`, `link`, `limit`, `order` |
| `[gl-chart-bar]` `[gl-chart-line]` | A bar or line chart of how many records arrived per `bucket` (`day`/`week`/`month`). | `table`, `field`, `bucket`, `limit`, `colour`, `height` |
| `[gl-chart-donut]` | A donut of the values of one field. | `table`, `field`, `limit` |
| `[gl-chart-stat]` | One number: a count, a sum or an average. | `table`, `metric` (`rows`/`sum`/`average`), `field`, `label`, `suffix` |

## Questions and answers

The owner answers questions in the panel; these draw the answered ones. They are rows, not markup, so an answer written once
appears on every page that shows its list - and a page with `[gl-faq]` carries `FAQPage` structured data that search engines
and answer engines read. Nobody gains answers by writing markup: the owner adds them in the panel.

| Code | What it does | Parameters worth knowing |
|---|---|---|
| `[gl-faq]` | The site's published questions and answers in the page's language, each fold-out, with a helpful / not helpful button. | `group` (the list to show, default `general`), `title`, `limit` (default 50, at most 200), `open` (`yes` shows every answer open) |
| `[gl-faq-ask]` | The form a visitor asks a new question with (name, email, question). It mails a confirmation link and stores nothing public until the owner answers. | `group` (the list the answer joins, default `general`), `title`, `button` |

A question nobody has answered yet appears only to the owner in the panel - so a page with `[gl-faq]` and no answers shows its
heading and nothing under it. Put `[gl-faq-ask]` below the list, or on a contact page.

## Social feeds (YouTube and Instagram)

The owner connects a YouTube channel or an Instagram Business/Creator account in the panel (**Social feeds**) and presses **Sync now**; the videos, posts and playlists are copied into the
site, so a page never calls YouTube while it renders. Until the owner has connected and synced, both codes draw a short "nothing to show yet"
note - do not write video markup by hand, and do not invent playlist names: call `get_social_status` first. It says whether each account is
connected and synced (and what to ask the owner to press if not) and lists the exact YouTube playlist names and ids to use in `playlist="..."`.

| Code | What it does | Parameters worth knowing |
|---|---|---|
| `[gl-youtube]` | The channel's videos as a grid of cards, newest first. Each card links to the video on YouTube; no script and no third-party cookie is on the page. | `limit` (how many, default 6, at most 50), `playlist` (only the videos of one playlist: its name or id; a name that matches none shows nothing), `columns` (1-6, default 3), `title` |
| `[gl-youtube-playlists]` | The channel's playlists as cards, with the number of videos in each. | `limit` (default 6), `columns` (1-6), `title` |
| `[gl-instagram]` | The connected Instagram account's newest posts as square tiles; each links to the post on Instagram. | `limit` (default 6, at most 50), `columns` (1-6, default 3), `title` |

To build a page per playlist, use one `[gl-youtube playlist="..."]` per section, and `[gl-youtube-playlists]` as the index.

## Commerce and plans

Only when the owner has set them up; ask before using.

| Code | What it does |
|---|---|
| `[gl-store-products]` | The store's product list. `columns`, `limit`. |
| `[gl-store-product slug="…"]` | One product by its slug. |
| `[gl-pay plan="…" currency="…" label="…"]` | A pay button for a plan, through the owner's payment account. |
| `[gl-plans highlight="pro" cta="Start" columns="3" only="free,pro"]` | The installation's public plans as cards. **Installation's own site only**; on a tenant's site it draws nothing and is not in `list_global_shortcodes`. Prices and limits come from the plan rows: never retype them. |
| `[gl-signup plan="pro" label="Start with Pro"]` | A sign-up button. Same restriction as `gl-plans`. |

## Page effects (theme layout, once, only if the owner wants them)

| Code | What it does |
|---|---|
| `[gl-cursor]` | A custom cursor dot that follows the pointer. |
| `[gl-network]` | A particle-network background on a canvas (`count`, `distance`, `z`). Costs CPU: leave it out on content-heavy pages. `z` is the layer the canvas sits on: `-1` never draws over the content, `0` (default) is the background a theme's `main`/footer stack above, and `20` shows the particles over sections that paint their own opaque background — the usual reason it seems not to appear in a theme you wrote yourself. |
| `[gl-anim]` | Turns on the entrance animations for the elements that ask for them. |

Prefer a built-in over writing the same markup: it follows the site's branding and works on the designer's canvas.
