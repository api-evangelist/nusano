---
name: nusano-read-newsroom
description: Pull Nusano's news releases, event notices and Nu Blog posts from nusano.com as JSON, filter them by category, date or tag, and resolve author and featured image.
api: nusano:posts
operations:
  - getPosts
  - getPostsById
  - getCategories
  - getTags
  - getUsers
  - getMediaById
generated: '2026-08-26'
method: generated
source: openapi/nusano-posts-api-openapi.yml, openapi/nusano-categories-api-openapi.yml, openapi/nusano-tags-api-openapi.yml, openapi/nusano-users-api-openapi.yml, openapi/nusano-media-api-openapi.yml, conventions/nusano-conventions.yml
---

# Read the Nusano newsroom

Nusano is a radioisotope manufacturer, not a software vendor. It ships no product API. The only
callable surface on `nusano.com` is the WordPress content REST API, which serves the company
newsroom — press releases, event notices and Nu Blog entries — as JSON. Use it to track milestones
(the "Beam on Target" LINAC milestone, the West Valley City facility, isotope supply agreements)
without scraping HTML.

Base URL: `https://nusano.com/wp-json`. No credentials. Read is anonymous; every write is
`401 rest_forbidden`.

## Steps

1. **List recent posts.** `getPosts` — `GET /wp/v2/posts?per_page=20&orderby=date&order=desc`.
   Trim the payload with the API's own sparse-fieldset parameter, because the full record embeds
   rendered HTML for the whole article:
   `&_fields=id,date,slug,link,title,excerpt,categories,tags,author,featured_media`.
   109 posts were readable on 2026-08-26.

2. **Page through.** Pagination is page-number style. Read `X-WP-Total` and `X-WP-TotalPages` from
   the response headers rather than counting, and follow `Link: <...>; rel="next"` until it is
   absent. `per_page` is capped at 100.

3. **Filter to a beat.** Call `getCategories` (`GET /wp/v2/categories`) once and cache the id→slug
   map — 14 categories, including `blog`, `awards-honors` and `community`. Then filter with
   `GET /wp/v2/posts?categories=<id>`. For finer slicing call `getTags` (310 tags) and use
   `?tags=<id>`. Date windows use `after=` and `before=` with ISO 8601 values.

4. **Read one post in full.** `getPostsById` — `GET /wp/v2/posts/{id}`. `title.rendered`,
   `excerpt.rendered` and `content.rendered` all contain HTML entities and markup; decode before
   using the text.

5. **Resolve the relations in one shot.** Append `&_embed` to any of the calls above and the
   response carries `_embedded` with the author record and the featured media object inline,
   instead of making you follow `author` → `getUsers` and `featured_media` → `getMediaById`
   separately.

## Rules

- **Read-only.** `createPosts`, `updatePostsById`, `deletePostsById` and every other write in the
  spec require a WordPress Application Password that only Nusano can issue. Do not attempt them.
- **No idempotency, no rate-limit signal.** Nusano publishes neither. No `X-RateLimit-*` or
  `Retry-After` header is returned, so back off on your own budget and do not hammer the origin.
- **Errors are not RFC 9457.** Failures return `{"code": "...", "message": "...", "data": {"status": N}}`
  with `content-type: application/json`. Branch on `code`, not on the message string. See
  `errors/nusano-problem-types.yml`.
- **This is a CMS, not a product API.** Nothing here exposes isotope inventory, production schedules
  or supply-agreement data. Commercial enquiries go through
  <https://nusano.com/company/contact-us/contact-sales/>.
