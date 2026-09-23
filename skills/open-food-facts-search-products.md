---
name: open-food-facts-search-products
description: Search the Open Food Facts product collection by text and facets, using either the v2 search endpoint or Search-a-licious.
api: Search-a-licious API
operations:
  - search_search_post
  - search_get_search_get
  - get-search
  - taxonomy_autocomplete_autocomplete_get
  - get-api-v3-taxonomy-canonicalize-tags
generated: '2026-09-17'
method: generated
source: openapi/open-food-facts-search-a-licious-openapi.json, openapi/open-food-facts-api-v2-openapi.yml
---

# Search products

## Pick the right surface

- **Search-a-licious** (`https://search.openfoodfacts.org`) — `search_search_post` (POST /search) or `search_get_search_get` (GET /search). This is the full-text and facet engine; it returns a `SuccessSearchResponse` with `count`, the page echo and facet buckets.
- **v2 search** (`https://world.openfoodfacts.org`) — `get-search` (GET /api/v2/search). Simpler tag-filter search against the product API.

Both are capped at 10 requests/minute per IP. Send the custom `User-Agent`.

## Steps

1. If the user gave you a category, label or brand in their own language, canonicalize it first with `get-api-v3-taxonomy-canonicalize-tags` (or complete it with `taxonomy_autocomplete_autocomplete_get`). Filters match canonical taxonomy ids such as `en:breakfast-cereals`, not free text.
2. Call the search operation with your query plus `page` and `page_size`.
3. Page with `page` / `page_size`; read `count` from the response envelope to know when to stop.
4. Ask for only the fields you need — a page of full product records is heavy.

## Errors

- `422` with a `{detail:[{loc,msg,type}]}` body means a parameter failed validation; `detail[].loc` names it.
- `503` means the rate ceiling. Back off.
