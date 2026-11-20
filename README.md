# @pipeworx/sgx-announcements

Singapore Exchange company announcements, newest first, by issuer name, date range and category, with a link to each announcement page. Read live from the exchange per request; nothing is stored.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2016+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `sgx_announcements(company?, date_from?, date_to?, category?, limit?)` - issuer, security, SGX category, title, date and time (Singapore) and the announcement link. `company` is an issuer name ("DBS GROUP HOLDINGS LTD") or part of one ("Singtel"); omit it for the whole market.

## Auth

Keyless.

## Data sources

- Singapore Exchange (SGX), company announcements - the exchange's public announcements listing.

Notes for the next person:

- The free listing covers roughly the most recent three months; every response says so (`window_note`) and gives `earliest_in_result` / `latest_in_result`.
- The listing identifies issuers by name only. There is no stock code and no company-profile record in it, so there is no `sgx_company` tool; a name that matches nothing returns `found: false` with an `empty_reason`.
- A name filter first asks the exchange for the exact issuer name; if that finds nothing it reads the whole window and matches names containing the text (`matched_by` says which happened). The whole-market read is a few MB, so prefer exact names.
- The exchange's empty answer is not valid JSON (a plain "No Document Found" inside the body); the pack treats that as zero rows and treats any other unreadable body as an error.
- Category names are the exchange's own; the `category` filter matches by substring.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "sgx-announcements": {
      "url": "https://gateway.pipeworx.io/sgx-announcements/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/sgx-announcements/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2016+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/sgx_announcements \
  -H 'Content-Type: application/json' \
  -d '{"company":"DBS GROUP HOLDINGS LTD","date_from":"2026-11-01","limit":10}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/sgx_announcements`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-sgx-announcements` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/sgx-announcements/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "sgx-announcements": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-sgx-announcements"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Sgx Announcements data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
