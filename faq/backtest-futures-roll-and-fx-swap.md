---
title: Can I backtest futures with roll handling, or FX with swap costs?
description: Yes, but only if the backtest handles two things a plain price series hides. In a continuous futures series, the price jump at each contract roll must not be counted as profit or loss. An FX position held overnight must be charged or credited swap every night it is held. A backtest that skips either can report a result the account would never have produced.
stage: 3
order: 22
product: research
next: stop-and-target-in-same-bar, deterministic-backtesting-with-slippage-and-spread, backtest-with-exogenous-data, research-engine-for-mid-frequency-strategies
date: 2026-10-01
---

Yes, but only if the backtest handles two things a plain price series hides. In a continuous futures series, the price jump at each contract roll must not be counted as profit or loss. An FX position held overnight must be charged or credited swap every night it is held. Both are easy to miss, because the bars look normal and the backtest runs without complaint.

Neither is a small detail. A strategy that holds futures across rolls can show a steady gain or loss that is nothing but the gap between two contracts. A carry strategy in FX can earn most of its return from swap, or lose it all to swap.

## Why futures rolls distort a backtest

A futures contract expires, so a long history has to be built from a chain of contracts. At each roll date, the series switches from the expiring contract to the next one. The two rarely trade at the same price, so the series jumps at the switch, even though no position gained or lost anything.

There are three common ways to build the series, and each one affects a backtest differently:

- **Unadjusted, or spliced.** The contracts are joined as they traded. Prices are real, but every roll leaves a jump. A position held across it shows profit or loss that did not happen, and stops or limits set before the roll sit at the wrong distance from the new price.
- **Back-adjusted by difference.** Every price before a roll is shifted by the gap, so the series is smooth. Point profit is right, but old prices are no longer real. They can drift far from where the market traded, even below zero, which breaks percentage returns and any rule tied to price levels.
- **Back-adjusted by ratio.** Every price before a roll is scaled by the ratio between the contracts. Percentage moves are right, but old point values are not, so point-based stops and costs are off in early history.

Three more things go wrong often:

- **The roll date.** The date your data vendor rolled on and the date you would roll on live should be the same. A series rolled on volume, tested against a strategy that rolls on a fixed calendar day, measures neither.
- **The cost of rolling.** Rolling live means closing one contract and opening the next, which costs spread and commission twice. A backtest on a continuous series that simply holds through the roll charges nothing.
- **Contract size.** A price move of one point is worth a fixed amount per contract, set by the exchange. A backtest that treats one contract as one unit of price understates or overstates profit, costs and margin by that factor.

## Why FX swap distorts a backtest

An FX position still open at the daily cutoff, usually 5 p.m. New York time, is rolled to the next settlement date. The account pays or earns the interest difference between the two currencies, plus the broker's markup. This is swap, also called rollover or financing.

What a backtest needs to get right:

- **Direction.** A long position and a short position in the same pair get different rates. With the broker's markup, both sides can pay.
- **The weekend charge.** Swap for the weekend is usually charged on one weekday, often Wednesday, at about three times the normal rate. A strategy that holds over that one night pays three nights of swap at once.
- **Size against the edge.** For a strategy that holds for days or weeks, swap can be larger than the spread and commission together. For a carry strategy, it is the return.
- **The rollover hour.** Spreads widen around the daily cutoff. A strategy that trades then should be tested with that wider spread. See [How realistic do slippage and spread need to be in a backtest?](/faq/how-realistic-slippage-and-spread.html)

Bid and ask matter more in FX than almost anywhere else, because the spread is a large share of a typical move. The note [Modeling bid and ask correctly](/notes/modeling-bid-ask-correctly.html) covers how a backtest should derive both from one series.

## What a backtest needs from the engine

To test futures and FX properly, an engine should give you:

1. A way to stop roll jumps from counting as profit, matched to how your series was built.
2. Roll dates that you set, so they match your data and your live rule.
3. Overnight financing that differs by day of the week and can be set for each instrument.
4. Separate costs for each instrument, because a futures contract and a currency pair in the same portfolio do not share a spread, commission or swap.
5. A record of every roll and every swap charge, so you can check them.

## How Reamer Research handles it

[Reamer Research](/products/reamer-research.html) runs any OHLCV series, so futures, FX, equities and other instruments can share one portfolio in one run. Commission, slippage, spread, bid or ask pricing, swap and the roll settings can each be set for every instrument in the run. Instruments without their own settings use the run's defaults. What follows is documented in the execution specification that ships with the kit.

**Futures rolls.** The roll feature is for an unadjusted, spliced series. It is off by default, and a run with it off is byte-identical to a run on a build without it.

- **Your roll calendar.** You set the months the roll happens in, for example March, June, September and December. You also set how many business days before the end of that month it happens; the default is 5.
- **Positions and orders move with the roll.** When a roll date passes, the engine measures the jump as the ratio of the open of the first bar on or after the roll date to the close of the bar before it. It scales the open position's entry price, stop and target, and every working order's prices, by that ratio. Profit from then on is measured against the new contract's prices, so the jump itself is not counted.
- **Every roll is recorded.** Each one is written to `roll_log` in the result with the instrument, the time and the ratio, so you can check every adjustment.

**FX swap.** You set one swap amount per unit for each day of the week, Sunday to Saturday, and it can differ for each instrument. It is charged once each time the bar sequence crosses into a new calendar date. A long position pays the amount and a short position receives it. A setting three times the normal amount on the right weekday covers the weekend charge. The run's total appears as `total_swap_cost`.

## Limits to know

- **No contract-size field.** Profit is price change times quantity, and margin is checked as quantity times price against equity times leverage. Exchange margin is not modelled. A futures contract's point value has to be carried in your quantities or prices, with per-unit costs set to match.
- **A roll is not a trade.** The engine adjusts prices; it does not close and reopen the position, so it charges no spread or commission for the roll. To test that cost, close and reopen in your strategy at the roll.
- **The bars you read are not adjusted.** Your strategy still sees the raw jump in the price series at a roll, so indicators computed from it need their own handling.
- **The ratio is close to open.** It includes any real price move between those two bars, not only the gap between the contracts.
- **One bar is not rechecked.** A stop or target on the bar where the roll happens was already scheduled against the old levels, and is not recalculated for that bar. The specification states this as a deliberate edge case.
- **Swap is the same rate for long and short.** A long pays exactly what a short receives on the same night, so a broker's markup on both sides cannot be set directly.
- **Off in the worked references.** The C++ and Python reference integrations leave the roll calendar empty and swap at zero. You set them in your own integration.
- **Bars only.** No options, no order-book data, and no corporate-action adjustment: data must arrive already adjusted for splits and dividends.

The $225 trial includes the full execution specification and the kit, so you can test roll and swap handling on your own data before buying a licence.
