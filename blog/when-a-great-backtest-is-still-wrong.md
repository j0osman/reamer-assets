---
title: When a Great Backtest Is Still Wrong
description: A backtest can pass every check covered so far — realistic execution, enough data, no lookahead — and still be sitting at the fortunate edge of what its own trades could plausibly have produced, a fact only Monte Carlo actually reveals.
date: 2026-07-17
---

A backtest can do everything right and still be misleading. Realistic execution costs, enough trades to mean something, no lookahead bias, nothing curve-fit about the parameters — and the result can still be sitting at the fortunate edge of what the strategy's own trades could plausibly have produced, rather than near the middle of it. That's a different failure mode than any of the ones already covered, and it's specifically the one Monte Carlo exists to catch.

## Passing every earlier check isn't the same as being robust

[Each of the checks covered elsewhere in this loop](https://reamerlabs.com/blog/trusting-a-backtest) rules out a specific way a result can be wrong — unrealistic fills, insufficient data, a fitted rather than a real pattern. A strategy can clear every one of them honestly and still have gotten a favorable roll of the dice in exactly which trades happened to occur, in exactly what order and mix. None of the earlier checks are built to catch that; they're answering different questions.

## What this actually looks like

A strategy backtests well, the execution model is realistic, the sample size is reasonable, nothing about the parameters looks tuned — and then [a Monte Carlo pass](https://reamerlabs.com/blog/why-monte-carlo-matters) shows a wide, uncomfortable spread of plausible outcomes, with the actual historical result sitting well toward the favorable end of it. Nothing about the original backtest was dishonest. It just happened to be a good draw from a distribution that also contains meaningfully worse ones, and the single historical run never had a way to reveal that on its own.

## Why this is worth stating on its own

It would be convenient if clearing every other check meant a result was automatically safe. It doesn't, and treating it as though it does is how a strategy that's been carefully, honestly tested can still turn out to be riskier than its one visible history suggested. The specific thing missing from an otherwise well-built backtest is a look at the spread of outcomes its own trades could have produced — not a flaw in how it was built, just a question it was never going to answer by itself.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
