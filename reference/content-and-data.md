# Content and data: blog, menus, media, forms, SEO

## The blog

A post has a title, a slug (its address under `/blog/`), an HTML body, one category, any number of tags, search metadata and a
language. It is a draft until `publish_post`.

**Order of work:** categories and tags first (they are per language), then posts.

Every category and tag should have a **picture** (upload it first, use the returned `url` as `image`) and search text: `/category` and
`/tag` list them as cards (picture, name, number of posts), and `/category/<slug>` is an indexed page whose title and description are the
`seoTitle` / `seoDescription`. One that has no picture is drawn with a plain placeholder, so a missing picture is never broken, only plain.

```
list_categories {locale: "en"}                       → reuse what exists
create_category {name: "Roofing", slug: "roofing", image: "/uploads/…", seoTitle: "…", seoDescription: "…"}   → ids come back
create_tag      {name: "Flat roof", slug: "flat-roof"}
update_category / update_tag {id, image, seoTitle, seoDescription}   → add or change the picture and search text later
create_post {
  title: "How to choose a flat roof membrane",
  slug: "choose-flat-roof-membrane",
  content: "<p>…</p><h2>…</h2>…",
  categoryId: "<id>", tagIds: ["<id>", "<id>"],
  seoTitle: "…(50-60 chars)", seoDescription: "…(150-160 chars)"
}
```

Writing a post:

- The **title is the `<h1>`** - do not put another `<h1>` in the body; start sections at `<h2>`, then `<h3>`.
- Open with the answer or the promise in the first two sentences; people and search engines both read them first.
- Short paragraphs, lists where the content is a list, one idea per heading. 600-1,500 words for a substantial article; shorter is fine
  when that is all there is to say.
- Images from the media library with alt text: `<img src="/uploads/…" alt="…" width="1200" height="630" loading="lazy">`.
- Link to other pages and posts with their real addresses; check they exist first (`list_pages`, `list_posts`).
- One **category** per post (broad: 3-8 across the site), several **tags** (specific). Do not create a tag used once.
- Slug: lower case, hyphens, words not dates, stable. Changing it later breaks links.
- Shortcodes work in the body (`[el-cta-button …]`), but plain HTML is usually enough.
- Write in the post's language and in the site's voice - read two existing posts first with `get_post`.
- `update_post` keeps what you do not send (it reads the post first). To replace the tag list send the **complete** `tagIds`.
- There is no tool to delete a post. If the user wants one removed, unpublish it (`publish_post` with `false`) and tell them to
  delete it in the panel.

## Menus

A menu is a tree of entries, one drop-down level deep. Each entry points at a page, a post, a category or a tag (by **id**), or is a
custom link (a `url`). It appears wherever `[gl-menu-<slug>]` is written, usually in the theme layout.

```
create_menu {name: "Main", slug: "main", language: "en", location: "header"}
set_menu_items {menuId, items: [
  {label: "Home",     type: "PAGE", targetId: "<home page id>"},
  {label: "Services", type: "PAGE", targetId: "<services id>", children: [
      {label: "Web design", type: "PAGE", targetId: "<web id>"},
      {label: "SEO",        type: "PAGE", targetId: "<seo id>"}]},
  {label: "Blog",     type: "CUSTOM", url: "/blog"},
  {label: "Contact",  type: "PAGE", targetId: "<contact id>"}
]}
```

- `set_menu_items` **replaces the whole menu**: send the complete outline every time (read it first with `get_menu`).
- Create the **pages first**, then the menu, so every entry has an id. A menu entry pointing at a page keeps working if the page's slug
  later changes; a custom `url` does not.
- Keep a main menu to 4-7 entries; put the rest in a footer menu (`slug: "footer"`).
- A menu with language `all` is shared by every language - only for entries whose labels are the same in each.
- The menu's `slug` is part of the shortcode: change it and every `[gl-menu-main]` that names it renders nothing.
- A menu for the customer panel: an entry pointing at a panel page must carry the `/dashboard/...` address as its `url`, because a
  `PAGE` entry resolves to the public address. The same menu in a public page is visible to visitors.

## Media

```
upload_media {sourceUrl: "https://…/photo.jpg", folder: "team", optimize: true}   → { url: "/uploads/…" }
create_upload_link {folder: "team"}   → { url } — for a file on the PERSON's computer: show them the link, they upload in their browser (10 min, 5 files), then list_media finds it
resize_media {url, percent: 50}   → { url: "/uploads/photo-50pct.png" } — a smaller copy beside the original; the original stays, so nothing that already points at it changes
set_media_alt {url, altText: "Ana Ruiz, founder, in the workshop"}
list_media {path: "team"}
```

- Upload **before** you write markup that uses the file, then use the **returned `url` exactly** (do not rebuild it).
- Alt text: what the image shows or what it is *for*, in the page's language, under about 125 characters, no "image of". Decorative
  images get `alt=""` in the markup.
- Choose sensible sizes (a hero rarely needs more than ~1600px wide); `optimize: true` lets the platform recompress.
- A file already in the library that is far bigger than it is drawn: `resize_media` with a `percent` (10-90) makes a smaller copy
  and answers its url - point the page at the copy. Pictures (jpg, png, webp, avif, gif) and clips (mp4, webm) only.
