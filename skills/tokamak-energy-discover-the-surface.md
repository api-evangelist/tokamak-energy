---
name: tokamak-energy-discover-the-surface
description: Enumerate everything the Tokamak Energy content API exposes — namespaces, routes, content types, taxonomies and their parameters — starting from a single request, so an agent never has to guess a path.
api: tokamak-energy:tokamak-energy-discovery-api
operations:
  - getApiIndex
  - listTypes
  - getType
  - listTaxonomies
  - getTaxonomy
  - listStatuses
generated: '2026-08-30'
method: generated
source: openapi/tokamak-energy-discovery-api-openapi.yml
---

# Discover the Tokamak Energy API surface

This API describes itself. You never need to guess a route, a parameter name or a constraint —
ask the host.

Base URL: `https://tokamakenergy.com/wp-json`

## 1. Start at the root

```
GET /
```

Returns the site name and description, the registered namespaces, the **complete route table with
per-route HTTP methods and per-argument descriptors** (type, description, enum, default, minimum,
maximum), and the supported authentication methods. Every OpenAPI document in this repository was
derived from exactly this response.

Namespaces live on 2026-08-30: `wp/v2`, `oembed/1.0`, `yoast/v1`, `duplicate-post/v1`,
`wp-site-health/v1`, `wp-block-editor/v1`, `wp-abilities/v1`.

## 2. Find the content types, including the site-specific ones

```
GET /wp/v2/types
```

Read `rest_base` off each type — that is the path segment, and it is not always the slug. Alongside
WordPress core types this site registers two of its own: `portfolio` and `area-item`.

## 3. Find the taxonomies and what they apply to

```
GET /wp/v2/taxonomies
```

`types` on each taxonomy names the content types it classifies. `rest_base` gives you its path.

## 4. Read a route's real constraints before you call it

The argument descriptors in the root document are authoritative for this host. `per_page` has a
maximum of 100 and a default of 10; `page` has a minimum of 1; `context` is one of `view`, `embed`,
`edit`. Violating a declared constraint returns `400 rest_invalid_param`.

## Rules

- **Registered does not mean populated.** `portfolio`, `portfolio_category`, `testimonial_category`
  and `post_tag` are all registered and all return HTTP 200 with `X-WP-Total: 0`. Always read
  `X-WP-Total` before concluding a collection has content.
- **Namespaces beyond `wp/v2` and `oembed/1.0` are not for you.** `yoast/v1`, `duplicate-post/v1`,
  `wp-site-health/v1`, `wp-block-editor/v1` and `wp-abilities/v1` are operator tooling; they either
  require authentication or exist to serve the site's own admin. `GET /wp-abilities/v1/abilities`
  returns `401 rest_forbidden` anonymously.
- **`/wp/v2/settings` returns 401 and `/wp/v2/comments` returns 403.** Both are permanent for an
  anonymous caller. Do not retry them.
- **The site-specific types are the fragile part of this contract.** Core `wp/v2` routes have been
  stable since 2016; `area-item` can be renamed or unregistered by a theme change with no notice.
