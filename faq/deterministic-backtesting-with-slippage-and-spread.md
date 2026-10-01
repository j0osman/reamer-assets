---
title: Which backtesting engine gives byte-identical results with realistic slippage and spread?
description: One whose randomness is seeded and tied to each position in the data, not drawn from a running stream. Then spread and slippage can vary with the market and still repeat exactly for a fixed seed. Reamer Research works this way. In its published test, 20 of 20 runs with spread and slippage noise switched on were byte-identical by SHA-256.
stage: 3
order: 16
product: research
next: what-makes-a-backtest-deterministic, how-realistic-slippage-and-spread, stop-and-target-in-same-bar, research-engine-for-mid-frequency-strategies
date: 2026-10-01
---

One whose randomness is seeded and tied to each position in the data, not drawn from a running stream. Then spread and slippage can vary with the market and still repeat exactly for a fixed seed. [Reamer Research](/products/reamer-research.html) works this way. In its published test, 20 of 20 runs with spread and slippage noise switched on were byte-identical by SHA-256.

Most tools make you choose. Fixed costs repeat but are not realistic, and random costs are realistic but change on every run. Getting both takes a few specific design choices, and they can be tested before you buy.

## Why the two usually pull apart

Real spread and slippage are not constant. Spread widens when the market moves fast, slippage is sometimes worse than expected and sometimes better, and both are worst around the open. A backtest that charges one fixed amount on every fill understates costs exactly when they matter. See [How realistic do slippage and spread need to be in a backtest?](/faq/how-realistic-slippage-and-spread.html)

Modelling that variation means randomness, and randomness is what usually breaks repeatability:

- **No seed.** Each run draws different numbers, so two runs of the same strategy disagree, and a change you made cannot be told apart from the noise.
- **One seed, one shared stream.** The run repeats, but every random number depends on how many were drawn before it. Add one order, or one instrument, and every cost after that point shifts. Comparing two versions of a strategy then compares two different draws of the noise.
- **Order that is not fixed.** If instruments or events are processed in an order that can vary, such as across threads, the same seed hands the numbers out differently.

For the difference between two runs to mean something, both problems have to be solved: the run must repeat, and a small change must move only what it touches.

## What an engine needs to give both

1. **A seed you set.** Every random element, including spread and slippage, comes from it.
2. **Noise keyed to where it is used.** Each sample is computed from the seed and its position, such as the instrument and the bar, rather than taken next from a stream. Then an extra order or instrument does not reshuffle everyone else's costs.
3. **A fixed order of events.** Instruments and fills are processed in a defined order, whatever the hardware or thread count.
4. **Noise that scales with the market.** Variation should be larger on volatile bars and small on calm ones, rather than one fixed width.
5. **Bounds where they make sense.** A spread that can go negative or grow without limit is not realistic. Slippage, on the other hand, can be favourable and can spike.
6. **A switch.** With noise off, costs are exactly your base values, for a clean comparison.

## How to test a vendor's claim

During a trial, with spread and slippage noise switched on:

1. **Repeat.** Run the same backtest at least ten times and hash the full output, including the trade and order logs, not just the summary. Every hash should match.
2. **Change one thing.** Add one order, or one instrument, and compare the logs. Trades the change cannot reach should be identical to the last decimal.
3. **Change the seed.** The results should move, and by a plausible amount. If they do not move, the noise is not doing anything.
4. **Turn noise off.** Fills should land at your base spread and slippage exactly.

## Use the seed as a sample, not a setting

A fixed seed makes a run repeatable. It does not make the result true. One seed is one draw of how the costs could have gone. Run the same strategy over many seeds and look at the spread of results: if the edge holds across them, the costs are not deciding it. If it disappears for some seeds, it was close to the cost line all along.

This is only possible because each seed's run is repeatable. A seed that gives a bad result can be rerun and inspected trade by trade. See [What makes a backtest deterministic, and why does it matter?](/faq/what-makes-a-backtest-deterministic.html)

## How Reamer Research does it

Everything below is stated in the execution specification that ships with the kit:

- **Two settings drive all randomness.** `price_volatility` turns spread and slippage noise on; at `0`, every fill uses your base values exactly, whatever the seed. `rng_seed` (42 by default) fixes the noise; with it fixed, a run is byte-identical on every repeat.
- **Noise keyed to the bar and the instrument.** Each synthetic tick is sampled from the seed, the bar's index and a hash of the ticker. Spread and slippage use their own salts, so neither disturbs the other.
- **Scaled to each bar.** The width of the noise follows that bar's own high-low range. Spread noise is capped at four times the base spread, and the spread never goes below zero. Slippage noise is not capped and can occasionally be favourable, as real price improvement is.
- **Wider at the open.** Separately from the noise, spread is widest at the start of each bar and narrows to its base by the close. See [Spread and Slippage Widen at the Open](/notes/spread-and-slippage-widen-at-the-open.html).
- **Bid and ask.** Buys fill at the ask plus slippage and sells at the bid minus slippage, stop and target exits included. See [Modeling Bid/Ask Correctly](/notes/modeling-bid-ask-correctly.html).
- **Costs per instrument, seed for the run.** Spread, slippage and the amount of noise can be set for each ticker. The seed is one for the whole run; instruments differ because the ticker is hashed into every sample.
- **Measured.** On an AMD EPYC 9575F, 20 of 20 full runs with noise switched on were byte-identical by SHA-256. See [What 2,000 Runs on a 64-Core EPYC Show](/blog/reamer-research-findings-epyc.html).
- **Seed sweeps at full speed.** Each backtest is single-threaded, and separate backtests are safe to run at the same time from your own threads, so a sweep across seeds uses every core.

## Limits to know

- **The guarantee is for repeated runs.** This page makes no claim about identical bytes across different operating systems or processors.
- **A per-instrument seed is ignored.** The seed is always set for the whole run.
- **Slippage does not grow with order size.** It is a price amount per fill. For orders that are large against the volume traded, set a higher slippage for those instruments yourself.
- **No queue position.** A limit order fills once the price reaches it. See [How Limit Orders Really Behave](/notes/how-limit-orders-really-behave.html).
- **No built-in seed sweep.** You run the seeds from your own loop and compare the results yourself.
- **Your own code.** If the strategy draws its own unseeded random numbers or reads the clock, the run will not repeat, however the engine behaves.

The $225 trial includes the full execution specification and the kit, so you can run the four checks above on your own strategy before buying a licence.
