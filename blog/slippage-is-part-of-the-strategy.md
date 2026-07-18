---
title: Slippage Is Part of the Strategy
description: Slippage isn't a fudge factor subtracted from an otherwise-real result. A strategy that only works when slippage is assumed away isn't a slightly-better version of the real strategy — it's evidence the real one doesn't hold up.
date: 2026-07-17
---

Slippage tends to get treated as an annoying tax on an otherwise-correct number — something to subtract at the end to be conservative. That framing gets the relationship backwards. The price a strategy decides to trade at and the price it actually fills at are rarely identical, and how much a strategy's result depends on that gap is itself a real property of the strategy, not a correction applied on top of it.

## Where the gap comes from

Between the moment a signal fires and the moment an order actually fills, the market has usually moved, even if only slightly — a few seconds pass, other participants react, liquidity at the exact desired price may not be there in the size needed. That gap is slippage, and it isn't a modeling inconvenience. It's what actually happens to a real order, every time, to some degree.

## Why zero slippage isn't a clean baseline

It's tempting to test a strategy with slippage set to zero "to see the pure signal first," treating that as a neutral starting point to build up from. It isn't neutral — it's a specific, unrealistic assumption, and a strategy's apparent quality can shift substantially once that assumption is removed. A signal that only produces a good result under zero slippage isn't a cleaner version of a good strategy. It's a strategy whose entire apparent edge might live inside a gap that doesn't exist in real trading.

## Slippage tolerance is information about the strategy

How much a result degrades as slippage increases is itself worth knowing, independent of whatever the headline return looks like at any one setting. A strategy that stays profitable across a reasonable range of slippage assumptions is telling you something different than one that only works at exactly zero and collapses the moment any friction is introduced. Reamer's execution model supports both a fixed, [deterministic](https://reamerlabs.com/blog/deterministic-research) slippage value and optional stochastic noise scaled to the bar's own volatility — the second isn't there to make results worse for its own sake, it's there so a strategy's sensitivity to realistic variation in fill quality is something that can actually be observed, rather than assumed away by testing at a single, forgiving number.

## Part of the strategy, not a discount applied to it

None of this is about being pessimistic for its own sake. It's that the fill a strategy actually gets, slippage included, is the fill the strategy actually gets — not a worse version of some purer, frictionless result. A strategy's real economics already include it.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/execution-spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
