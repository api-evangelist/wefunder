---
name: Manage a Wefunder syndicate
description: Read a syndicate, manage its members, and review deals and statistics on behalf of a syndicate lead.
api: openapi/wefunder-openapi-original.yml
operations: [listSyndicates, getSyndicate, listSyndicateMembers, inviteSyndicateMember, approveSyndicateMember, listSyndicateDeals, getSyndicateDeal, listSyndicateDealInvestors, getSyndicateStatistics]
generated: '2026-07-21'
method: generated
---

# Manage a Wefunder syndicate

1. **Authenticate as the user** with `authorization_code` + PKCE (S256). Request
   `read:syndicates` (view) and `write:syndicates` (manage) as needed. Refresh tokens
   ROTATE — persist the new refresh token on every refresh; the old one is invalidated.
2. **Find the syndicate**: `listSyndicates` (`GET /syndicates`), then `getSyndicate`
   (`GET /syndicates/{syndicate_id}`).
3. **Manage members**: `listSyndicateMembers`, invite with `inviteSyndicateMember`
   (`POST .../members/invite`), approve pending members with `approveSyndicateMember`.
   Member writes require `write:syndicates`.
4. **Review deals**: `listSyndicateDeals`, drill in with `getSyndicateDeal`
   (`deals/{fundraise_id}`) and `listSyndicateDealInvestors`.
5. **Report**: `getSyndicateStatistics` for portfolio statistics.

Rules:
- Deal-closing writes go through intents with an `idempotency_key` — a duplicate key
  returns the existing intent (409). Include an `Idempotency-Key` header on retryable
  writes.
- Paginate with `meta.next_cursor` (opaque). Never auto-retry writes; only idempotent
  GETs are safe to retry on 5xx/429.
- `403` means a missing scope or an unapproved OAuth application tier; `404` means the
  resource is not visible to this token.
