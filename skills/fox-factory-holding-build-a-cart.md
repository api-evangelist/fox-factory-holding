---
generated: '2026-09-14'
method: generated
name: Build and revise a FOX cart
description: Create, update, inspect and cancel a cart on the FOX storefront over the UCP/MCP commerce endpoint.
api: mcp/fox-factory-holding-mcp.yml
operations: [create_cart, get_cart, update_cart, cancel_cart]
source: >-
  Grounded in mcp/fox-factory-holding-mcp-tools.json — the verbatim tools/list returned by
  https://ridefox.com/api/ucp/mcp on 2026-09-14.
---

# Build and revise a FOX cart

The reversible half of the write surface. Nothing here moves money.

## Auth
- Anonymous, but every call needs `meta["ucp-agent"].profile`.

## Steps
1. **Create** — `create_cart` with `cart.line_items[]` (`{item, quantity}`). Add `cart.buyer` (`email`, `phone_number`) when you have consent, `cart.context` (`address_country`, `address_region`, `postal_code`, `currency`, `language`, `intent`) for accurate pricing, and `cart.attribution` (`utm_*`, `referring_domain`, `click_id_*`) to carry marketing provenance. Capture the returned id — format `gid://shopify/Cart/abc123`.
2. **Inspect** — `get_cart` with `id`.
3. **Revise** — `update_cart` with `id` and the changed `cart` fields. Discounts go in `cart.discounts.codes[]`; codes are case-insensitive and **each update replaces the previously applied set**, so resend every code you want to keep.
4. **Abandon** — `cancel_cart` with `id`.

## Idempotency — read this before retrying
- `create_cart` and `update_cart` expose **no** idempotency key. A retry after a timeout can create a second cart. Read back with `get_cart` before retrying, and cancel any duplicate. Only `complete_checkout` accepts `meta["idempotency-key"]`. See `conventions/fox-factory-holding-conventions.yml`.

## Reversibility
- `cancel_cart` reverses `create_cart`. No window is published for it — treat it as best-effort and verify with `get_cart`.

## Errors
- See `errors/fox-factory-holding-problem-types.yml`.
