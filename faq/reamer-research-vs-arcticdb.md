---
title: How does Reamer Research compare with ArcticDB?
description: ArcticDB from Man Group is a dataframe database for Python, built for storing and reading very large time-series data sets. A backtest on it is something you write in Python. Reamer Research is a backtesting engine for strategies on time, tick, volume, dollar or custom bars, with fill rules, costs and accounting already in place, and it stores no data. If you already keep your data in ArcticDB, the two can work together, with ArcticDB supplying the bars.
stage: 4
order: 35.66
product: research
next: arcticdb-alternatives, reamer-research-vs-kdb, research-engine-vs-backtest-script, market-data, pricing
date: 2026-11-08
topics: backtesting, market-data, quantitative-analysis
---

ArcticDB is a database, and Reamer Research is a backtesting engine. ArcticDB stores dataframes and reads them back quickly, and a backtest is something you write on top. Reamer Research supplies the fill rules, costs and accounting, and leaves data storage to you.

Details below were checked against ArcticDB's own pages in November 2026.

## What ArcticDB is

- **A dataframe database for Python.** ArcticDB reads and writes pandas dataframes and NumPy arrays from Python. Man Group built it, and Bloomberg has integrated it into its BQuant platform.
- **No database server.** It runs inside your Python process and keeps data in storage you choose: S3 or a compatible store, Azure Blob Storage, or local LMDB files.
- **Versions and snapshots.** You can read earlier versions of your data and keep named snapshots, which helps when you need to repeat research on the data as it was.
- **Processing on read.** You can filter, aggregate and resample as you read, for example to build bars from finer data.
- **Licensing.** ArcticDB uses the Business Source License 1.1. It is [free for non-commercial, personal or academic use](https://docs.arcticdb.io/dev/licensing/). Business use, including research and development, needs a commercial licence from ArcticDB Limited, priced by quote. Each version converts to Apache 2.0 two years after release.
- **Backtesting.** ArcticDB has no fill model, cost model or order handling. Your backtest reads data from it and does the rest in your own code.

## How Reamer Research differs

- **The hard parts of the backtest come built.** [Reamer Research](/products/reamer-research.html) has bid and ask, spread, slippage, commission, overnight swap, margin, gaps, order lifetimes and brackets in place. A public [execution specification](/docs/execution-spec.html) states how each one works. See [What is a quant research engine, and how is it different from a backtesting script?](/faq/research-engine-vs-backtest-script.html)
- **Bar by bar.** Your strategy runs on each bar and can react to its open position, pending orders and account. Several instruments share one wall clock and one account.
- **An engine, not a database.** You load bars from any source and pass them in as arrays, or use memory-mapped `.bin` files for data larger than RAM. It has no versioning and no storage of its own.
- **Your own language.** Strategies are written in Python or C++, or any language that can call C.
- **Byte-identical runs.** With a fixed seed, output matches to the byte, including randomised spread and slippage.
- **Platforms.** It runs on Linux x86-64 and on macOS on Apple Silicon. ArcticDB also runs on Windows, and Reamer Research does not.

## Using both

If your data already lives in ArcticDB, you do not have to move it. Read each symbol into a dataframe, convert the columns to arrays, and pass them to Reamer Research. Timestamps go in as whole seconds, so convert ArcticDB's nanosecond index first. Rows for different symbols do not need to line up. ArcticDB stays the data store, and the fill rules and accounting stop being code your team has to maintain. You write that loader yourself.

## Which to choose

- **ArcticDB** if you need to store, version and read large time-series data sets from Python.
- **Reamer Research** if you trade strategies on bars, held from minutes to days, and want modelled fills and costs without writing and defending your own simulator.
- **Both** if ArcticDB is already your data layer and you want the simulator to be someone else's maintained code.

## Price

Reamer Research costs $1,800 per seat per year, with a $225 30-day trial. See [pricing](/pricing.html).
