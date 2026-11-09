---
title: What are the best Backtrader alternatives in 2026?
description: Backtrader's last release was in April 2023. backtesting.py is the simplest step for one instrument. bt suits portfolio rebalancing, vectorbt suits parameter sweeps, and NautilusTrader and LEAN cover ticks, more asset classes and live trading. Reamer Research from Reamer Labs is a paid engine library for strategies on bars, and it keeps Backtrader's bar-by-bar shape. In a published benchmark on 500,000 bars, Backtrader took 15.04 s and Reamer Research took 3.38 s from Python.
stage: 4
order: 35.91
product: research
next: backtesting-py-alternatives, reamer-research-vs-backtrader, python-backtest-too-slow, what-is-reamer-research, pricing
date: 2026-11-04
topics: backtesting, algorithmic-trading, benchmark
---

Backtrader is a free Python framework with years of tutorials and a simple `next()` method that runs on every bar. It has not had a release since April 2023.

## Why traders look for Backtrader alternatives

- **Maintenance.** The last PyPI release, 1.9.78.123, came out in April 2023. The package metadata lists Python versions only up to 3.7, and backtesting.py's own [list of alternatives](https://github.com/kernc/backtesting.py/blob/master/doc/alternatives.md) files it under obsolete or unmaintained.
- **Costs that are off by default.** Its default broker starts with zero slippage, and commission is something you configure.
- **Older live connectors.** Live trading through Backtrader relies on connectors from its active years.
- **Speed on big runs.** It runs entirely in Python, which adds up across long intraday histories and parameter sweeps.

## Backtrader alternatives compared

Details below were checked against each project's own pages in November 2026.

| Tool | Type | Best for | Last release | Live trading | Price |
|---|---|---|---|---|---|
| backtesting.py | Open-source library (AGPL-3.0) | One strategy on one instrument | July 2026 | Not covered in its documentation | Free |
| bt | Open-source framework (MIT) | Portfolio weights and rebalancing | October 2026 | None advertised | Free |
| vectorbt, vectorbt PRO | Python libraries | Large parameter sweeps | October 2026 (open-source edition) | Signals only; you supply execution | Open-source edition free, PRO paid |
| NautilusTrader | Open-source platform (LGPL-3.0) | Ticks, order books, many venues | August 2026 (stable) | Adapters for crypto venues, Interactive Brokers and others | Free |
| LEAN | Open-source engine (Apache 2.0) | Many asset classes, QuantConnect's model | October 2026 (LEAN CLI) | Self-run | Free |
| Zipline Reloaded | Open-source engine (Apache 2.0) | Existing Zipline code | October 2026 | None advertised | Free |
| Reamer Research | Paid local library | Strategies on bars, with written fill rules and modelled costs | October 2026 | Reamer Server, sold separately, with your own connector | $20 per month |

## Closest to how Backtrader works

- **backtesting.py** has the same bar-by-bar idea and a smaller API, with interactive charts. It trades one instrument per strategy, so a Backtrader strategy with several data feeds will not port directly.
- **[Reamer Research](/products/reamer-research.html)** calls your strategy on every bar, from Python or C++, and returns every order, fill and closed trade. Spread, slippage, commission, overnight swap and margin are set per instrument. A public [execution specification](/docs/execution-spec.html) states how orders fill. It has no charts, ready indicators or analyzers of the kind Backtrader ships, and it runs on Linux x86-64, on macOS on Apple Silicon and on Windows through WSL2. See [How does Reamer Research compare with Backtrader?](/faq/reamer-research-vs-backtrader.html)

## Different models

- **bt** builds a strategy from allocation blocks. It suits rebalancing a portfolio more than order-level trading rules.
- **vectorbt** tests large parameter grids on whole arrays. Rules that depend on the current position or on bracket orders are harder to express.

## Larger platforms

- **NautilusTrader** has a Rust core and Python strategies, and the same code runs in backtest and live. Its README warns that breaking changes can occur between releases.
- **LEAN** is the engine behind QuantConnect, and you can run it locally with your own data.
- **Zipline Reloaded** is for existing Zipline code. Its maintainers now point new projects to ML4T Backtest.

## What a published benchmark measured

[What a Backtest Costs](https://reamerlabs.com/papers/what-a-backtest-costs.pdf) ran four engines on the same strategies and data, on one AMD EPYC 9575F on 2026-10-25. This is one backtest of a 20-bar breakout on 500,000 bars, with no commission, spread or slippage.

| Engine | Time |
|---|---|
| Reamer Research from C++ | 0.36 s |
| Reamer Research from Python | 3.38 s |
| NautilusTrader 1.231.0, one function patched | 13.61 s |
| Backtrader | 15.04 s |
| LEAN, built from source | 22.10 s |
| NautilusTrader 1.231.0, stock | 76.85 s |

Backtrader finished ahead of LEAN and stock NautilusTrader, and behind patched NautilusTrader and Reamer Research. Your own indicators and any model inside the loop also count in every engine.

The study's author builds and sells Reamer Research, so the [runners and raw results are public](https://github.com/j0osman/research-engine-benchmark) and you can check every figure. It is one machine, one date, one instrument and two simple strategies. It did not test backtesting.py, bt, vectorbt or Zipline.

## How to choose

| If you want | Look at |
|---|---|
| To learn, and your strategy already works | Stay on Backtrader |
| A maintained free library for one instrument | backtesting.py |
| Portfolio weights and rebalancing | bt |
| Large parameter sweeps in Python | vectorbt |
| Ticks, order books or ready venue adapters | NautilusTrader |
| Many asset classes | LEAN |
| A maintained engine for strategies on bars, with modelled costs and fill rules you can read | Reamer Research |

Reamer Research costs $20 per seat per month, cancel any time, so a month on your own strategy is the test. See [pricing](/pricing.html).
