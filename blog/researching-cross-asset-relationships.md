---
title: "Researching Cross-Asset Relationships"
description: The moment a decision compares tickers to each other — ranking, correlation, a spread — a per-ticker loop stops being merely slower and becomes structurally the wrong shape for the question being asked.
date: 2026-07-19
tier: Practical Research Workflows
---

A strategy that decides independently for each ticker and a strategy that decides by comparing tickers to each other look almost identical in code — both iterate over a ticker list, both read from the same `data` object. The difference only matters once the logic actually needs to rank, weight, or correlate instruments against one another, and at that point a per-ticker loop with conditionals isn't just an inconvenient way to write it — it's the wrong shape for the question being asked.

## Why the loop shape breaks the question

A loop processes one ticker at a time, in some fixed order, making a decision at each step before moving to the next. "Which four tickers out of twenty have the strongest momentum right now" isn't a question any single iteration of that loop can actually answer — answering it requires looking at all twenty first, then deciding. Writing that logic as sequential per-ticker conditionals invites exactly the kind of subtle bug that's hard to notice in a code review: a decision at ticker three that should depend on tickers one through twenty being compared against each other, quietly implemented instead as if ticker three could be judged in isolation.

## Build the cross-section first, decide second

The fix is mechanical: build one array per ticker into a numpy array, compute the cross-sectional comparison across the whole array, and only then decide per instrument — rank, correlation coefficient, spread z-score, whatever the relationship actually is.

```python
closes = np.array([data[t].close[-1] for t in tickers])
oldest = np.array([data[t].close[0]  for t in tickers])
momentum = closes / oldest - 1
ranked = np.argsort(momentum)[::-1]   # now genuinely comparing all tickers at once
```

The arrays exist before any decision is made, so "how does this ticker compare to the others" has an actual answer computed from the full set — not an approximation reconstructed one ticker at a time inside a loop that never had access to the others while it was running.

## The result is the honest version of the same idea

None of this changes what the strategy is trying to test — a genuine belief that some tickers behave differently, or predictably relative to each other. What it changes is whether the backtest is actually testing that belief or a distorted version of it introduced by the way the comparison got implemented. A cross-asset strategy that "backtests well" written the loop-and-conditionals way is [a weaker result than it looks](https://reamerlabs.com/blog/curve-fitting-vs-evidence) — the same idea tested against a real cross-section can disagree with it, and the loop version never throws an error to say so.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
