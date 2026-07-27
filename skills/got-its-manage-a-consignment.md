---
name: Create and monitor a consignment (shipment)
description: Create a consignment in a workspace, add pieces (assets) and gateways, then monitor and update it through delivery.
api: openapi/got-its-openapi-original.yml
base_url: https://api.reelables.com/v1
operations:
  - POST /workspaces/{workspaceId}/consignments
  - POST /consignments/{consignmentId}/pieces
  - POST /consignments/{consignmentId}/gateways
  - GET /consignments/{consignmentId}
  - PATCH /workspaces/{workspaceId}/consignments
---

# Create and monitor a consignment (shipment)

Use this skill to track a shipment as a Reelables consignment — a collection of pieces (assets) followed together, optionally with mobile gateways in the trailer.

## Authentication

OAuth 2.0 client credentials — obtain a bearer token from `https://auth.reelables.com/oauth2/token` (see the asset-tracking skill) and send `Authorization: Bearer ACCESS_TOKEN`. Tokens last 1 hour.

## Steps

1. **Create the consignment** — `POST /workspaces/{workspaceId}/consignments`. Capture the returned `consignmentId`. (You can also correlate later by tracking reference with `PATCH /workspaces/{workspaceId}/consignments` using `trackingRef`.)
2. **Add pieces** — `POST /consignments/{consignmentId}/pieces`. Each piece is an existing asset; add every asset that ships together. Remove one with `DELETE /consignments/{consignmentId}/pieces/{pieceId}`.
3. **Add gateways (optional)** — `POST /consignments/{consignmentId}/gateways` to attach mobile gateways that provide cellular connectivity in transit.
4. **Monitor** — `GET /consignments/{consignmentId}` for the current state; `GET /consignments/{consignmentId}/pieces` and `.../gateways` to enumerate contents.
5. **Update by tracking reference** — `PATCH /workspaces/{workspaceId}/consignments` with a `trackingRef` to update a consignment discovered by its external reference.

## Conventions and error handling

- **Pagination:** `{ items, nextToken }`; page with `limit` + `nextToken`.
- **Errors:** JSON `errors[]` with `code`/`id`/`title`/`detail`/`status`. Handle `401`/`403`/`404`/`429` as in the asset-tracking skill.
- Send an optional `request-id` (UUID) header for traceability.
