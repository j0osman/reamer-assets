---
title: How does Reamer Research compare with DolphinDB?
description: DolphinDB is a time-series database with its own scripting language, streaming and a backtest plugin that simulates order matching down to tick-by-tick and Level-2 data. It suits teams that want to store and compute on large market-data sets in one system. Reamer Research is a backtesting engine only, for strategies on OHLCV bars, with a written execution specification and byte-identical runs, driven from Python, C++ or C. You bring your own data and keep it however you like.
stage: 4
order: 35.6
product: research
next: what-is-reamer-research, market-data, supported-languages, who-its-for, pricing
date: 2026-10-08
---

DolphinDB is a time-series database with its own scripting language, stream processing and a backtest plugin that simulates order matching down to tick-by-tick and Level-2 data. Reamer Research is a backtesting engine only, for strategies on OHLCV bars. DolphinDB is a data platform that can also backtest. Reamer Research does one job and leaves data storage to you.

## What DolphinDB is

- **A database first.** Distributed storage and computing for time series, queried with a SQL-compatible scripting language that has over 2,000 built-in functions.
- **Streaming.** Historical data can be replayed through stream tables, so the same script can serve backtesting and real-time use.
- **A backtest plugin.** Event-driven callbacks on bars, snapshots and trades, with a matching-engine simulator that follows price-time priority. It works on daily, minute, snapshot and tick-by-tick data, across stocks, futures, options and crypto.
- **Editions.** A free community edition, limited to 2 nodes with 2 CPU cores and 8 GB of memory each, and commercial editions for larger deployments. Cloud, on-premises or hybrid.

## How Reamer Research differs

- **An engine, not a database.** [Reamer Research](/products/reamer-research.html) is a library with a stable C interface. Your program hands it bars from CSV files, settings and a strategy, and gets back the full result. Where you store data is up to you. See [Does it include market data?](/faq/market-data.html)
- **Your own language.** Strategies are written in Python or C++, or any language that can call C, not in a database's scripting language. See [Which languages can I write strategies in?](/faq/supported-languages.html)
- **Bars, not order books.** It models what happens inside a bar on a deterministic tick path, but it does not replay Level-2 data or simulate a queue. Order-book and high-frequency strategies are out of scope.
- **Fill rules in writing.** An execution specification ships with the kit and covers bid and ask, spread, slippage, commission, overnight swap, margin, gaps, order lifetimes and what fills first inside a bar.
- **Byte-identical runs.** With a fixed `rng_seed`, output is identical to the byte, including randomised spread and slippage.
- **One machine.** It runs on one Linux x86-64 or Apple Silicon Mac, offline after activation. No cluster to run.

## Which to choose

- **DolphinDB** if you need to store and compute on large tick or Level-2 data sets, want research, factor computation and backtesting in one database, or trade at frequencies where order-book matching matters.
- **Reamer Research** if your strategies run on bars, held from minutes to days, you already keep your data somewhere, and you want a backtest whose fill rules you can read and rerun to the byte, from the language you already use. See [Who are Reamer Research and Reamer Server for?](/faq/who-its-for.html)

## Price

Reamer Research costs $1,800 per seat per year, with a $225 30-day trial. See [pricing](/pricing.html).
