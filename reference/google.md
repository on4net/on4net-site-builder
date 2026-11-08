# Google: Analytics, Search Console and Business Profile

Everything here is for a site that has **its own verified domain** (`get_site` answers it as `domain`; it is `null` while the site
only has a subdomain of the installation). Until then every Google tool is refused and says why. All of them need a key with the
**Google** scope - the owner grants it on purpose, so if a call is refused with `403`, tell the owner instead of retrying.

## Look before you act

1. `get_google_status` - `domain`, `available` (and `reason` when it is not), whether the **Analytics** tag and the **Search Console**
   verification tag are in the head, and whether a **Business Profile** location is chosen. No secret is ever in it.
2. If `available` is false, stop: the owner has to add the domain in *Settings, Addresses* and verify it first.
3. The owner must also have **connected Google** in *Settings, Google* **after** the domain was verified, so the Search Console and
   Analytics permissions were granted. A refusal that says "Connect again" means exactly that - say so; do not retry.

## Analytics

- `setup_analytics` - finds the GA4 property that already measures this domain (www counts as the same site) or creates a property
  and a web stream, and stores the measurement id as the page's tag. Pressing it again reuses what exists.
- `analytics_report` - users, sessions and views over the last days, grouped by `pages` (default), `sources`, `countries`,
  `devices` or `days`.
- `set_google_tags` writes the `G-…` id by hand when the owner already has one; send only what changes, `""` removes it.

## Search Console

- `setup_search_console` - one call: Google gives a verification tag, the page serves it, Google checks it, the property
  (`https://<domain>/`) is added and the sitemap is submitted. The answer lists each step; a step that failed stops the rest.
- `search_console_queries` - clicks, impressions, click rate and average position by `query` (default), `page`, `country`,
  `device` or `date`. The data lags about two days, and the site has to be in Search Console first.

## Business Profile

- `business_reviews` - the newest reviews of the location the owner chose, with the average and the total.
- `reply_to_review` and `create_business_post` are **public**: customers read them under the business's name. Write the text, **show
  it to the owner and wait for their agreement**, then call. A reply is at most 4096 characters, a post 1500, in the language of the
  review or of the audience.
- There is no tool to change opening hours or the description: a wrong value there is seen by every customer.

## What to tell the owner

Say which of the three are connected, what each shows now, and - when something is missing - the one step they must take (verify the
domain, connect Google again, or choose a Business Profile location). Do not paste numbers you did not read.
