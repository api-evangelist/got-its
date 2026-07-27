---
name: Register and track a Reelables asset
description: Create an asset in a workspace, attach a Reelables smart label, then read its location, temperature and facility history.
api: openapi/got-its-openapi-original.yml
base_url: https://api.reelables.com/v1
operations:
  - POST /workspaces/{workspaceId}/assets
  - POST /assets/{assetId}/labels
  - GET /assets/{assetId}/locations
  - GET /assets/{assetId}/temperatures
  - GET /assets/{assetId}/facilities
---

# Register and track a Reelables asset

Use this skill to onboard a physical item into the Reelables platform and follow it through the supply chain.

## Authentication

The Reelables API uses OAuth 2.0 **client credentials**. Provision a `CLIENT_ID`/`CLIENT_SECRET` from Reelables support, then:

1. `POST https://auth.reelables.com/oauth2/token?grant_type=client_credentials` with header `Authorization: Basic base64(CLIENT_ID:CLIENT_SECRET)` and `Content-Type: application/x-www-form-urlencoded`.
2. Use the returned token as `Authorization: Bearer ACCESS_TOKEN` on every request. Tokens expire after 1 hour — re-authenticate on 401.

Optionally send a `request-id` header (UUID) on each call for tracing.

## Steps

1. **Create the asset** — `POST /workspaces/{workspaceId}/assets`. Choose the `workspaceId` you have permission in (list them with `GET /workspaces`). Capture the returned asset id.
2. **Attach a label** — `POST /assets/{assetId}/labels` with the label's `bleId`. This binds a physical Reelables smart label to the asset so its signals are attributed correctly.
3. **Read location history** — `GET /assets/{assetId}/locations`. Pass `summaryLocations=true` for merged locations, or omit for raw. Use `limit` + `nextToken` to page.
4. **Read temperature history** — `GET /assets/{assetId}/temperatures` (cellular labels report temperature).
5. **Read facility history** — `GET /assets/{assetId}/facilities` to see which facilities the asset has passed through.

## Conventions and error handling

- **Pagination:** list responses return `{ items: [...], nextToken }`. Pass `nextToken` back as a query param to fetch the next page; stop when it is absent.
- **Errors:** failures return a JSON `errors[]` array with `code`, `id`, `title`, `detail`, `status`. Handle `401` (re-auth), `403` (permission), `404` (unknown id), `429` (back off — no Retry-After header is provided).
- Do not assume idempotent retries; the API documents no idempotency key.
