---
name: Subscribe a contact and send a transactional message
description: Create or update a Mapp Engage contact, subscribe it to a group, send a prepared message as a transactional message, and read back the send statistics.
api: openapi/mapp-engage-openapi.yml
generated: '2026-08-12'
method: generated
source: openapi/mapp-engage-openapi.yml + https://docs.mapp.com/apidocs/getting-started-with-engage-api
operations:
  - create
  - update
  - subscribeByEmail
  - sendTransactional
  - sendTransactionalWithEventDetails
  - getStatisticsByExternalMessageId
  - unsubscribeByEmail
---

# Subscribe a contact and send a transactional message

## Before you start

- **Base URL is tenant-specific.** Take the host you log in to Mapp Engage with, drop the page path, and
  append `/api/rest/v19`. There is no shared `api.mapp.com` host for Engage.
- **Auth is HTTP Basic**, not a token. Base64-encode `<system-user-email>:<password>` and send it as
  `Authorization: Basic …` on **every** request. The user must be a system user of type **API** (a Hybrid
  user works but Mapp advises against it in production).
- **Send four headers on every call**: `Host` (must match the request host — omitting it can trip Mapp's
  anti-intrusion systems), `Accept`, `Content-Type` (on POST), `Authorization`.
- **There is no idempotency key.** `sendTransactional` and `create` are not safe to blind-retry. Keep your
  own dedupe ledger keyed on your own message/contact id before you retry anything. See
  `conventions/mapp-conventions.yml`.

## Steps

1. **Create the contact** — `create` (`POST /contact/create`) with `emailAddress` and an `attributes` array.
   If the contact may already exist, call `update` (`POST /contact/update`) instead, or call `get`
   (`POST /contact/get`) first and branch. `create` on an existing address is not an upsert.
2. **Subscribe it to the sending group** — `subscribeByEmail`
   (`POST /membership/subscribeByEmail?email=…&groupId=…&subscriptionMode=OPT_IN`). Choose the
   `subscriptionMode` deliberately; `OPT_IN` records consent.
3. **Send the message** — `sendTransactional` (`POST /message/sendTransactional`) with the prepared-message
   id and a `parameters` array of name/value personalisations. Attachments go in the same JSON body as
   base64 in `attachments[].content`.
   Use `sendTransactionalWithEventDetails` when the send is driven by an event payload you also want stored.
   **Side effect to know:** sending a prepared message transactionally *adds the recipient as a member of
   the group that prepared message belongs to* — a contact can be subscribed or re-subscribed without you
   asking. Verify group membership expectations before you call this in bulk.
4. **Read the result** — `getStatisticsByExternalMessageId`
   (`GET /message/getStatisticsByExternalMessageId`) when you passed your own external id, otherwise
   `getStatistics` (`GET /message/getStatistics`).
5. **Honour opt-out** — `unsubscribeByEmail` (`POST /membership/unsubscribeByEmail`) for the group, or
   `unsubscribeFromMessageByEmail` (`POST /membership/unsubscribeFromMessageByEmail`) for a single message.

## Errors and retries

- `200` returns a body, `204` succeeds with no body. Both are success — do not treat `204` as a failure.
- `400` names the offending parameter in the body. Fix and resend; do not retry unchanged.
- `403` is almost always bad credentials or the wrong system-user type, not a permissions edge case.
- `500 "Database Access Limit Exceeded"` **is a rate limit wearing a server-error status code.** Mapp
  publishes an approximate ceiling of ~10 transactions per second per account and returns no `429`, no
  `Retry-After` and no `RateLimit-*` headers. Back off on any 500 whose description mentions the database
  access limit.
- Documented retry ladder: retry after ≥30 seconds, then after ≥15 minutes, then stop and contact support.
  Set client timeouts above 30 seconds — some calls legitimately take longer.
