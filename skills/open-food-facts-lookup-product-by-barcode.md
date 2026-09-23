---
name: open-food-facts-lookup-product-by-barcode
description: Look up a food product in Open Food Facts by its GS1 barcode and read only the fields you need.
api: Open Food Facts API v3
operations:
  - get-api-v3-product-code
  - get-product-by-code
generated: '2026-09-17'
method: generated
source: openapi/open-food-facts-api-v3-openapi.yml, openapi/open-food-facts-api-v2-openapi.yml
---

# Look up a product by barcode

## Before you call

- Send a custom `User-Agent` in the form `AppName/Version (ContactEmail)`. It is not optional — unidentified traffic gets blocked.
- No API key exists. Reads are anonymous.
- Stay under 15 requests/minute from one IP for product reads. There are no `RateLimit-*` headers to read, so pace yourself.

## Steps

1. Normalize the barcode. Open Food Facts canonicalizes short codes and leading zeros to EAN-13; a raw 8-digit code may not match as typed. See the barcode-normalization reference before deciding a product is absent.
2. Call `get-api-v3-product-code` — `GET /api/v3/product/{code}` on `https://world.openfoodfacts.org`.
3. Pass `fields` with only the facets you need (for example `product_name,brands,nutriscore_grade,nova_group,ingredients_text`). A full product record composes fifteen facet schemas and can be very large.
4. Read the v3 envelope: `errors[]` entries fail the request, `warnings[]` do not.

## Handling the answer

- `404` means the product is not in the database — from product schema 996 onward this is a real 404, not a 200 with `status: 0`. Re-check normalization before reporting it missing.
- `503` means you hit the per-IP or global rate ceiling. No `Retry-After` is published; back off and retry.
- On v2 (`get-product-by-code`) the envelope is `{code, status, status_verbose}` instead, and `status_verbose: "no code or invalid code"` means the barcode itself was rejected.

## Attribution

Data is ODbL, contents DbCL, images CC BY-SA. If you surface this data to users, attribute Open Food Facts.
