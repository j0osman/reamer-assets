---
title: "What a Rolling Window Actually Costs"
description: A lookback window isn't free just because the syntax to request one is a single number. What a rolling window of OHLCV history actually costs in memory and compute, as that number grows, shapes how large a lookback a research loop can actually afford to run.
date: 2026-07-19
tier: Engineering
---

Requesting a hundred bars of lookback instead of ten looks like a one-character change in a strategy's code. What it costs underneath is a different question, and it's the one that decides whether a research idea that wants a long lookback is actually affordable to run at scale.

## The shape of the cost

Reamer's `on_bar` path keeps a rolling window of OHLCV history per ticker, sized to whatever lookback a strategy declares. The cost of that window is a function of two things multiplied together: how many bars of history it holds, and how many timesteps the backtest runs for — not a fixed cost paid once. A strategy with a 20-bar lookback over a year of 1-minute bars is doing meaningfully less work per step than the same strategy with a 500-bar lookback over the same data, and that difference compounds across every single step of the run, not just the first one.

## Why a bounded window, not the full history

The alternative — handing a strategy the entire history up to the current bar, every step — would make the per-step cost grow across the *whole backtest*, not just with the declared lookback, since the window a strategy could look at would keep getting larger as the run progressed. Reamer bounds the window explicitly to what the strategy actually declared it needs, so the per-step cost stays flat across the length of the run instead of growing with it. A strategy that only reasons about the last 20 bars never pays for the ability to see bar 1 from bar 100,000.

## Why this is worth understanding before choosing a lookback

None of this is a reason to keep lookback windows small out of caution — a strategy that genuinely needs 500 bars of context should ask for 500. It's a reason to treat lookback as a real cost parameter, the same way a database query's row count is, rather than an implementation detail invisible until a backtest that used to take seconds starts taking minutes for reasons that aren't obvious from the strategy code itself.

## Distinct from running many tickers

This is a cost that scales with lookback depth per ticker, independent of how many tickers a backtest runs — that's a different scaling question, covered separately in [Scaling Multi-Asset Backtests](https://reamerlabs.com/blog/scaling-multi-asset-backtests). A strategy with a deep lookback on one instrument and a strategy with a shallow lookback across a hundred instruments are stressing two different parts of the engine, not the same one.

---

Full reference: [docs](https://reamerlabs.com/docs)
