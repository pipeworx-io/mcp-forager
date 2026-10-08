# @pipeworx/forager

Find a person's phone number from their LinkedIn profile URL, via Forager
(forager.ai). Part of `enrich-waterfall`'s new `enrich_find_phone` finder
rotation (fleet #2860).

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1743+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `forager_find_phone(linkedin_url?, person_id?, _accountId, _apiKey)` —
  phone number + type. Requires BOTH a Forager account id and an API key.

## Auth

BYO only. Pass your Forager account id as `_accountId` and your API key as
`_apiKey` (header `X-API-KEY`, with the account id as a PATH segment:
`/api/{account_id}/...`). Both are on your Forager dashboard. No platform
key (D4).

## Data sources

- `https://api-v2.forager.ai/api/{account_id}/datastorage/person_contacts_lookup/phone_numbers/`

## Verification notes (fleet #2860, 2026-10-08, no vendor key held) — THIS PACK HAS THE WEAKEST CONFIRMATION IN THE BATCH

- **The URL path is confirmed live.** Every auth shape tried against
  `.../phone_numbers/` with account id `"0"` returns HTTP 403 (never 404),
  so the path and the account-id-in-path convention are real.
- **The header name is NOT independently confirmed.** No header,
  `X-API-KEY: invalidkey123`, `Authorization: Api-Key invalidkey123`, and
  `Authorization: Bearer invalidkey123` all return the byte-identical
  `{"detail":"Authentication credentials were not provided."}`. A
  placeholder-key probe cannot tell "wrong header" apart from "right header,
  garbage key" when the server's generic-scheme message is this generic.
  `X-API-KEY` is used here because that is what search results describing
  Forager's own docs name, but **this is the one auth shape in this batch
  that should be treated as a hypothesis, not a confirmed fact**, until a
  real key either works or returns a DIFFERENT 401/403 body.
- **The request/response body fields are a best-effort guess, not a
  confirmed schema.** We could not read `docs.forager.ai`'s own schema page
  for this specific endpoint (only sibling endpoints — detail lookup,
  reverse-by-email, reverse-by-phone, role search — turned up in what we
  could fetch). This pack sends `{"linkedin_url": ...}` or
  `{"person_id": ...}` and reads a `phone_numbers` array (string or
  `{number, type}` objects) back, by analogy with Forager's REST
  conventions elsewhere. If a real key gets a 200 with a different shape,
  `findPhone()` in `src/index.ts` is the only place that needs fixing.
- **Not verified (no key held), and more load-bearing than usual for this
  pack:** the auth header name, the request body field name, the response
  field name, and whether `_accountId` is even the right concept (vs. the
  account id being derivable from the key itself).

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "forager": {
      "url": "https://gateway.pipeworx.io/forager/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/forager/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1743+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/forager_find_phone`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "forager": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-forager"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-forager
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Forager data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
