---
name: tokamak-energy-harvest-media
description: Locate and retrieve Tokamak Energy press imagery, technical diagrams and documents — ST40, Demo4 and TE Magnetics assets — from the public media library, with alt text, captions and the correct size variant.
api: tokamak-energy:tokamak-energy-media-api
operations:
  - listMedia
  - getMediaItem
  - listPosts
generated: '2026-08-30'
method: generated
source: openapi/tokamak-energy-media-api-openapi.yml, openapi/tokamak-energy-posts-api-openapi.yml
---

# Harvest Tokamak Energy media assets

The media library behind tokamakenergy.com is anonymously readable as JSON. Verified live at 853
items on 2026-08-30.

Base URL: `https://tokamakenergy.com/wp-json`

## 1. Narrow before you download

853 items at the 100-per-page maximum is nine round trips of mostly metadata you do not need.
Filter and trim first.

```
GET /wp/v2/media?search=ST40&media_type=image&per_page=100&_fields=id,date,source_url,alt_text,title,media_details
```

`media_type` accepts `image`, `video`, `text`, `application`, `audio` and `file`. Use
`mime_type=application/pdf` to isolate documents.

## 2. Take the right size

`media_details.sizes` carries the generated variants, each with its own `source_url`, `width` and
`height`. `source_url` at the top level is the **original upload**, which for press photography can
be several megabytes. Pick a named size unless you actually need the original.

## 3. Get an asset's context

```
GET /wp/v2/media/{id}?_fields=id,source_url,alt_text,caption,post,date
```

`post` is the id of the news item or page the asset was uploaded to — resolve it at
`/wp/v2/posts/{id}` for the story the image belongs to. It is `null` for library-only assets.

## 4. Or work the other way, from a story to its image

```
GET /wp/v2/posts/{id}?_embed&_fields=id,title,link,featured_media,_links
```

The featured image arrives inline under `_embedded["wp:featuredmedia"]`.

## Rules

- **Attribution and reuse are not granted by this API.** The media library being readable is a
  consequence of how the site is built. It is not a licence. Tokamak Energy publishes no image
  licence terms; check `https://tokamakenergy.com/contact-us/` before republishing anything.
- **`alt_text` is frequently empty.** Do not treat it as a reliable description; fall back to
  `title.rendered` and to the caption on the owning post.
- **`caption.rendered` is HTML**, not plain text.
- **Upload, edit, sideload, finalize and post-process routes exist and require authentication.**
  They are not part of this skill and will return `rest_forbidden` to an anonymous caller.
- **Read-only, so nothing here is reversible or needs to be.**
