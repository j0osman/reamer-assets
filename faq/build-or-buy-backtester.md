---
title: Should I build my own backtester or buy one?
description: Build the parts where your edge lives and buy the parts it does not depend on. For most bar-based strategies the edge is in the signals, the data and the sizing, not in how fills, costs and accounting are simulated, so buying or adopting an engine and building the strategy is usually the better use of time. Build the engine yourself when the engine is the edge, when nothing available models what you trade, or when you need to own every line of it.
stage: 2
order: 10
product: research
next: independent-quant-research-to-live-stack, performance, research-engine-for-mid-frequency-strategies, research-engine-vs-backtest-script
date: 2026-09-30
---

Build the parts of your research where your edge lives, and buy the parts it does not depend on. A strategy makes money from its signals, the data behind them and how it sizes positions. It rarely makes money from how the backtester fills a stop through a gap or annualises a Sharpe ratio, but it can lose money if either is done wrong. For most people trading on bars, that points to buying or adopting an engine and spending the saved time on the strategy.

There are real exceptions, and the cost of buying is not only the price. Both sides are below.

## Where the edge usually lives

Split a research setup into layers and ask which of them a competitor could copy without hurting you:

- **Signals.** What the strategy looks for. Usually the edge.
- **Data.** What you feed it, how it is cleaned, and any non-price data. Often part of the edge.
- **Sizing and portfolio rules.** How much to trade and how strategies combine. Often part of the edge.
- **Execution simulation.** How orders fill, what they cost, how margin is checked. Has to be right, but rarely an edge.
- **Accounting and metrics.** Cash, positions, profit, drawdown, ratios. Has to be right, never an edge.
- **Plumbing.** Loading files, aligning timestamps, writing reports, running sweeps. Never an edge.

The layers that have to be right but are rarely an edge are where building costs most for least return.

## The real cost of building

The first version of a backtester is quick: a loop over bars, a buy, a sell and a profit total can be written in a day. The cost is in everything after that.

### The rules that take the time

A backtester that gives trustworthy numbers needs a decision, written into code, for each of these:

- Which price a market order pays, and when it can first fill.
- How a stop fills when the price gaps straight through it.
- Which exit wins when a stop and a target both fall inside one bar.
- How spread and slippage are charged, and whether they widen when the market is volatile.
- How partial closes, reversals and adding to a position are accounted for.
- How margin is checked, and what happens to an order that fails the check.
- How instruments with different trading hours are aligned onto one timeline.
- How futures rolls, overnight financing and non-price data are handled, if you trade them.

Each one is a small decision. Together they are weeks of work, and each wrong one quietly moves the result.

### Knowing it is right

There is no answer key for a backtest. To trust a home-built engine you have to write tests for every rule above, check that the accounting always adds up, check that the same run gives the same output twice, and check it again after every change. This is usually the part that is skipped, and the part that decides whether the numbers can be believed. See [What makes a backtest deterministic, and why does it matter?](/faq/what-makes-a-backtest-deterministic.html)

### Speed

A backtester that is fast enough for one run can be too slow for a sweep of thousands. Making it fast without breaking it is its own project.

### Maintenance

A new instrument, a new order type, a library upgrade, a bug found a year later that changes old results. An engine you build is an engine you maintain for as long as you use it.

### Time not spent on research

The largest cost is often the least visible. Every week spent on the engine is a week not spent testing strategies. To put a number on it, estimate the weeks the list above would take you, multiply by what your time is worth, and compare that with the price of an engine.

## When building is the right choice

- **The engine is the edge.** If your advantage comes from modelling fills or market microstructure better than anyone else, that model is your strategy, and you should own it.
- **Nothing available models what you trade.** Full order-book strategies, options priced across strikes, or an unusual market structure may simply have no engine that fits.
- **You need to own every line.** Some firms require the source code of anything that produces a number they act on.
- **You are learning.** Writing a backtester once is the best way to understand what one does, even if you later use someone else's.

## The real cost of buying

Buying is not free of cost either:

- **The price,** every year you use it.
- **Its limits.** An engine models some things and not others. A strategy outside its scope cannot be tested on it.
- **Its rules become yours.** Every result depends on its fill and cost rules, so you need to read them.
- **Less visibility.** With a closed engine you rely on its documentation and tests rather than reading its code.
- **Dependence on the vendor.** If the vendor stops, you need a plan.

A free, open-source framework is a third option: no licence fee, and the code is there to read. The questions to ask of it are the same: are its fill and cost rules written down, does it give identical output for identical input, is it fast enough to sweep, and who maintains it.

## How to decide

1. List the layers above and mark which ones hold your edge.
2. For the layers that do not, check whether an available engine models what you trade.
3. Read its execution rules before anything else. If they are not written down, you are buying code you have to test yourself.
4. Run it on a strategy you already know the answer to, and run it twice to check the output matches.
5. Check what happens to your work if the vendor stops or the licence ends.

If an engine passes, buy the layers that need to be right and build the ones that make you money. Most setups end up in between: a bought or adopted engine, with your own data pipeline, strategies, sweep scripts and reporting built around it. See [What is a quant research engine, and how is it different from a backtesting script?](/faq/research-engine-vs-backtest-script.html)

## Where Reamer Research fits

[Reamer Research](/products/reamer-research.html) is the bought core in that in-between setup. It covers the layers that have to be right but are rarely an edge: execution simulation, accounting, metrics, reporting and speed. Your strategies, data pipeline, sweep scripts and analysis stay yours to build around it. Everything below is documented in the kit:

- **The rules, already written.** Market, limit and stop fills, gaps, bracket collisions, spread and slippage, commission, margin, partial closes, overnight financing, futures rolls, multi-asset alignment and non-price data are each specified in `EXECUTION_SPEC.md`. The document ships in the kit rather than on the web, so the 30-day trial below is the way to read it, and test against it, before committing to a year.
- **Checkable output.** With a fixed seed, a run is byte-identical on every repeat. The quickstart asks you to run it twice and compare.
- **Speed for sweeps.** About 1.7 million bars per second per run on a 64-core EPYC machine, with the hardware, method and a benchmark tool to reproduce it in `BENCHMARK.md`.
- **A stable interface.** Programs written against an earlier version of the interface keep working with the current library without changes.
- **Integration in minutes.** An uncut recording shows an AI coding agent, working from the kit documents alone, taking a strategy through the full research loop in about 13 minutes.

The price is USD 1,800 per seat per year, paid once, with nothing renewing automatically, or USD 225 for a 30-day trial to check fit first. See [Pricing](/pricing.html).

What you do not get is the source code: the engine is a precompiled library behind a C interface. Reamer Labs is one person, and a source escrow can be arranged at the licensee's own cost, with the release conditions set out in `LICENSING.md`. The engine's scope is OHLCV bars for mid-frequency strategies. If your strategy needs the order book or options modelling, or the engine itself is your edge, this is one of the cases where building is right.
