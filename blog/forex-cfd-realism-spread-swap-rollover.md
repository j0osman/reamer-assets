---
title: "Forex and CFD Realism: Spread, Swap, and Rollover"
description: Forex and CFD cost structure maps onto Reamer's execution model as-is, not as an approximation — but one silent unit mistake is enough to mis-price every fill by five orders of magnitude without ever throwing an error.
date: 2026-07-19
tier: Practical Research Workflows
---

Most backtesting cost models are built around equities first, then stretched to cover everything else — commission per share, a flat slippage assumption, done. A forex or CFD strategy tested against that shape quietly inherits an equities-shaped cost model, and the two markets don't share a cost structure. Spread, not commission, is the dominant cost on most forex trades. A position held overnight accrues a real, direction-dependent financing cost that a same-day equity strategy never has to think about. Neither of those is a precision issue — get either one wrong and the backtest is measuring a different, easier market than the one the strategy would actually trade in.

## The mistake that doesn't throw an error

`qty` in Reamer is always individual base-asset units, not lots — for forex, that means 1 unit of base currency, not the market's default of a 100,000-unit standard lot. Forex quotes and broker fee schedules are conventionally expressed in lots, so a strategy sized the way a trader would naturally think about it — "one standard lot" — has to be translated before it goes in as `qty=100000`, and `commission_per_unit` or a swap rate quoted per lot has to be divided by 100,000 to land in the same units. Skip that conversion and nothing crashes. The backtest just runs at costs five orders of magnitude off from what was intended, and a strategy that "works" under that mistake isn't a real edge — it's an artifact of a decimal point nobody moved.

## Swap is a real cost, not a rounding error

Every position held across a calendar date boundary accrues swap — longs pay, shorts receive, and the rate is set per day of the week, because many brokers roll a multiple of the normal rate on one specific day to account for weekend settlement. A strategy that holds positions for days rather than hours can have its entire apparent edge come from a backtest that zeroed this out, the same way [ignoring slippage anywhere else in the model quietly inflates results](https://reamerlabs.com/blog/slippage-is-part-of-the-strategy) — it isn't a smaller version of the real strategy, it's evidence the real one doesn't hold up once financing costs are actually charged.

```python
cfg = reamer_py.DefaultExecutionModelConfig()
cfg.set_swap(3, 4.5)   # Wednesday — many brokers roll triple-swap here
```

## Why forex is the closest fit, not just a supported one

Forex/CFD cost structure maps onto Reamer's execution model directly — spread, slippage, commission, and swap correspond to real broker fee-schedule fields with no approximation layer in between. Equities and futures are both well-supported too, but each carries a caveat forex doesn't: equities need pre-adjusted input to avoid a fake price break at every split or dividend, futures need an explicit roll-cycle configuration to avoid a fake price jump at every contract expiry. Forex has neither problem — the model is already shaped like a forex broker's own cost sheet, once the lot-to-unit conversion is made correctly and once for good.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
