---
name: tokamak-energy-track-fusion-news
description: Monitor Tokamak Energy's public news archive for new announcements about ST40, Demo4, TE Magnetics, the STEP programme or funding, and return each item with its publication date and canonical link.
api: tokamak-energy:tokamak-energy-posts-api
operations:
  - listPosts
  - getPost
  - listCategories
generated: '2026-08-30'
method: generated
source: openapi/tokamak-energy-posts-api-openapi.yml, openapi/tokamak-energy-taxonomy-api-openapi.yml
---

# Track Tokamak Energy fusion news

Tokamak Energy publishes its announcements at `tokamakenergy.com/latest-news/`. The same archive is
readable as JSON, anonymously, with no key. Verified live at 84 posts on 2026-08-30.

Base URL: `https://tokamakenergy.com/wp-json`

## 1. Find what changed since you last looked

Use `listPosts` with the `modified_after` parameter and trim the payload with `_fields` — a full
posts page carries rendered HTML bodies and is tens of kilobytes; this is a few hundred.

```
GET /wp/v2/posts?modified_after=2026-08-01T00:00:00&per_page=100&_fields=id,date,modified,link,title,categories
```

Read `X-WP-Total` from the response headers before you decide to walk anything. If it is 0, stop —
nothing changed.

## 2. Page correctly

`per_page` maxes at 100 and `page` starts at 1. Do not increment `page` yourself: follow the
`Link` header's `rel="next"`, which this API emits per RFC 8288 and exposes to browsers via
`Access-Control-Expose-Headers`. When there is no `rel="next"`, you are done.

## 3. Get the full item

```
GET /wp/v2/posts/{id}?_embed
```

`_embed` inlines the author, featured media and category terms into `_embedded`, which collapses
three extra round trips into one. Without it you get bare numeric ids: `author`, `featured_media`
and `categories` are references, not values.

## 4. Filter by subject

```
GET /wp/v2/categories?per_page=100&_fields=id,name,slug,count
GET /wp/v2/posts?categories={id}&_fields=id,date,link,title
```

Eight categories were live on 2026-08-30. Do not filter on `tags` — the `post_tag` taxonomy is
registered on this site but has zero terms, so `GET /wp/v2/tags` returns an empty array and any
tag filter returns nothing.

## 5. Search instead, when you have a phrase not a category

```
GET /wp/v2/posts?search=ST40&_fields=id,date,link,title
```

## Rules

- **Identify items by `link` or `slug`, never by `id`.** Numeric ids are site-local and are not a
  durable identifier for a piece of subject matter.
- **`title.rendered`, `content.rendered` and `excerpt.rendered` contain HTML entities**, not plain
  text — `&amp;`, `&#8217;` and similar. Decode before you quote them.
- **Do not poll hard.** No rate limit is documented and no `RateLimit-*` or `Retry-After` header is
  returned, so you get no warning before you hit a ceiling. `robots.txt` on this host asks for
  `Crawl-delay: 2`; treat that as the pacing contract.
- **No caching validators.** No `ETag`, `Last-Modified` or `Cache-Control` is returned, so
  conditional requests are unavailable. Detect change by comparing `modified` timestamps, not by
  re-downloading bodies.
- **On error, branch on `code`, not `message`.** Errors are the flat WordPress envelope
  `{"code":…,"message":…,"data":{"status":…}}` — not RFC 9457 problem+json. `rest_post_invalid_id`
  means the id does not exist; `rest_forbidden` means the route is not anonymous and retrying will
  not help. See `errors/tokamak-energy-problem-types.yml`.
- **This is read-only.** Every operation here is a GET. Nothing you do can change anything, so there
  is nothing to undo.
