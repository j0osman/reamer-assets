---
title: How does Reamer Research compare with vectorbt PRO?
description: vectorbt PRO is a vectorised Python library, very fast at testing thousands of parameter combinations of a strategy whose shape is already fixed. Reamer Research is an event-driven engine that steps through time with order-level fill rules from a written specification, for checking whether a strategy survives realistic costs in the first place. Some researchers use both.
stage: 4
order: 35.2
product: research
next: what-is-reamer-research, is-my-backtest-overfit, how-realistic-slippage-and-spread, pricing
date: 2026-10-08
---

vectorbt PRO is a vectorised Python library that is very fast at testing thousands of parameter combinations of a strategy whose shape is already fixed. Reamer Research is an event-driven engine that steps through time and fills each order by a written specification. They answer different questions, and some researchers use both.

## What vectorbt PRO is

- **A Python library** built on NumPy, pandas and Numba. It works on whole price histories as arrays, so one call can run a large grid of parameters at once.
- **Fast at sweeps.** Few tools test 500 lookback lengths of a breakout rule faster.
- **Broad.** Indicators, signal generation, portfolio simulation, plotting and analysis in one package.
- **A paid membership** (monthly, yearly or lifetime) that gives access to a private repository. The older open-source vectorbt is free.

## Where they differ

- **How a strategy is written.** In vectorbt PRO you usually express a strategy as arrays of signals, or as Numba-compiled callbacks. In Reamer Research you write ordinary code that is called once per time step with every instrument's recent bars and current positions, and returns orders.
- **How orders fill.** Reamer Research ships a written execution specification: bid and ask, spread and slippage, commission, overnight swap, margin, gaps, order lifetimes, and what fills first inside a bar on a deterministic tick path. The engine is tested against that document.
- **Repeatability.** With a fixed `rng_seed`, Reamer Research output is byte-identical, including randomised spread and slippage.
- **Language.** vectorbt PRO is Python. Reamer Research is a compiled library with a stable C interface, called from Python, C++ or any language that can call C.

## Why order matters

A sweep assumes the strategy's shape is worth sweeping. Whether it survives realistic costs and fills comes first. A fast sweep over a shape that never held up only finds the best-fitting version of an idea that was not real. See [How do I know if my backtest is overfit?](/faq/is-my-backtest-overfit.html)

Reamer Research can also sweep: each backtest is single-threaded, and separate backtests can run at the same time from your own threads, so a sweep uses every core. See [How fast is it?](/faq/performance.html)

## Which to choose

- **vectorbt PRO** if the shape is settled and the job is exploring a large parameter space quickly in Python, or if you want analysis and plotting in the same library.
- **Reamer Research** if you need order-level fills you can read, costs modelled the way your broker charges them, and results you can rerun to the byte. It is for systematic strategies on OHLCV bars, held from minutes to days. See [What is Reamer Research?](/faq/what-is-reamer-research.html)
- **Both**, if you validate a shape in Reamer Research and then sweep it at scale in vectorbt PRO.

## Price

Reamer Research costs $1,800 per seat per year, with a $225 30-day trial. See [pricing](/pricing.html).
