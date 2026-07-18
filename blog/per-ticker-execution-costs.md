---
title: "Per-Ticker Execution Costs in a Multi-Asset Portfolio"
description: A multi-asset backtest sharing one execution config across every ticker is itself an assumption of uniform costs — and real portfolios never actually have that.
date: 2026-07-19
tier: Practical Research Workflows
---

Every ticker in a portfolio backtest shares one execution config by default — same spread, same slippage, same commission, same swap, applied uniformly across every instrument in the run. That default is a real assumption, not a neutral starting point: a broker's overnight financing rate on one FX pair is not its rate on another, and a commission schedule that's flat for one instrument on an account isn't necessarily flat for a different one on the same account. Averaging those differences away doesn't just lose precision — it can make a strategy that leans on one specific instrument's actual cost profile look better, or worse, than it would trading for real.

## Overriding without disturbing the rest

`ticker_overrides` sets a different execution config for specific tickers, leaving every other ticker on the shared config untouched:

```python
eurusd_cfg = reamer_py.DefaultExecutionModelConfig()
eurusd_cfg.spread = 0.0002

result = reamer_py.run_backtest(
    data={"EURUSD": "eurusd.bin", "GBPUSD": "gbpusd.bin"},
    strategy=MyStrategy(),
    exec_config=global_cfg,          # GBPUSD has no override, so it uses this unchanged
    ticker_overrides={"EURUSD": eurusd_cfg},
)
```

Spread and swap are the two most common reasons to reach for this, but slippage, commission, and even which side of the quote the data represents can all differ per instrument the same way — whenever an instrument's real cost structure is different enough from the portfolio default that averaging it in would misrepresent it.

## The fallback is explicit, never silent

A ticker with no entry in `ticker_overrides` always falls back to the shared config unchanged — never a silent zero-cost default. If any override is set at all, a warning fires for any traded ticker that isn't covered by one, and separately for any override key that doesn't match a real ticker in the run — catching a typo'd symbol before it silently does nothing. The random seed governing synthetic tick generation is deliberately excluded from per-ticker overrides too: it always comes from the top-level config, so cost differences between instruments never accidentally introduce cost-correlated randomness into [how ticks are generated](https://reamerlabs.com/blog/why-ohlcv-execution-is-harder-than-it-looks).

The same override dict has to travel with the saved result, not just the run that produced it — a `.reamer` file replayed without it would silently fall back to uniform costs on replay, understating or overstating specific instruments relative to what the original run actually charged.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
