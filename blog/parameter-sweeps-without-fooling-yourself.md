---
title: "Parameter Sweeps Without Fooling Yourself"
description: There's no dedicated sweep engine in Reamer — a sweep is just a loop over configs — and that absence of built-in guardrails means the discipline against overfitting to the sweep itself has to come from whoever's running it, not from the tool.
date: 2026-07-19
tier: Practical Research Workflows
---

Sweeping a parameter — trying a moving-average length across a range of values, say, and keeping whichever one performed best — isn't a special operation in Reamer. It's an ordinary Python loop calling `run_backtest` once per combination and comparing the results afterward. That's a deliberate absence, not a missing feature: a dedicated sweep-optimization engine is a different kind of tool built around exploring a fixed parameter space efficiently, and that's not the problem Reamer is built to solve. The problem worth being honest about is what a loop like that actually proves.

## The sweep itself is where the overfitting happens

Running fifty parameter combinations against the same historical window and keeping the best one isn't the same as finding a genuinely better strategy — it's closer to running fifty separate hypotheses and reporting only the one that happened to work, on data all fifty saw. The best-performing combination out of fifty candidates will look artificially strong even if every single candidate had a real, roughly equal edge, purely because the best of many noisy draws is expected to look better than any one of them individually. A sweep result reported without accounting for how many combinations were tried isn't a stronger backtest — it's a backtest with the multiple-comparisons problem quietly built into how it was produced.

```python
best = None
for lookback in range(5, 60, 5):
    cfg = StrategyConfig(lookback=lookback)
    result = reamer_py.run_backtest(data, MyStrategy(cfg), exec_config, ...)
    if best is None or result.net_pnl > best.net_pnl:
        best = result
```

## What actually earns trust here

The loop above finds a candidate. It doesn't validate one. The parameter that won the sweep still has to clear the same bar any single strategy would — [Monte Carlo robustness testing](https://reamerlabs.com/blog/why-monte-carlo-matters) against out-of-sample behavior, not just the in-sample number that made it win the sweep in the first place. Treating the winning parameter as already-proven because it beat forty-nine alternatives skips exactly the step that would catch whether it beat them for a real reason or because it happened to fit this particular slice of history best. The sweep narrows the search. It was never supposed to be the validation.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
