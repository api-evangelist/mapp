---
name: Schedule and control a Mapp Engage mobile push campaign
description: Create and schedule a push message, send a single push, and pause, resume, cancel or update it in flight.
api: openapi/mapp-engage-openapi.yml
generated: '2026-08-12'
method: generated
source: openapi/mapp-engage-openapi.yml
operations:
  - createAndSchedulePushMessage
  - getPushMessage
  - updatePushMessage
  - SendSingleMobilePush
  - MultiTenantSendSingleMobilePush
  - pausePushSend
  - resumePushSend
  - cancelPushSend
  - deletePushMessage
---

# Schedule and control a Mapp Engage mobile push campaign

## Steps

1. **Create and schedule** — `createAndSchedulePushMessage`
   (`POST /mobilePush/createAndSchedulePushMessage`). This creates the prepared push and puts it on the
   schedule in one call.
2. **Read it back** — `getPushMessage` (`GET /mobilePush/getPushMessage`) before you act on it. Amend with
   `updatePushMessage` (`PUT /mobilePush/updatePushMessage`).
3. **One-off sends** — `SendSingleMobilePush` (`POST /mobilePush/sendSingle`), or
   `MultiTenantSendSingleMobilePush` (`POST /mobilePush/multiTenantSendSingle`) when you are sending across
   tenants from a shared integration.
4. **In-flight control** — `pausePushSend`, then `resumePushSend`, or `cancelPushSend`
   (all `POST /mobilePush/…`). Pause is the safe first move; cancel cannot be undone.
5. **Remove a prepared push** — `deletePushMessage` (`DELETE /mobilePush/deletePushMessage`).

## Rules

- Always `getPushMessage` before pause/resume/cancel. There is no conditional-request support (no ETag, no
  If-Match), so a blind state transition can act on a send that already finished.
- `cancelPushSend` is destructive and not idempotent-safe to retry: if the first call succeeded and the
  response was lost, the retry may error against a cancelled send. Re-read state instead of re-cancelling.
- The device SDKs that receive these messages have a live end-of-life clock: Engage Android SDK v6 reaches
  End of Support 2026-10-15 and End of Life 2027-04-15. Check `packages/mapp-packages.yml` before assuming
  a fleet can render a new payload.
