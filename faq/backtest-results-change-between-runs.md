---
title: Why do I get different backtest results each time I run it?
description: Because something in the run is not fixed. Usually it is unseeded randomness, data that changed between runs, an order of operations that is not guaranteed, or a library or setting that drifted. Run the same backtest twice, compare the trade logs, and find the first trade that differs. A seeded engine gives byte-identical output, which leaves your own code and data as the only things to check.
stage: 1
order: 3
product: research
next: what-makes-a-backtest-deterministic, is-my-backtest-overfit, how-realistic-slippage-and-spread, research-engine-vs-backtest-script
date: 2026-09-29
---

If the same strategy on the same data gives a different answer each time, something in the run is changing that you did not change. Until that is found, no result can be compared with any other. You cannot tell whether an improvement came from your edit or from the run, and every parameter sweep mixes the strategy's variation with the tool's.

The cause is almost always one of six things.

## 1. Randomness without a fixed seed

Anything that draws a random number gives a new answer each run unless its seed is fixed: simulated slippage, random fill ordering, sampled spreads, a machine-learning model's starting weights, a random train and test split.

Randomness in a backtest is not the problem. Realistic cost noise is useful, because real fills vary. The problem is randomness you cannot replay. With a fixed seed, the noise is the same on every run, so you keep the realism and still get comparable results. To see how much a result depends on the noise, change the seed on purpose and compare, rather than letting it change by accident.

## 2. Data that changed

- **A re-download.** Vendors revise history: corrected prints, restated fundamentals, new adjustments for splits and dividends. Pulling the same date range twice does not guarantee the same numbers.
- **A bar still forming.** A backtest that runs up to "now" includes a last bar that is still changing.
- **Time handling.** A change of time zone, or a daylight-saving boundary handled differently, moves bars relative to each other.

Keep a fixed copy of the data used for each study, with a checksum, and run against that copy rather than a live source.

## 3. Order of operations that is not guaranteed

- **Unordered collections.** Looping over instruments stored in a set or hash map can visit them in a different order each run. If the order decides which order is placed first, or which position is sized from the remaining cash, the results change.
- **Parallel work.** Threads that finish in a different order each run can change which event is handled first.
- **Floating-point sums.** Adding the same numbers in a different order gives slightly different totals. The difference is tiny, but a tiny difference on a threshold can flip a signal, and one flipped signal changes every trade after it.

## 4. The environment drifted

A library upgrade, a different machine, a changed default in a configuration file, or a different version of the backtester can each change the result without any change to the strategy. Record the version of every component and the full configuration alongside each result, so that two results can be checked for like-for-like before they are compared.

## 5. The run depends on the clock or the machine

A backtest that reads the current time, calls a network service, or stops a slow step after a fixed number of seconds will behave differently under different machine load. The history being tested does not change, so nothing in the test should depend on when or where it runs.

## 6. State left over from a previous run

Caches, global variables, files written by an earlier run, or indicators that were not reset can carry one run's state into the next. The first run after a restart then differs from the second.

## How to find the cause

1. Run the same backtest twice with nothing changed.
2. Compare the full trade logs, not the final return. Totals can match by coincidence while the trades differ.
3. Find the first trade that differs. The cause is at or just before that bar.
4. Fix one variable at a time: seed, data copy, versions, instrument order, until two runs match exactly.

The target is identical output, byte for byte. "Close enough" hides the next change in the same noise.

## How Reamer Research handles it

[Reamer Research](/products/reamer-research.html) is built so the engine is never the variable. Each rule below is written down in `EXECUTION_SPEC.md` and `DEPLOYMENT.md`, which ship in the kit:

- **Seeded execution noise.** Spread and slippage noise are off by default, and with noise off a run is fully deterministic whatever the seed. With noise on, every sampled price comes from a fixed seed (`rng_seed`, 42 by default), and the run is reproducible byte for byte for that seed. The quickstart in the kit asks you to run it twice and compare: the output is byte-identical. See [Why Replay Matters](/notes/why-replay-matters.html) and [Why Synthetic Ticks Instead of Stored Ticks](/notes/why-synthetic-ticks-instead-of-stored-ticks.html).
- **Nothing depends on the clock.** The engine deliberately has no time limit on a strategy's callback, because a wall-clock limit would make identical inputs give different results under different machine load. If you need a time limit, you set it on the process, outside the result.
- **One thread per backtest.** A single run has no scheduler inside it to reorder events. Across 2,000 runs on a 64-core machine, run-to-run throughput varied by a coefficient of variation of 0.59%, so timing is as stable as the output. The figures are in `BENCHMARK.md`.

What the engine cannot fix is inside your own code and data. If a strategy callback draws unseeded random numbers, reads the clock, calls the network, or keeps state between runs, its results will vary however deterministic the engine is. A fixed data copy, recorded versions and a recorded configuration stay your responsibility. What the engine removes is the doubt about itself: when two runs differ, the difference is in something you can find.
