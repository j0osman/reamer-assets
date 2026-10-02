---
title: What is Reamer Research?
description: Reamer Research is a deterministic research engine for mid-frequency quants. It is a backtesting library with a stable C interface that you call from Python, C++ or any language that can call C, on your own machine. The same data, settings and seed give byte-identical results every run. It costs $1,800 per seat per year, with a $225 30-day trial.
stage: 4
order: 24
product: research
next: who-its-for, supported-languages, performance, what-is-reamer-server, pricing
date: 2026-10-02
---

Reamer Research is a deterministic research engine for mid-frequency quants. It is a backtesting library with a stable C interface that you call from Python, C++ or any language that can call C, on your own machine. The same data, settings and seed give byte-identical results every run, including the randomised slippage and spread.

It is a library, not an application. Your program hands it bars, settings and a strategy, and gets back the full result.

## The loop it covers

1. **Test.** Run a strategy against years of OHLCV bars, intraday to multi-day, with spread, slippage, commission and overnight swap set for each instrument, and a leverage limit on the account.
2. **Diagnose.** Every order, fill and closed trade comes back, so any result can be traced trade by trade.
3. **Sweep.** Run the same strategy across many parameter sets to see whether a result holds or survived on one lucky setting. Each backtest is single-threaded, and separate backtests can run at once from your own threads.
4. **Report.** 31 summary metrics and the full trade and order logs, written as a schema-versioned JSON file.
5. **Connect.** The researched logic goes live through [Reamer Server](/products/reamer-server.html), a separate product. Moving over is a port of the logic, not a copy of the code: see [How do I take a strategy from backtest to live trading without a rewrite?](/faq/backtest-to-live-without-rewrite.html)

How fills are decided, including stops and targets inside one bar, is set out in an execution specification that ships with the kit.

## What it is not

- Not for high-frequency or order-book (level 2 or level 3) strategies, and not for options.
- Not a live trading system. It runs backtests in batches; live trading is Reamer Server's job.
- No market data. You bring your own bars, already adjusted for splits and dividends.
- Linux x86-64 and macOS on Apple Silicon only, with no Windows version.

## Price

$1,800 per seat per year, paid once, with no automatic renewal. The $225 30-day trial includes the full kit. See the [product page](/products/reamer-research.html) and [pricing](/pricing.html).
