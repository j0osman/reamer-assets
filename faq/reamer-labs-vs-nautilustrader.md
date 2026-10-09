---
title: How do Reamer Research and Reamer Server compare with NautilusTrader?
description: NautilusTrader is a free, open-source trading platform with a Rust core and a Python API that runs the same strategy code in backtest and live, down to tick and order-book data, with ready-made venue adapters. Reamer Research and Reamer Server are two paid libraries for strategies on bars, a research engine with a written execution specification and an order management engine around your own gate and broker connector. NautilusTrader fits order-book and multi-venue trading; Reamer Labs fits mid-frequency bar strategies where you own the risk gate and the connection.
stage: 4
order: 35.4
product: both
next: what-is-reamer-research, what-is-reamer-server, research-to-server-move, who-its-for, pricing
date: 2026-10-08
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

## Which to choose

- **NautilusTrader** if you trade on ticks or order books, want one codebase for backtest and live, need its venue adapters, or want it free and open source.
- **Reamer Labs** if you trade systematic strategies on bars, want a backtest whose fill rules you can read, and want the live side built around your own risk rules and broker connection. See [Who are Reamer Research and Reamer Server for?](/faq/who-its-for.html)

## Price

NautilusTrader is free. Reamer Research costs $1,800 per seat per year and Reamer Server $7,200, each with a 30-day paid trial. See [pricing](/pricing.html).
