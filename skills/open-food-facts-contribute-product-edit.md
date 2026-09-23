---
name: open-food-facts-contribute-product-edit
description: Write to Open Food Facts — create or update a product, upload an image, and undo a bad edit — with the account auth, staging rehearsal and revert path the project documents.
api: Open Food Facts API v3
operations:
  - patch-api-v3-product-code
  - post-api-v3-product-code-images
  - delete-api-v3-product-code-images-uploaded-imgid
  - post-api-v3-product_revert
  - get-cgi-session.pl
  - post-cgi-product_jqm2.pl
generated: '2026-09-17'
method: generated
source: openapi/open-food-facts-api-v3-openapi.yml, openapi/open-food-facts-api-v2-openapi.yml, https://openfoodfacts.github.io/openfoodfacts-server/api/
---

# Contribute a product edit

Writes change a public, shared, crowdsourced database. Treat every call here as consequential.

## Authenticate

1. Get a session with `get-cgi-session.pl` — `POST /cgi/session.pl`. The session cookie is preferred, is IP-restricted, and a user may hold at most 10 sessions.
2. Alternatively pass `user_id` and `password` as request parameters. `user_id` is the account **username**, never the email address.
3. Include `app_name`, `app_version` and `app_uuid` on write requests so the edit is attributable.

## Rehearse first

Run the whole flow against the staging deployment at `https://world.openfoodfacts.net` before touching production. It mirrors the API and sits behind HTTP basic auth specifically so staging content is not indexed. There is no dry-run parameter on any write operation — staging is the rehearsal mechanism.

## Write

- `patch-api-v3-product-code` — `PATCH /api/v3/product/{code}` creates or updates a product on v3.
- `post-api-v3-product-code-images` — `POST /api/v3/product/{code}/images` uploads an image.
- On v2 the equivalent is `post-cgi-product_jqm2.pl`; `code`, `user_id` and `password` are all required.

## Know what you can take back

There is **no idempotency key** on this API. A retried write re-applies the same values and creates another product revision; it is not deduplicated. Retry only when you know the first call did not land.

Reversal paths that do exist, none of them with a published time window:

- `post-api-v3-product_revert` — `POST /api/v3/product_revert` reverts a product to a previous revision. This is the undo for a bad edit.
- `delete-api-v3-product-code-images-uploaded-imgid` removes an image you uploaded.
- `post-cgi-product_image_unselect.pl` unselects an image on v2.

Because no window is published, do not promise a user that an edit will stay revertible for any particular period.

## Errors

- `401` — missing or invalid session/credentials.
- `403` — the account is not permitted to make this edit.
- `302` — the legacy CGI write endpoints redirect instead of returning JSON. Prefer the v3 operations.
- `503` — rate limited.
