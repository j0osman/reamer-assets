---
title: Why is my Python backtest so slow?
description: Usually because a Python loop does expensive work on every bar, and that cost is multiplied by every bar and every run. Reading a value through pandas inside the loop is often the largest part. In one test, reading a single column row by row took about 12 seconds per 500,000 bars with iterrows and about 0.06 seconds with a NumPy array. Profile first, move data out of pandas before the loop, precompute what you can, run sweeps in separate processes, and move the loop to compiled code only if that is still not enough.
stage: 1
order: 1.5
product: research
next: backtesting-py-alternatives, backtrader-alternatives, performance, research-engine-vs-backtest-script
date: 2026-11-04
topics: backtesting, benchmark
---

A backtest that looks at each bar in turn runs its loop body once per bar, and a sweep repeats the whole loop once per parameter set. A small cost on each bar therefore becomes a large one. Five hundred thousand one-minute bars is about a year of a market that trades around the clock, and a sweep of 1,000 settings makes that 500 million loop passes.

In most slow backtests, the strategy's idea is not the expensive part. The expense is in how the loop reads data.

## What one read costs

We timed one job: read the closing price on each bar and add it to a total. There was no strategy logic at all.

| How the loop reads the close | Time per bar | Time per 500,000 bars |
|---|---|---|
| `df.iloc[i]['close']` | 25.0 µs | 12.50 s |
| `df.iterrows()` | 24.2 µs | 12.12 s |
| `df['close'].iloc[i]` | 3.5 µs | 1.76 s |
| `df.itertuples()` | 0.59 µs | 0.29 s |
| NumPy array, `closes[i]` | 0.13 µs | 0.06 s |
| Python list, `closes[i]` | 0.04 µs | 0.02 s |

Across a sweep of 1,000 runs, that is about 3.4 hours with `iterrows` and about one minute with a NumPy array, before the strategy has done anything.

This was measured once on a laptop with an Intel Core i5-8250U, using Python 3.14, pandas 3.1 and NumPy 2.5. Each figure is the median of five runs. The two slowest rows were timed on 20,000 bars, `df['close'].iloc[i]` on 100,000 and the rest on 500,000, then scaled. Your machine will differ, and the ratios matter more than the absolute times.

## How to speed it up, in order

1. **Profile before you change anything.** Run `python -m cProfile -s cumtime your_backtest.py` and read the top of the list. The slow line is often not the one you expect.
2. **Take data out of pandas before the loop.** Convert each column once with `df['close'].to_numpy()`, then index the array. If you need pandas, `itertuples` is far quicker than `iterrows`.
3. **Precompute indicators.** Work out a moving average once, across the whole series, and read it by index. Do not recalculate it from a slice on every bar. If a window must be recalculated, keep a running total and update it one value at a time.
4. **Convert timestamps once.** Time zone conversion and date parsing are slow inside a loop. Do them when you load the data, then compare plain integers.
5. **Cut logging and object creation in the loop.** A print, a log line or a new DataFrame on every bar can cost more than the strategy.
6. **Test on less data first.** Use a recent year to reject weak ideas, and keep the full history for the ideas that survive.
7. **Run sweeps in separate processes.** Python threads share one interpreter lock, so threaded Python strategies do not run at once. `multiprocessing` or `concurrent.futures.ProcessPoolExecutor` gives each run its own process. Each process holds its own copy of the data unless you memory-map it, so watch your memory.

## When a faster loop is not enough

If the profile shows the loop itself is the problem, you have three choices.

- **Vectorise.** If the strategy needs no knowledge of its current position, work on whole arrays. vectorbt is built for this. Rules that depend on the open position, or brackets that move, are hard to express this way.
- **Compile the hot path.** Numba can compile numeric Python functions, and it works best on plain arrays and simple loops.
- **Use an engine with a compiled core.** The engine's own loop, fills and accounting then stop costing Python time. Your strategy's own time still counts.

The last point has limits. A strategy that does slow pandas work on every bar stays slow in any engine, so do the steps above first.

## How much a compiled engine changes

[What a Backtest Costs](https://reamerlabs.com/papers/what-a-backtest-costs.pdf) ran the same 20-bar breakout on 500,000 bars, on one AMD EPYC 9575F on 2026-10-25, with no commission, spread or slippage.

- **Backtrader, entirely in Python:** 15.04 s.
- **Reamer Research, strategy in Python:** 3.38 s.
- **Reamer Research, strategy in C++:** 0.36 s.

The gap between the last two is the cost of crossing into Python on every bar. The study's author builds and sells Reamer Research, so the [runners and raw results are public](https://github.com/j0osman/research-engine-benchmark). It is one machine, one date, one instrument and two simple strategies, and its strategy did little work per bar. A heavier strategy would narrow these differences.

## Where Reamer Research fits

[Reamer Research](/products/reamer-research.html) is a compiled engine you call from Python, C++ or any language that can call C. Bars can come from memory-mapped `.bin` files, so a dataset larger than RAM runs. Each backtest runs on one thread, and you run several at once for a sweep. It covers strategies on time, tick, volume, dollar or custom bars, on Linux x86-64, on macOS on Apple Silicon and on Windows through WSL2. See [How fast is it?](/faq/performance.html) for the conditions behind each figure.

A $20 month lets you time your own strategy on your own machine, and cancel if it does not fit. See [pricing](/pricing.html).
