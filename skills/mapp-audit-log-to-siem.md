---
name: Pull Mapp Engage audit events into a SIEM
description: Retrieve Log Tracker audit events for a time window and ship them to Splunk, Microsoft Sentinel or another log platform.
api: openapi/mapp-engage-openapi.yml
generated: '2026-08-12'
method: generated
source: openapi/mapp-engage-openapi.yml + https://docs.mapp.com/docs/mapp-engage-new-public-api-endpoints-in-api-v19
operations:
  - auditLogEvents
  - getUsageStatistics
  - ExportGetUsageStatistics
---

# Pull Mapp Engage audit events into a SIEM

Added to Engage API v19 on 2026-07-31 for exactly this purpose.

## Steps

1. **Pull the window** — `auditLogEvents` (`GET /auditlog/events`) with a bounded time range. Events cover
   logins, message operations, group changes, imports and exports, and automation actions. Access requires
   the same permission as the Log Tracker in the UI, and events are scoped to your own account only.
2. **Page forward by time, not by cursor.** There is no cursor and no `Link` header. Keep a high-water mark
   of the last event timestamp you ingested and request the next window from there, with a small overlap,
   then dedupe on the event identity in your SIEM.
3. **Correlate usage** — `getUsageStatistics` (`GET /usagestatistics/get`) and
   `ExportGetUsageStatistics` (`GET /usagestatistics/export`) for volume context alongside the audit trail.

## Rules

- Schedule this well under the ~10 tps account ceiling; audit polling competes with production message
  traffic for the same budget and there is no separate quota.
- Because the retry ladder is 30 seconds then 15 minutes, a poller that fails twice will leave a ~15-minute
  hole. Make the high-water mark durable so the next successful run backfills it rather than skipping it.
- Mapp publishes no `Sunset`/`Deprecation` headers, so a removed field will not announce itself in the
  response. Watch https://docs.mapp.com/docs/news, which is where every Mapp deprecation has been announced.
