---
title: How Limit Orders Really Behave
description: A limit order isn't a market order with a nicer price attached. It waits for a specific level to be crossed, might never fill at all, and modeling it any other way changes what a strategy's results actually mean.
date: 2026-07-17
---

A limit order is sometimes modeled as though it were just a market order with a preferred price attached — assumed to fill the moment it's placed, at roughly the price requested. That isn't what a limit order actually does, and the difference changes what a backtest using them is really measuring.

## What a limit order actually promises

A limit order sits and waits. It fills only when the market actually reaches the specified price — at that price or better, never worse — and if the market never gets there, it never fills at all. That's a fundamentally different object than a market order, which trades immediately at whatever the current price allows. A strategy built around limit entries is implicitly betting that price will reach a specific level; treating that order as though it fills instantly, regardless of whether the level was ever actually touched, tests a different strategy than the one that was actually designed.

## The fill is exact, but arrival isn't guaranteed

When a limit order does fill, it fills at the level specified — [not a worse price, the way a stop order can](https://reamerlabs.com/blog/stop-orders-gaps-and-reality). The uncertainty isn't in the price, it's in whether the order gets touched at all within the time it's live. A backtest needs to genuinely check, tick by tick, whether the specified level was crossed — not assume it, and not silently skip that check because the strategy's premise made it feel likely.

## Time-in-force changes what "never fills" means

A limit order doesn't wait forever by default. Depending on how it's submitted, an unfilled limit order might persist until explicitly cancelled, expire at a specific timestamp, or, if marked immediate-or-cancel, simply disappear if it doesn't fill within the same bar it was submitted on. A strategy's real behavior depends on which of these applies — an order that quietly expires unfilled produces a very different trade history than one that stays live indefinitely waiting for a level that might not arrive for weeks.

## Why this matters for what a result means

A strategy that only enters on pullbacks to a specific level is making a claim about where price goes, not just when a signal fires. Modeling that claim honestly — checking for the actual crossing, respecting the actual time-in-force, allowing for the real possibility of no fill at all — is what keeps a backtest of a limit-based strategy actually testing the strategy that was designed, rather than a market-order strategy wearing a limit order's name.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://github.com/reamerlabs/Reamer/blob/main/EXECUTION_SPEC.md) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
