# Theme layouts, colours and style

## The theme layout

A theme layout is the **chrome** around every page that wears it. You write it as ONE canvas:

```html
<div class="theme-layout-wrapper min-h-screen flex flex-col">
  <header class="theme-header">
    <a href="/">[gl-logo class="site-logo" alt="Acme"]</a>
    <nav aria-label="Main">[gl-menu-main]</nav>
    [gl-lang-switcher]
  </header>
  <main class="page-content-area flex-1">
    [SYS-PAGE-CONTENT]
  </main>
  <footer class="theme-footer">
    <p>© Acme</p>
    [gl-menu-footer]
  </footer>
</div>
```

Rules the platform depends on:

- **`[SYS-PAGE-CONTENT]` appears exactly once**, inside the `<main>`. It is where the page's content goes. A layout without it
  shows no page.
- Keep the **three regions** - a `<header class="theme-header">`, a `<main class="page-content-area">`, a
  `<footer class="theme-footer">` - as direct children of the wrapper. The platform splits the canvas into header / content
  wrapper / footer by those, and renders the header and footer around each page.
- Use **shortcodes for the logo, menus and language switcher**, never fixed copies: the logo follows the brand settings and
  dark mode, a menu follows `set_menu_items`.
- The layout's own `css` and `js` ride with the layout. Keep them small; everything specific to a band belongs in that section.
- A layout is per language. Create `Main (en)` and `Main (fa)`; the right-to-left one mirrors the header.
- To change an existing layout, `get_theme_layout`, edit the canvas you read, `update_theme_layout`.
- The **blog, search, contact and category pages** are not drawn in your layout but in the **site frame**: also call
  `set_site_frame` with the same header and footer markup (without `[SYS-PAGE-CONTENT]`, which is only for layouts).

### Different layouts per viewport

`sameInAllViews` is true by default: one layout everywhere. To use another on phones, create it and set the page's
`mobileThemeId` and `sameInAllViews: false`. A viewport with no layout of its own borrows desktop, then tablet, then mobile.
Prefer **one responsive layout** (a menu that collapses with CSS) over several layouts.

## Brand colours and dark mode

The site's colours are CSS variables the platform writes for every page, so use them instead of hard-coding hex values:

```css
:root                  { --accent:#f97316; --brand-primary:#f97316; --blue:#3b82f6; --brand-secondary:#3b82f6; }
html:not(.dark)        { --bg:#f8fafc; --brand-background:#f8fafc; }
html.dark              { --bg:#030712; --brand-background:#030712; }
```

- `var(--brand-primary)` is the main brand colour, `var(--brand-secondary)` the second, `var(--bg)` the page background (it
  changes with the theme).
- Dark mode is the class `dark` on `<html>`: write dark rules as `html.dark .my-piece { … }`, and let the colours above do most of
  the work. If you include `[gl-theme-toggle]` in the header, visitors can switch.
- These values are the owner's brand settings. You cannot change them with a tool; a blueprint's `settings` can carry
  `brand_color_primary`, `brand_color_secondary`, `brand_color_light`, `brand_color_dark` if the user asks you to set a palette.

- **Never write a brand colour as a hex value** in a theme, layout, section or element: the owner changes the colours in the settings and
  everything that reads the variables follows; a hard-coded `#f97316` stays orange forever. Only neutral greys, white and black may be literal.
  Derive shades from the variables (`color-mix(in srgb, var(--brand-primary) 15%, transparent)`).
  After creating or changing a theme, run `audit_theme_colors {id}`: it lists the hex colours left in the css, and `apply: true` turns the ones
  that are exactly the brand's primary or secondary into variables.

## The blog, categories, tags and posts wear your theme

These screens are drawn by the platform inside the theme's header and footer (the theme linked to the language, or the language's default
design). Their markup is fixed, so you style them **by class** in the theme's CSS:

`.blog-page`, `.category-page`, `.tag-page`, `.post-page` (the wrapper) · `.page-title` (the h1) · `.blog-list` (the list) · `.post-card`
with `.post-card__title`, `.post-card__category`, `.post-card__excerpt`, `.post-card__date` · `.category-list`, `.tag-list` (the chips).

Style these in the theme and the whole blog matches the rest of the site; check `/blog` after attaching a theme.

**To design them as pages** (any theme layout, sections, your own markup), create ordinary pages with these slugs:

- slug `blog` - the blog's front page: put `[gl-blog-list limit="12" columns="title,excerpt,date,category"]` in it.
- slug `category` - the **template for every category**. `create_page` with `queryParams: [{key: "category"}]` and
  `[gl-blog-list category="{{category}}"]` in the html. `/category/news` then shows this page with `news` filled in.
- slug `tag` - the same for tags: `queryParams: [{key: "tag"}]` and `[gl-blog-list tag="{{tag}}"]`.

A site with no such page keeps the platform's own list, so create them only when the user wants these screens designed. A single
post (`/blog/<slug>`) is always drawn by the platform inside the theme's header and footer.

## Tailwind utilities (`framework: "tailwind-jit"`)

