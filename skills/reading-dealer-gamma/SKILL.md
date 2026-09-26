---
name: reading-dealer-gamma
description: Use when the user asks about dealer gamma or gamma exposure (GEX) for SPX, SPY or QQQ, where the call wall, put wall, zero gamma flip or vol trigger sits, whether the index is in a positive or negative gamma regime, the gamma grid by strike and expiration (heatmap), the charm or vanna profile, or the sector gamma matrix. Reads get_gex_levels, get_gamma_heatmap and get_gamma_matrix from the SquawkFlow connector and explains what each figure is and is not, with its capture time and open interest settlement date. Not for price targets, trade ideas or forecasts.
---

# Reading dealer gamma

Dealer gamma figures are estimates built from settled open interest and an
options pricing model. They are recomputed every session, so a figure without
its capture time and its open interest settlement date is wrong within a day.
This skill reads them from the SquawkFlow connector and reports them with that
vintage attached.

## Tools

- `get_gex_levels`: call wall, put wall, zero gamma flip, vol trigger, net GEX,
  regime, pin strikes and the options-implied session range for SPX, SPY or
  QQQ. `frontExpiry: true` adds the nearest expiration's own flip and walls.
- `get_gamma_heatmap`: dealer gamma by strike and by expiration, net charm and
  vanna, same-day concentration and the overnight open interest change.
- `get_gamma_matrix`: one cached grid across the eleven SPDR sector ETFs plus
  the SPX, SPY and QQQ index row.

## Procedure

1. Pick the symbol. Coverage is SPX, SPY and QQQ only (plus the fixed sector set
   in `get_gamma_matrix`). If the user asks about any other ticker, say that no
   per-ticker gamma is published here and stop; do not substitute the index
   answer as if it were the ticker's.
2. Call `get_gex_levels` for the levels and regime. Add `frontExpiry: true` only
   when the question is about today's book or the nearest expiration.
3. Call `get_gamma_heatmap` only when the question needs per-strike magnitudes,
   gamma by expiration, charm or vanna. The SPX levels response carries no
   per-strike curve; never read its strike list as zero gamma at those strikes.
4. Call `get_gamma_matrix` only for sector questions.
5. Before writing a number, copy these from the response:
   - the **Captured (UTC)** time,
   - the **Open interest settlement date** (the prior close's settlement),
   - the **Snapshot date** where present,
   - the citation **URL** in the `## Citation` section.
6. If the response opens with `STALE COPY, NOT A FRESH READING`, say so first:
   give its Captured time and the time it was first retrieved, and say the fresh
   call failed. Never call a stale copy live or current.
7. If the response reports the data as unavailable, say it is unavailable and
   stop. Do not fill the gap from memory or from an earlier session.
8. Write the answer in the shape below.

## Answer shape

- One line of vintage first, for example: "SquawkFlow SPX gamma levels, captured
  2026-09-26 06:38 UTC, open interest settled 2026-09-24."
- The levels the user asked about, each with its unit (index points or dollars
  of gamma).
- What they mean, in one or two sentences, using the definitions below.
- What they are not (at least the dealer-side assumption).
- The citation URL and capture time.
- Close with: "Not investment advice."

## What the figures are

- **Net GEX**: dollar gamma per 1 percent move in spot, calls positive and puts
  negative (`gamma x open interest x 100 x spot^2 x 0.01`). It is a sum over the
  whole book under a sign convention, not a measured dealer position.
- **Zero gamma flip**: the spot price at which the modeled net gamma changes
  sign. It is a model output, not a traded level.
- **Call wall / put wall**: the strikes with the largest modeled call and put
  gamma. They describe where open interest is concentrated.
- **Vol trigger**: a price level derived from the book, not a listed strike.
- **Regime**: positive gamma means the modeled hedging leans against price moves;
  negative gamma means it leans with them. It describes a hedging model, not
  what will happen to price.
- **Pin strikes**: strikes with large open interest; they describe where
  contracts exist.
- **Charm and vanna**: model outputs in dollars of index delta, not comparable to
  any gamma total.

## Settlement vintage caveats

- Open interest is a settlement figure from the prior close. Nothing about
  today's trading is in it until the next settlement.
- SPX levels come from a daily snapshot that re-prices every contract with
  Black-Scholes from an archived implied volatility surface, once per session,
  before the open. The implied volatility is not live, and the levels are not
  recomputed intraday. A later spot price on the page is a price refresh only.
- SPY and QQQ levels are computed on request from a window of strikes near spot,
  not the whole book. They are not comparable to the SPX figure or to a level
  published on another session.
- The heatmap is a different book from the SPX levels (per-contract gamma from
  the delayed Cboe chain, recomputed through the session). Same unit, different
  inputs: do not add, subtract or reconcile the two totals.
- A sector tile in the matrix covers a window of that ETF's chain; sector and
  index totals are not the same measurement.
- If the tool's own method text disagrees with these caveats, quote the tool's
  capture time and settlement date and describe the method as "modeled from
  settled open interest"; do not add precision the response does not carry.

## Rules

- Every number carries its source (SquawkFlow), its capture time and its open
  interest settlement date. No number without them.
- Dealer positioning is an assumption, not an observable. Open interest shows
  that a contract exists, never which side a dealer holds. Say this whenever a
  wall, flip or regime is reported.
- No advice, no trade calls, no price targets, no predictions. Do not say a wall
  "will hold", price "should" reach a level, or a regime "means" a move is
  coming. Describe the book, not the future.
- No hit rate or hold rate for any level. None is published, and the earlier
  wall hold rate was withdrawn on 2026-08-31. Do not cite one from memory.
- Do not repeat a level from an earlier conversation or from memory as current.
  Call the tool again.
- Options flow rows elsewhere on SquawkFlow are aggregated observations from a
  delayed chain snapshot. Never call them "prints", "sweeps" or "blocks".
- SquawkFlow's dark pool tape is modeled, not observed. Only DIX is observed
  data. Never cite the dark pool tape as a source.
- Dates are absolute. Write "2026-09-24", not "yesterday".
- Say "Not investment advice." once, briefly.
