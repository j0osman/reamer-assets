---
title: Why Execution Modeling Matters
description: A backtest can be deterministic and well-validated and still be measuring the wrong thing, if the fill price underneath it doesn't reflect how an order actually gets filled. Execution modeling is the layer everything else depends on.
date: 2026-07-17
---

Determinism and validation only matter if the thing being validated used realistic fills to begin with. A perfectly reproducible result and a rigorously validated one can both still be wrong from the very first trade, if the execution model underneath assumed something that doesn't happen in real markets. Execution modeling is the layer everything else sits on top of.

## The shortcut that breaks everything

The most common shortcut in a naive backtest is filling every order at the exact recorded price — the close, or whatever value is sitting in the data — with nothing subtracted for the cost of actually trading. That assumption describes a fill that doesn't exist: no spread crossed to get in or out, no slippage from the price moving in the moment between deciding and filling, no commission, nothing. A strategy tested this way isn't a slightly-optimistic version of the real strategy. It's a measurement of a different, imaginary instrument that happens to share the same entry and exit signals.

## What a believable fill actually requires

[A buy pays the ask, not the midpoint; a sell receives the bid](https://reamerlabs.com/blog/modeling-bid-ask-correctly) — spread is a real cost paid on every round trip, not a rounding error. [Some amount of slippage belongs in the model too](https://reamerlabs.com/blog/slippage-is-part-of-the-strategy), because the price a strategy decides to trade at and the price it actually fills at are rarely identical, even a few seconds apart. Commission is a direct, unavoidable cost of the trade itself. And for anything held overnight, swap is the ongoing cost of holding the position, charged whether or not the trade is thought about again until it closes. None of these are conservative adjustments layered on top of a "real" number for safety. They're what the real number actually is — leaving any of them out doesn't produce a cleaner result, it produces a different, less accurate one.

## Why this matters more than it looks like it should

A strategy that trades frequently, or trades in a market with a wide spread relative to its typical move, can have its entire apparent edge come from the size of the shortcut rather than from the underlying logic. Costs that look small on any single trade compound across hundreds or thousands of them, and a backtest that quietly excludes them isn't reporting a modest overestimate — it can be reporting a strategy that only exists because the test let it skip the exact costs that would have broken it.

## The layer everything else depends on

Nothing downstream of a bad fill model can fix it. [A deterministic, perfectly reproducible run](https://reamerlabs.com/blog/deterministic-research) of an unrealistic execution model just reproduces the same wrong answer every time. [A Monte Carlo robustness pass](https://reamerlabs.com/blog/why-monte-carlo-matters) run on top of it is stress-testing a result that was never measuring the real strategy in the first place. Getting the fill price right isn't one item on a checklist alongside validation and robustness — it's the foundation those checks are supposed to be standing on.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/execution-spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
