---
generated: '2026-09-14'
method: generated
name: Check out on FOX with buyer approval
description: Convert a cart into a checkout, set fulfillment and payment, and complete the purchase — with the human-approval invariant the store requires.
api: mcp/fox-factory-holding-mcp.yml
operations: [create_checkout, get_checkout, update_checkout, complete_checkout, cancel_checkout, get_order]
source: >-
  Grounded in mcp/fox-factory-holding-mcp-tools.json (verbatim tools/list, 2026-09-14) and the
  provider's own rules published at https://ridefox.com/robots.txt and https://ridefox.com/llms.txt.
---

# Check out on FOX with buyer approval

This is the money-moving surface. The provider states a hard constraint on it.

## The invariant — non-negotiable
Both `robots.txt` and `llms.txt` on ridefox.com state that checkout, payment and order placement must **not** be completed automatically. An agent must obtain **explicit, contemporaneous human approval** at the moment of payment, or route the purchase through Shop Pay via the Shopify shopping skill instead. Do not script an end-to-end purchase.

## Steps
1. **Open** — `create_checkout` with either `checkout.cart_id` (converting an existing cart) or `checkout.line_items[]`. Include `checkout.buyer`, `checkout.context` and `checkout.attribution`. Capture the id — `gid://shopify/Checkout/abc123`.
2. **Fulfil** — `update_checkout` with `id` and `checkout.fulfillment.methods[]` (shipping address and method) and any `checkout.discounts.codes[]`.
3. **Read the real total** — `get_checkout` with `id`. Quote the buyer from this response: line items, totals, discounts and taxes. Convert minor units before quoting.
4. **Attach payment** — `update_checkout` with `checkout.payment.instruments[]`. Each instrument needs `id`, `handler_id` and `type`. Handlers declared by this store: `com.google.pay` (`gpay`), `dev.shopify.card` (`shopify.card`), `dev.shopify.shop_pay` (`shop_pay`); `apple-pay` additionally requires `billing_address` and an `apple_pay_token` credential.
5. **Get approval** — stop. Present the total and obtain the buyer's explicit approval now.
6. **Complete** — `complete_checkout` with `id`, `checkout` and `meta["idempotency-key"]`. **Always send the idempotency key** — this is the only tool on the surface that accepts one, and it is the only protection against a double charge on retry.
7. **Confirm** — `get_order` with the resulting order id (`gid://shopify/Order/123`).

## If it goes wrong
- **Before completion** — `cancel_checkout` with `id` reverses `create_checkout`. No window is published.
- **After completion** — there is **no** reversal tool. Refunds and returns are a human process under https://ridefox.com/policies/refund-policy. Tell the buyer this before step 6.
- **Timeout during step 6** — retry with the *same* `meta["idempotency-key"]`. Never retry with a new one.

## Errors
- HTTP 422 + JSON-RPC error; branch on `error.data.code`. `error.data.continue_url` is a human-handoff link. See `errors/fox-factory-holding-problem-types.yml`.
- 429: rate limited per IP, back off.
