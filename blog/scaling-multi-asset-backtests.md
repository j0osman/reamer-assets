---
title: "Scaling Multi-Asset Backtests"
description: Running a strategy across fifty instruments isn't fifty single-asset backtests glued together — it's one aligned timeline, one shared equity curve, and a per-step cost that scales with ticker count regardless of how deep any single ticker's lookback is.
date: 2026-07-19
tier: Engineering
---

A strategy that reasons across many instruments at once — a cross-sectional ranking, a spread, a portfolio sized against total equity — needs every ticker's current bar available at the same step, aligned to the same timestamp, not reconstructed from N separate single-asset runs after the fact. That requirement changes what "scaling" means for a multi-asset backtest: the question isn't just "how fast is one bar," it's "how does cost grow as the ticker count grows."

## Alignment has to happen once, not per strategy

Before a multi-asset `on_bar` call can fire, every ticker's timeline has to be reconciled against every other ticker's — union or intersection, depending on whether a step should fire when *any* ticker has a bar or only when *all* of them do. That alignment work happens once, up front, as part of loading the data, rather than being re-derived by every strategy that runs against the same set of instruments. A strategy author never hand-rolls "what timestamp do these fifty tickers actually share" — that's infrastructure, not strategy logic.

## What grows with ticker count, and what doesn't

Per-step cost inside `on_bar` scales with how many tickers have data to hand over at that step — accessing fifty tickers' current bars costs more than accessing five, in a way that's linear in ticker count, not incidental. What doesn't scale with ticker count is each ticker's own lookback-window cost, which is a separate axis covered in [What a Rolling Window Actually Costs](https://reamerlabs.com/blog/what-a-rolling-window-actually-costs). A hundred-ticker backtest with a 10-bar lookback and a five-ticker backtest with a 500-bar lookback are stressing genuinely different parts of the engine, and conflating the two makes it easy to misdiagnose which axis is actually the bottleneck in a slow run.

## Zero-copy access, not per-ticker Python objects

Every ticker's data inside a multi-asset `on_bar` call arrives as zero-copy numpy array views, not as Python objects the interpreter constructs fresh per ticker per step. That distinction matters more as ticker count grows: the cost of handing a strategy a hundred tickers' worth of data is the cost of exposing existing memory, not the cost of allocating and later garbage-collecting a hundred small objects every single step.

## The actual constraint on "how many tickers"

The practical ceiling on portfolio size in a Reamer backtest is set by how much per-step work an aligned, cross-sectional view of that many tickers actually requires — not by an arbitrary cap on ticker count. A strategy asking a genuinely large question — a broad cross-sectional universe, not a handful of correlated pairs — is exactly the case this architecture is built to keep affordable.

---

Full reference: [docs](https://reamerlabs.com/docs) · Related: [What a Rolling Window Actually Costs](https://reamerlabs.com/blog/what-a-rolling-window-actually-costs)
