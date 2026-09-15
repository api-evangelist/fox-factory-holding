---
generated: '2026-09-14'
method: generated
name: Find a FOX product
description: Search the FOX storefront catalog and read full product detail, using the live UCP/MCP commerce endpoint at ridefox.com.
api: mcp/fox-factory-holding-mcp.yml
operations: [search_catalog, lookup_catalog, get_product]
source: >-
  Grounded in mcp/fox-factory-holding-mcp-tools.json — the verbatim tools/list returned by
  https://ridefox.com/api/ucp/mcp on 2026-09-14. Every tool name and parameter below appears in that
  document; nothing is invented.
---

# Find a FOX product

Read-only catalog work against Fox Factory Holding's FOX storefront. No account, no API key.

## Endpoint
- `POST https://ridefox.com/api/ucp/mcp`
- `Content-Type: application/json`, `Accept: application/json, text/event-stream`
- JSON-RPC 2.0, MCP protocol `2024-11-05`, UCP `2026-08-25`

## Auth
- No credential. `tools/list` answers anonymously (HTTP 200).
- Every `tools/call` must carry `meta["ucp-agent"].profile` — a fetchable https URI identifying your agent. Omitting it returns JSON-RPC `-32001` / `invalid_profile_url` with HTTP 422. See `authentication/fox-factory-holding-authentication.yml`.

## Steps
1. **Search** — `search_catalog` with `catalog.query` (free text). Pass `catalog.context.address_country` and `catalog.context.currency` — the store's own `llms.txt` says pricing and availability are only accurate with them. Narrow with `catalog.filters`.
2. **Batch lookup** — `lookup_catalog` with `catalog.ids[]` when you already hold identifiers. Hard cap of **10 ids per call** (`minItems: 1`, `maxItems: 10`); chunk larger sets.
3. **Full detail** — `get_product` with `catalog.id`, plus `catalog.selected[]` (`{name, label}` option pairs) to resolve a specific variant and `catalog.preferences[]` to shape the response.

## Money
- Amounts are **integers in ISO 4217 minor units** paired with a currency code: `{"amount": 2500, "currency": "USD"}` is $25.00. Divide by 100 for two-decimal currencies before quoting a buyer; zero-decimal currencies such as JPY are already whole units.

## Errors
- HTTP 422 with a JSON-RPC error object; branch on `error.data.code`. See `errors/fox-factory-holding-problem-types.yml`.
- 429 means you are rate limited per IP — back off. No numeric limit is published.

## Sibling storefronts
The same 13 tools are served by other Fox Factory brands: `https://www.raceface.com/api/ucp/mcp`, `https://eastoncycling.com/api/ucp/mcp`, `https://www.methodracewheels.com/api/ucp/mcp`. Each has its own catalog.
