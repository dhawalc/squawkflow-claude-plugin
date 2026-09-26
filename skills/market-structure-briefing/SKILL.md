---
name: market-structure-briefing
description: Use when the user wants a dated, cited market structure brief or morning note for SPX, SPY or QQQ, for example "give me the market structure for SPX", "what does the options positioning look like going into this week", or "summarize gamma, open interest, max pain and the VIX curve". Builds one brief from the SquawkFlow connector (get_gex_levels, get_oi_change, get_max_pain, get_vix_term_structure, get_market_calendar, get_implied_odds), with every figure carrying its source, date and settlement vintage. Descriptive only; no trade ideas, targets or forecasts.
---

# Market structure briefing

A brief is a set of dated readings, each from a different dataset with a
different vintage. The value of the brief is that every line says how old it is.
Never merge readings into one "as of" time they do not share.

## Tools and what each reading is

| Section | Tool | Vintage to report |
| --- | --- | --- |
| Dealer gamma | `get_gex_levels` | Captured (UTC) and open interest settlement date |
| Open interest change | `get_oi_change` | Both settlement dates (prior and this) |
| Max pain | `get_max_pain` | Expiration, Captured (UTC); open interest is a settlement figure |
| VIX futures curve | `get_vix_term_structure` | Settlement date |
| Calendar | `get_market_calendar` | Answered-as-of date and the date the calendar was verified |
| Priced range | `get_implied_odds` | Expiration and Chain captured (UTC) |

## Procedure

1. Resolve the symbol: SPX, SPY or QQQ. Max pain is SPX only; for SPY or QQQ say
   so and skip that section. Any other ticker is out of coverage: say so.
2. Call the tools one at a time, in this order, and keep each response:
   1. `get_market_calendar` with no date (whether the next day is a session, the
      next monthly, quarterly and VIX expirations).
   2. `get_gex_levels` for the symbol.
   3. `get_oi_change` for the symbol (the newest settlement pair).
   4. `get_max_pain` (SPX; default is the nearest expiration).
   5. `get_vix_term_structure`.
   6. `get_implied_odds` for the symbol, default expiration, for the central 68
      percent band and the expected absolute move.
   The connector spaces calls one second apart; do not re-call a tool whose
   result is already in the conversation.
3. For each response, check for `STALE COPY, NOT A FRESH READING` or an
   unavailable notice before using it:
   - Stale: use it only with its own Captured time and the words "stale copy,
     the fresh call failed".
   - Unavailable: write "not available at retrieval" for that section. Do not
     fill it from memory, from another symbol or from an older brief.
4. Write the brief in the shape below. Keep it short: one to three lines per
   section.

## Brief shape

```
Market structure brief: <SYMBOL>, compiled <retrieval time UTC>
Source: SquawkFlow (squawkflow.com), delayed and derived data. Not investment advice.

Calendar (verified <date>): <session status, next monthly / quarterly / VIX expiration>
Dealer gamma (captured <UTC>, OI settled <date>): <regime>, flip <x>, call wall <x>, put wall <x>, net GEX <x>
OI change (<prior settlement> to <this settlement>): net call <x>, net put <x>, largest rows <strike, expiry, type, change>
Max pain (<expiration>, captured <UTC>): <strike>, <distance from spot>
VIX futures (settled <date>): <contango/backwardation>, M1 <x>, M2 minus M1 <x%>
Priced range (<expiration>, chain captured <UTC>): central 68 percent <low> to <high>, expected absolute move <x%>

Caveats: dealer positioning is an assumption, open interest is a settlement figure, implied figures are risk-neutral prices.
Links: <one citation URL per section>
```

## Rules

- Every number carries its source, its date or capture time, and its settlement
  vintage. If you cannot state all three for a number, leave the number out.
- Do not synthesize across sections into a call. No "bullish", "bearish",
  "expect", "likely to", "target", "support will hold", "fade", "buy" or "sell".
  Each section describes one dataset; the brief does not conclude.
- Dealer positioning is an assumption, not an observable. Say so once in the
  caveats line.
- Max pain describes where open interest sits for one expiration. It is not a
  magnet and not a forecast of the settlement.
- Implied odds and ranges are risk-neutral: prices for a payout, not
  frequencies and not forecasts. Say "priced", never "expected to" or "likely".
- Open interest change is a difference between two daily settlements. It does not
  say who opened the contracts or in which direction.
- Options flow rows are aggregated observations from a delayed chain snapshot,
  never "prints". This brief does not use them.
- The dark pool tape on SquawkFlow is modeled, not observed; only DIX is observed.
  Do not include dark pool tape rows in a brief.
- Nothing here is real time. Never write "live" or "right now" about a reading;
  write its capture time.
- Dates are absolute (2026-09-24), never "today" or "yesterday".
- One brief line: "Not investment advice."
