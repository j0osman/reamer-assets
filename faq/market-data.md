---
title: Does it include market data?
description: No. Neither Reamer Research nor Reamer Server includes market data, but Reamer Research ships fetchers that pull historical bars from your own Alpaca or Databento account straight into a backtest. Otherwise you bring your own bars of any type in OHLCV format, as arrays from any source or as memory-mapped files built from CSV, and equities must already be adjusted for splits and dividends. The engine checks that bars are well formed, not that the data is right. Reamer Server takes orders from your strategies, which get their prices from your own feed.
stage: 4
order: 29
product: both
next: tick-volume-dollar-bars, integration-time, results-and-metrics, paper-trading, performance
date: 2026-10-02
topics: market-data, ohlc
---

No. Neither Reamer Research nor Reamer Server includes market data, but Reamer Research ships fetchers that pull historical bars from your own Alpaca or Databento account straight into a backtest. Otherwise you bring your own bars of any type in OHLCV format, as arrays from any source or as memory-mapped files built from CSV, and equities must already be adjusted for splits and dividends. The engine checks that bars are well formed, not that the data is right. Reamer Server takes orders from your strategies, which get their prices from your own feed.

The only data in the kit is a 100-bar daily sample, so the quickstart runs on its first try.

## Getting bars into Reamer Research

- **From Alpaca or Databento.** `reference-python/fetchers/` downloads historical bars from your own Alpaca account (US stocks and crypto) or Databento account (OHLCV, including continuous futures) straight into a backtest, and `examples/fetch_and_run.py` does it in one command. Alpaca crypto needs no account. The engine itself makes no network calls. Alpaca stock bars arrive adjusted for splits and dividends. Each Databento request is priced first, and one over your limit is refused.
- **Arrays from any source.** Load bars from CSV with pandas or numpy, a database or a vendor API, and pass them in as arrays: timestamp, open, high, low, close and volume for each bar. This works from Python, C++ or any language that can call C.
- **`.bin` files.** `bin/reamer-csv-build AAPL.csv AAPL.bin` converts a CSV. It detects the delimiter, columns and timestamp format, and reports what it skipped. Timestamps are whole seconds, so two rows at the same timestamp stop it unless `--keep-first` keeps the first bar of each second. It needs no license. Through the C interface, `reamer_run_backtest_files()` memory-maps these files, so a dataset larger than RAM runs, and `start_ts` and `end_ts` select a date range without rewriting them. The Python binding takes arrays.
- **One instrument per file.** If a CSV holds several symbols, the converter stops until `--symbol` selects one.
- **Any bar type.** Time, tick, volume, dollar, imbalance or your own custom rule. You build the bars, and the engine treats every type the same way. OHLCV is the record format, not a limit on how a bar was built.
- **Several instruments at once.** Each instrument has its own series, and timestamps do not need to line up across them. Every instrument is cut to the range they all share, so the shortest series sets the run's length.
- **Non-price data,** such as earnings dates or an index level, can be attached to each instrument through the C interface. A bundled tool converts it from a two-column CSV. The Python binding does not pass it.

## What stays your job

- **Data quality.** A bar with a NaN or infinite price stops the run with an error. Wrong prices, missing bars or bad prints do not; they produce wrong results.
- **Corporate actions.** Unadjusted equity data gives wrong results with no warning.
- **Futures rolls.** Supply a continuous series. If it is unadjusted, you can declare the roll months through the C interface, and the engine stops treating the jump at each roll as a real price move.

See [How do I backtest a strategy that uses earnings dates or other non-price data?](/faq/backtest-with-exogenous-data.html) and [What is Reamer Research?](/faq/what-is-reamer-research.html)
