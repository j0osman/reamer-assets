---
title: What is Reamer Research?
description: Reamer Research is a deterministic research engine for mid-frequency quants. It is a library with a stable C interface that you call from Python, C++ or any language that can call C, on your own machine. It tests a strategy, diagnoses it trade by trade, sweeps it, reports it, and sends its orders live to Reamer Server. The same data, settings and seed give byte-identical results every run. It costs $1,800 per seat per year, with a $225 30-day trial.
stage: 4
order: 24
product: research
next: who-its-for, supported-languages, performance, what-is-reamer-server, pricing
date: 2026-10-02
---

Reamer Research is a deterministic research engine for mid-frequency quants. It is a library with a stable C interface that you call from Python, C++ or any language that can call C, on your own machine. The same data, settings and seed give byte-identical results every run, including the randomised slippage and spread.

It is a library, not an application. Your program hands it bars, settings and a strategy, and gets back the full result.

## The loop it covers

1. **Test.** Run a strategy against years of OHLCV bars, intraday to multi-day, with spread, slippage, commission and overnight swap set for each instrument, and a leverage limit on the account. Bars come from memory or from memory-mapped `.bin` files, so a dataset larger than RAM runs. On every bar the strategy sees its full position state and every resting order, and can re-price an order, move a position's stop and target, close everything or cancel everything.
2. **Diagnose.** Every order, fill, modification and closed trade comes back, so any result can be traced trade by trade.
3. **Sweep.** Run the same strategy across many parameter sets to see whether a result holds or survived on one lucky setting. Each backtest is single-threaded, and separate backtests can run at once from your own threads. A progress callback reports each run and can stop it early. `reamer_run_monte_carlo()` resamples a run's trades to show how much of a drawdown was trade-order luck.
4. **Report.** 31 summary metrics, the realised equity curve, and the full trade and order logs, written as a schema-versioned JSON file.
5. **Connect.** `reamer_relay_*` sends the orders your strategy returns to a running [Reamer Server](/products/reamer-server.html), a separate product, and reads the fills back as position state. The order is the same structure in both places. See [How do I take a strategy from backtest to live trading without a rewrite?](/faq/backtest-to-live-without-rewrite.html)

How fills are decided, including stops and targets inside one bar, is set out in an execution specification that ships with the kit.

## Scope

- Mid-frequency strategies on OHLCV bars. Order-book (level 2 or level 3), high-frequency and options strategies are a different kind of engine.
- Batch runs over historical bars. Live order handling, sequencing and your pre-trade gate run in Reamer Server, which the relay connects to.
- Your own data. You bring your own bars, already adjusted for splits and dividends.
- Linux x86-64 and macOS on Apple Silicon.

## How it compares

- [Backtrader](/faq/reamer-research-vs-backtrader.html), a free, open-source Python framework.
- [vectorbt PRO](/faq/reamer-research-vs-vectorbt-pro.html), a paid vectorised research library for Python.
- [DolphinDB](/faq/reamer-research-vs-dolphindb.html), a time-series database with a backtest plugin.
- [QuantConnect](/faq/reamer-labs-vs-quantconnect.html), a hosted platform with data, backtesting and live trading.
- [NautilusTrader](/faq/reamer-labs-vs-nautilustrader.html), an open-source platform for backtest and live, down to order books.
- [EPAM Deltix QuantOffice](/faq/reamer-labs-vs-deltix.html), an enterprise platform from research to execution.

## Price

$1,800 per seat per year, paid once, with no automatic renewal. The $225 30-day trial includes the full kit. See the [product page](/products/reamer-research.html) and [pricing](/pricing.html).
