---
title: My strategy was profitable in the backtest but loses money live. How do I find out why?
description: Rerun the backtest over the exact dates you traded live, then match each live trade to its backtest trade. Missing or extra trades point to the code or the data. Same trades at worse prices point to spread, slippage and costs. If the backtest also lost money over those dates, the market changed or the strategy was never robust. If you have only a few trades, the loss may still be normal variation, so count them before you change anything.
stage: 1
order: 1.1
product: both
next: backtest-vs-live-results, is-my-backtest-overfit, how-realistic-slippage-and-spread, backtest-results-change-between-runs
date: 2026-11-04
topics: backtesting, slippage, transaction-cost
---

Do not start by changing the strategy. First compare what the backtest would have done with what actually happened, over the same dates. That comparison usually narrows the problem to a few causes, and each one has a different fix.

For the full list of causes, see [Why don't my backtest results match live trading?](/faq/backtest-vs-live-results.html) This page is the procedure for finding which one is yours.

## 1. Count your live trades

Check that there is a problem to find. With few trades, a working strategy can easily look broken.

A strategy that truly wins 55% of its trades will usually show between 35% and 75% over 20 trades. A losing month inside that range tells you very little. Also compare the live drawdown with the worst drawdowns in the backtest. If the live loss is smaller than losses the backtest has already been through, keep collecting trades unless the money at risk says otherwise.

## 2. Rerun the backtest over the live dates

Use the same settings, the same instruments and the same starting balance, on data covering the dates you traded. Then ask one question: did the backtest make money over those dates?

- **The backtest also lost money.** Your live system probably did what it should. Go to step 5.
- **The backtest made money.** Live trading differs from the test. Go to step 3.

## 3. Match the trades one by one

Export both trade lists and line them up by instrument and time. Sort every trade into one of these groups:

- In both lists.
- In the backtest only.
- Live only.

Trades that appear in only one list come first, because a price difference means little if the two systems made different decisions. Common causes:

- An indicator is computed differently live, or starts from a different history.
- The live data feed differs from the backtest data, such as bid against last price, or a different session or time zone.
- The backtest decided on a bar's close and filled inside that same bar.
- An order was rejected live, or a limit order never filled.

## 4. Measure the price gap on matched trades

For each matched trade, take the live entry and exit prices and subtract the backtest prices, then average the difference per trade. Add real commission and overnight financing from your broker statement.

Compare that total with the backtest's average profit per trade. If live costs eat most of the average trade, the strategy has too little edge at its current frequency, or the cost model was too kind. Put the measured costs into the backtest and rerun the whole history. If the result no longer makes money, you have your answer.

Check when the worst slippage happens, too. Fills at the open or around news usually cost more than the average suggests.

## 5. Test whether the strategy was ever robust

If the backtest also lost money over the live dates, the question moves from execution to the strategy itself.

- Move each parameter a little either way. A result that exists at only one setting was probably noise.
- Split the history by period, and see whether the profit comes from one short stretch.
- Test on any data you kept out of the design.

See [How do I know if my backtest is overfit?](/faq/is-my-backtest-overfit.html)

## What each result means

| What you see | Likely cause | What to do |
|---|---|---|
| Too few trades to judge | Normal variation | Keep collecting trades, within your risk limit |
| Backtest also lost over the same dates | Overfitting or a changed market | Test parameters and periods, then rethink the strategy |
| Trades missing or extra | Different code, data or order handling | Fix the difference, then repeat the match |
| Same trades, worse prices | Spread, slippage, commission or financing | Put measured costs in the backtest and rerun all of it |
| Same trades, same prices, but different profit | Account or position sizing differences | Compare balances, sizing and currency conversion |

## What makes this comparison possible

The procedure needs two things from a backtest: a full trade list, and the same answer each time you run it. If a rerun changes the trades, you cannot tell a live difference from noise in the tool. See [Why do I get different backtest results each time I run it?](/faq/backtest-results-change-between-runs.html)

[Reamer Research](/products/reamer-research.html) returns every order, fill and closed trade, with the reason for every rejected order, and a fixed seed gives byte-identical output on every run. Spread, slippage, commission and overnight swap are set per instrument, so you can put your measured costs back into the model. `reamer_run_monte_carlo()` resamples a run's trades to show how much of a drawdown was trade-order luck.

It cannot tell you why your broker filled you where it did, and it will not give a strategy an edge it does not have. It runs on Linux x86-64 and on macOS on Apple Silicon. The $225 30-day trial lets you run this comparison on your own strategy and data. See [pricing](/pricing.html).
