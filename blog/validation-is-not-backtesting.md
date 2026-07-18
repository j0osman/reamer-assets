---
title: Validation Is Not Backtesting
description: A backtest is a single measurement. Validation is a test of whether that measurement can be trusted. Treating "it backtested well" as equivalent to "it's validated" is a common, costly mix-up.
date: 2026-07-16
---

"I backtested it and it made money" gets treated, a lot of the time, as though it already means "I validated the idea." It doesn't. Backtesting and validation answer two different questions, and conflating them is one of the more common ways a strategy ends up trusted well before it's earned that trust.

## Two different questions

A backtest answers: *what happened when this strategy ran against this specific sequence of historical data?* That's a single measurement — one path through one dataset, with one specific sequence of trades, one specific equity curve, one specific number at the end.

Validation answers a different question entirely: *would this result hold up if the data had been even slightly different?* Not "what happened," but "how much should this one measurement be trusted." A backtest can be executed flawlessly — accurate fills, realistic costs, no bugs anywhere — and still be a single lucky draw that happens to look like a real edge. Validation is the process of checking for that possibility instead of assuming it away.

## Why the mix-up is costly

A backtest, on its own, has no way to distinguish a real, repeatable edge from a strategy that happened to fit the specific noise in one particular dataset. Both can produce an identical-looking equity curve. The measurement can't tell them apart — only a test of how sensitive that measurement is to the exact data it ran against can. Treating a good backtest as proof, without ever asking that question, is how a strategy that's actually indistinguishable from noise ends up funded with real capital and real confidence.

## What this looks like concretely

`run_backtest()` produces the measurement — a specific result, from a specific run, against specific data. [`run_monte_carlo()` is a separate, deliberate step](https://reamerlabs.com/blog/why-monte-carlo-matters): it bootstrap-resamples that result's trade sequence to build a distribution of plausible alternative outcomes, and reports how the actual result sits inside that distribution — risk of ruin, drawdown percentiles, how much of the apparent edge survives across variations rather than depending on the exact sequence that happened to occur.

These are two different function calls because they're two different questions. A backtest can pass the first one — run cleanly, produce a strong number — and still fail the second, if the strength of that number turns out to depend heavily on the specific path the data happened to take.

## A good backtest and a validated idea are not the same milestone

This is the practical consequence worth holding onto: an excellent-looking backtest is the beginning of the question, not the end of it. Nothing about a clean run, on its own, tells you whether the result would survive being asked the harder question. Only actually asking it does.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model behind the backtest itself: [execution specification](https://reamerlabs.com/spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
