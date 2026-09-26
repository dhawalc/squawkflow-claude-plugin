---
name: expiration-and-event-calendar
description: Use when the user asks which options expirations, VIX expirations, exchange holidays or other scheduled exchange-calendar events fall in a date window (for example "what expires the week of Nov 2", "is Nov 26 a trading day", "when is the next quarterly expiration"), or what the option market prices for a given expiration. Reads get_market_calendar and get_implied_odds from the SquawkFlow connector and reports each date with the exchange document date it was checked against. Covers the exchange calendar only, not economic releases or earnings.
---

# Expiration and event calendar

SquawkFlow's calendar is read entry by entry off Cboe and NYSE documents. No
date is worked out from a rule (the third-Friday rule is wrong for June 2026,
when the standard expiration moved to Thursday June 18 for Juneteenth). This
skill answers "what falls in this window" and "what does each expiration price".

## Tools

- `get_market_calendar` (optional `date`, ISO): whether that date is a session,
  the next session after it, the next monthly, quarterly and VIX futures
  expiration, the SPX settlement rules in the exchange wording, and the upcoming
  NYSE full-day closures.
- `get_implied_odds` (`symbol` SPX, SPY or QQQ; optional `expiration`, optional
  `level`): what the chain prices for one expiration, and the list of
  expirations the tool answers for.

## Procedure

1. Fix the window as two absolute dates. If the user gives a relative window
   ("next week"), resolve it to dates and state them.
2. Call `get_market_calendar` with `date` set to the window start. Record:
   - "Calendar verified on" (the date the exchange documents were checked),
   - sessions and closures inside the window from the closure table,
   - the next monthly, quarterly and VIX expirations.
3. If a listed next expiration falls inside the window and the window extends
   past it, call `get_market_calendar` again with `date` set to the day after that
   expiration, until the window is covered. Stop at the end of the published
   table; a date past it is "not published", never computed.
4. For daily and weekly SPX, SPY or QQQ expirations, call `get_implied_odds` for
   the symbol with no expiration and read "Expirations this tool answers for".
   That list may be truncated; an expiration missing from it can still be
   requested by date. The tool answers only for expirations settling within 90
   days.
5. When the user asks what an expiration prices, call `get_implied_odds` with
   that `expiration` and report: the central 68 percent band, the implied
   median, the expected absolute move from the forward, the option root and its
   settlement (A.M. open or P.M. close), and "Chain captured (UTC)".
6. If any response is a stale copy, report it with its own capture time and say
   the fresh call failed. If one is unavailable, say so; do not fill it in.

## Answer shape

- The window, as absolute dates.
- A table: Date, Weekday, What (session closure, monthly expiration, quarterly
  expiration, VIX expiration, daily expiration), Source and checked date.
- For each expiration the user asked about: what it prices, with the chain
  capture time, labelled as a risk-neutral price.
- One line on what this calendar does not cover (below).
- The citation URL(s) and "Not investment advice."

## What this calendar does not cover

- No scheduled economic releases (jobs, CPI, FOMC and so on) and no earnings
  dates in `get_market_calendar`. Say that plainly if the user asks; do not add
  dates from memory and present them as SquawkFlow's. SquawkFlow does publish a
  few dated event-week pages (for example `/calendar/election-week-2026`). Use
  `search` to find one for the window; if it exists, `fetch` it and cite that
  page with the date the page states for itself. If none exists, say so.
- No half sessions: they are not published, so a listed session may be a half
  session.
- No market hours: the calendar answers whole days, not whether the exchange is
  open at this moment.

## Rules

- Every date carries its source (Cboe or NYSE via SquawkFlow) and the date it was
  checked. Every priced figure carries its expiration and chain capture time.
- Never compute an expiration from a rule. If the table does not state it, it is
  "not published".
- Implied figures are risk-neutral prices, not forecasts and not frequencies.
  Write "the chain prices", never "the market expects".
- Standard monthly SPX options are A.M.-settled at the open of expiration day;
  SPXW options settle at the 4:00 pm ET close (1:00 pm on a half day). Use the
  settlement wording the tool returns.
- No advice, no trade calls, no predictions about what happens on any date.
- Dates are absolute. Say "Not investment advice." once.
