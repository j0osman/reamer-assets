---
title: What is a quant research engine, and how is it different from a backtesting script?
description: A research engine is one shared program that does everything around a strategy (loading and aligning data, simulating fills and costs, accounting, metrics, logs and reports) so that each study contains only the strategy itself. A backtesting script does all of that on its own, once per study. The engine is what lets you test, diagnose, sweep, report and move to live with the same rules every time. A script is quicker to start and is the right tool for a single rough idea.
stage: 2
order: 9
product: research
next: build-or-buy-backtester, independent-quant-research-to-live-stack, research-engine-for-mid-frequency-strategies, what-makes-a-backtest-deterministic
date: 2026-09-30
---

A backtesting script is one program that tests one idea: it loads the data, loops over the bars, decides when to trade, works out the fills, adds up the profit and prints a result. A research engine takes everything in that script except the strategy and puts it in one place that every study calls. The strategy becomes a small piece of code the engine runs, and the rest (fills, costs, accounting, metrics, records) is the same for every study.

The difference is less about features than about where the rules live. In a script, the rules live inside each study. In an engine, they live once, outside all of them.

## What a backtesting script is good at

Nearly everyone starts with a script, and for good reason:

- **It is quick.** A first test of an idea can be written in an afternoon.
- **Everything is visible.** The fill logic is right there, next to the signal.
- **It fits the question.** For one rough idea, tested once, a script is enough.

The trouble starts with the second study. It begins as a copy of the first, with small changes, and within a year the fill, cost and metric rules differ a little in every copy. That drift is covered in [How do I manage many research scripts without results drifting apart?](/faq/research-scripts-drifting-apart.html)

## What a research engine does

A research engine supports a whole cycle of work on a strategy, not just one run of it. The cycle has five steps.

### 1. Test

Run the strategy over years of bars with realistic costs. The engine owns the loop over time, the alignment of several instruments onto one timeline, the account's open orders and positions, and the cost of each fill. The strategy sees the bars in order and says what it wants to trade.

### 2. Diagnose

When a result looks wrong, find out why. An engine returns every order, fill and closed trade, with the reason for every rejection, so any number in the summary can be traced back to the trades behind it. With a deterministic engine, any single trade can be reproduced exactly. See [What makes a backtest deterministic, and why does it matter?](/faq/what-makes-a-backtest-deterministic.html)

### 3. Sweep

Run the strategy many times across a range of parameters, instruments and periods, to see whether a result holds across its neighbourhood or only at one lucky setting. A sweep is only as good as the consistency of the runs in it. If every run uses the same rules and gives the same output for the same input, the differences across the sweep belong to the strategy. See [How do I know if my backtest is overfit?](/faq/is-my-backtest-overfit.html)

### 4. Report

Write each run's result in one fixed, documented format, with metrics whose formulas are written down. Then any two studies can be compared, by a person or by another program, without asking how each number was computed.

### 5. Hand off to live

Take the strategy logic that was researched into live trading without starting again. The research engine does not trade live itself, but it should state exactly what carries over and what has to change, so the move is a port of known size rather than a second project.

## What sits underneath

To support that cycle, an engine has parts a script rarely builds properly:

- **An execution model that is written down.** Which price a buy pays, how a stop fills through a gap, which exit wins when a stop and a target fall inside one bar, how margin is checked. The rules are a document you can check the results against, not whatever the code happens to do. See [What Is an Execution Specification?](/notes/what-is-an-execution-specification.html)
- **Accounting that adds up.** Cash, positions, margin and fees reconciled on every bar, so the equity curve and the trade list always agree.
- **Metrics with fixed definitions.** Sharpe annualised the same way in every study, drawdown measured the same way, so the headline numbers mean the same thing everywhere.
- **Identical output for identical inputs.** Without it, neither diagnosis nor sweeps can be trusted.
- **A stable interface.** Studies written against last year's version still run against this year's, so upgrading the engine does not mean rewriting the research.
- **Speed.** A sweep of thousands of runs is only practical if each run is fast. Speed changes which questions are worth asking.

## When a script is still the right choice

- **The idea is rough and may be dropped tomorrow.** Test it quickly and decide whether it deserves the full treatment.
- **The strategy is outside what an engine models.** If an engine is built for bars and the idea needs the full order book, or options priced across strikes, a purpose-built script may be the only way to test it.
- **You are learning.** Writing a backtester once, with its fill and cost rules, is the best way to understand what an engine does for you.

## Signs you have outgrown scripts

- Two studies of similar strategies give results you cannot compare, and you are not sure why.
- A bug fix in one script has not reached the others.
- An old result no longer reproduces.
- A parameter sweep takes long enough that you run fewer of them than you should.
- Moving a strategy to live means writing it again from scratch.

## What an engine does not do

An engine does not find an edge, supply market data or decide which costs are realistic for your market. It applies the rules you give it, the same way every time. The quality of the data, the choice of costs and the strategy itself remain yours. For how to set costs, see [How realistic do slippage and spread need to be in a backtest?](/faq/how-realistic-slippage-and-spread.html)

## How Reamer Research fits

[Reamer Research](/products/reamer-research.html) is a research engine for mid-frequency strategies on OHLCV bars, from intraday to multi-day. It is a precompiled library with a stable C interface, running on your own machine. It covers the five steps like this, each documented in the kit:

- **Test.** Your strategy is a callback the engine calls on each bar, written in Python through the reference binding, in C++, or in any language that can call C. Every fill, cost, margin and accounting rule is written in `EXECUTION_SPEC.md`. Several instruments are aligned onto one timeline, and bars of any construction (time, tick, volume or dollar bars) are accepted in the same format.
- **Diagnose.** Each run returns every order, fill and closed trade, and each rejected order carries its reason in text. With a fixed seed, a run is byte-identical on every repeat, so any trade can be reproduced exactly.
- **Sweep.** Backtests can run concurrently from several threads, each with its own configuration, and any number of runs share one seat on one machine. On a 64-core EPYC machine, a single run processed about 1.7 million bars per second, and run-to-run throughput varied by 0.59% across 2,000 runs. See `BENCHMARK.md` and [What 2,000 Runs on a 64-Core EPYC Show](/blog/reamer-research-findings-epyc.html).
- **Report.** Each run can write one JSON document with a version number: 31 summary metrics, drawdown and streaks, and the full trade and order logs. `RESULT_JSON_SCHEMA.md` defines every field, including how each ratio is annualised.
- **Hand off.** `reamer_relay_*` sends the orders a strategy returns to [Reamer Server](/products/reamer-server.html), the live product, and reads the fills back as position state. `RESEARCH_TO_SERVER.md` ships in the kit and states what carries over and what changes: the strategy logic and the order carry over, and indicators that read a lookback window in research become running state live.

The scope is stated in the kit as well. Reamer Research is built for mid-frequency strategies on OHLCV bars; high-frequency, order-book (L2/L3) and options strategies call for a different engine. You bring your own bars, already adjusted for splits and dividends. Running a sweep is your own loop over runs; the engine makes each run fast and repeatable, and does not choose the parameters for you.
