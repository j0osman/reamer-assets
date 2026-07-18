---
title: "Building a Research Workflow in Python"
description: The parts of a research setup that should stay fixed across every idea — data loading, cost assumptions, evaluation — and the one part that should actually change between ideas. Most one-off research scripts blur that line without meaning to.
date: 2026-07-19
tier: Practical Research Workflows
---

The fastest way to test a new trading idea is to copy last week's script, change the logic in the middle, and run it. It's also how cost assumptions quietly drift between ideas without anyone deciding they should — a slippage value tweaked to make one test converge faster, a commission setting left out because that script didn't need it, small differences accumulating until two ideas tested six weeks apart aren't actually being held to the same standard anymore, just because they came from different copy-pasted files.

## What should never move between ideas

A research setup has a small number of pieces that have nothing to do with any specific idea: how data gets loaded, what execution costs apply, how a result gets judged. None of that should be re-decided per script.

```python
def load_universe(tickers):
    return {t: f"data/{t}.bin" for t in tickers}

def default_exec_config():
    cfg = reamer_py.DefaultExecutionModelConfig()
    cfg.slippage, cfg.spread, cfg.commission_per_unit = 0.05, 0.10, 0.01
    return cfg

def run(strategy, tickers):
    return reamer_py.run_backtest(
        data=load_universe(tickers), strategy=strategy,
        exec_config=default_exec_config(), initial_capital=100_000,
    )
```

## What should be the only thing that changes

The strategy class — the actual idea — is the one piece meant to be different every time. Once loading, costs, and the run call are pulled into shared functions like the ones above, testing a new idea means writing a new strategy and calling `run(NewStrategy(), tickers)`, not re-deriving the whole setup from scratch and hoping nothing was quietly changed along the way.

## Why this is the concrete version of [a principle already established](https://reamerlabs.com/blog/building-a-repeatable-research-process)

Deciding cost assumptions and evaluation standards once, rather than relitigating them for every idea, is a principle worth holding regardless of what tool implements it. What changes at this level is that it becomes a specific, checkable fact about a specific codebase — either every strategy in a research project actually calls the same `default_exec_config()`, or it doesn't, and that's something a diff can catch, not something that has to be taken on faith about how carefully each script was written.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
