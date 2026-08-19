---
name: Sync a Braze catalog for personalization
description: Create a catalog, load and maintain its items through the synchronous or asynchronous endpoints, and know which one to use for which batch size.
api: openapi/braze-catalogs-catalog-management-synchronous-api-openapi.yml, openapi/braze-catalogs-catalog-items-synchronous-api-openapi.yml, openapi/braze-catalogs-catalog-items-asynchronous-api-openapi.yml
generated: '2026-08-13'
method: generated
source: openapi/ + https://www.braze.com/docs/api/api_limits/ + conventions/braze-conventions.yml
operations:
  - GET /catalogs
  - POST /catalogs
  - DELETE /catalogs/{catalog_name}
  - GET /catalogs/{catalog_name}/items
  - GET /catalogs/{catalog_name}/items/{item_id}
  - POST /catalogs/{catalog_name}/items/{item_id}
  - PUT /catalogs/{catalog_name}/items/{item_id}
  - PATCH /catalogs/{catalog_name}/items/{item_id}
  - DELETE /catalogs/{catalog_name}/items/{item_id}
  - POST /catalogs/{catalog_name}/items
  - PUT /catalogs/{catalog_name}/items
  - PATCH /catalogs/{catalog_name}/items
  - DELETE /catalogs/{catalog_name}/items
---

# Sync a Braze catalog for personalization

A catalog is the reference data Liquid pulls from at send time — products, offers, locations, store
hours. Catalog storage is capped at **500 MB** per workspace (raised in the 2026-07-23 release).

## The one decision that matters: sync or async

Braze exposes the **same paths** twice, with different semantics and wildly different limits.

| | Item-level (`/items/{item_id}`) | Bulk (`/items`) |
|---|---|---|
| Semantics | synchronous — applied inline | asynchronous — accepted and processed in the background |
| Batch | one item | many items per call |
| Rate limit | 50/minute (shared with catalog management) | 16,000/minute (shared) |

**The distinction is not visible in a response field.** It lives in the path shape and in the API
titles. An agent must know which one it called; do not infer completion from a 200 on the bulk path —
that 200 means *accepted*, exactly as it does everywhere else in this API.

## Step 1 — create the catalog

`POST /catalogs` with the field names and types. `GET /catalogs` lists what exists;
`DELETE /catalogs/{catalog_name}` removes one.

Catalog management calls share a **50 requests/minute** pool with the synchronous item endpoints —
this is the tightest limit you will meet in a sync job.

## Step 2 — load items

Initial load and any large update: use the bulk path.

- `POST /catalogs/{catalog_name}/items` — create many
- `PUT /catalogs/{catalog_name}/items` — replace many
- `PATCH /catalogs/{catalog_name}/items` — edit many
- `DELETE /catalogs/{catalog_name}/items` — delete many

Single-record corrections: use the item path with `{item_id}` (`POST`/`PUT`/`PATCH`/`DELETE`).

## Step 3 — read back

- `GET /catalogs/{catalog_name}/items` lists items.
- `GET /catalogs/{catalog_name}/items/{item_id}` reads one.

Because bulk writes are asynchronous, a read immediately after a write can return stale data. Poll
the item, or wait — Braze recommends a 5-minute gap between consecutive calls to an endpoint for
consistency.

## Step 4 — retries

There is no idempotency key on any Braze endpoint. A bulk `POST` that times out may or may not have
been accepted. Prefer `PUT` (replace) over `POST` (create) when replaying a batch, because replace is
naturally idempotent in effect even though the API offers no guarantee.

## Agent note

`get_catalogs`, `get_catalog_items` and `get_catalog_item` are exposed as read tools on the Braze MCP
server (`mcp/braze-mcp.yml`). **No catalog write tool exists on MCP** — every write above is REST-only.
