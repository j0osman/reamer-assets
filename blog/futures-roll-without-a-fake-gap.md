---
title: "Rolling a Continuous Futures Contract Without a Fake Gap"
description: Splicing front-month futures contracts together at expiry creates a price jump that has nothing to do with the market. Reamer rebases execution bookkeeping around that jump automatically — with one real limit worth knowing before relying on it.
date: 2026-07-19
tier: Practical Research Workflows
---

A continuous futures series is a splice, not a single instrument. Front-month contracts expire, trading rolls to the next one, and if that handoff isn't adjusted for, the price series has an artificial jump sitting in it at every roll date — sized by whatever the front-to-next-month spread happened to be that day, with no relationship to anything the strategy is actually reacting to. A stop-loss set the day before a roll is now measured against a price series that moved for a reason unrelated to the market.

## Configuring the roll

`roll_cycle_months` and `roll_days_before_monthend`, set per-ticker via [`ticker_overrides`](https://reamerlabs.com/docs#ticker-overrides), control when a continuous contract rolls and how far ahead of month-end it happens — quarterly, monthly, whatever the underlying contract's own cycle actually is. Left empty, the default, rolling is disabled entirely, identical to today's behavior for a ticker that doesn't need it.

```python
cfg = reamer_py.DefaultExecutionModelConfig()
cfg.roll_cycle_months         = [3, 6, 9, 12]   # quarterly cycle
cfg.roll_days_before_monthend = 5               # business days before month-end; default
```

## What the roll actually adjusts

The roll is a ratio-rebase applied to execution-model bookkeeping — an open position's entry price, any attached take-profit or stop-loss, and any resting order price levels are all ratio-adjusted forward from the roll date, so a stop set relative to the old contract stays correctly positioned relative to the new one. That's what actually prevents the fake-gap problem from corrupting a strategy's risk levels across a roll, and every applied roll is logged — ticker, timestamp, and the exact ratio used — so which adjustments fired is always inspectable after the fact rather than trusted blindly.

## The one real limit — this doesn't rewrite history

Roll adjustment fixes execution bookkeeping. It does not rewrite historical bars — a strategy's own price history still shows the raw splice jump across the roll boundary exactly as it was in the source data. Anything computed from a lookback window that spans a roll date — a moving average, a breakout level, a volatility measure — sees the same discontinuity it would see with roll adjustment turned off entirely. Calling this a general-purpose continuous-contract price adjustment would be exactly the kind of overclaim [Reamer's approach to execution modeling exists to avoid](https://reamerlabs.com/blog/why-execution-modeling-matters) — it protects the bookkeeping around a position, not the price series a strategy reads from.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
