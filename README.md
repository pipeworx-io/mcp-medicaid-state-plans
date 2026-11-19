# @pipeworx/medicaid-state-plans

Medicaid State Plan Amendments and the State Waivers List (section 1115 demonstrations, 1915 waivers) from CMS Medicaid.gov, searchable by state, topic and date, with the text of an amendment's approval package returned in pages.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2015+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `spa_search(state?, topic?, q?, approval_from?, approval_to?, effective_from?, effective_to?, sort?, limit?, page?)` -- finds amendments. Returns transmittal number (e.g. FL-25-0015), state, summary, approval and effective dates, topic tags and the approval-package link. An unknown state or topic returns the valid values, not an empty list.
- `spa_get(transmittal_number, offset?, max_chars?, include_text?)` -- one amendment: metadata plus the package text as a window (default 8,000 characters), with `text_next_offset` to continue. A scanned package with no text layer says so.
- `waiver_search(state?, authority?, status?, q?, approval_*?, effective_*?, limit?, page?)` -- finds waivers and demonstrations. Authority: 1115, 1915 (b1)-(b4), 1915 (c). Status: Approved, Disapproved, Expired, Pending, Terminated, Withdrawn.
- `waiver_get(waiver_id)` -- one waiver: authority, status, approval, effective and expiration dates, and its supporting documents with dates, links and sizes.

## Auth

Keyless.

## Data sources

- CMS, Medicaid.gov Medicaid State Plan Amendments list (per-state transmittals and their approval packages).
- CMS, Medicaid.gov State Waivers List.

Called live per request; nothing is stored. Federal government data.

## Notes

- Transmittal numbers are strings: state code, two-digit year, sequence (FL-25-0015, OH-22-0017).
- `approval_date` can be later than the year in the number: FL-25-0015 was submitted in 2025 and approved 2026-10-08.
- Approval-package text comes from the PDF's text layer. Line breaks the PDF producer put mid-word are repaired, so tables and column layouts flatten to running text. Older packages that are scans have no text layer; the response sets `text_reason: "no_text_layer"` and gives the package link.
- Waiver documents are mostly ZIP or PDF downloads; this pack lists them, it does not read them.
- Result pages are server-rendered by the publisher, so the pack parses result cards. If the layout changes the tools return an explicit "unparsed" or "layout may have changed" message rather than an empty list.
- The publisher's site answers requests that carry a descriptive client name; the pack sends its own and does not disguise itself.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "medicaid-state-plans": {
      "url": "https://gateway.pipeworx.io/medicaid-state-plans/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/medicaid-state-plans/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2015+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/spa_search \
  -H 'Content-Type: application/json' \
  -d '{"state":"Florida","q":"FQHC","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/spa_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-medicaid-state-plans` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/medicaid-state-plans/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "medicaid-state-plans": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-medicaid-state-plans"]
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
ask_pipeworx({ question: "your question about Medicaid State Plans data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
