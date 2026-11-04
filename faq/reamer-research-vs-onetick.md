---
title: How does Reamer Research compare with OneTick?
description: OneTick, now part of KX, is a tick data and analytics platform with managed market data from over 200 venues, order book history, transaction cost analysis and trade surveillance, sold to institutions without a public price list. Reamer Research is a backtesting engine for strategies on OHLCV bars, with a written execution specification, byte-identical runs and published prices. It holds no market data. The two do different jobs, and bars from OneTick can feed Reamer Research.
stage: 4
order: 35.62
product: research
next: onetick-alternatives, reamer-research-vs-kdb, reamer-labs-vs-deltix, market-data, who-its-for, pricing
date: 2026-11-04
topics: backtesting, market-data, transaction-cost
---

OneTick is a tick data and analytics platform. Reamer Research is a backtesting engine for strategies on OHLCV bars. OneTick answers what happened in the market, down to individual orders. Reamer Research answers what your strategy would have earned on bars, under fill rules you can read.

Details below were checked against the vendors' own pages in November 2026.

## What OneTick is

- **Part of KX.** OneMarketData, the owner of OneTick, [merged with KX](https://kx.com/news-room/kx-and-onetick-merge-to-unite-capital-markets-data-analytics-ai-and-surveillance-on-one-platform/) in September 2025.
- **Managed market data.** OneTick Cloud offers normalized, cleansed exchange data from over 200 venues, including Level 1, Level 2 and Level 3 order book data, updated on a T+1 basis.
- **Analytics on ticks.** It runs historical and real-time queries from SQL or Python, with `onetick-py` as a pandas-like Python API. It can rebuild the order book to any depth.
- **Transaction cost analysis and surveillance.** Best execution, MiFID II and trade surveillance products sit alongside the data platform.
- **Backtesting.** OneTick lists backtesting as a use case and says it can act as an exchange simulator and strategy engine. Its recent writing presents it mainly as the data foundation for backtesting. We did not find public detail of its fill rules.
- **Built for institutions.** It names proprietary traders, hedge funds and investment banks as users. OneTick Cloud has a 30-day free trial, and prices come through sales.

## How Reamer Research differs

- **An engine, not a data platform.** [Reamer Research](/products/reamer-research.html) holds no market data. You load bars from any source and pass them in as arrays. See [Does it include market data?](/faq/market-data.html)
- **Bars, not order books.** It models what happens inside a bar on a deterministic tick path. It does not replay order books or simulate a queue, so order-book and high-frequency strategies are out of scope.
- **Fill rules in writing.** The public [execution specification](/docs/execution-spec.html) covers spread, slippage, commission, overnight swap, margin, gaps, order lifetimes and what fills first inside a bar.
- **Byte-identical runs.** With a fixed seed, output matches to the byte, including randomised spread and slippage.
- **Your strategy in your language.** Strategies run in Python or C++, or any language that can call C, on each bar. Each run returns every order, fill and closed trade.
- **One machine, published prices.** It runs on one Linux x86-64 machine or Apple Silicon Mac, offline after activation, and you buy it self-serve. There is no Windows version.

## Using both

The two can work together. Build bars from OneTick data, adjusted as your instruments need, and pass them to Reamer Research as arrays or write them to `.bin` files. OneTick stays the source of market data and execution analysis, and Reamer Research runs the strategy. You write that loader yourself.

## Which to choose

- **OneTick** if you need managed tick history across many venues, order book research, transaction cost analysis or surveillance, or if your strategies depend on the order book.
- **Reamer Research** if your strategies run on bars, held from minutes to days, you already have data, and you want a backtest whose fill rules you can read and rerun to the byte. See [Who are Reamer Research and Reamer Server for?](/faq/who-its-for.html)

## Price

Reamer Research costs $1,800 per seat per year, with a $225 30-day trial. See [pricing](/pricing.html).
