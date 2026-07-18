---
title: "Adding Macro Events to Strategy Research"
description: A CPI print or a rate decision doesn't belong to one ticker the way an earnings surprise does — but the exogenous data mechanism is keyed per-ticker. Deciding which instruments a macro event actually reaches is a research question, not a technical detail to skip past.
date: 2026-07-19
tier: Practical Research Workflows
---

An earnings surprise has a natural owner — it happened to one company, so it attaches to one ticker. A macro release doesn't work that way. A CPI print or a central bank rate decision doesn't belong to any single instrument, and a strategy reacting to one is usually reacting across several tickers at once — a rate-sensitive equity, a currency pair, a bond proxy — not attaching a fact to a single company's timeline. That's [a portfolio-level question](https://reamerlabs.com/blog/multi-asset-portfolio-research) before it's a data-wiring one: which instruments in the run actually have a real, arguable relationship to the event.

## There's no broadcast mechanism, and that's fine

Exogenous data is a plain per-ticker dictionary — one sidecar file path per ticker key, nothing more. There's no dedicated "attach this to every instrument" feature, because none is needed: pointing several tickers' keys at the same sidecar file reaches all of them with the same event data, since the underlying mechanism doesn't care whether two keys happen to resolve to the same file.

```python
result = reamer_py.run_backtest(
    data={"SPY": "spy.bin", "TLT": "tlt.bin", "EURUSD": "eurusd.bin"},
    strategy=RateDecisionStrategy(),
    exogenous={
        "SPY": "fomc_events.exo.bin",
        "TLT": "fomc_events.exo.bin",
        "EURUSD": "fomc_events.exo.bin",
    },
)
```

## The real decision is scope, not syntax

The part worth taking seriously isn't the dictionary — it's deciding which tickers a given macro event should actually reach before wiring it up. Attaching a rate decision to every instrument in a twenty-ticker portfolio because it's mechanically easy to do is a different research claim than attaching it only to the handful with a real, arguable rate sensitivity. The first quietly turns "does this event matter to this instrument" into an assumption baked into the data-wiring step, before the strategy logic ever gets a chance to test it — which is exactly the kind of decision that should be made deliberately and stated, not left as a side effect of which sidecar paths happened to get passed in.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
