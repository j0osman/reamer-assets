---
title: Can Reamer Research backtest tick, volume or dollar bars?
description: Yes. Reamer Research takes bars of any type: time, tick, volume, dollar, imbalance or your own custom rule. You build the bars and the engine treats every type the same way. Each bar's real trade count can drive its fills, and instruments sampled at different moments can share one portfolio.
stage: 3
order: 50
product: research
next: market-data, what-is-reamer-research, what-makes-a-backtest-deterministic, backtest-vs-live-results
date: 2026-11-06
topics: backtesting, market-data
---

Yes. Reamer Research takes bars of any type: time, tick, volume, dollar, imbalance or your own custom rule. OHLCV is the record format, not a limit on how a bar was built. You build the bars, and the engine treats every type the same way.

The harder question is whether a backtest stays correct once bars stop arriving on a clock. Accepting the file is the easy part. Two assumptions usually break.

## What breaks when bars stop being time bars

**Activity gets read from the clock.** A simulator that walks through a bar often decides how much happened from the bar's length. On a dollar bar that is backwards. A bar that fills in seconds at the open can hold hundreds of trades, and one that takes half an hour at lunch can hold a few.

**Instruments stop lining up.** Two instruments with their own dollar bars almost never close at the same moment. An engine that wants every instrument on each step finds few steps, or none. One that forces them onto a fixed grid brings back the time sampling you built activity bars to avoid.

## How Reamer Research handles them

- **One record for every bar type.** Each bar holds a timestamp, open, high, low, close, volume, notional and tick count. Volume and notional are separate fields, so neither has to change meaning with the bar type.
- **The real trade count drives the fills.** Inside each bar the engine walks a seeded synthetic tick path, and limit and stop orders fill on the first tick that qualifies. Supply `tick_count` and the path uses it as given: a dollar bar built from 40 trades gets 40 ticks, however long it took. Without it, the tick count comes from the time gap to the next bar.
- **Instruments do not need to line up.** One wall clock runs the whole backtest, and every bar of every instrument is an event on it at its own timestamp. Your strategy runs the moment a bar closes. Instruments with a new bar at that moment are marked through `window_valid`, and the rest carry their last known state, so your strategy can tell a new price from an old one.
- **The same rules apply to every bar.** Spread, slippage, commission, swap and margin work as they do on time bars, under the public [execution specification](/docs/execution-spec.html). Fixed seeds still give byte-identical runs.

## What stays your job

- **Building the bars.** The engine does not turn ticks into bars. Buy and sell classification, threshold estimates and bucketing belong in your own pipeline or a vendor's tool. So does where each bar is cut. Index futures trade overnight, so a volume bar that starts at 9:20 ET can close at 9:31 and put the open inside it. Cut bars at the session boundary.
- **Rewriting rules counted in bars.** "The first six bars after the open" means 30 minutes on 5-minute bars and perhaps a minute on dollar bars, and a 20-bar lookback no longer covers a fixed stretch of time. Rules keyed to the clock survive the switch: the strategy sees each step's timestamp as `data.ts` in Python or `d.ts()` in C++, in epoch seconds UTC. Convert exchange times such as 9:30 ET to UTC yourself, daylight saving included.
- **Keeping the future out of them.** A threshold estimated over the whole dataset leaks later data into earlier bars. The engine cannot see this, because it only receives finished bars.
- **Annualising the ratios.** Bars at irregular intervals do not map to a calendar. Pass `periods_per_year` to `reamer_get_summary()` to set the convention for Sharpe and Sortino.

Pass finished bars as arrays from any language, or convert a CSV with `bin/reamer-csv-build`, which picks up notional and trade count columns. Timestamps are whole seconds, so each instrument can have one bar per second.

Order-book, high-frequency and options strategies are a different kind of engine. For anything built on bars, the rule that decides what a bar contains is yours. See [Does it include market data?](/faq/market-data.html)
