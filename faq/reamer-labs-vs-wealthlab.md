---
title: How do Reamer Research and Reamer Server compare with WealthLab?
description: WealthLab (formerly Wealth-Lab) is a Windows application for portfolio backtesting and automated trading, with strategies built from visual blocks or written in C#, and ready data and broker extensions. Reamer Research and Reamer Server are libraries you build your own system around, in Python or C++, with written fill rules and byte-identical runs, on Linux, Apple Silicon Macs and, for Reamer Research, Windows through WSL2. Reamer Labs suits quants who write their own code.
stage: 4
order: 35.67
product: both
next: wealthlab-alternatives, what-is-reamer-research, what-is-reamer-server, supported-platforms, who-its-for, pricing
date: 2026-11-08
topics: backtesting, algorithmic-trading, automated-trading
---

WealthLab, formerly Wealth-Lab, is a finished Windows application for building, backtesting and trading strategies. Reamer Research and Reamer Server are libraries you build your own system around. WealthLab gives you the whole workflow in one program. Reamer Labs supplies the research and order-management core, and you write the program around it.

Details below were checked against WealthLab's own pages in November 2026.

## What WealthLab is

- **A portfolio backtesting application.** Backtests share capital and position sizing across a portfolio. MetaStrategy combines several strategies and shows how each one contributes.
- **Blocks or C#.** Strategies are built visually from Building Blocks or written in C#, and a block strategy can be opened as C# code. An AI strategy generator is also included.
- **Optimization and walk-forward.** Optimizers, walk-forward optimization and rolling-window backtests are built in. Commission and slippage are modelled. With intraday data, a granular setting gives a closer test of limit and stop orders.
- **Data and brokers through extensions.** Wealth-Data supplies end-of-day history with past index members. Extensions add Norgate Data, IQFeed and others, and brokers including Interactive Brokers, TradeStation, Schwab, Tradier, Alpaca and Binance.
- **Live trading in the same program.** Strategy Monitor and Order Manager watch signals and send orders through a supported broker.
- **Windows only.** WealthLab runs on 64-bit Windows 10 or 11.
- **Public prices.** $49.95 a month, $399.95 a year, $499.95 a year for Premium, or a $1,595 lifetime licence with two years of maintenance. There is a 14-day free trial. See their plans page for current figures.

## How Reamer Labs differs

- **Libraries, not an application.** [Reamer Research](/products/reamer-research.html) and [Reamer Server](/products/reamer-server.html) each have a stable C interface, and you write a short program around them. There are no charts or visual builders.
- **General-purpose languages.** Strategies are written in Python or C++, or any language that can call C, so your own libraries and models run inside the strategy. The kit ships a Python binding, a C++ wrapper, 6 templates and 20 examples.
- **Fill rules in writing.** The public [execution specification](/docs/execution-spec.html) states how orders fill on bars. A seeded tick path runs through every bar, so you do not need intraday data to resolve a stop and a target in one bar.
- **Byte-identical runs.** With a fixed seed, output matches to the byte, which suits audit and regression tests.
- **Bars of any type.** Time, tick, volume or dollar bars all run, and several instruments share one wall clock and one account in every run. Monte Carlo resampling of a run's trades is built in, and parameter sweeps and walk-forward splits are loops in your own program.
- **Your own data.** The engine reads your CSV or binary files and fetches no data itself. The kit includes Alpaca and Databento fetchers that write those files.
- **Live through your own gate.** Reamer Server runs your pre-trade gate on every order and sends accepted orders through a broker connector you write. The kit has a worked FIX 4.4 example. WealthLab ships ready broker extensions.
- **Different machines.** Both products run on Linux x86-64. Reamer Research also runs on macOS on Apple Silicon and on Windows through WSL2; Reamer Server does not run on Windows.

## Which to choose

- **WealthLab** if you want a native Windows application, want to build strategies visually or in C#, want ready data and broker extensions, or do not want to run your own software.
- **Reamer Labs** if you write Python or C++, run on Linux, an Apple Silicon Mac or Windows through WSL2, need strategies that call your own code, and want fills you can read and rerun to the byte. See [Who are Reamer Research and Reamer Server for?](/faq/who-its-for.html)

## Price

Reamer Research costs $20 per seat per month, cancel any time. Reamer Server costs $24,000 per seat per year, with a $2,000 30-day trial. See [pricing](/pricing.html).
