---
title: What are the best kdb+ alternatives in 2026?
description: The main kdb+ alternatives are time-series and columnar databases such as QuestDB, ClickHouse, DolphinDB, TimescaleDB and ArcticDB. They differ in licence, query language and how much market data work they do for you. If you use kdb+ mainly to backtest strategies on bars, you may need an engine and not a database. Reamer Research from Reamer Labs is one, and it can take bars from any of these.
stage: 4
order: 35.93
product: research
next: reamer-research-vs-kdb, onetick-alternatives, dolphindb-alternatives, research-engine-vs-backtest-script, pricing
date: 2026-11-04
topics: backtesting, market-data, quantitative-analysis
---

kdb+ is a column-based time-series database from KX, queried with q and long used for tick data. KDB-X is its next generation, and it adds Python and SQL routes. Most alternatives replace the database. If your real need is a backtest, the alternative may be a different kind of tool.

## Why teams look for kdb+ alternatives

- They want SQL, or Python, and not q.
- They want an open-source licence or a published price.
- Their data is too large for the free KDB-X Community Edition, which is [limited](https://code.kx.com/licensing/usage-restrictions.html) to 16 GB of memory for q.
- They use it mainly to backtest, and have to write the fills and costs themselves.

## kdb+ alternatives compared

Details below were checked against each project's own pages in November 2026.

| Tool | Type | Query language | Best for | Licence |
|---|---|---|---|---|
| QuestDB | Time-series database | SQL | Market data with SQL | Apache 2.0, paid enterprise edition |
| ClickHouse | Columnar database | SQL | General analytics at scale | Apache 2.0, paid cloud and enterprise |
| DolphinDB | Time-series database | Own scripting language | Data and order-book backtests in one system | Proprietary, free community edition with limits |
| TimescaleDB | PostgreSQL extension | SQL | Teams already on PostgreSQL | Apache 2.0 core, Timescale License for other features |
| ArcticDB | Dataframe database | Python | Python research teams | Business Source License |
| OneTick | Data and analytics platform | SQL, Python | Managed data, transaction cost analysis, surveillance | Commercial |
| Reamer Research | Backtesting engine, not a database | Python, C++ or C | Backtests of strategies on bars | $20 per seat per month |

## Databases

- **QuestDB** is open source and names capital markets as a main use. It has SQL, nanosecond timestamps, order book analytics and materialized views from ticks to OHLCV bars. High availability needs the paid edition.
- **ClickHouse** is a general columnar database, so market data functions and conventions are yours to add.
- **DolphinDB** has the same shape as kdb+: a database with its own language. It adds a backtest plugin with a matching-engine simulator. See [How does Reamer Research compare with DolphinDB?](/faq/reamer-research-vs-dolphindb.html)
- **TimescaleDB** extends PostgreSQL, so existing tools and skills carry over.
- **ArcticDB** from Man Group stores dataframes and suits Python teams. Its licence is free for non-commercial use, and firms need a commercial licence.
- **OneTick** is now from KX too, so it is not a change of vendor. See [What are the best OneTick alternatives in 2026?](/faq/onetick-alternatives.html)

## If you use kdb+ to backtest

A database computes on your data. It does not decide how orders fill. KX's own [October 2026 tutorial](https://kx.com/blog/backtesting-a-moving-average-crossover-strategy-with-kdb-x-python/) leaves out commission, spread and slippage, and tells the reader to add them.

[Reamer Research](/products/reamer-research.html) is an engine for that part, for strategies on time, tick, volume, dollar or custom bars. Bid and ask, spread, slippage, commission, swap, margin and order lifetimes are in place under a public [execution specification](/docs/execution-spec.html). It stores no data, so you can keep kdb+ or any database here and pass in bars. It works on bars, not ticks. See [How does Reamer Research compare with kdb+?](/faq/reamer-research-vs-kdb.html)

## How to choose

| If you want | Look at |
|---|---|
| Very large tick data, with q skills on the team | Stay on kdb+ or KDB-X |
| Open source with SQL, for market data | QuestDB |
| General analytics at scale | ClickHouse |
| A database and order-book backtests in one system | DolphinDB |
| To stay on PostgreSQL | TimescaleDB |
| Modelled fills and costs for strategies on bars | Reamer Research, with any database |

See [Who are Reamer Research and Reamer Server for?](/faq/who-its-for.html) Reamer Research is $20 a month, cancel any time; see [pricing](/pricing.html).
