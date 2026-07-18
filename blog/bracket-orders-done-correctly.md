---
title: Bracket Orders Done Correctly
description: A take-profit exit is easy to assume fills at exactly the target price, like a limit order. It doesn't — it uses the same slippage-inclusive formula as a stop-loss exit, and when both are close together, which one actually triggers first is a real question with a real answer.
date: 2026-07-17
---

A bracket — a take-profit and a stop-loss attached to the same position — looks like two simple, symmetric exits: hit the good price, or hit the bad one. The assumption that quietly sneaks in is that the take-profit side behaves like a limit order, filling at exactly the target. It doesn't, and treating it that way overstates exactly the side of the trade that's supposed to represent the win.

## Both exits use the same formula

A take-profit exit fills using the same slippage-inclusive formula as a stop-loss exit — [not an exact-price fill the way a genuine limit order works](https://reamerlabs.com/blog/how-limit-orders-really-behave). That means even a "successful" take-profit, one that gets hit as intended, still incurs realistic slippage on the way out, the same as a loss would. Modeling the take-profit side as a clean limit fill while modeling the stop-loss side realistically overstates the win side of every bracketed trade, which quietly inflates a strategy's apparent edge in a very specific, easy-to-miss way.

## What happens when both levels are close together

A bracket with a tight stop and a tight target creates a real question whenever both could plausibly be hit within the same bar: which one actually got touched first? OHLCV data alone doesn't say. The answer has to come from [an actual, ordered sequence of prices within the bar](https://reamerlabs.com/blog/why-ohlcv-execution-is-harder-than-it-looks) — a deterministic tick sequence, generated the same way every time for the same seed, so that whichever level is genuinely touched first in that sequence is the one that resolves the trade. Guessing which side "should" win based on which one is closer to the entry, or picking one arbitrarily, produces a different, less honest answer than actually checking the order events occurred in.

## Why this level of care matters here specifically

Brackets are exactly the mechanism most strategies lean on to define their own risk — a stop-loss is often the only thing standing between a strategy and an unbounded loss. Modeling the exit level accurately, and resolving the collision case honestly instead of by assumption, is not a minor implementation detail on top of the strategy. For a bracket-based strategy, it's close to the entire execution model that matters.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
