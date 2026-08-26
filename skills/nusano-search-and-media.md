---
name: nusano-search-and-media
description: Search everything published on nusano.com and pull matching assets out of the Nusano media library — facility photography, logos and media-kit files — with real URLs and MIME types.
api: nusano:search
operations:
  - getSearch
  - getPages
  - getPagesById
  - getMedia
  - getMediaById
  - getTypes
  - getTaxonomies
generated: '2026-08-26'
method: generated
source: openapi/nusano-search-api-openapi.yml, openapi/nusano-pages-api-openapi.yml, openapi/nusano-media-api-openapi.yml, openapi/nusano-discovery-api-openapi.yml, conventions/nusano-conventions.yml
---

# Search Nusano and fetch its media

`nusano.com` exposes a site-wide search projection and a 1,127-item media library through the
WordPress content REST API. Use this when you need to answer a question about Nusano's technology,
HALEU knowledge center or facility from the company's own words, or to fetch a real asset URL
instead of guessing one.

Base URL: `https://nusano.com/wp-json`. Anonymous. No key.

## Steps

1. **Search.** `getSearch` — `GET /wp/v2/search?search=<terms>&per_page=20`. It returns a light
   projection: `id`, `title`, `url`, `type`, `subtype`. 156 objects were searchable on 2026-08-26.
   Narrow with `&subtype=post` or `&subtype=page`.

2. **Fetch the hit.** The result's `type`/`subtype` tells you which collection to call. A `page`
   subtype goes to `getPagesById` (`GET /wp/v2/pages/{id}`); a `post` subtype goes to
   `getPostsById`. Read `content.rendered` and strip the markup.

3. **Walk the page tree instead of searching.** `getPages` — `GET /wp/v2/pages?per_page=100&_fields=id,slug,link,title,parent,menu_order`
   returns all 47 pages, and `parent` gives you the hierarchy (Company, Technology, Life Sciences,
   HALEU Knowledge Center, Careers). This is the reliable way to enumerate the site.

4. **Find assets.** `getMedia` — `GET /wp/v2/media?search=<terms>&per_page=20&_fields=id,slug,link,title,media_type,mime_type,source_url,alt_text`.
   `source_url` is the direct, fetchable file URL; `mime_type` tells you whether it is an image,
   a PDF or a video before you download it. `getMediaById` returns one asset with its full size
   ladder in `media_details.sizes`.

5. **Ask the API what it holds.** `getTypes` (`GET /wp/v2/types`) and `getTaxonomies`
   (`GET /wp/v2/taxonomies`) are the self-describing layer — call them before assuming a collection
   exists, because the registered types on this site are set by the CMS configuration and change
   when Nusano changes its plugins.

## Rules

- **Respect the media licence.** `source_url` being fetchable is not permission to republish.
  Nusano publishes a media kit at <https://nusano.com/news/nusano-media-kit/>; use it for brand
  assets and read <https://nusano.com/terms-of-service/> before redistributing anything.
- **Do not attempt uploads.** `createMedia` is in the spec because the route index declares it, but
  it is authentication-gated and returns `401 rest_forbidden` anonymously.
- **The MCP endpoint will not help you.** `https://nusano.com/wp-json/mcp/mcp-adapter-default-server`
  is live but returns `401 rest_forbidden` to an anonymous `tools/list`. Use the REST calls above.
