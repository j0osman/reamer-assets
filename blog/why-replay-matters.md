---
title: Why Replay Matters
description: A finished backtest result is a final-stage view — trades, fills, a summary. When something in that result looks wrong, a few outsized losses, an order that seemingly should have filled and didn't, replay is how you go find the exact moment and diagnose why.
date: 2026-07-17
---

A finished backtest result is a final-stage view. Trades happened, orders filled or didn't, a summary got produced — and all of that is presented after the fact, with the actual tick-by-tick mechanics that produced it already collapsed into a result you're meant to read, not watch. Most of the time that's fine. It stops being fine the moment something in that result doesn't match what the strategy's logic was supposed to do.

## The specific problem replay solves

A strategy runs, and something in the output doesn't look right — a couple of trades with unusually large losses sitting among otherwise ordinary ones, or an order that, according to the strategy's own logic, should clearly have filled and just didn't. Reading the summary statistics again doesn't answer why. Reading the order log might show that something happened, but not the tick-level reason it happened that way. Replay is what actually answers it: stepping through the exact bar, [the exact synthetic tick sequence](https://reamerlabs.com/blog/why-ohlcv-execution-is-harder-than-it-looks), and watching precisely what price the order was checked against and why it did or didn't fill.

## Surgical diagnosis, not general reassurance

This isn't about confirming a result looks reasonable in general. It's about taking one specific, suspicious trade and finding the exact mechanical reason behind it — a stop that triggered on a tick the strategy's own bar-level view wouldn't have shown, a limit order that technically never got touched despite the bar's close looking like it should have, a fill that happened at a worse price than expected because of how slippage or spread applied at that exact moment. None of these are visible from the aggregate result. All of them are visible the instant the specific bar in question gets replayed.

## Why this matters more than it sounds like it should

Without the ability to go find that exact moment, a strange result either gets quietly trusted (because the summary number still looked fine overall) or quietly distrusted (because something felt off, with no way to confirm what). Replay removes the guessing from both of those outcomes — a specific concern about a specific trade gets a specific, checkable answer, instead of a shrug in either direction.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
