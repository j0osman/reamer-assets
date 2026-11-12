---
title: How do Reamer Research and Reamer Server compare with MultiCharts?
description: MultiCharts is a Windows charting and trading platform with ready broker connections, an optimizer and its own scripting language, built for traders who want one application from chart to live order. Reamer Research and Reamer Server are libraries for Linux and Apple Silicon Macs that you build your own system around, in Python or C++, with written fill rules and byte-identical runs. Reamer Labs suits quants who write their own code.
stage: 4
order: 35.64
product: both
next: multicharts-alternatives, what-is-reamer-research, what-is-reamer-server, supported-platforms, who-its-for, pricing
date: 2026-11-04
topics: backtesting, algorithmic-trading, automated-trading
---

MultiCharts is a finished Windows application for charting, testing and live trading. Reamer Research and Reamer Server are libraries you build your own system around. MultiCharts gives you the whole desk in one window. Reamer Labs supplies the research and order-management core, and you write the program around it.

Details below were checked against MultiCharts' own pages in November 2026.

## What MultiCharts is

- **A charting and trading application.** Charts, strategy testing and automated trading sit in one program.
- **Its own scripting language.** Strategies are written in PowerLanguage, which follows EasyLanguage style. A separate .NET edition is also sold.
- **Detail inside the bar.** Bar Magnifier replays finer data inside a bar, down to ticks if you have them. Without finer data, a strategy reacting inside a bar is limited to the open, high, low and close.
- **Optimization and portfolios.** The Advanced plan adds Portfolio Trader, and matrix and walk-forward optimization.
- **Windows only.** It has no Mac or Linux version, and MultiCharts suggests a virtual machine for Macs.
- **Public prices.** Plans start at $99 per month, with a lifetime licence also sold, and there is a free trial. See their purchase page for current figures.

## How Reamer Labs differs

- **Libraries, not an application.** [Reamer Research](/products/reamer-research.html) and [Reamer Server](/products/reamer-server.html) are libraries you call from your own program: Reamer Research as a Python package or one C++ header, both products through a stable C interface. There are no live charts. Reamer Research's offline trade viewer charts each closed trade on its bars, but it is not a charting or trading application.
- **General-purpose languages.** Strategies are written in Python or C++, or any language that can call C, so your own libraries and models run inside the strategy.
- **Fill rules in writing.** The public [execution specification](/docs/execution-spec.html) states how orders fill on bars. A seeded tick path runs through every bar, so you do not need tick data to resolve a stop and a target in one bar.
- **Byte-identical runs.** With a fixed seed, output matches to the byte, which suits audit and regression tests.
- **Several instruments on one account.** Instruments share one wall clock and one account in every Reamer Research run.
- **Live through your own gate.** Reamer Server runs your pre-trade gate on every order and sends accepted orders through a broker connector you write. The kit has a worked FIX 4.4 example. MultiCharts ships ready broker connections.
- **Different machines.** Both products run on Linux x86-64. Reamer Research also runs on macOS on Apple Silicon and on Windows through WSL2; Reamer Server does not run on Windows.

## Which to choose

- **MultiCharts** if you want a native Windows application, want charts and ready broker connections, write strategies in an EasyLanguage style, or do not want to run your own software.
- **Reamer Labs** if you write Python or C++, run on Linux, an Apple Silicon Mac or Windows through WSL2, need strategies that call your own code, and want fills you can read and rerun to the byte. See [Who are Reamer Research and Reamer Server for?](/faq/who-its-for.html)

## Price

Reamer Research costs $20 per seat per month, cancel any time. Reamer Server costs $24,000 per seat per year, with a $2,000 30-day trial. See [pricing](/pricing.html).
