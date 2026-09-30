---
title: What makes a backtest deterministic, and why does it matter?
description: A backtest is deterministic when the same strategy, data, settings and seed give the same output, byte for byte, on every run. That requires fixed randomness, a guaranteed order of events, and nothing that depends on the clock, the machine or a previous run. It matters because every comparison depends on it: when the tool adds no variation of its own, any difference between two results comes from something you changed, and you can check what.
stage: 2
order: 8
product: research
next: build-or-buy-backtester, research-engine-vs-backtest-script, deterministic-backtesting-with-slippage-and-spread, backtest-results-change-between-runs
date: 2026-09-30
---

A deterministic backtest is a function: the same strategy, the same data, the same settings and the same seed go in, and the same output comes out, down to the last byte, however many times it runs. Not roughly the same Sharpe ratio. The same trades, at the same prices, on the same bars.

That sounds like a technical nicety. It is the property every other conclusion from a backtest rests on.

## What determinism requires

An engine is only deterministic if every one of these holds. One missing is enough to break it.

### 1. Randomness comes from a seed

Realistic backtests use randomness on purpose: noisy spreads, sampled slippage, the order of ticks inside a bar. Determinism does not mean removing that noise. It means every random number comes from a seed you set, so the same seed replays the same noise.

How the numbers are drawn matters too. If every random draw comes in sequence from one shared stream, one extra draw early in the run, from one extra order, shifts every random number after it. Every later fill changes, and a small edit to the strategy looks like a large change in the result. The sturdier design ties each random number to where it is used, such as the bar and the instrument, so a change in one place leaves the noise elsewhere untouched.

### 2. Events happen in a guaranteed order

When two things happen on the same bar, a stop and a target inside one candle, or orders on two instruments at the same timestamp, the engine has to decide which comes first. A deterministic engine decides by a written rule. A non-deterministic one decides by whatever order a hash map or a thread pool happened to produce this time.

### 3. Nothing depends on the clock or the machine

A historical test has no reason to know what time it is now, how busy the machine is, or whether a network call came back quickly. Anything that reads the current time, waits on the network, or gives up after a fixed number of seconds can behave differently on the next run, even though the history it is testing has not changed.

### 4. Each run starts clean

No caches, globals, files or indicator state carried over from the run before. The first run after a restart should match the tenth.

### 5. The inputs are pinned

The engine can only promise the same output for the same input. The data file, the configuration and the versions of every component have to be fixed and recorded, or two runs are not the same test. [Why do I get different backtest results each time I run it?](/faq/backtest-results-change-between-runs.html) walks through finding which of these has slipped.

## Why it matters

### Every comparison depends on it

A backtest is useful for comparison: this version against the last, this parameter against its neighbour, with costs against without. If the tool adds its own variation, every difference is a mix of the change you made and noise from the tool, and nothing separates the two. With a deterministic engine, a difference between two runs is caused by a difference between the two inputs.

### Parameter sweeps become meaningful

A sweep runs hundreds or thousands of backtests and compares them. If each run varies on its own, the sweep measures that variation as much as the strategy, and a result that holds across a neighbourhood of parameters cannot be told apart from one that only looked stable. See [How do I know if my backtest is overfit?](/faq/is-my-backtest-overfit.html)

### A strange trade can be investigated

When one trade in a result looks wrong, a deterministic engine can reproduce that exact trade, on that bar, with the same simulated prices, as many times as needed while you find the cause. Without determinism the trade may not come back on the next run, and the question is never answered. See [Why Replay Matters](/notes/why-replay-matters.html).

### Someone else can check the result

A colleague, an allocator or your own future self can rerun the study from the same inputs and get the same output. A result nobody else can reproduce has to be taken on trust.

## What determinism does not mean

- **It does not mean the backtest is right.** A deterministic engine with optimistic fills gives the same wrong answer every time. Determinism makes errors repeatable so they can be found. Realistic costs are a separate question: see [How realistic do slippage and spread need to be in a backtest?](/faq/how-realistic-slippage-and-spread.html)
- **It does not mean the result is robust.** One seed is one sample of the noise. To see how much a result depends on it, change the seed on purpose and compare. Determinism is what makes that test possible, because each seed's run is repeatable.
- **It does not cover your own code.** If the strategy draws unseeded random numbers or reads the clock, it breaks determinism however well the engine behaves.

## How to test a tool's claim

1. Run the same backtest, with noise switched on, at least ten times.
2. Hash the full output, the trade log and the order log, not just the summary. Totals can match while the trades differ.
3. Compare the hashes. They should all be identical.
4. Restart the machine, or use a fresh process, and repeat.
5. Make one small change, such as a single extra order, and check how much of the rest of the result moves. It should be only what that change can reach.

## How Reamer Research handles it

Determinism is the property [Reamer Research](/products/reamer-research.html) is built around. Each point below is stated in the documents that ship in the kit:

- **Seeded, position-keyed noise.** Spread and slippage noise are off by default, and with noise off a run is deterministic whatever the seed. With noise on, a run is byte-identical for a fixed `rng_seed` (42 by default). Each simulated tick is sampled from the seed, the bar's index and a hash of the ticker, not drawn in turn from one shared stream, and a rejected order uses no random numbers. `EXECUTION_SPEC.md` §0a and §1 give the rules; the rejection rule is in `CHANGELOG.md`.
- **Written ordering rules.** When a bracket's stop and target both fall inside one bar, the engine settles it on a simulated tick path fixed by the seed and the bar data, and the first exit that path reaches wins. The rule is `EXECUTION_SPEC.md` §7. See [Bracket Orders Done Correctly](/notes/bracket-orders-done-correctly.html).
- **No clock in the result.** The engine deliberately sets no time limit on a strategy callback, because a wall-clock limit would let machine load change the result. `DEPLOYMENT.md` explains how to bound a run from outside instead.
- **Measured, not only designed.** On a 64-core EPYC machine, 20 of 20 full runs from CSV to result were byte-identical by SHA-256, with the noise switched on. Across 2,000 runs, throughput varied by a coefficient of variation of 0.59%. See [What 2,000 Runs on a 64-Core EPYC Show](/blog/reamer-research-findings-epyc.html) and `BENCHMARK.md`.
- **Checkable on day one.** The quickstart asks you to run the example twice and compare: the output is byte-identical. Each result's JSON report records the version of the library that produced it.

The limits are the ones above: the engine removes its own variation, not yours. Your strategy code, a fixed copy of your data, and a record of your configuration stay your responsibility. The byte-identical guarantee is stated for repeated runs; this page makes no claim about identical bytes across different operating systems or processors.
