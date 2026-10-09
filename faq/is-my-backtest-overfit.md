---
title: How do I know if my backtest is overfit?
description: You cannot prove a backtest is not overfit, but you can test for the signs. An overfit result holds at one parameter set and collapses at its neighbours, fails on data it was not tuned on, rests on a handful of trades, or disappears once realistic costs are applied. Check each one, on a tool that gives the same answer every run.
stage: 1
order: 2
product: research
next: backtest-results-change-between-runs, what-makes-a-backtest-deterministic, how-realistic-slippage-and-spread, research-engine-vs-backtest-script
date: 2026-09-29
---

A backtest is overfit when it has learned the noise in one stretch of history instead of a pattern that will repeat. It looks excellent on the data it was built on and does little or nothing on data it has not seen. No single number tells you whether that has happened. What you can do is look for the signs, and each has a test.

## Why it happens

Every choice you make while looking at results is a small fit to the data: the lookback, the threshold, the stop distance, the instruments, the date range, the filter you added because it removed three bad trades. Each choice is reasonable on its own. Together they can shape a strategy to one piece of history so closely that the result says more about the history than about the strategy.

The more variations you try, the more certain it is that one of them looks good by chance. A strategy picked as the best of two hundred is not the same evidence as a strategy that worked first time.

## 1. Count what you tried

Keep a record of every variation you ran, not only the one you kept. If the winner was one of many, judge it against the others: a result far above the rest is more believable than one slightly ahead of a crowd of similar numbers. If you cannot say how many variations you tried, assume it was more than you think.

## 2. Look at the neighbourhood, not the peak

Sweep each parameter across a range around the chosen value and look at the whole map of results, not the best cell.

- **A plateau is a good sign.** If a 20-bar lookback works and 18 and 23 work nearly as well, the strategy is responding to something real.
- **A spike is a warning.** If 20 works and 19 and 21 lose money, the backtest found a coincidence in the data.
- **Pick from the middle of the plateau,** not its highest point. The highest point is where noise helped most.

A sweep only means something if every run in it is comparable. The costs, the data and the fill rules have to be identical across runs, and the tool has to give the same answer when nothing has changed. Otherwise the map shows the tool's variation mixed in with the strategy's.

## 3. Hold data back

Set aside a period of data before you start, and do not look at it while you develop the strategy. When you are finished, run the final version on it once.

- **Out-of-sample.** If the result holds up on the held-back period, that is real evidence. If it only holds after you adjust something, the held-back data has become part of the fit, and it no longer counts.
- **Walk-forward.** Tune on one window, test on the next, move both forward, and repeat. The joined-up test windows show how the strategy would have done if you had re-tuned it on a schedule, which is closer to how it will actually be run.
- **Different conditions.** A period with a different trend, volatility or rate environment is a harder test than a neighbouring month.

## 4. Look at where the profit comes from

A total return hides how it was made. Break it down:

- **By trade.** If a few trades make most of the profit, remove them and see what is left. A strategy that depends on three trades has a sample size of three.
- **By period.** If one year or one event carries the result, the strategy may be a bet on that year.
- **By instrument.** If it works on one symbol and none of its close relatives, be suspicious of the one.
- **By count.** Few trades means wide uncertainty. Fifty trades cannot tell a real edge from a lucky one with much confidence.

## 5. Apply realistic costs before trusting anything

Overfitting and optimistic costs make each other worse. Parameter searches tend to drift toward settings that trade more often, and more trades mean more cost. A strategy tuned with no spread and no slippage has usually been tuned toward the settings that costs hurt most. Set costs per instrument before you sweep, not after you have chosen a winner. See [Why Execution Modeling Matters](/notes/why-execution-modeling-matters.html) and [Spread and Slippage Widen at the Open](/notes/spread-and-slippage-widen-at-the-open.html).

## 6. Make sure the tool is not a variable

If running the same backtest twice gives two different results, every comparison in steps 2 to 5 is unreliable, because you cannot separate a change in the strategy from a change in the run. A fixed seed that reproduces the same output exactly is the base the other tests stand on. See [Why Replay Matters](/notes/why-replay-matters.html).

## A short checklist

1. Record every variation you tried.
2. Sweep each parameter and choose from a plateau, not a peak.
3. Keep a period of data untouched until the end, and test on it once.
4. Break the profit down by trade, period and instrument.
5. Set realistic costs per instrument before sweeping.
6. Confirm that identical inputs give identical output.

A strategy that passes all six can still fail live. One that fails any of them is likely to.

## How Reamer Research helps, and what it leaves to you

[Reamer Research](/products/reamer-research.html) does not decide whether a strategy is overfit. That judgement, and the design of your in-sample, out-of-sample and walk-forward windows, stay with you: the engine runs the windows you give it and does not choose them. What it does is make the tests above cheap, comparable and exact:

- **Identical output for identical inputs.** With a fixed seed, the same strategy, data and settings produce byte-identical results on every run, so a difference between two runs in a sweep is always a difference in the parameters.
- **Sweeps that finish.** Each backtest runs on one thread, and one seat covers any number of concurrent backtests on the licensed machine, so a sweep runs as many parallel processes as the machine has cores. Measured throughput on the native path is about 1.72 million bars per second per run, and a Python strategy runs 884,130 bars in about 65 seconds. Both figures are published in `BENCHMARK.md`, which ships in the kit.
- **Costs fixed before the sweep.** Spread, slippage and commission are set per instrument, and every run in a sweep applies the same rules from `EXECUTION_SPEC.md`. When price volatility is enabled, spread and slippage vary bar by bar from the seed, so re-running a finished strategy across several seeds shows whether its result depends on one lucky draw of fills.
- **Results you can break down.** The result JSON carries every closed trade, every fill, every order, and a per-trade return series, the same series the Sharpe, Sortino and Calmar figures are computed from, so the breakdown in step 4 is a query rather than a rebuild.
- **Trade-order luck, measured.** `reamer_run_monte_carlo()` resamples a run's per-trade returns and reports final-equity, loss, ruin and max-drawdown percentiles, seeded and reproducible, so you see how much of a drawdown came from the order the trades happened to arrive in.

Data quality and the statistical judgement stay yours. What the engine gives you is a result you can trust to mean the same thing every time you run it, which is what the tests above need.
