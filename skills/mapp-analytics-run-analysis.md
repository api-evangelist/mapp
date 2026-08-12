---
name: Run a Mapp Intelligence analysis through the Analytics API
description: Get an OAuth token, discover the available dimensions, metrics and segments, submit an analysis query, poll it, and retrieve the result.
api: openapi/mapp-intelligence-analytics-openapi.yml
generated: '2026-08-12'
method: generated
source: openapi/mapp-intelligence-analytics-openapi.yml + https://docs.mapp.com/apidocs/how-to-grant-access-to-the-intelligence-analytics-api
operations:
  - GET /query-objects
  - GET /segments
  - GET /dynamic-timefilters
  - POST /analysis-query
  - GET /analysis-query/{correlationId}
  - GET /analysis-result/{calculationId}
  - DELETE /analysis-query/{correlationId}
  - GET /analysis-usage/current
  - createReportQuery
  - status
  - deleteReportAnalysis
---

# Run a Mapp Intelligence analysis through the Analytics API

> **Spec caveat.** Most Analytics API operations ship with **no `operationId`** in Mapp's published
> fragments, so they are addressed here by method + path. Only `createReportQuery`, `status` and
> `deleteReportAnalysis` carry ids. Bind tools to the path, not to a generated name.

## Before you start

- The Analytics API is an **optional feature**. If it has not been enabled on the account by a Customer
  Success Manager, valid credentials still fail.
- Base URL: `https://api.mapp.com/api/analytics`. The legacy `intelligence.eu.mapp.com` host still
  redirects, but **responses always reference the new hosts**, so an integration that allowlists only the
  old domain will succeed on the request and then fail to fetch its own result.

## Steps

1. **Get a token** — `POST https://auth.mapp.com/oauth2/token` with HTTP Basic `client-id:client-secret`
   and body `grant_type=client_credentials`. **Do not send a `scope` parameter** — earlier versions
   required one and the current endpoint rejects it. Default validity is 60 minutes.
   Send the result as `Authorization: Bearer <token>` on every call.
2. **Discover what you can ask for** — `GET /query-objects` (dimensions and metrics), `GET /segments`,
   `GET /dynamic-timefilters`. Build the query from these; do not hardcode object names across tenants.
3. **Submit** — `POST /analysis-query` with the analysis JSON. Two success shapes, and you must handle both:
   - `200` → the result was already computed: take `calculationId` + `resultUrl`.
   - `201` → queued: take `correlationId` + `statusUrl`.
4. **Poll** — `GET /analysis-query/{correlationId}` until a `resultUrl` appears. `DELETE` the same path to
   cancel a query you no longer need — do this, because analyses are metered.
5. **Fetch** — `GET /analysis-result/{calculationId}`.
6. **Reports** — the same lifecycle with `createReportQuery` (`POST /report-query`), `status`
   (`GET /report-query/{reportCorrelationId}`) and `deleteReportAnalysis` (`DELETE` on that path).
7. **Watch your budget** — `GET /analysis-usage/current` returns analyses run this month. Mapp does not
   publish the allowance as a number, so treat this counter as the only visibility you have.

## Errors

- Errors are **RFC 9457 `application/problem+json`** with `type`, `title`, `status`, `detail`, `instance`
  and a Mapp-specific `code` (e.g. `AUTH_MISSING_OR_INVALID_HEADER`). Parse `code`, not `title`.
- `422` means the analysis JSON is invalid — a client error, never retry unchanged.
- A value that renders as `0` in the Mapp Intelligence UI comes back as `null` from the API. Do not treat
  `null` as missing data.
