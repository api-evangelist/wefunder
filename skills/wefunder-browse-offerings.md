---
name: Browse public offerings on Wefunder
description: Authenticate server-to-server and browse the public deal explore surface.
api: openapi/wefunder-openapi-original.yml
operations: [listOfferings, getOffering]
generated: '2026-07-21'
method: generated
---

# Browse public offerings on Wefunder

1. **Authenticate with `client_credentials`.** POST to the token host
   (`https://api.wefunder.com/oauth`, or `https://wefunder.com/oauth/token` per the
   spec) with your `pk_test_`/`pk_live_` client credentials and scope `read:public`.
   A client_credentials token can ONLY hold `read:public` — it acts as the app with
   no user context; user-scoped calls will fail `403 insufficient_scope`.
2. **List offerings** with `listOfferings` (`GET /explore`). Optional `sort` query
   param (e.g. `most_raised`, `newest`). Responses paginate with an opaque cursor —
   read `meta.next_cursor` and pass it back; never construct cursors.
3. **Fetch one offering** with `getOffering` (`GET /offerings/{external_id}`), using
   the offering's `external_id`; the resource carries an `etag`.

Rules:
- Send `Wefunder-Version: 2025-01-15` on every request (dated-version model).
- On `429`, honor `X-RateLimit-Reset` (standard tier: 1,000 requests/hour/token).
- Errors arrive as `error.{type,message,request_id,remediation}` — log `request_id`.
