---
title: How do Reamer Research and Reamer Server compare with QuantRocket?
description: QuantRocket is a paid Python platform that runs in Docker, with the Moonshot and Zipline backtesters, data services and ready connections to Interactive Brokers and Alpaca. Reamer Research and Reamer Server from Reamer Labs are libraries you call from your own program, in Python or C++, with a public fill specification, byte-identical runs and a live order core built around your own gate and broker connector. Both suit quants who run their own systems. QuantRocket supplies the platform, and Reamer Labs supplies the core.
stage: 4
order: 35.65
product: both
next: quantrocket-alternatives, what-is-reamer-research, what-is-reamer-server, research-to-server-move, who-its-for, pricing
date: 2026-11-05
topics: backtesting, algorithmic-trading, automated-trading
---

QuantRocket and Reamer Labs both serve quants who run their own systems on their own machines. QuantRocket is a platform of services in Docker that includes backtesters, data and broker connections. Reamer Research and Reamer Server are libraries that you call from a program you write.

Details below were checked against QuantRocket's own site and documentation in November 2026.

## What QuantRocket is

- **A Python platform in Docker.** It runs as a set of containers wherever Docker runs, on Linux, Mac or Windows, and you work in JupyterLab.
- **Two main backtesters.** Moonshot is vectorized and based on Pandas, and suits cross-sectional and factor strategies with regular rebalancing. MoonshotML adds machine learning. Zipline is event-driven, in a version QuantRocket customized for live trading, and takes 1-day or 1-minute bars.
- **Research tools.** Pipeline screens large universes point in time, and Alphalens tests alpha factors.
- **Data services.** US stock prices at end of day and 1 minute, back to 2007, are available to all customers. Other datasets, including Sharadar fundamentals, are sold or connected separately, and real-time data comes through Interactive Brokers, Alpaca or Massive.
- **Ready broker connections.** Interactive Brokers and Alpaca connect directly, and other brokerages connect through SnapTrade.
- **Stocks, futures and FX.**
- **Proprietary, by subscription.** There are Non-Professional and Professional licences, billed monthly or annually. Prices appear after you log in and select a licence. A free tier gives sample data, with no live or paper trading.

## How Reamer Labs differs

- **Libraries, not services.** [Reamer Research](/products/reamer-research.html) and [Reamer Server](/products/reamer-server.html) each have a stable C interface and link into your own program. There are no containers to run. You pass the engine arrays or memory-mapped files.
- **One strategy model.** Your `on_bar` method runs on each bar and can react to open positions, pending orders and the account. QuantRocket's documentation warns that vectorized backtesters need care to avoid look-ahead bias, and that event-driven ones generally protect against it.
- **Costs set per instrument.** Spread, slippage, commission, swap and margin are set for each instrument, and with volatility enabled, spread and slippage scale with each bar's range. QuantRocket's Zipline has commissions and slippage disabled by default until you set them. Moonshot documents fixed slippage in basis points, and leaves limit-order fills for your own code to model.
- **Fill rules in writing.** The public [execution specification](/docs/execution-spec.html) states how every order type fills on bars, including bid and ask, gaps and brackets. A seeded tick path runs through each bar, so a stop and a target in one bar resolve the same way on every run.
- **Byte-identical runs.** With a fixed `rng_seed`, output matches to the byte, which suits audit and regression tests.
- **Any language that can call C.** Strategies can be written in Python or C++, and your own libraries and models run inside the strategy.
- **Live through your own gate.** Reamer Server sequences orders, tracks their state and calls your pre-trade gate on every order, and `reamer_relay_*` sends research orders to it. You write the broker connector, and the kit has a worked FIX 4.4 example.
- **Your own data.** Reamer Labs sells no data. You bring your own bars of any type, from minutes to days.

Reamer Research runs on Linux x86-64 and on macOS on Apple Silicon. Reamer Server runs on Linux. Neither runs on Windows.

## Speed

QuantRocket's backtesters were not tested in [What a Backtest Costs](https://doi.org/10.6084/m9.figshare.34787745), so this page gives no speed figures for them.

## Which to choose

- **QuantRocket** if you want data, research tools and broker connections in one platform, you trade through Interactive Brokers or Alpaca, or your strategies are cross-sectional and suit Pandas.
- **Reamer Labs** if you want an engine inside your own system, fills you can read and rerun to the byte, per-bar logic with costs modelled per instrument, and a live side that passes through your own risk rules. See [Who are Reamer Research and Reamer Server for?](/faq/who-its-for.html)

## Price

Reamer Research costs $1,800 per seat per year and Reamer Server $7,200, each with a 30-day paid trial. See [pricing](/pricing.html).
