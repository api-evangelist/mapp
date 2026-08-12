---
name: Build, preview and publish a Mapp Engage segment
description: Fetch the selection-plan schema, create a segment, preview it, count it, and publish it — the headless equivalent of the Segmentation Builder UI.
api: openapi/mapp-engage-openapi.yml
generated: '2026-08-12'
method: generated
source: openapi/mapp-engage-openapi.yml + https://docs.mapp.com/apidocs/engage-api-creating-and-publishing-a-segment
operations:
  - schema
  - create_5
  - find_2
  - get_2
  - preview
  - triggerCount
  - getCount
  - update_5
  - publish
  - delete_3
---

# Build, preview and publish a Mapp Engage segment

These operations were added to Engage API v19 on 2026-07-31 specifically to make segmentation available to
headless and agent-driven integrations. Everything here is additive to v19 — no version bump is required.

## Steps

1. **Get the grammar first** — `schema` (`GET /segmentation/schema`) returns the selection-plan schema.
   Build the plan against this, not against a hand-written example; the accepted operators and attribute
   references are tenant-dependent.
2. **Create the plan** — `create_5` (`POST /segmentation/create`). The spec exposes this operation twice
   under the same short name in Mapp's own fragments; in this repo the second occurrence carries the
   `_5` suffix so operationIds stay unique. The path is the contract.
3. **Preview before you count** — `preview` (`GET /segmentation/preview`) returns a sample of matching
   contacts. Cheap; use it to catch an inverted filter before you spend a count.
4. **Count asynchronously** — `triggerCount` (`POST /segmentation/triggerCount`) starts the count, then poll
   `getCount` (`GET /segmentation/getCount`). Counting is not instant on large databases.
5. **Amend if needed** — `update_5` (`PUT /segmentation/update`).
6. **Publish** — `publish` (`POST /segmentation/publish`). Only a published segment is usable by campaigns.
7. **Clean up** — `delete_3` (`DELETE /segmentation/delete`). Find existing plans with `find_2`
   (`POST /segmentation/find`) and read one with `get_2` (`GET /segmentation/get`).

## Rules

- Preview and count are the only safe read-backs; there is no dry-run publish.
- Nothing here is idempotent. A repeated `create` produces a second plan, not the same one. Record the
  returned plan id before you retry.
- Segment definitions reference attributes — if the attribute does not exist yet, create it first with the
  Profile/Member/Group attribute operations rather than letting the plan fail validation.