- Reuse what is in the library (`list_media`) before uploading a duplicate.
- Allowed: jpg, png, webp, gif, avif, svg, ico, mp4, webm, pdf and the fonts woff, woff2, ttf, otf - at most **10 MB** each, and **50 MB** for a
  video (mp4/webm). Anything else is refused.
- **Video**: use `<video controls muted playsinline preload="metadata" poster="…">`; a looping background clip also gets `autoplay loop`
  (browsers only autoplay when it is `muted`). Keep a clip short and compressed (H.264 mp4, ideally under ~10 MB), and give it a poster image.
- **Folders**: nobody needs to create a folder first - naming one in `folder` creates it, and the site's own root is already separate from
  every other site. Keep the library tidy with one folder per purpose and reuse the same names on every upload: `blog` (post images),
  `team` (people), `products`, `pages` (hero and section images), `icons` (svg/ico), `video`, `docs` (pdf). Lower case, one level deep.
  Never write into `logo` - the platform keeps its own files there. Look at `list_media` first: if the user already organised the library,
  follow their folders instead of adding yours.
- Only upload material the user gave you or that is clearly free to use. Say where an image came from if it is not theirs.

## Questions and answers

The owner writes and answers questions in **FAQ** in the panel; shortcodes draw them. You can see what a site has with
`list_global_shortcodes` (the codes) and `render_page` (what a page shows), but the answers themselves are panel work - tell the
user where to add them instead of writing an FAQ as markup.

```
[gl-faq group="general" title="Common questions"]        the answered questions, with the helpful buttons
[gl-faq-ask group="general" title="Ask us anything"]     the visitor's form: name, email, question, then a mailed confirmation
```

- A page with `[gl-faq]` carries `FAQPage` structured data for the answers it shows; a hand-written accordion of `[gl-el-details]`
  does not. Use the built-in when the questions are real and the owner keeps them up to date, and `[gl-el-details]` for copy that
  is part of the page itself.
- `group` is the list a question belongs to: `[gl-faq-ask group="pricing"]` puts the answer on the pricing page's list, so the two
  codes must agree.

## Forms and data tables

A **data table** (collection) is a named list of fields. A visitor fills in a form bound to it, the platform validates the submission
against the fields and stores an **entry**; the owner reads entries in the panel.

```
create_collection {
  name: "Contact", slug: "contact",
  fields: [
    {key: "name",    label: "Your name", type: "text",     required: true},
    {key: "email",   label: "Email",     type: "email",    required: true},
    {key: "topic",   label: "Topic",     type: "select",   options: ["Quote","Support","Other"]},
    {key: "message", label: "Message",   type: "textarea", required: true}
  ]
}
```

Field types: `text` `textarea` `number` `email` `tel` `url` `date` `time` `select` `radio` `checkbox` `file` `hidden`.
The field `key` is the form input's `name` and the key stored in each entry: lower case, no spaces, and **stable** (renaming a key
orphans the entries already stored; change only the `label`).

The form is plain HTML in a section, bound by `data-collection`:

```html
<form class="sk-contact__form" data-collection="contact" novalidate>
  <label>Your name <input name="name" autocomplete="name" required></label>
  <label>Email <input name="email" type="email" autocomplete="email" required></label>
  <label>Topic <select name="topic"><option>Quote</option><option>Support</option><option>Other</option></select></label>
  <label>Message <textarea name="message" rows="5" required></textarea></label>
  <button type="submit">Send</button>
  <p data-form-message role="status" aria-live="polite"></p>
</form>
```

- `data-collection` names the table's **slug**. Each input's `name` is a field `key`; an input with no matching field is ignored.
- `<p data-form-message>` receives the success or error text; give it `role="status"`. The form carries `data-state` of `sending`,
  `success` or `error` while it works - style those states.
- Put the form in a **section** (author markup), not in an `html`-typed parameter, which strips `<form>`.
- Label every input; mark required ones; use `autocomplete`; do not ask for more than you will use.
- **Tell the user** that the form stores personal data, that they should say in the page what happens with it (privacy line), and
  that where the entries are sent (a notification email, a webhook) is **theirs to set in the panel** - you cannot.
- You **cannot read the entries**: they are visitors' personal data and stay in the panel. If the user wants a summary, ask them to
  export it there. Never put a visitor's words or address on a page.

Showing a table's entries on a page (a testimonials list, an events table): `[gl-data-table table="events" columns="date,title" order="oldest" limit="10"]`
or the chart codes - see `shortcodes`.

## SEO, page by page

- **Title** (`seo.title`): 50-60 characters, the page's main topic first, brand last: "Flat roof installation in Leeds | Acme Roofing".
- **Description** (`seo.description`): 150-160 characters, a real sentence that makes someone click, no keyword stuffing.
- **One `<h1>`** per page, matching the title's meaning; headings in order.
- **Slug**: short, readable, stable.
- **Internal links** with descriptive text ("our flat roof service", not "click here").
- `indexable: false` only for pages that should not appear in search (thank-you pages, drafts you want reachable).
- Every image has alt text. The home page title says what the business is and where.
- Do not claim what the user did not tell you (prices, awards, guarantees, locations). Ask, or mark it as placeholder in your report.
- A page for signed-in customers is a **panel page** (`audience: "dashboard"`), not a public one: it opens at `/dashboard/<slug>` and
  is kept out of the sitemap. Build its menu with the `/dashboard/...` addresses written out.
