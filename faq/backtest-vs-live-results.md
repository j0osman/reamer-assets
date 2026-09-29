---
title: Why don't my backtest results match live trading?
description: Usually because the backtest assumed cheaper costs and easier fills than the market gives, used information that was not available at the time, or tested a slightly different strategy from the one that went live. Each cause can be found and closed before real money is at stake.
stage: 1
order: 1
product: both
next: how-realistic-slippage-and-spread, backtest-results-change-between-runs, backtest-to-live-without-rewrite, deterministic-backtesting-with-slippage-and-spread
date: 2026-09-29
---

A gap between backtest and live results almost never has one cause. It is usually several small ones adding up in the same direction: each makes the backtest a little more optimistic than the market, and together they turn a profitable-looking strategy into a flat or losing one. The causes fall into five groups, and each can be checked.

## 1. Costs were modelled too kindly

The most common cause. A backtest with zero spread, zero slippage and zero commission is measuring an idealised market that does not exist.

- **Spread is a cost you pay on every round trip.** A buy pays the ask and a sell receives the bid. If your data is bid prices and the backtest fills buys at them, every entry is too cheap. See [Modeling Bid/Ask Correctly](/notes/modeling-bid-ask-correctly.html).
- **Spread and slippage are not constant.** Both widen when the market is moving fast, and spread is widest near the open. A single fixed cost per trade understates exactly the moments a breakout or momentum strategy trades in. See [Spread and Slippage Widen at the Open](/notes/spread-and-slippage-widen-at-the-open.html).
- **Costs differ by instrument.** One cost setting for a whole universe overcharges liquid names and undercharges thin ones.
- **Holding costs add up.** Overnight financing on FX or leveraged positions is small per night and large over a year.

A strategy with a thin edge per trade is the most exposed. If the average trade earns less than a realistic round-trip cost, no amount of tuning fixes it.

## 2. The backtest gave fills the market would not

- **Trading on the bar that produced the signal.** A decision made from a bar's close cannot fill at a price from earlier in that same bar. A backtest that allows it is quietly trading on information it did not yet have.
- **Stops that fill at the stop price through a gap.** When the market opens beyond your stop, the fill is near the open, not at your level. See [Stop Orders, Gaps, and Reality](/notes/stop-orders-gaps-and-reality.html).
- **Brackets where both exits were touched in one bar.** If the take-profit and the stop-loss both sit inside one bar's range, which came first decides the trade. Assuming the take-profit always wins inflates results. See [Bracket Orders Done Correctly](/notes/bracket-orders-done-correctly.html).
- **Limit orders treated as market orders at a better price.** A limit order waits for its level and may never fill. See [How Limit Orders Really Behave](/notes/how-limit-orders-really-behave.html).

## 3. The backtest used information that was not available yet

- **Data that was revised later.** Earnings dates, fundamentals and economic figures are often stored as their final values, not as what was known on the day. A strategy that reads them without point-in-time alignment is looking ahead. See [Testing Earnings Strategies with Exogenous Data](/notes/testing-earnings-strategies-with-exogenous-data.html).
- **Price data that was not adjusted, or was adjusted the wrong way.** An unadjusted stock split looks like a crash. A universe built from today's index members leaves out every company that later failed, so the past looks safer than it was.

## 4. The result was luck, not an edge

- **One good parameter set.** If the result only holds at one lookback and one threshold, and falls apart at the neighbours, the backtest found noise. A robust strategy degrades gently as parameters move.
- **Results that change between runs.** If running the same backtest twice gives two answers, you cannot tell whether a change in results came from your strategy or from the tool. Every comparison you make afterwards is built on sand.

## 5. The live strategy is not quite the tested one

Even when the backtest was honest, the move to live can change the strategy without anyone noticing.

- **Indicators computed differently.** In a backtest, an indicator usually reads a full window of history on every bar. Live, it has to be maintained as running state, one update at a time. A small difference in how it starts or updates changes the signals.
- **Instruments mapped wrongly.** If a symbol in the live system silently refers to a different instrument than it did in the backtest, the strategy trades the wrong thing with the right logic.
- **Costs that do not carry over.** Backtest costs are a model. Live costs are whatever your broker and venue charge. Compare the two after the first weeks of trading, and update the model.

## How Reamer Research and Reamer Server handle each cause

[Reamer Research](/products/reamer-research.html) is built to close groups 1 to 4 inside the backtest, and every rule it applies is written down in `EXECUTION_SPEC.md`, which ships in the kit:

- Spread, slippage and commission are set per instrument. Spread is widest at the start of each bar and narrows toward its close, and with volatility enabled both spread and slippage scale with each bar's own range.
- An order decided on a bar's close is only eligible from the next bar onward. Stops that gap fill near the open. When both bracket exits are touched in one bar, the one reached first in the bar's price path wins.
- Non-price data such as earnings dates is resolved point-in-time: the strategy sees only the latest value dated at or before the current bar.
- A fixed seed gives byte-identical output on every run, so a change in results is always a change in the strategy or the inputs. Parameter sweeps then show whether a result is robust or one lucky configuration.

Data quality stays yours. The engine checks that bars are well formed, not that prices are correctly adjusted.

For group 5, [Reamer Server](/products/reamer-server.html) runs the same decision logic live, and `RESEARCH_TO_SERVER.md` lists exactly what changes on the way: indicators that read a lookback window become running state, instruments are mapped from backtest identifiers to live names, and no cost settings carry over. Backtest results carry your own instrument names, so the backtest and live instrument lists can be compared as a build step instead of trusted.

No backtest removes the gap entirely: live fills depend on your broker and venue, and the market will always differ from any model of it. What a careful backtest does is make the gap small and known in advance, so you find it before real money does.
