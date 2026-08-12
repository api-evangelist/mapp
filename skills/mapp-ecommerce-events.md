---
name: Feed e-commerce behaviour into Mapp Engage
description: Register abandoned cart, abandoned browse, wish list and transaction events against a contact so Engage automations can trigger on them.
api: openapi/mapp-engage-openapi.yml
generated: '2026-08-12'
method: generated
source: openapi/mapp-engage-openapi.yml
operations:
  - createAbandonedCartItem
  - createAbandonedBrowseItem
  - createWishlistItem
  - registerTransaction
  - removeAbandonedCartItem
  - removeAbandonedBrowseItem
  - removeWishlistItem
  - removeTransaction
---

# Feed e-commerce behaviour into Mapp Engage

Mapp Engage has **no inbound webhook and no event bus for this** — behaviour is pushed in by calling the
Ecommerce operations directly. The Kafka **Data Streams** surface is outbound-only (Mapp Intelligence → you)
and is a Custom-tier feature; see `asyncapi/mapp-data-streams.yml`.

## Steps

1. **Cart abandonment** — `createAbandonedCartItem` (`POST /ecommerce/createAbandonedCartItem`) per item as
   the shopper adds it. Call `removeAbandonedCartItem` (`DELETE /ecommerce/removeAbandonedCartItem`) when
   the item leaves the cart, otherwise the abandonment automation will fire on stale state.
2. **Browse abandonment** — `createAbandonedBrowseItem` / `removeAbandonedBrowseItem`, same pattern.
3. **Wish list** — `createWishlistItem` / `removeWishlistItem`.
4. **Purchase** — `registerTransaction` (`POST /ecommerce/registerTransaction`). Registering the transaction
   is what closes the loop; it does **not** implicitly clear the cart items, so pair it with
   `removeAbandonedCartItem` calls for the purchased lines.
5. **Correction** — `removeTransaction` (`DELETE /ecommerce/removeTransaction`) for a reversed order.

## Rules

- **Volume is the risk here.** These calls sit on the shopper's critical path and hit the ~10 tps
  per-account ceiling faster than anything else in the API. Buffer and batch on your side, parallelise
  submissions across the API cluster as Mapp recommends, and treat `500 "Database Access Limit Exceeded"`
  as a backpressure signal.
- **No idempotency.** A retried `registerTransaction` can double-count revenue. Key your own ledger on the
  order id and only retry entries you know did not land.
- These are fire-and-forget-shaped calls, and Mapp explicitly warns against building a fire-and-forget
  client. Log every response; queue and retry the failures.
