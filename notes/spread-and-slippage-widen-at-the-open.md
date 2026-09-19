---
title: "Spread and Slippage Widen at the Open"
description: A single spread number and a single slippage number are two constants. Real fills are noisier than that — wider right after a bar opens, calmer toward its close, and scaled by how volatile the bar actually was. Reamer Research models that noise instead of averaging it away.
date: 2026-09-04
tier: Execution Modeling
---

`spread` and `slippage` are two config fields, each a single number. Read literally, that says every fill in a backtest pays exactly the same spread and exactly the same slippage, on a calm bar and a violent one alike. Real markets don't work that way — spread widens when a bar is choppy and tightens when it's quiet, and it's typically wider in the seconds right after a bar opens than by the time it closes. Treating `spread` and `slippage` as fixed constants isn't a simplification that nets out to the same average cost. It understates exactly the trades that matter most: the ones filled during real volatility.

## The noise is not decoration

Reamer Research doesn't fill every order at the literal `spread` and `slippage` values from config. Those are the *base* — the mean the engine samples around, not the number every fill actually pays. When `price_volatility` is set above zero, each synthetic tick's ask spread, bid spread, and slippage are drawn independently around that base, with noise width scaled to that specific bar's own high-low range: a bar that barely moved barely perturbs the base, a bar that swung hard can push the sampled spread or slippage well past it. Spread noise is capped at four times the base value, matching typical real-world widening under stress, so it never runs away or lets the bid cross the ask. Slippage noise is deliberately left uncapped in both directions — real slippage during a liquidity event can spike far past ordinary spread widening, and an occasional favorable draw is realistic price improvement, not a bug to clamp away.

With `price_volatility` at zero, none of this fires — every fill uses the literal `spread` and `slippage` values, deterministically, and the run is fully reproducible regardless of seed. Turning `price_volatility` on doesn't turn the run random; it turns it into a specific, seeded draw from a distribution, reproducible byte-for-byte for a fixed `rng_seed` the same way the rest of [the synthetic tick path](https://reamerlabs.com/blog/why-synthetic-ticks-instead-of-stored-ticks) is.

## Wider at the open, tighter by the close

On top of that per-tick noise, spread carries a second, separate effect: it's structurally wider near a bar's open and narrows toward its close, scaled by how large the bar's own range is relative to the base spread. A bar that barely moved sees almost no widening. A bar with a wide range relative to its spread sees the open meaningfully wider than the close — the same shape real order books show right after a bar's first prints, before depth rebuilds. This widening applies to spread only, not slippage, and stacks with the per-tick noise above rather than replacing it.

## Why this is the harder, more honest default

A strategy that trades mean-reversion into volatility, or that clusters its entries near a bar's open, is exactly the strategy this noise model is hardest on — and exactly the strategy a flat, averaged cost would flatter the most. A single constant spread hides the fact that the worst fills and the best-looking trades often happen in the same volatile stretch of the market, and a strategy whose apparent edge depends on catching that stretch cheaply deserves to be tested against what that stretch actually costs, not against the average day.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/spec)
