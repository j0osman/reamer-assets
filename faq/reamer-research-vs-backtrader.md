---
title: How does Reamer Research compare with Backtrader?
description: Both run a strategy one bar at a time against historical data. Backtrader is a free, open-source Python framework whose last release was in April 2023; costs and slippage are settings you add. Reamer Research is a paid, compiled engine with a written execution specification, costs modelled by default, byte-identical runs per seed and active maintenance. Backtrader is fine for a first look at an idea; Reamer Research is for results you need to trust and repeat.
stage: 4
order: 35.1
product: research
next: what-is-reamer-research, research-engine-for-mid-frequency-strategies, what-makes-a-backtest-deterministic, pricing
date: 2026-10-08
---

Both run a strategy one bar at a time against historical data, and someone who has written a Backtrader strategy will recognise the shape of a Reamer Research one. The difference is underneath: Backtrader is a free, open-source Python framework where costs are settings you add, and Reamer Research is a paid, compiled engine built around a written specification of how orders fill.

## What Backtrader is

- **Free and open source** (GPL-3.0), written entirely in Python.
- **Widely used.** Years of tutorials and forum answers, and a simple `next()` method that is called on every bar.
- **Flexible.** Many data feeds, indicators, analyzers and older live-trading connectors.
- **Quiet.** The last release on PyPI, 1.9.78.123, came out in April 2023, and open issues and pull requests mostly go unanswered.

## Where they differ

- **Costs.** In Backtrader, commission and slippage are off until you configure them. In Reamer Research, bid and ask, spread, slippage, commission, overnight swap and margin are part of every run, set per instrument. See [How realistic do slippage and spread need to be in a backtest?](/faq/how-realistic-slippage-and-spread.html)
- **Fill rules you can read.** Reamer Research ships a written execution specification covering order types, gaps, order lifetimes and what fills first inside a bar. A deterministic tick path runs through every bar, so a stop and a target in the same bar resolve the same way every time. See [How should a backtest fill a stop and a target that both fall inside one bar?](/faq/stop-and-target-in-same-bar.html)
- **Repeatability with randomised costs.** With a fixed `rng_seed`, output is byte-identical, including randomised spread and slippage.
- **Engine.** Backtrader runs entirely in Python. The Reamer Research engine is compiled C++ with a stable C interface, called from Python, C++ or any language that can call C. Python strategies still pay Python's per-bar cost; the published speed figures are for C++. See [How fast is it?](/faq/performance.html)
- **Maintenance and support.** Reamer Research is maintained, versioned and supported by email from its author.

## When Backtrader is the better choice

- You are learning, or taking a first look at whether an idea has any shape at all.
- You need it to be free.
- You want live trading and market data in the same package as the research tool. With Reamer Labs, live orders go through [Reamer Server](/products/reamer-server.html), a separate product, and the market data is yours.

## When Reamer Research is

When a result will decide where real money goes, and you need to know which costs and fill rules produced it, rerun it to the byte, and sweep parameters around it. It is for systematic strategies on OHLCV bars, held from minutes to days, on Linux x86-64 or macOS on Apple Silicon. See [What is Reamer Research?](/faq/what-is-reamer-research.html)

## Price

Backtrader is free. Reamer Research costs $1,800 per seat per year, with a $225 30-day trial to run your own strategy first. See [pricing](/pricing.html).
