---
title: How do I backtest a strategy that uses earnings dates or other non-price data?
description: Stamp every record with the moment it became available to you, not the date it describes, then join it to your bars "as of": each bar sees the latest record stamped at or before it, never a later one. Most lookahead in event-driven backtests comes from the timestamps, not the join. Reamer Research attaches a non-price series to each ticker and enforces that rule, so a value appears exactly on the bar carrying its timestamp and never earlier.
stage: 3
order: 23
product: research
next: what-makes-a-backtest-deterministic, backtest-vs-live-results, research-engine-for-mid-frequency-strategies, what-is-reamer-research
date: 2026-10-01
---

Stamp every record with the moment it became available to you, not the date it describes, then join it to your bars "as of": each bar sees the latest record stamped at or before it, never a later one. Most lookahead in event-driven backtests comes from the timestamps, not the join. [Reamer Research](/products/reamer-research.html) attaches a non-price series to each ticker and enforces that rule, so a value appears exactly on the bar carrying its timestamp and never earlier.

## What counts as non-price data

Anything a strategy reads that is not in the bar itself:

- **Events.** Earnings releases, dividends, index additions, guidance changes.
- **Slow series.** Macro releases, credit ratings, a volatility index, positioning surveys.
- **Derived or alternative data.** Sentiment scores, regime labels, a model's output.

They share one problem. A bar has an obvious time. A record usually has several, and only one of them is safe to use.

## Three timestamps, one of them right

Every record can carry three different times:

1. **The period it describes.** "Q2 earnings", "March inflation".
2. **The event time.** When the company reported, or the agency published.
3. **When you could have had it.** When it reached you, in a form your strategy could read.

Use the third. A figure published on the 5th about the 1st belongs at the 5th. An earnings number released at 16:05 New York time belongs at 16:05 that day, not at the start of it. Joining on the first or second is how a backtest gets to trade on numbers the market had not seen.

## Where the lookahead hides

- **Date-only stamps.** A record stamped with a date and no time is usually read as midnight. An earnings release after the close, stamped with its date, then becomes visible at the start of that day, hours before it existed. On intraday bars, that is a large edge that never existed.
- **Time zones.** A release at 16:05 in New York is 20:05 or 21:05 UTC, depending on daylight saving. Mix local and UTC times and some records move across the day boundary.
- **Scheduled dates.** An earnings calendar is known weeks ahead, so the date itself is fair to use early. Whether it is before the open or after the close, and any change to it, may not be. Store what the calendar said on each day, not what finally happened.
- **Revisions.** Many figures are revised after their first release. A dataset that holds only the latest value gives the backtest numbers nobody had at the time. Use the first-release value, stamped at its release.
- **Reporting lag.** Fundamentals filed weeks after the quarter ends are often stored against the quarter's end date. Stamp them at the filing.
- **Coverage that changes.** A vendor that only covers companies that still exist, or that backfilled history when it added a name, hides the failures from the backtest.

## Joining it to bars

The join should be backward-looking only: for each bar, the latest record stamped at or before that bar's time. In pandas, that is `pd.merge_asof(bars, records, on="ts", direction="backward")`, with both sorted by time and both in UTC.

Three rules make it safe:

- **Never join on nearest.** A "nearest" join will happily take a record from just after the bar.
- **Carry values forward, not back.** Before the first record, the value is missing, not the first record's value.
- **Know what time a bar is stamped with.** If bars carry their opening time, a record released during a bar first appears on the next bar. If they carry their closing time, it appears on the same bar. Either can be right; check it matches what you could do live.

## Checking it

- **Spot-check by hand.** Take twenty records and compare each timestamp with the original release time from the source.
- **Count midnight stamps.** Many records at exactly 00:00 usually means date-only stamps that need real times.
- **Delay and rerun.** Move every record later by an hour, or a bar, and run again. Some loss is normal for a strategy that reacts fast. If the whole edge disappears, check whether it depended on seeing records before you really could.

The engine can stop a record being read before its timestamp. Nothing can stop a wrong timestamp. That part is work on the data, done once and kept.

## How Reamer Research handles it

Everything below is from `EXOGENOUS_FORMAT.md` in the kit:

- **A sidecar per ticker.** You write a two-column CSV, timestamp and value, and convert it with `bin/reamer-exo-build`, which needs no licence. The value is anything your strategy can parse: JSON, a number, packed binary. The engine never reads inside it.
- **Point in time, enforced.** On each bar, a ticker sees the latest entry stamped at or before the bar's timestamp. An entry appears exactly on the bar carrying its own timestamp, never one bar earlier, and stays until a newer one replaces it. Before the first entry, the strategy gets nothing, not a guess.
- **Explicit times.** Timestamps are UTC, with or without a time of day. A date-only stamp means 00:00 UTC, so after-close releases need their real time.
- **Duplicates reported.** If two records share a timestamp, the last one in the file wins, and the converter reports how many were collapsed.
- **Checked before the run.** Every sidecar is opened and validated before the backtest starts. A missing or malformed file fails the run; it never shows up partway through as a gap the strategy trades through.
- **Repeatable.** With the same bars, configuration, sidecars and `rng_seed`, results are byte-identical. See [What makes a backtest deterministic, and why does it matter?](/faq/what-makes-a-backtest-deterministic.html)
- **Cheap per bar.** Each ticker's cursor only moves forward, so finding the current value costs the same on every bar, and a run with no sidecars pays nothing.

The note [Testing earnings strategies with exogenous data](/notes/testing-earnings-strategies-with-exogenous-data.html) goes through the timestamp discipline for an earnings-surprise strategy.

## Limits to know

- **C interface only, for now.** Sidecars are attached through the current entry point, `reamer_research_run_backtest_v3()`, and read in the `on_bar_v4` callback. The Python reference binding calls the previous entry point and does not pass them; it can be extended to. Neither reference implementation includes a worked example.
- **One series per ticker.** Combine several sources into one value, such as a JSON object. Market-wide data, such as a volatility index, is attached to each ticker that reads it.
- **The current value, not a history.** The callback receives the latest visible value for each ticker. If the strategy needs earlier values, it keeps them itself.
- **Your timestamps are trusted.** The engine enforces the order of visibility; it cannot know when your data was really available.
- **No data supplied.** Earnings, fundamentals and every other series are yours to source and stamp.
- **No Windows.** It runs on Linux x86-64 and macOS on Apple Silicon.

The $225 trial includes the full kit, the converter and the format specification, so you can build a sidecar from your own data and check where each value first appears before buying a licence.
