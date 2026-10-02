---
title: Does it include market data?
description: No. Neither Reamer Research nor Reamer Server includes market data. For Reamer Research you bring your own OHLCV bars, loaded from CSV or any source, and equities must already be adjusted for splits and dividends. The engine checks that bars are well formed, not that the data is right. Reamer Server takes orders from your strategies, which get their prices from your own feed.
stage: 4
order: 29
product: both
next: integration-time, results-and-metrics, paper-trading, performance
date: 2026-10-02
---

No. Neither Reamer Research nor Reamer Server includes market data. For Reamer Research you bring your own OHLCV bars, loaded from CSV or any source, and equities must already be adjusted for splits and dividends. The engine checks that bars are well formed, not that the data is right. Reamer Server takes orders from your strategies, which get their prices from your own feed.

The only data in the kit is a 100-bar daily sample, so the quickstart runs on its first try.

## Getting bars into Reamer Research

- **Any source, any loader.** The engine reads no files. You load bars yourself, from CSV with pandas or numpy, a database or a vendor API, and pass them in as arrays: timestamp, open, high, low, close and volume for each bar.
- **Any bar type.** Time bars are the usual case. Tick, volume or dollar bars work too if you build them first.
- **Several instruments at once.** Each instrument has its own series, and timestamps do not need to line up across them.
- **Non-price data,** such as earnings dates or an index level, can be attached to each instrument through the C interface. A bundled tool converts it from a two-column CSV. The Python binding does not pass it.

## What stays your job

- **Data quality.** A bar with a NaN or infinite price stops the run with an error. Wrong prices, missing bars or bad prints do not; they produce wrong results.
- **Corporate actions.** Unadjusted equity data gives wrong results with no warning.
- **Futures rolls.** Supply a continuous series. If it is unadjusted, you can declare the roll months through the C interface, and the engine stops treating the jump at each roll as a real price move.

See [How do I backtest a strategy that uses earnings dates or other non-price data?](/faq/backtest-with-exogenous-data.html) and [What is Reamer Research?](/faq/what-is-reamer-research.html)
