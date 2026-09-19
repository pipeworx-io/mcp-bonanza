# @pipeworx/bonanza

Search Bonanza marketplace listings via the "Bonapitit" API
(api.bonanza.com/docs) — an eBay-Trading-API-compatible read/write API.
Orders are seller-scoped on this API (need a logged-in user's auth token), so
this pack is search + item detail only. No sold-price data.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1576+ live data sources.

## Tools

- `bonanza_search(keywords, entries_per_page?, page_number?, sort_order?)` —
  keyword search (`findItemsByKeywords`).
- `bonanza_search_by_category(category_id, entries_per_page?, page_number?)`
  — category browse (`findItemsByCategory`).
- `bonanza_get_item(item_id)` — single listing detail (`getSingleItem`).

## Auth

BYO key only (`_apiKey`), requires an API key. Register a free devID/certID
at <https://api.bonanza.com/accounts/new> — a business-identity account with
a ~1-2 day manual approval. Pass `?_apiKey=<devID>` or `<devID>:<certID>`
(colon-joined; certID is accepted but unused by these three read calls).

BYO is the settled posture, not a stopgap: the platform-registration question
was ruled on 2026-09-06 (fleet #1278) — Pipeworx will not fund or register a
platform devID, so callers always bring their own. A keyless call refuses
with "requires an API key" and points at the registration URL. Note **the
request shape has not been verified against a live response** (see Data
sources) — the first caller with a real devID exercises it live.

## Data sources

- <https://api.bonanza.com/docs> — Bonapitit reference. The docs site is a
  client-rendered SPA; the actual wire format (base URL, headers, body
  encoding) is NOT visible in the static HTML and had to be reconstructed
  from the open-source PHP SDK: <https://github.com/Shoplo/bonapitit-bonanza-php-sdk>.

Traps for the next person:

- Base URL is `https://api.bonanza.com/api_requests/standard_request` for
  every call below (secure/write calls use `secure_request` instead and also
  need a user auth token — out of scope here).
- The request body is `application/x-www-form-urlencoded` with exactly ONE
  field: the call name (lower-camel, e.g. `findItemsByKeywords`) whose value
  is the JSON-serialized request object. This is not a JSON body and not a
  REST-ish query string — easy to get wrong.
- Auth headers are `X-BONANZLE-API-DEV-NAME` / `X-BONANZLE-API-CERT-NAME`
  (Bonanza's old internal name, "Bonanzle") — not `X-BONANZA-*`.
- **This pack has never been called against a live Bonanza response.**
  Registering an account needs a password and business details Bruce has to
  enter himself; until that key exists, treat the exact response field names
  as unverified against the community SDK/docs, not against a real payload.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "bonanza": {
      "url": "https://gateway.pipeworx.io/bonanza/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/bonanza/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1576+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/bonanza_search \
  -H 'Content-Type: application/json' \
  -d '{"keywords":"vintage brass lamp","entries_per_page":10}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/bonanza_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "bonanza": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-bonanza"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-bonanza
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Bonanza data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
