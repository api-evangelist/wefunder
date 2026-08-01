---
name: Track campaign attribution with webhooks
description: Read attribution stats for a campaign and subscribe to investment events over signed webhooks.
api: openapi/wefunder-openapi-original.yml
operations: [getAttributionStats, listAttributedInvestments, listWebhookSubscriptions, createWebhookSubscription, testWebhookSubscription, regenerateWebhookSecret, reactivateWebhookSubscription]
generated: '2026-07-21'
method: generated
---

# Track campaign attribution with webhooks

1. **Authenticate** via `authorization_code` + PKCE with the attribution tier you are
   approved for: `read:attribution:aggregate` (Tier 0), `read:attribution:anonymized`
   (Tier 1, requires approval), or `read:attribution:full` (Tier 2, founders only).
   Webhook management needs `read:webhooks` / `write:webhooks`.
2. **Read stats**: `getAttributionStats` (`GET /campaigns/{campaign_id}/attribution/stats`)
   and `listAttributedInvestments` for per-investment rows (cursor-paginated).
3. **Subscribe**: `createWebhookSubscription` (`POST /campaigns/{campaign_id}/attribution/webhooks`)
   with a `target_url` and events from `investment.applied`, `investment.confirmed`,
   `investment.canceled`. The signing secret (`whsec_...`) is shown ONCE at creation —
   store it immediately.
4. **Verify deliveries**: check `X-Wefunder-Signature` =
   `sha256= + HMAC-SHA256(secret, X-Wefunder-Timestamp + "." + raw_body)`; reject
   mismatches and timestamps older than 5 minutes (replay protection). Use the delivery
   UUID for consumer-side idempotency. `@wefunder/sdk` ships `constructEvent()`.
5. **Operate**: `testWebhookSubscription` to send a test event, `regenerateWebhookSecret`
   to rotate the secret, `reactivateWebhookSubscription` after consecutive delivery
   failures deactivate a subscription.

Rules:
- `422` on create means invalid URL, unknown events, or subscription limit reached.
- Never log the signing secret; verify against the RAW request body, not re-serialized JSON.
