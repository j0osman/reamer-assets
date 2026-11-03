---
title: How do Reamer Research and Reamer Server compare with NautilusTrader?
description: NautilusTrader is a free, open-source trading platform with a Rust core and a Python API that runs the same strategy code in backtest and live, down to tick and order-book data, with ready-made venue adapters. Reamer Research and Reamer Server are two paid libraries for strategies on bars, a research engine with a written execution specification and an order management engine around your own gate and broker connector. In a published four-engine benchmark, 500,000 bars with trades took NautilusTrader 76.85 s stock and 13.61 s with one function patched, against 3.38 s for Reamer Research from Python. NautilusTrader fits order-book and multi-venue trading; Reamer Labs fits mid-frequency bar strategies where you own the risk gate and the connection.
stage: 4
order: 35.4
product: both
next: what-is-reamer-research, what-is-reamer-server, research-to-server-move, who-its-for, pricing
date: 2026-10-08
topics: systematic-trading, backtesting
---

NautilusTrader is a free, open-source trading platform with a Rust core and a Python API. It runs the same strategy code in backtest and live, down to tick and order-book data, and has ready-made adapters for many venues. Reamer Research and Reamer Server are two paid libraries for strategies on bars: a research engine with a written execution specification, and an order management engine you build your own gate and broker connector around.

## What NautilusTrader is

- **Open source** (LGPL-3.0), with a fast Rust core and strategies written in Python.
- **One engine for backtest and live.** The same strategy code runs in both, which removes a whole class of differences between the two.
- **Fine-grained data.** Quotes, trades, bars and order books, at nanosecond resolution.
- **Many venues.** Adapters for crypto exchanges, Interactive Brokers, betting exchanges and others, with order management and risk checks built in.

## How Reamer Labs differs

- **Two products.** [Reamer Research](/products/reamer-research.html) is the research engine and [Reamer Server](/products/reamer-server.html) takes a strategy live. Each is a library with a stable C interface, licensed separately.
- **Bars, not order books.** Reamer Labs targets strategies on OHLCV bars, held from minutes to days. It does not do high-frequency or order-book strategies.
- **Fill rules in writing.** Reamer Research fills orders by an execution specification that ships with the kit: bid and ask, spread, slippage, commission, overnight swap, margin, gaps, and what fills first inside a bar. Output is byte-identical for a fixed `rng_seed`.
- **One call per time step.** A Reamer Research strategy is called once per time step with every instrument's recent bars and positions, so a cross-sectional rule reads all instruments in one place.
- **Your gate and your connector.** Reamer Server supplies the order core: sequencing, order state, and a call to your pre-trade gate on every order. You write the gate's rules and the broker connector. It ships starting points in C++, Rust and Go, including a FIX 4.4 example. Strategies connect over a socket from any language.
- **Research orders go live through a relay.** `reamer_relay_*` in Reamer Research sends the same order structure a backtest uses to Reamer Server. Indicators move from a bar window to running state. See [What does it take to move from Reamer Research to Reamer Server?](/faq/research-to-server-move.html)
- **A vendor behind it.** A commercial licence, versioned releases and email support from the author.

## What a published benchmark measured

[What a Backtest Costs](https://doi.org/10.6084/m9.figshare.34787745) ran Backtrader, NautilusTrader, LEAN and Reamer Research on the same strategies and data, on one AMD EPYC 9575F on 2026-10-25. There was no commission, spread or slippage, and the realistic strategy was a 20-bar breakout. The study's author builds and sells Reamer Research, so the [runners and raw results are public](https://github.com/j0osman/research-engine-benchmark) and you can check every figure.

The study tested NautilusTrader 1.231.0 on bars only, so it says nothing about its order-book or live performance.

- **One backtest, 500,000 bars.** Stock NautilusTrader took 76.85 s. With `record_trade` patched out it took 13.61 s and 717 MB. Reamer Research took 3.38 s and 309 MB from Python, and 0.36 s and 185 MB from C++.
- **Why stock is slower.** The cost of `record_trade` grows with the square of the trade count. This was already reported as [issue 4434](https://github.com/nautechsystems/nautilus_trader/issues/4434), and the maintainers state that 2.0 fixes it. The 2.0 release candidates were not tested.
- **No trades, smaller gap.** On a strategy that never trades, NautilusTrader took 6.43 s and Reamer Research 2.74 s from Python.
- **100 tickers of 35,064 bars each.** Patched NautilusTrader took 675.92 s and 4.7 GB. Reamer Research took 14.85 s and 942 MB from Python. On 500,000 bars the gap was 4 times; here it was about 45 times, from a cost in NautilusTrader's position snapshots on netting accounts.
- **64 copies at once.** Patched NautilusTrader finished 11,242 backtests per hour. Reamer Research finished 42,468 from Python and 332,249 from C++, though the C++ path lost the most speed per copy.
- **Agreement.** NautilusTrader and LEAN agreed to the cent at −1,981.89. Reamer Research ended at −1,937.16 on the same trades, because they fill on the current bar and it fills on the next one.

NautilusTrader repeated every result, as did the other engines. This is one machine, one date, one instrument and two simple strategies. Heavy indicators or a model inside the loop would narrow the differences.

## Which to choose

- **NautilusTrader** if you trade on ticks or order books, want one codebase for backtest and live, need its venue adapters, or want it free and open source.
- **Reamer Labs** if you trade systematic strategies on bars, want a backtest whose fill rules you can read, and want the live side built around your own risk rules and broker connection. See [Who are Reamer Research and Reamer Server for?](/faq/who-its-for.html)

## Price

NautilusTrader is free. Reamer Research costs $1,800 per seat per year and Reamer Server $7,200, each with a 30-day paid trial. See [pricing](/pricing.html).
