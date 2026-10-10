---
title: How realistic do slippage and spread need to be in a backtest?
description: Realistic enough that the result survives them. Set spread, slippage and commission per instrument from what your broker actually charges, let spread and slippage widen when the market moves fast, and then rerun at one and a half and two times those costs. A strategy whose edge disappears under slightly higher costs was never a reliable edge.
stage: 1
order: 4
product: research
next: what-makes-a-backtest-deterministic, backtest-vs-live-results, deterministic-backtesting-with-slippage-and-spread, research-engine-vs-backtest-script
date: 2026-09-29
---

Costs are the part of a backtest most often set to zero, and the part most likely to decide whether a strategy makes money. They do not need to be perfect. They need to be realistic enough, and varied enough, that the result still holds when they are a little worse than expected.

## Why costs matter more than they look

Costs are paid on every trade, so their weight grows with how often a strategy trades and shrinks with how much each trade earns.

Take a strategy that makes an average of 0.06% per trade before costs and trades 1,000 times a year. A round-trip cost of 0.02% leaves two thirds of the edge. A round-trip cost of 0.05% leaves one sixth. A cost of 0.07% turns it into a loser. None of those cost levels is unusual. The same strategy is good, marginal or losing depending on a number that many backtests leave at zero.

The simplest check follows from this: compare the average profit per trade before costs with a realistic round-trip cost. If they are close, no amount of tuning will make the strategy reliable.

## The four costs to model

1. **Spread.** Buys pay the ask and sells receive the bid. Know what your price data is: bid, ask or midpoint. If your data is bid prices and your backtest buys at them, every entry is too cheap by a full spread. See [Modeling Bid/Ask Correctly](/notes/modeling-bid-ask-correctly.html).
2. **Slippage.** The difference between the price you saw and the price you got. It grows with speed of movement, with order size relative to what is trading, and with order types that chase the market, such as stops.
3. **Commission and fees.** Usually the easiest to get right, because your broker publishes them.
4. **Holding costs.** Overnight financing on leveraged or FX positions. Small per night, large over a year of holding.

## What "realistic" means in practice

- **Per instrument.** One cost for a whole universe overcharges liquid instruments and undercharges thin ones, so the backtest drifts toward trading the thin ones.
- **From your own fills.** Base values should come from your broker's quotes and your own trade statements, not a round number. If you have no live fills yet, take typical quoted spreads for the hours you trade, and err on the high side.
- **Wider when the market moves.** Spread and slippage both widen when prices move fast, and spread is usually widest just after a bar opens. A single fixed cost understates the trades made in exactly those moments. See [Spread and Slippage Widen at the Open](/notes/spread-and-slippage-widen-at-the-open.html).
- **Set before you tune.** Parameter searches drift toward settings that trade more. Tuning with no costs selects exactly the settings that costs hurt most, so set costs first, then sweep. See [How do I know if my backtest is overfit?](/faq/is-my-backtest-overfit.html)

## Stress-test the costs

Once the base costs are set, rerun the finished strategy with every cost multiplied by 1.5, then by 2.

- **The result degrades gently.** The edge is larger than the uncertainty in your cost estimate. Good.
- **The result collapses at 1.5.** The edge is about the same size as the cost, and any error in the estimate decides the outcome.

The multiplier at which the strategy stops making money is its break-even cost. The further that is from your realistic estimate, the more room you have for the live market to be worse than the model.

The comparison only means something if the runs differ in nothing but the costs. If the backtest gives different results each time it runs, the stress test measures noise. See [Why do I get different backtest results each time I run it?](/faq/backtest-results-change-between-runs.html)

## Strategies most exposed to cost errors

- High-turnover strategies, where cost is paid often.
- Strategies that enter on breakouts or through stop orders, which fill as the market moves against them.
- Strategies that trade near the open, or into volatility, where spread is widest.
- Strategies that trade thin instruments or large sizes relative to volume.

A slow strategy that trades a few times a month in liquid instruments can tolerate a rough cost model. A fast one cannot.

## How Reamer Research models costs

[Reamer Research](/products/reamer-research.html) applies the rules in [`EXECUTION_SPEC.md`](/docs/execution-spec.html), which ships in the kit and is published in the docs:

- **Per instrument.** Spread, slippage, commission, commission mode, overnight swap, the type of price data, and the amount of noise are each set per ticker, with a default for any ticker left unset.
- **Bid and ask, not one price.** You declare whether your bars are bid, ask or midpoint prices. Buys fill at the ask plus slippage, sells at the bid minus slippage, including limit, stop and bracket exits, so a fill can land outside the bar's own high and low by up to the spread plus slippage.
- **Costs that move with the market.** With noise enabled, spread and slippage are drawn around your base values on every synthetic tick, scaled by that bar's own high-low range. Spread noise is capped at four times the base. Slippage noise is uncapped and can occasionally be favourable, as real price improvement is. Spread is also widest at the start of each bar and narrows toward its close.
- **Stress tests that compare cleanly.** With a fixed seed every run is byte-identical, so a rerun at higher costs differs only in the costs. The result reports commission, slippage and swap totals separately, so you can see what each cost took.

Two limits are worth knowing. Slippage is a price amount per fill and does not grow with order size, so if your orders are large relative to the volume traded, set a higher slippage for those instruments yourself. And a limit order fills once the price reaches its level, with no model of where the order stood in the queue, so a limit-heavy strategy should be judged with that in mind. See [How Limit Orders Really Behave](/notes/how-limit-orders-really-behave.html).

The base values are always yours. The engine applies them honestly and the same way every run. Whether they match your broker is something only your own fills can confirm, so compare them after your first weeks of live trading and update the model.
