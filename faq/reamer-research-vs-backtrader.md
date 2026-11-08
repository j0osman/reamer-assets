---
title: How does Reamer Research compare with Backtrader?
description: Both run a strategy one bar at a time against historical data. Backtrader is a free, open-source Python framework whose last release was in April 2023; costs and slippage are settings you add. Reamer Research is a paid, compiled engine with a written execution specification, costs modelled by default, byte-identical runs per seed and active maintenance. In a published four-engine benchmark, 500,000 bars took Backtrader 15.04 s and Reamer Research 3.38 s from Python. Backtrader is fine for a first look at an idea; Reamer Research is for results you need to trust and repeat.
stage: 4
order: 35.1
product: research
next: what-is-reamer-research, research-engine-for-mid-frequency-strategies, what-makes-a-backtest-deterministic, pricing
date: 2026-10-08
topics: backtesting
---

Both run a strategy one bar at a time against historical data, and someone who has written a Backtrader strategy will recognise the shape of a Reamer Research one. The difference is underneath: Backtrader is a free, open-source Python framework where costs are settings you add, and Reamer Research is a paid, compiled engine built around a written specification of how orders fill.

## What Backtrader is

- **Free and open source** (GPL-3.0), written entirely in Python.
- **Years of tutorials.** Forum answers, and a simple `next()` method that is called on every bar.
- **Flexible.** Many data feeds, indicators, analyzers and older live-trading connectors.
- **Quiet.** The last release on PyPI, 1.9.78.123, came out in April 2023, and open issues and pull requests mostly go unanswered.

## Where they differ

- **Costs.** In Backtrader, commission and slippage are off until you configure them. In Reamer Research, bid and ask, spread, slippage, commission, overnight swap and margin are part of every run, set per instrument. See [How realistic do slippage and spread need to be in a backtest?](/faq/how-realistic-slippage-and-spread.html)
- **Fill rules you can read.** Reamer Research ships a written execution specification covering order types, gaps, order lifetimes and what fills first inside a bar. A deterministic tick path runs through every bar, so a stop and a target in the same bar resolve the same way every time. See [How should a backtest fill a stop and a target that both fall inside one bar?](/faq/stop-and-target-in-same-bar.html)
- **Repeatability with randomised costs.** With a fixed `rng_seed`, output is byte-identical, including randomised spread and slippage.
- **Engine.** Backtrader runs entirely in Python. The Reamer Research engine is compiled C++ with a stable C interface, called from Python, C++ or any language that can call C. Python strategies still pay Python's per-bar cost, so the C++ path is about nine times faster than the Python path at 500,000 bars. See [How fast is it?](/faq/performance.html)
- **Maintenance and support.** Reamer Research is maintained, versioned and supported by email from its author.

## What a published benchmark measured

[What a Backtest Costs](https://doi.org/10.6084/m9.figshare.34787745) ran Backtrader, NautilusTrader, LEAN and Reamer Research on the same strategies and data, on one AMD EPYC 9575F on 2026-10-25. There was no commission, spread or slippage, and the realistic strategy was a 20-bar breakout. The study's author builds and sells Reamer Research, so the [runners and raw results are public](https://github.com/j0osman/research-engine-benchmark) and you can check every figure.

- **One backtest, 500,000 bars.** Backtrader took 15.04 s. Reamer Research took 3.38 s from Python and 0.36 s from C++.
- **100 tickers of 35,064 bars each.** Backtrader took 94.82 s. Reamer Research took 14.85 s from Python.
- **64 copies at once.** Backtrader finished 11,125 backtests per hour. Reamer Research finished 42,468 from Python and 332,249 from C++.
- **Memory went Backtrader's way in places.** On a strategy that never trades, Backtrader used 209 MB against 309 MB for Reamer Research's Python path at 500,000 bars, and 513 MB against 739 MB at 100 tickers. With trades, the two tied at 309 MB at 500,000 bars.
- **Parallel efficiency.** At 64 copies Backtrader kept 0.72 of its single-copy speed per copy. Reamer Research kept 0.63 from Python and 0.53 from C++, the lowest of the engines, though its total throughput was still rising.
- **The results differ by fill price.** The two made the same trades. Backtrader fills at the next bar's open and ended at −1,956.83. Reamer Research fills at the first tick of a seeded path through the next bar and ended at −1,937.16. Repricing its fills at the open gives Backtrader's figure exactly.

Both engines repeated every result. This is one machine, one date, one instrument and two simple strategies. Heavy indicators or a model inside the loop would narrow the speed differences, because your strategy's own time counts in both engines.

## When Backtrader is the better choice

- You are learning, or taking a first look at whether an idea has any shape at all.
- You need it to be free.
- You want an engine written entirely in Python that you can read and change.
- You want live trading and market data in the same package as the research tool. With Reamer Labs, live orders go through [Reamer Server](/products/reamer-server.html), a separate product, and the market data is yours.

## When Reamer Research is

When a result will decide where real money goes, and you need to know which costs and fill rules produced it, rerun it to the byte, and sweep parameters around it. It is for systematic strategies on time, tick, volume, dollar or custom bars, held from minutes to days, on Linux x86-64, macOS on Apple Silicon or Windows through WSL2. See [What is Reamer Research?](/faq/what-is-reamer-research.html)

## Price

Backtrader is free. Reamer Research costs $1,800 per seat per year, with a $225 30-day trial to run your own strategy first. See [pricing](/pricing.html).
