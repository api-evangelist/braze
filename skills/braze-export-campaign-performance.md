---
name: Export Braze campaign and Canvas performance
description: Pull the campaign/Canvas inventory and their analytics time series out of Braze, paginating correctly across the two different pagination idioms in the API.
api: openapi/braze-export-campaign-api-openapi.yml, openapi/braze-export-canvas-api-openapi.yml, openapi/braze-export-kpi-api-openapi.yml
generated: '2026-08-13'
method: generated
source: openapi/ + https://www.braze.com/docs/api/api_limits/ + conventions/braze-conventions.yml
operations:
  - GET /campaigns/list
  - GET /campaigns/details
  - GET /campaigns/data_series
  - GET /canvas/list
  - GET /canvas/details
  - GET /canvas/data_series
  - GET /canvas/data_summary
  - GET /sends/data_series
  - GET /kpi/dau/data_series
  - GET /kpi/mau/data_series
note: >-
  These are exactly the operations the Braze MCP server exposes as read tools
  (get_campaign_list, get_canvas_data_series, …). If an MCP client is available, prefer it —
  see mcp/braze-tool-crosswalk.yml. This skill is the REST path.
---

# Export Braze campaign and Canvas performance

## Step 1 — enumerate

`GET /campaigns/list?page=0` and `GET /canvas/list?page=0`

- **Zero-indexed `page`, up to 100 items per page.** There is no total count and no `Link` header.
  You are done when the array comes back empty.
- `include_archived` controls whether archived assets appear.
- Use `last_edit.time[gt]` to fetch only what changed since your last sync.

The other pagination idiom in this API is `limit`/`offset` (used by `/templates/email/list`,
`/content_blocks/list`, `/email/unsubscribes`, `/subscription/user/status`,
`/sms/invalid_phone_numbers`). Do not assume one style across endpoints.

## Step 2 — resolve metadata

`GET /campaigns/details?campaign_id=...` and `GET /canvas/details?canvas_id=...`

These return the variants and steps you need to interpret the series in step 3
(`message_variation_id` per campaign variant, step ids per Canvas).

## Step 3 — pull the series

`GET /campaigns/data_series` and `GET /canvas/data_series` page by **time**, not by record:

- `length` — how many units back to return
- `ending_at` — the end of the window (ISO 8601)
- `unit` — day or hour where supported
- `include_variant_breakdown`, `include_step_breakdown`, `include_deleted_step_data` widen the shape

`GET /canvas/data_summary` gives the rollup for a window instead of the series.

## Step 4 — budget your calls against the shared pool

- `/campaigns/data_series` has its own generous limit (50,000/minute).
- Almost everything else here draws on a **shared 250,000 requests/hour per workspace** pool, which
  the export, messaging, template and subscription endpoints all share. A wide backfill can starve
  your production sends out of the same bucket.
- Braze recommends a **5-minute gap** between consecutive calls to the same endpoint for data
  consistency — analytics are not read-your-writes.

Read `X-RateLimit-Remaining` on every response and stop before you hit zero; on **429** sleep until
`X-RateLimit-Reset` (UTC epoch seconds; the window resets on the clock hour).

## Step 5 — join

`campaign_id` → `message_variation_id` → `send_id` is the join path from a campaign to its sends
(`GET /sends/data_series`). See `data-model/braze-data-model.yml` for the full entity graph.
