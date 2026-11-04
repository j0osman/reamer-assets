---
title: What are the best backtesting.py alternatives in 2026?
description: backtesting.py is a good small library for testing one strategy on one instrument, so the best alternative depends on what you have outgrown. For several instruments in one strategy, look at Backtrader or bt. For large parameter sweeps, vectorbt. For ticks, order books and live trading, NautilusTrader or LEAN. For strategies on bars with a written fill specification and modelled costs, Reamer Research is a paid engine library from Reamer Labs. If one instrument is enough, backtesting.py is still maintained and may be all you need.
stage: 4
order: 35.9
product: research
next: backtrader-alternatives, python-backtest-too-slow, what-is-reamer-research, how-realistic-slippage-and-spread, pricing
date: 2026-11-04
topics: backtesting, algorithmic-trading
---

backtesting.py is a free Python library with a small API, interactive charts and a built-in optimizer. It was last released in July 2026, so age is not a reason to leave it. People look elsewhere when a strategy grows past what it is built for.

## Why traders look for backtesting.py alternatives

- **One strategy, one instrument.** Each `Backtest` takes one dataset. The request to trade several instruments together has been [open since 2019](https://github.com/kernc/backtesting.py/issues/20), and `MultiBacktest` runs a separate backtest per instrument, not a portfolio.
- **Simple costs.** You set a commission and one constant spread rate. `Backtest()` has no slippage setting, and spread does not change with the bar or the time of day.
- **One rule inside the bar.** When a stop-loss and a take-profit both fall inside one bar, the stop-loss is processed first. That is a cautious rule, but it is a fixed assumption, not a model of the bar.
- **Research only.** Its documentation does not cover live trading.
- **The licence.** It is AGPL-3.0, so check the terms if you plan to ship software built on it.

## backtesting.py alternatives compared

Details below were checked against each project's own pages in November 2026.

| Tool | Type | Best for | Several instruments in one strategy | Live trading | Price |
|---|---|---|---|---|---|
| Backtrader | Open-source framework (GPL-3.0) | Learning, event-driven strategies | Yes | Older connectors | Free |
| bt | Open-source framework (MIT) | Portfolio weights and rebalancing | Yes | None advertised | Free |
| vectorbt, vectorbt PRO | Python libraries | Large parameter sweeps | Yes | Signals only; you supply execution | Open-source edition free, PRO paid |
| NautilusTrader | Open-source platform (LGPL-3.0) | Ticks, order books, many venues | Yes | Adapters for crypto venues, Interactive Brokers and others | Free |
| LEAN | Open-source engine (Apache 2.0) | Many asset classes, QuantConnect's model | Yes | Self-run | Free |
| Zipline Reloaded | Open-source engine (Apache 2.0) | Existing Zipline code | Yes | None advertised | Free |
| Reamer Research | Paid local library | Strategies on bars, with written fill rules and modelled costs | Yes | Reamer Server, sold separately, with your own connector | $1,800 per year |

## Free Python frameworks

- **Backtrader** has many tutorials and supports several data feeds in one strategy. Its last PyPI release was in April 2023, and backtesting.py's own [list of alternatives](https://github.com/kernc/backtesting.py/blob/master/doc/alternatives.md) files it under obsolete or unmaintained. See [What are the best Backtrader alternatives in 2026?](/faq/backtrader-alternatives.html) for where to go if that matters to you.
- **bt** builds a strategy from small allocation blocks. It suits portfolio weights and rebalancing better than order-level trading rules.
- **vectorbt** tests large parameter grids on whole arrays at once. Rules that depend on the current position or on bracket orders are harder to express that way.
- **Zipline Reloaded** is the community-kept successor to Quantopian's engine. Its maintainers now fix bugs and compatibility problems, and point new projects to ML4T Backtest.

## Larger platforms

- **NautilusTrader** has a Rust core and Python strategies, and the same code runs in backtest and live, down to ticks and order books. It is more to learn than backtesting.py.
- **LEAN** is the engine behind QuantConnect. You can run it locally with your own data, and strategies can be written in Python or C#.

## Paid engine library

[Reamer Research](/products/reamer-research.html) from Reamer Labs is a compiled engine you call from Python, C++ or any language that can call C. It differs from backtesting.py in these ways:

- Several instruments share one account and one timeline.
- Spread, slippage, commission, overnight swap and margin are set per instrument. With volatility enabled, spread and slippage scale with each bar's range.
- Fill rules are written down in a public [execution specification](/docs/execution-spec.html). A seeded tick path runs through every bar, so a stop and a target in one bar resolve the same way on every run.
- Each run returns every order, fill and closed trade.

It covers strategies on OHLCV bars, and runs on Linux x86-64 and on macOS on Apple Silicon. You bring your own data. There are no built-in charts or optimizer: a sweep is your own loop over runs, and separate runs can use every core.

## What a published benchmark measured

backtesting.py was not tested in [What a Backtest Costs](https://doi.org/10.6084/m9.figshare.34787745), so this page gives no speed figures for it. The study ran Backtrader, NautilusTrader, LEAN and Reamer Research on the same strategies and data. See [What are the best Backtrader alternatives in 2026?](/faq/backtrader-alternatives.html) for the results.

## How to choose

| If you want | Look at |
|---|---|
| One strategy on one instrument, with charts, for free | Stay on backtesting.py |
| Several instruments in one strategy, for free | Backtrader or LEAN |
| Portfolio weights and rebalancing | bt |
| Large parameter sweeps in Python | vectorbt |
| Ticks, order books or ready venue adapters | NautilusTrader |
| Strategies on bars, with several instruments, modelled costs and fill rules you can read | Reamer Research |

Reamer Research costs $1,800 per seat per year, with a $225 30-day trial to run your own strategy first. See [pricing](/pricing.html).
