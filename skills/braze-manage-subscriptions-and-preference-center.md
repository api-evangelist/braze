---
name: Manage Braze subscriptions and the preference center
description: Read and write a user's email, SMS and WhatsApp subscription state, keep the suppression lists clean, and hand a user a hosted preference center URL.
api: openapi/braze-subscription-groups-sms-and-whatsapp-api-openapi.yml, openapi/braze-email-lists-addresses-api-openapi.yml, openapi/braze-preference-center-api-openapi.yml, openapi/braze-sms-api-openapi.yml
generated: '2026-08-13'
method: generated
source: openapi/ + conventions/braze-conventions.yml + errors/braze-problem-types.yml
operations:
  - GET /subscription/status/get
  - GET /subscription/user/status
  - POST /v2/subscription/status/set
  - POST /email/status
  - GET /email/unsubscribes
  - GET /email/hard_bounces
  - POST /email/bounce/remove
  - POST /email/blocklist
  - GET /sms/invalid_phone_numbers
  - POST /sms/invalid_phone_numbers/remove
  - GET /preference_center/v1/list
  - GET /preference_center_v1/{PreferenceCenterExternalID}/url/{UserID}
---

# Manage Braze subscriptions and the preference center

This flow touches consent state. Every write here is legally consequential — get it wrong and you
either message someone who opted out or silence someone who did not.

## Step 1 — read current state

- `GET /subscription/status/get` — status for a given subscription group.
- `GET /subscription/user/status` — every subscription group for a user, paginated with
  `limit`/`offset`.

## Step 2 — write subscription group state

**Use `POST /v2/subscription/status/set`, not `POST /subscription/status/set`.** Both exist in the
contract; the v2 path is the current one and the unversioned path is the superseded predecessor
(`lifecycle/braze-lifecycle.yml`). Braze does not emit `Sunset` or `Deprecation` headers, so nothing
at runtime will tell you the old path is on the way out.

Rate limit: 5,000 requests/minute.

## Step 3 — email subscription state is a separate axis

`POST /email/status` sets the global email subscription state (subscribed / unsubscribed / opted-in).
This is **not** the same thing as subscription group membership — a user can be globally subscribed
and out of every group, or the reverse. Set both deliberately.

## Step 4 — suppression list hygiene

- `GET /email/unsubscribes` and `GET /email/hard_bounces` (both `limit`/`offset`) read the
  suppression lists.
- `POST /email/bounce/remove` and `POST /email/spam/remove` release addresses you have verified.
- `POST /email/blocklist` blocks addresses. **Prefer `/email/blocklist` over `/email/blacklist`** —
  the latter is the same operation under the old name and is retained only for compatibility.
- `GET /sms/invalid_phone_numbers` / `POST /sms/invalid_phone_numbers/remove` are the SMS equivalents.

## Step 5 — hand the user a preference center

- `GET /preference_center/v1/list` lists the preference centers configured in the workspace.
- `GET /preference_center_v1/{PreferenceCenterExternalID}/url/{UserID}` mints a **per-user** URL to a
  Braze-hosted page. Note the path irregularity: the URL-generation route is
  `/preference_center_v1/...` (underscore) while the management routes are `/preference_center/v1/...`
  (slash). That is in the contract, not a typo here.
- Management writes are rate limited to **10 requests/minute**; reads to 1,000/minute.

## Errors to expect

`400` with a prose `message`, `401` for a key without the endpoint permission, `429` when the group
write limit trips. There are no machine-readable error codes — match on the documented strings in
`errors/braze-problem-types.yml` and treat anything unrecognized as fatal rather than retryable.
