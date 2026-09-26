# SquawkFlow for Claude

Free, read-only market structure data for SPX, SPY and QQQ, with every figure
dated. This plugin connects Claude to SquawkFlow's hosted MCP server and adds
skills that tell Claude how to read the data honestly: each number comes with
its source, its capture time and the settlement date of the open interest
behind it, and nothing is presented as a forecast or a trade idea.

No account, no API key, no sign-up. Nothing here places an order, and SquawkFlow
has no order execution. Not investment advice.

## What it does

Ask Claude things like:

- "Where are the SPX call wall, put wall and gamma flip, and when were they captured?"
- "Give me a dated market structure brief for SPY."
- "Which expirations and exchange holidays fall between Nov 2 and Nov 13, and what does the Nov 6 SPX expiration price?"

Claude calls the SquawkFlow tools, then answers with the readings, their dates
and a link to the published page each one came from.

## What is included

**Connector.** One remote MCP server, `squawkflow`, over streamable HTTP at
`https://mcp.squawkflow.com/claude/mcp`. No authentication.

**Skills.**

| Skill | Use it for |
| --- | --- |
| `reading-dealer-gamma` | GEX levels, call and put walls, the zero gamma flip, the vol trigger, the gamma grid by strike and expiration, and the sector gamma matrix: what each is, what it is not, and the settlement vintage behind it |
| `market-structure-briefing` | A dated, cited brief for SPX, SPY or QQQ built from gamma levels, open interest change, max pain, the VIX futures curve, the exchange calendar and implied odds |
| `expiration-and-event-calendar` | Which options and VIX expirations, sessions and exchange closures fall in a window, and what each expiration prices |

## Tools

The connector exposes these read-only tools. Each declares a title and
`readOnlyHint: true`.

| Tool | What it returns |
| --- | --- |
| `list_squawkflow_tools` | The catalog: every tool, its coverage, data vintage and limits, and what the server deliberately does not publish |
| `get_gex_levels` | SPX, SPY or QQQ call wall, put wall, zero gamma flip, vol trigger, net GEX, regime, pin strikes and the options-implied session range |
| `get_gamma_heatmap` | Dealer gamma by strike and expiration, charm and vanna, same-day concentration and the overnight open interest change |
| `get_gamma_matrix` | One cached grid across the eleven SPDR sector ETFs plus the SPX, SPY and QQQ index row |
| `get_oi_change` | Per-strike open interest change between two named daily Cboe settlements |
| `get_max_pain` | The SPX settlement strike that minimises aggregate option payout for one expiration |
| `get_implied_odds` | The risk-neutral probability the chain prices on settling above a level by an expiration, with the implied distribution |
| `get_vix_term_structure` | The Cboe VIX futures settlement curve and its contango or backwardation shape |
| `get_market_calendar` | Sessions, NYSE full-day closures and the next monthly, quarterly and VIX expirations, each read off an exchange document |
| `get_session_record` | One dated SPX session: the levels published before it traded and the verdict on each |
| `get_filing_receipt` | A Form 13F receipt: CIK, period of report, acceptance date, accession number and the reported positions |
| `get_positioning` | Weekly CFTC Traders in Financial Futures positioning for the published index futures |
| `get_lab_record` | Dated receipts from SquawkFlow's simulated Lab (no money, no orders) |
| `search` | Find the published SquawkFlow page that answers a question, with its canonical URL |
| `fetch` | Read one published page verbatim, with the date the page states for itself |

## Install

### From the Claude directory

Find **SquawkFlow** in the plugin directory on claude.ai and add it. The plugin
then works in Claude chat, Cowork and Claude Code.

### Claude Code, manually

Clone this repository and load it for a session:

```bash
git clone https://github.com/dhawalc/squawkflow-claude-plugin
claude --plugin-dir ./squawkflow-claude-plugin
```

Or add only the connector, without the skills:

```bash
claude mcp add --transport http squawkflow https://mcp.squawkflow.com/claude/mcp
```

### Claude on the web or the desktop app, as a connector

Paid plans: Settings, Connectors, Add custom connector. Name it SquawkFlow,
paste `https://mcp.squawkflow.com/claude/mcp`, leave authentication empty, and
save. Enable it from the plus menu in a chat. This adds the tools without the
skills.

## Data sources and dating

- **Options chains and open interest:** delayed Cboe data. Open interest is the
  daily settlement figure from the prior close, so every gamma, max pain and
  open interest figure names the settlement date it was built from.
- **SPX gamma levels:** a daily snapshot, re-priced once per session before the
  open from an archived implied volatility surface. Not recomputed intraday.
- **VIX futures:** Cboe official daily settlement prices.
- **Calendar:** Cboe and NYSE published documents, each entry with the date it
  was checked. No date is computed from a rule.
- **13F filings:** SEC EDGAR. **Futures positioning:** CFTC, as of each Tuesday,
  published the following Friday.
- Every data tool response ends with one citation: the squawkflow.com page URL
  and the capture and retrieval times. `search` and `fetch` return JSON instead,
  with the page's canonical `url` on every result and the date the page states
  for itself. When a live call fails and the server serves its last good answer,
  the response starts with `STALE COPY, NOT A FRESH READING` and gives that
  copy's own capture time.

## Limits

- **Delayed and derived.** Nothing is real time. Dealer positioning is an
  assumption: open interest shows that a contract exists, never which side a
  dealer holds.
- **Coverage.** SPX, SPY and QQQ for gamma and implied odds; max pain is SPX
  only; no per-ticker gamma, screeners, fundamentals, earnings dates or
  economic release calendar in the tools.
- **Rate limits.** SquawkFlow's public endpoints allow 60 requests per minute per
  IP address (30 for the VIX curve). The hosted server spaces each session's
  calls one second apart and shares a budget across all users, so a busy period
  can return a "busy" or "stale copy" answer.
- **Read-only.** No tool writes anything, and there is no account to connect.
- **No advice.** No price targets, trade calls, forecasts or statistics about how
  often a published level was right.
- **Not included.** SquawkFlow's dark pool tape is modeled rather than observed
  and is not exposed.

## What the plugin sends, and privacy

The plugin contains no code that runs on your machine. When Claude uses a tool,
the tool name and its arguments (for example a symbol and a date) go to
`https://mcp.squawkflow.com/claude/mcp`, which reads SquawkFlow's own API and
published pages. Nothing else is sent, and nothing is sent anywhere else.

The server keeps a local log of the client name and version once per session
and, for each tool call, the tool name, the outcome and a coarse symbol label
(SPX, SPY, QQQ or "unsupported"). It writes no IP address, no raw arguments and
no response content to that log. Traffic passes through Cloudflare, which sees
connection metadata including IP addresses.

Privacy policy: <https://squawkflow.com/privacy>

## Support

Email admin@squawkflow.com, or see <https://squawkflow.com/contact>.

## License

MIT. See [LICENSE](LICENSE).
