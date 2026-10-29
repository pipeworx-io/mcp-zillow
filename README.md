# mcp-zillow

Zillow Research MCP — housing market data from Zillow's public CSV dumps.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

| Tool | Description |
|------|-------------|
| `zillow_home_values` | Zillow Home Value Index (ZHVI) — monthly home value estimates as a TIME SERIES. PREFER for any "how have home prices/values TRENDED" question — "home price trend in Denver over the past few years", "how has the Austin housing market moved", "are home values rising in Phoenix" — a multi-year monthly series answers a trend question where a single-year census snapshot cannot; metro grain answering a county question is the right trade, said openly. Returns observation_date + value pairs for the matching region (case-insensitive substring match on region name). Coverage: US metros, states, national. |
| `zillow_rent_prices` | Zillow Observed Rent Index (ZORI) — monthly rent estimates as a time series. US metros only. Returns observation_date + value pairs. |
| `zillow_market_snapshot` | Latest available value across every metric (home value, rent, inventory, sales count, median sale price, new listings) for a single region. Useful for a one-call "tell me everything about this market" view. Coverage is US METRO areas, states and national — ask for the metro that contains a place ("Miami"), because ZIP codes and counties are not in this dataset and census_acs answers those instead. |
| `zillow_top_markets` | Ranked list of regions by a housing metric. sort_by="value" (default) ranks by the latest value: metric=zhvi direction=top → most expensive metros, direction=bottom → cheapest. sort_by="growth" ranks by percent change over a trailing window (growth_window_months, default 12): direction=top → FASTEST-GROWING / fastest-appreciating metros, direction=bottom → biggest decliners. metric defaults to zhvi (home values). Returns up to 100 rows. |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "zillow": {
      "url": "https://gateway.pipeworx.io/zillow/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/zillow/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1857+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/zillow_home_values \
  -H 'Content-Type: application/json' \
  -d '{"region":"Los Angeles"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/zillow_home_values`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-zillow` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/zillow/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "zillow": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-zillow"]
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
ask_pipeworx({ question: "your question about Zillow data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