Create the theme layout with `framework: "tailwind-jit"` and write ordinary Tailwind classes in the markup (`flex items-center gap-4 p-6
md:grid-cols-3 rounded-xl`, `hover:`, `md:`, `dark:`). The platform generates the stylesheet for each page from the classes that page
actually uses, so nothing is shipped that is not used, and you write far less css.

- The brand colours are the palette: `bg-primary`, `text-primary`, `border-primary`, `bg-secondary`, `bg-background`, … read the owner's
  settings, so they follow when the owner changes them. **Use these, never `bg-orange-500` or `[#f97316]`** for a brand colour.
- Opacity on those three does not work (`bg-primary/50`); use plain css `color-mix(in srgb, var(--brand-primary) 50%, transparent)`.
- There is no Tailwind reset (the site keeps its own base styles) and dark mode is the `dark` class on `<html>` (`dark:bg-gray-900`).
- It applies to pages that wear that theme layout. The visual builder's canvas does not draw these utilities, so check with `render_page`.
- Classes built at run time in JavaScript (`"bg-" + name`) are not seen; write whole class names.

## The CSS you can rely on

The platform serves **one fixed stylesheet** plus whatever css you attach to a theme layout, section or element. That means:

- A **utility class** (`flex`, `grid`, `min-h-screen`, `gap-4`, `text-center`) works only if that stylesheet contains it. The default
  theme canvas uses a few. For anything beyond what you have seen working in the site's existing pages, write **plain CSS** in the
  piece's own `css`.
- Choose the layout's framework with `framework`: unset or `tailwind` is the base layer; `daisyui` adds the DaisyUI component
  classes (`btn`, `card`, `navbar`). A framework's sheet is a subset, loaded **before** your css so yours wins.
- Put **all of a piece's CSS in the piece**. A section that works anywhere is a section the owner can move.

### CSS conventions that keep a site coherent

```css
/* scoped to the piece; BEM-ish; values from variables */
.sk-pricing { padding: clamp(2rem, 6vw, 5rem) 1rem; background: var(--bg); }
.sk-pricing__grid { display: grid; gap: 1.5rem; grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr)); }
.sk-pricing__card { border: 1px solid color-mix(in srgb, currentColor 15%, transparent); border-radius: 1rem; padding: 1.5rem; }
.sk-pricing__cta { background: var(--brand-primary); color: #fff; }
@media (prefers-reduced-motion: no-preference) { .sk-pricing__card { transition: transform .2s } .sk-pricing__card:hover { transform: translateY(-2px) } }
```

- Scale: use a few spacing steps (`.5 1 1.5 2 3 5rem`) and `clamp()` for fluid type and padding rather than breakpoint soup.
- Layout: `grid` with `auto-fit/minmax` or `flex` with `wrap`; one media query for the big step, not five.
- Type: a font from the platform's repository (see **Fonts**), else a system stack; `line-height: 1.6` for body text; `max-width: 65ch` for reading text.
- Images: `max-width: 100%; height: auto`; give `width`/`height` attributes to stop layout shift; `loading="lazy"` below the fold.
- Motion: gentle, and wrapped in `prefers-reduced-motion`.
- Never `!important` to win a fight with another piece - fix the other piece's scope.

## Fonts

The platform hosts open-licence web fonts on its own CDN, so a site never calls Google Fonts or another third party. Call **`list_fonts`**:
each entry gives `scripts`, `weights`, a `stylesheet` URL and the `fontFamily` to use.

```css
@import url("<stylesheet>");           /* first line of the theme's css */
body { font-family: <fontFamily>; }
h1, h2, h3 { font-weight: 700; }
```

- Pick by the language: a `rtl` Persian/Arabic site needs a font whose `scripts` include `arabic` (e.g. Vazirmatn); a Latin-only font falls back for the letters it lacks.
- One body family and at most one display family; load only the weights the entry lists (400/500/700).
- The stylesheets use `font-display: swap` and split by script, so the browser fetches only the parts the page uses. Do not link the same font twice.

## Right-to-left languages

The language's `dir` (from `get_site`) is applied to the document by the platform. For a `rtl` language (Persian, Arabic, Hebrew):

- Write **logical** CSS: `margin-inline-start`, `padding-inline`, `text-align: start`, `inset-inline-end`, `border-inline-start`; avoid `left` /
  `right` and `float: left`.
- Flex and grid reverse by themselves; do not add manual `row-reverse` for direction.
- Icons that mean direction (arrows, chevrons) need `transform: scaleX(-1)` under `[dir="rtl"]`.
- Write the **copy** in the language, not translated word for word from English; numbers and dates use the language's conventions.
- Persian typography: a font with Arabic-script support; do not letter-space (`letter-spacing`) Arabic-script text.

## Accessibility checklist for a theme

- A "skip to content" link first in the header: `<a class="skip-link" href="#main">Skip to content</a>`, with `id="main"` on `<main>`.
- `<nav aria-label="…">` around menus.
- Focus is visible (`:focus-visible { outline: 2px solid var(--brand-primary); outline-offset: 2px }`); never `outline: none` alone.
- Contrast of text on its background is at least 4.5:1 in **both** light and dark.
- Interactive targets are at least 44×44px on touch.
