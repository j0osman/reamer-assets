---
title: How does Reamer Research compare with kdb+?
description: kdb+ and its successor KDB-X from KX are time-series databases with the q language, built for storing and computing on very large market data sets. A backtest on them is something you build, using q or Python. Reamer Research is a backtesting engine for strategies on OHLCV bars, with fill rules, costs and accounting already in place, and it stores no data. If your firm already runs kdb+, the two can work together, with kdb+ supplying the bars.
stage: 4
order: 35.63
product: research
next: kdb-alternatives, reamer-research-vs-onetick, reamer-research-vs-dolphindb, research-engine-vs-backtest-script, market-data, pricing
date: 2026-11-04
topics: backtesting, market-data, quantitative-analysis
---

kdb+ is a time-series database, and Reamer Research is a backtesting engine. kdb+ stores market data and computes on it, and a backtest is something you write on top. Reamer Research supplies the fill rules, costs and accounting, and leaves data storage to you.

Details below were checked against KX's own pages in November 2026.

## What kdb+ and KDB-X are

- **A database and a language.** kdb+ is a column-based time-series database, queried with q, and it has long been used for tick data in capital markets. KDB-X is its next generation from KX, and it adds Python and SQL routes alongside q.
- **Python access.** KDB-X Python, formerly PyKX, works with q data from Python.
- **Streaming and history together.** The same system can hold historical data and process real-time data.
- **Licensing.** The KDB-X Community Edition is free for personal and commercial use. Its [listed limits](https://code.kx.com/licensing/usage-restrictions.html) include 16 GB of memory for q, a single instance, 4 secondary threads per process and 24 CPU cores. Larger use is paid, and KX prices it by memory.
- **Backtesting.** Backtests are built from queries. KX's [October 2026 tutorial](https://kx.com/blog/backtesting-a-moving-average-crossover-strategy-with-kdb-x-python/) works on whole columns, with no loops. It leaves out commission, spread and slippage, and tells the reader to add transaction costs, position sizing and stops. We did not find a published KX fill model for bar strategies.

## How Reamer Research differs

- **The hard parts of the backtest come built.** [Reamer Research](/products/reamer-research.html) has bid and ask, spread, slippage, commission, overnight swap, margin, gaps, order lifetimes and brackets in place. A public [execution specification](/docs/execution-spec.html) states how each one works. See [What is a quant research engine, and how is it different from a backtesting script?](/faq/research-engine-vs-backtest-script.html)
- **Bar by bar, not column by column.** Your strategy runs on each bar and can react to its open position, pending orders and account. This suits rules that are hard to write as a query.
- **An engine, not a database.** You load bars from any source and pass them in as arrays, or use memory-mapped `.bin` files for data larger than RAM. It has no query language and no streaming data store.
- **Your own language.** Strategies are written in Python or C++, or any language that can call C. You do not need to learn q.
- **Byte-identical runs.** With a fixed seed, output matches to the byte, including randomised spread and slippage.
- **Built for bars.** It handles OHLCV bars, and it runs on Linux x86-64 and on macOS on Apple Silicon.

## Using both

If your firm already runs kdb+, you do not have to choose. Query bars from kdb+ in Python, convert the columns to arrays, and pass them to Reamer Research. kdb+ stays the data store, and the fill rules and accounting stop being code your team has to maintain. You write that loader yourself.

## Which to choose

- **kdb+ or KDB-X** if you store and query large tick data sets, work on order book signals, or need real-time analytics.
- **Reamer Research** if you trade strategies on bars, held from minutes to days, and want modelled fills and costs without writing and defending your own simulator.
- **Both** if kdb+ is already your data layer and you want the simulator to be someone else's maintained code.

## Price

Reamer Research costs $1,800 per seat per year, with a $225 30-day trial. See [pricing](/pricing.html).
