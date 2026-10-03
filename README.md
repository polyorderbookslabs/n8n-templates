# n8n templates for Polymarket order book data

Workflows for [n8n](https://n8n.io) that use historical Polymarket **order book
depth** — the resting bids and asks at every price level, at 1-second
resolution.

## Why these are different from the other Polymarket templates

There are already good n8n templates for Polymarket, and they mostly pull from
Polymarket's own Gamma and CLOB APIs. Those are free and work well for prices.

They cannot do order books. Polymarket's `/book` endpoint returns the current
state only, and nothing archives it, so no workflow built on that API can look at
what the book was yesterday. These templates read an archive captured live.

If you want prices, use the free Gamma templates — they are simpler and cost
nothing. Use these when the question needs depth.

---

## Measure real slippage on Polymarket order books

`polymarket-order-book-slippage.json`

Ask what an order would actually have cost, filled against the real resting
ladder, versus the midpoint at that moment.

**Input:** a market slug, an order size, and how many minutes before settlement
to examine.

**Output:** per outcome — how often the order could not be filled at all, median
slippage, and the worst moment.

A real result, 100 shares on a BTC 5-minute market in its final five minutes:

| Outcome | Snapshots | Median slippage | Worst |
| --- | --- | --- | --- |
| Yes | 200 | 1.54% | **26.63%** |
| No | 200 | 0.75% | 5.76% |

The midpoint said one thing; filling 100 shares cost up to a quarter more. That
gap is invisible in any price series, because a price series does not know how
much size was resting.

### Setup

1. Import the JSON into n8n.
2. Set `POLYORDERBOOKS_API_KEY` as an environment variable on your n8n instance.
   Get a free key at [polyorderbooks.com/signup](https://polyorderbooks.com/signup)
   — the free tier queries at the same 1-second resolution as paid plans.
3. Run the form trigger and enter a market slug, e.g. `btc-updown-5m-1787597700`.

### Notes

Binary markets go one-sided as they resolve — nobody offers the losing outcome —
so some snapshots have no asks to fill against. The workflow reports those as
`could_not_fill` rather than silently skipping them, because "you could not have
traded at all" is usually the more interesting answer.

Keep the window short. The workflow requests 1-second resolution, which is 60
snapshots per minute per outcome.

---

## Open data

If you want to check the approach before signing up for anything, 897,192
snapshots across 805 resolved markets are published under CC BY 4.0 with a DOI:
[10.5281/zenodo.22084114](https://doi.org/10.5281/zenodo.22084114). The same
ladder-walking logic runs fine on those Parquet files.

## Links

- [polyorderbooks.com](https://polyorderbooks.com) — the API
- [docs.polyorderbooks.com](https://docs.polyorderbooks.com) — endpoint reference,
  parameters and response shapes
- [Order book data explained](https://polyorderbooks.com/polymarket-order-book-data)
  — what an L2 snapshot contains, with a worked slippage example
- [All datasets](https://polyorderbooks.com/datasets) — every free download,
  including the DOI-carrying research dataset
- [Python client](https://pypi.org/project/polyorderbooks/) — `pip install polyorderbooks`
- [MCP server](https://github.com/polyorderbookslabs/mcp-server) — the same data in
  Claude, Cursor and other MCP clients

## Questions

Open an issue on this repository for anything about the workflows themselves.

For the API — a plan, a coverage question, a window these files do not reach —
email **contact@polyorderbooks.com**. Bug reports against the API are welcome and
usually answered the same day.

## Licence

MIT. Not affiliated with, endorsed by, or connected to Polymarket or n8n.
