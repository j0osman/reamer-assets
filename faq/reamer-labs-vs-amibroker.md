---
title: How do Reamer Research and Reamer Server compare with AmiBroker?
description: AmiBroker is a Windows application for charting, scanning and portfolio-level backtesting, with strategies written in its own AFL language, sold as a lasting licence. Reamer Research and Reamer Server are libraries you build your own system around, in Python or C++, with written fill rules and byte-identical runs, on Linux, Apple Silicon Macs and, for Reamer Research, Windows through WSL2. Reamer Labs suits quants who write their own code.
stage: 4
order: 35.68
product: both
next: amibroker-alternatives, what-is-reamer-research, what-is-reamer-server, supported-platforms, who-its-for, pricing
date: 2026-11-08
topics: backtesting, algorithmic-trading, automated-trading
---

AmiBroker is a finished Windows application for charting, scanning and backtesting, scripted in its own AFL language. Reamer Research and Reamer Server are libraries you build your own system around. AmiBroker gives you charts and testing in one program. Reamer Labs supplies the research and order-management core, and you write the program around it.

Details below were checked against AmiBroker's own pages in November 2026.

## What AmiBroker is

- **A charting, scanning and backtesting application.** Charts, scans, alerts and strategy tests sit in one program. The current version is 7.00.
- **Its own formula language.** Strategies are written in AFL, the AmiBroker Formula Language, which works on whole price arrays at once and has a built-in debugger. Version 7 adds an AI code assistant for AFL.
- **Portfolio-level backtesting.** Many symbols trade together, each with its own position sizing, and a ranking score picks trades when buying power runs short. Rotational systems and a custom backtester are supported.
- **Optimization, walk-forward and Monte Carlo.** Exhaustive and smart optimizers, walk-forward testing and Monte Carlo simulation are built in.
- **Data and trading.** It reads data from many vendors, including eSignal, IQFeed and Interactive Brokers, and the AmiQuote downloader comes with the top edition. Orders can be placed from charts or through an auto-trading interface.
- **Windows only.** It runs on Windows 7 SP1 through 11. Mac users run it through Boot Camp, Parallels or VMware.
- **A lasting licence.** Standard $299 (32-bit only), Professional $379, Ultimate Pack Pro $499. Each includes 24 months of free upgrades, then upgrades at half price. See their purchase page for current figures.

## How Reamer Labs differs

- **Libraries, not an application.** [Reamer Research](/products/reamer-research.html) and [Reamer Server](/products/reamer-server.html) each have a stable C interface, and you write a short program around them. There are no charts or scanners.
- **General-purpose languages.** Strategies are written in Python or C++, or any language that can call C, so your own libraries and models run inside the strategy. The kit ships a Python binding, a C++ wrapper, 6 templates and 20 examples.
- **Fill rules in writing.** The public [execution specification](/docs/execution-spec.html) states how orders fill on bars. A seeded tick path runs through every bar, so you do not need tick data to resolve a stop and a target in one bar.
- **Byte-identical runs.** With a fixed seed, output matches to the byte, which suits audit and regression tests.
- **Bars of any type.** Time, tick, volume or dollar bars all run, and several instruments share one wall clock and one account in every run. Monte Carlo resampling of a run's trades is built in, and parameter sweeps and walk-forward splits are loops in your own program.
- **Your own data.** The engine reads your CSV or binary files and makes no network calls. The kit includes Alpaca and Databento fetchers that write those files.
- **Live through your own gate.** Reamer Server runs your pre-trade gate on every order and sends accepted orders through a broker connector you write. The kit has a worked FIX 4.4 example.
- **Different machines.** Both products run on Linux x86-64. Reamer Research also runs on macOS on Apple Silicon and on Windows through WSL2; Reamer Server does not run on Windows.

## Which to choose

- **AmiBroker** if you want a native Windows application with charts and scans, write AFL or want to, and prefer a one-time licence.
- **Reamer Labs** if you write Python or C++, run on Linux, an Apple Silicon Mac or Windows through WSL2, need strategies that call your own code, and want fills you can read and rerun to the byte. See [Who are Reamer Research and Reamer Server for?](/faq/who-its-for.html)

## Price

Reamer Research costs $1,800 per seat per year and Reamer Server $7,200, each with a 30-day paid trial. See [pricing](/pricing.html).
