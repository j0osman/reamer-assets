---
title: How should a backtest fill a stop and a target that both fall inside one bar?
description: The bar alone cannot say which was hit first, so no answer is certain. A backtest should settle it by one stated rule, apply that rule the same way on every run, and let you check how much the result depends on it. A strategy whose result changes when the rule changes depends on something the data does not contain.
stage: 3
order: 16.5
product: research
next: deterministic-backtesting-with-slippage-and-spread, backtest-futures-roll-and-fx-swap, research-engine-for-mid-frequency-strategies
date: 2026-10-01
---

The bar alone cannot say which was hit first, so no answer is certain. When a bar's high reaches the target and its low reaches the stop, the open, high, low and close do not record which came first. A backtest has to settle it by a rule. What matters is that the rule is stated, applied the same way on every run, and tested for how much the result depends on it.

This is not a rare case. A strategy with a tight bracket on long bars meets it often, and every time, the backtest chooses between a full win and a full loss on that trade.

## Why the bar cannot answer it

A bar is a summary of an interval: where it opened, the highest and lowest prices it reached, and where it closed. It does not record the order of the high and the low. A bar that opened at 100, reached 103 and 97, and closed at 101 could have gone up first or down first. Both paths give the same four numbers.

If the target is at 102 and the stop is at 98, both were touched. One path makes the trade a winner, the other a loser, and the data is identical.

## The usual rules, and what each does to the result

- **Stop first, always.** The worst case for every ambiguous trade. It understates the strategy, which is the safer error, but it can reject a strategy that works.
- **Target first, always.** The best case. It overstates the strategy, and a bracket strategy tested this way can look profitable when it is not.
- **Guess from the bar's shape.** For example, assume a bar that closed higher went down first and then up. This is a rule of thumb, not a fact about that bar, and it is right only part of the time.
- **Whichever level is closer to the open.** Plausible, but the closer level is not always hit first, and the rule favours tight stops or tight targets depending on how the bracket is set.
- **Drop to shorter bars.** Check the ambiguous bar against one-minute bars, or real ticks, to find the true order. This is the most accurate answer where that data exists, and the costliest to get for every instrument and every year.
- **A simulated path through the bar.** Build a sequence of prices within the bar's range and fill whichever level the sequence reaches first. It is not the real path, but if it is the same every run it gives an answer you can check and repeat.

Whatever the rule, the target should not fill at exactly the target price unless the strategy really used a resting limit order. A take-profit exit that is a market order after a trigger has slippage like any other exit. The note [Bracket orders done correctly](/notes/bracket-orders-done-correctly.html) covers why.

## How much it matters for your strategy

The rule matters in proportion to how many trades it decides. Two checks show you that:

1. **Count the ambiguous trades.** Find the exits on bars whose range reached both the stop and the target. If they are a small share of all trades, the rule barely matters. If they are a large share, the bracket is too tight for the bar length.
2. **Run the best case and the worst case.** Run once with stop first and once with target first. If the strategy is good under both, the rule does not decide it. If it is good only under target first, the edge is in the bars the data cannot see.

If the answer depends on the rule, the fixes are a wider bracket, shorter bars, or data that records what happened within the bar.

## Related details that change the fill

- **Gaps.** A bar that opens beyond the stop fills near the open, not at the stop. See [Stop orders, gaps and reality](/notes/stop-orders-gaps-and-reality.html).
- **Spread at the open.** Spreads are often widest early in a bar, which is when many stops trigger. See [Spread and slippage widen at the open](/notes/spread-and-slippage-widen-at-the-open.html).
- **Bid and ask.** A long position's stop and target both exit by selling, so both should trigger and fill on the bid, not on the bar's printed price.

## How Reamer Research resolves it

[Reamer Research](/products/reamer-research.html) uses a simulated path. What follows is documented in the execution specification that ships with the kit:

- **A tick path through every bar.** The engine builds a sequence of synthetic ticks within each bar, kept inside the bar's high and low. The number of ticks is the seconds to the next bar, or the bar's own trade count if you supply it.
- **First touch wins.** When both the target and the stop are reached in the same bar, the exit whose level is reached at the earlier tick is the one that fills.
- **The same every run.** The path is set by `rng_seed` and the bar data, so a fixed seed gives the same path, the same exits and byte-identical results every run.
- **The stop and the target fill the same way.** Both are triggered and filled on the bid for a long position and the ask for a short one, with slippage. Neither fills at exactly its level.
- **Brackets that move.** A strategy can replace a position's stop and target from `on_bar` with a `MODIFY_POSITION` action, to breakeven or into profit. The new levels apply from the next bar, so a bar the strategy has already seen is settled against the levels it had.
- **Every exit can be checked.** Each closed trade in the result records its stop and target, the exit price, and the bar and tick index of the exit. You can find the trades whose exit bar reached both levels and see how each was settled.

The tick path is not the real path, and the specification does not claim it is. The note [Why synthetic ticks instead of stored ticks](/notes/why-synthetic-ticks-instead-of-stored-ticks.html) explains the reasoning.

## Limits to know

- **It is not real tick data.** If a strategy's edge depends on the true order of prices within a bar, it needs real ticks or much shorter bars, not a better rule.
- **No built-in best-case and worst-case switch.** The engine has one rule, first touch on the tick path. To get the stop-first or target-first bounds, compute them yourself from the closed trades and your bars.
- **No exit-reason field.** Which exit fired is read from the exit price against the stop and target levels each trade records.
- **The path's construction is not published.** The specification states what the path guarantees and how fills are taken from it, not how it is generated.

The $225 trial includes the full execution specification and the kit, so you can run your own bracket strategy and inspect every exit before buying a licence.
