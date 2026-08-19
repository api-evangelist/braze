---
name: Track a user and trigger an API campaign in Braze
description: Write user attributes and a custom event to a Braze profile, then fire an API-triggered campaign or Canvas for that user, safely and without duplicating sends.
api: openapi/braze-user-data-api-openapi.yml, openapi/braze-messaging-send-messages-api-openapi.yml
generated: '2026-08-13'
method: generated
source: openapi/ + https://www.braze.com/docs/api/basics/ + conventions/braze-conventions.yml
operations:
  - POST /users/track
  - POST /campaigns/trigger/send
  - POST /canvas/trigger/send
  - POST /sends/id/create
note: >-
  The harvested Braze specs declare no operationId on any of their 95 operations, so every
  step below references METHOD + path, which is what the contract actually provides.
---

# Track a user and trigger an API campaign

## Before you start

- **Base URL is per-workspace.** Braze runs one REST host per instance
  (`https://rest.iad-01.braze.com`, `https://rest.fra-01.braze.eu`, `https://rest.au-01.braze.com`, …).
  A valid key against the wrong host returns **401**. The instance is configuration; there is no
  discovery endpoint. See `conventions/braze-conventions.yml` for the full host list.
- **Auth** is `Authorization: Bearer <REST API key>`. The key carries per-endpoint permissions set
  in the dashboard — a missing permission is a **401/403**, not a scope error.
- **There is no idempotency key.** Braze publishes none. Retrying a step below after a timeout can
  double-log an event or double-send a message. Treat every write as at-most-once from your side:
  record that you sent it before you retry.

## Step 1 — write the profile and the event

`POST /users/track`

- One call carries `attributes[]`, `events[]` and `purchases[]`.
- **Hard cap: 75 objects combined** across all three arrays. Over that you get
  `400 Max Input Length Exceeded`.
- Payload limit 4 MB (2 MB on the bulk variant); over it you get **413**.
- Address the user by `external_id`, by `user_alias` (`alias_label` + `alias_name`), or by
  `braze_id`. Max 50 identifiers per request.

Rate limit: `/users/track` is contract-dependent with a burst of 3,000 requests per 3 seconds.

## Step 2 — check what "success" actually meant

Braze returns **200** with:

```json
{"message": "success"}
```

or, when some records were partially bad:

```json
{"message": "success", "errors": ["<minor error message>"]}
```

**`"success"` means accepted and queued — not applied, and definitely not delivered.** Always read
`errors[]` even on a 200. Errors are prose strings; there is no stable machine code and no RFC 9457
`problem+json`. See `errors/braze-problem-types.yml`.

## Step 3 — mint a send id if you need to attribute the send later

`POST /sends/id/create`

- Returns a `send_id` you can pass to the trigger call, so the resulting analytics roll up under an
  identifier you chose.
- Rate limit: **100 per day**. This is the tightest limit in the whole API — do not mint one per
  message. Mint one per campaign run.

## Step 4 — trigger the message

For a campaign: `POST /campaigns/trigger/send` with `campaign_id`, optional `send_id`,
`trigger_properties`, and recipients.

For a journey: `POST /canvas/trigger/send` with `canvas_id` and `canvas_entry_properties`.

- Max **50** specific `external_ids` per call; beyond that, target a segment or a connected audience.
- Broadcast sends are capped at **250 requests/minute**; non-broadcast shares the 250,000/hour
  workspace pool.
- A campaign id with no variant returns `400 Message Variant Unspecified`; a variant that does not
  belong to that campaign returns `400 Invalid Message Variant`.

## Step 5 — handle throttling

Every response carries `X-RateLimit-Limit`, `X-RateLimit-Remaining` and `X-RateLimit-Reset`
(UTC epoch seconds). On **429**, sleep until `X-RateLimit-Reset` — windows reset on the clock hour,
not on a rolling window, so exponential backoff alone will keep hitting the wall. `Retry-After` is
not documented; do not depend on it.

On **5xx**, retry with exponential backoff — but see the idempotency warning above before retrying
step 1 or step 4.
