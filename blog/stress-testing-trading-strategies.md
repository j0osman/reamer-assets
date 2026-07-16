---
title: Stress Testing Trading Strategies
description: Monte Carlo resamples what already happened under the same assumptions. Stress testing asks a different question — what happens if the assumptions themselves get worse, not just the specific sequence of trades drawn from them.
date: 2026-07-17
---

[Monte Carlo asks how sensitive a result is to which specific trades happened to occur](https://reamerlabs.com/blog/why-monte-carlo-matters), holding everything else fixed. Stress testing asks a different, complementary question: what happens if the conditions themselves get worse — costs a bit higher, positions a bit larger, correlations a bit less favorable than assumed — rather than just resampling from the same fixed assumptions.

## Assumptions, not just outcomes, deserve scrutiny

Every backtest runs on a specific set of assumptions about costs and conditions — a particular slippage value, a particular spread, a particular leverage level. Those assumptions are estimates, not guarantees, and a strategy's real-world performance depends on how wrong those estimates are allowed to be before the result stops holding up. Re-running a strategy with meaningfully worse cost assumptions than the base case — not because those worse numbers are expected, but to see how much room for error the strategy actually has — is a different exercise than resampling the trades that already happened under the original assumptions.

## What this looks like in practice

Deliberately widening slippage and spread beyond the base-case estimate and re-running the backtest shows how much of the apparent edge depends on costs staying close to their assumed values. Testing at reduced leverage, or with per-instrument costs pushed toward their less favorable end, shows whether the strategy's margin for error is comfortable or razor-thin. None of this is about predicting the future correctly — it's about knowing, in advance, how much the strategy actually depends on today's assumptions being right.

## Why this matters alongside Monte Carlo, not instead of it

Monte Carlo and stress testing catch different failure modes. A strategy can be robust to which specific trades happened to occur and still be fragile to costs turning out to be worse than assumed — or the reverse. [Checking both](https://reamerlabs.com/blog/when-to-trust-an-edge) is what distinguishes a strategy whose edge holds up under real-world variation from one that only survives inside the exact conditions it happened to be measured under.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
