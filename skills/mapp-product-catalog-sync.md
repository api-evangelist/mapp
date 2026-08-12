---
name: Sync a product catalog into Mapp Cloud
description: Read catalog metadata and attributes, then bulk upsert, patch and delete product variants in the Mapp Product Catalog API.
api: openapi/mapp-product-catalog-openapi.yml
generated: '2026-08-12'
method: generated
source: openapi/mapp-product-catalog-openapi.yml + https://docs.mapp.com/apidocs/product-catalog-api-data-limits-and-validation-rules
operations:
  - getCatalogMetadataByWorkspace
  - getCatalogAttributes
  - bulkUpsertVariants
  - bulkAddVariants
  - bulkPartialUpdateVariants
  - bulkDeleteVariants
  - upsertVariant
  - partialUpdateVariant
  - getVariant
  - getPaginatedVariants
  - listVariants
  - deleteVariant
  - deleteVariantAttributes
  - deleteAllCatalogData
---

# Sync a product catalog into Mapp Cloud

Base URL `https://api.mapp.com`, paths under `/api/product-catalog/v1`. Auth is a **Bearer JWT** issued
from the same Mapp Cloud API Client ID / Secret used by the Analytics API (`Keycloak` security scheme).
API access must be enabled on the account first.

## Steps

1. **Find the catalog** — `getCatalogMetadataByWorkspace`
   (`GET /api/product-catalog/v1/catalogs/metadata/catalog/workspace`) to resolve the `catalogId`.
2. **Read the attribute contract** — `getCatalogAttributes`
   (`GET /api/product-catalog/v1/catalogs/metadata/attribute/{catalogId}`). Validate your feed against
   this before you send anything; the API rejects unknown and malformed attributes rather than dropping them.
3. **Full sync** — `bulkUpsertVariants`
   (`PUT /api/product-catalog/v1/catalogs/{catalogId}/variants/bulk`). This is the operation to build a
   scheduled sync on: it is a **replace-or-insert** and therefore the one genuinely re-runnable call in the
   Mapp API surface.
4. **Incremental sync** — `bulkPartialUpdateVariants` (`PATCH …/variants/bulk`) for price and stock deltas;
   `bulkAddVariants` (`POST …/variants/bulk`) only for known-new rows.
5. **Single-row work** — `upsertVariant` (`PUT …/variants/{variantId}`), `partialUpdateVariant`
   (`PATCH …`), `getVariant` (`GET …`), `deleteVariant` (`DELETE …`),
   `deleteVariantAttributes` (`DELETE …/variants/{variantId}/attributes`).
6. **Read back** — `getPaginatedVariants` (`GET …/variants`) or `listVariants`
   (`GET …/products/{productId}/variants`).
7. **Only if you mean it** — `deleteAllCatalogData` (`DELETE …/variants/`) wipes the catalog.

## Validation limits to enforce client-side

Mapp publishes hard limits; failing them wastes a whole bulk call:

| Field | Limit |
|---|---|
| Product / Variant / Style ID | 255 characters |
| Title | 255 characters |
| Description | 5000 characters |
| Brand | 100 characters |
| MPN | 70 characters |
| GTIN | 8–14 digits |
| Color | 40 characters |
| Size | 100 characters |
| Price / sales_price | 0.01 – 9,999,999.99, two decimals |
| Attribute name | 64 chars, `[a-zA-Z0-9_-]*` |
| Display name | 255 characters |
| Feed name | `^[a-zA-Z0-9_-]*$` |

## Rules

- Prefer `PUT …/variants/bulk` over `POST …/variants/bulk` for anything scheduled — upsert survives a
  retry, add does not.
- There is still no idempotency key. Bulk upsert is safe because of its semantics, not because Mapp
  guarantees replay safety.
