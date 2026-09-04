---
title: Why Replay Matters
description: A finished backtest result is a final-stage view: trades, fills, a summary. When something in that result looks wrong, a few outsized losses, an order that seemingly should have filled and didn't, replay is the deterministic re-run and per-order diagnosis that finds the exact moment and shows why.
date: 2026-07-17
---

A finished backtest result is a final-stage view. Trades happened, orders filled or didn't, a summary got produced, and all of that is presented after the fact, with the actual tick-by-tick mechanics that produced it already collapsed into a result that's meant to be read, not watched. Most of the time that's fine. It stops being fine the moment something in that result doesn't match what the strategy's logic was supposed to do.

## The specific problem replay solves

A strategy runs, and something in the output doesn't look right: a couple of trades with unusually large losses sitting among otherwise ordinary ones, or an order that, according to the strategy's own logic, should clearly have filled and just didn't. Reading the summary statistics again doesn't answer why. Replay is what actually answers it, and it does so from the CLI or from `reamer_py` directly: the run is deterministic for a given `rng_seed`, so the exact same result, down to the same [synthetic tick sequence](https://reamerlabs.com/blog/why-ohlcv-execution-is-harder-than-it-looks) for the bar in question, reproduces byte-for-byte on request. Nothing about diagnosing a specific trade requires a separate visual tool. The order CSV and the exported HTML report carry the per-order detail; a narrowed re-run against the same seed reproduces the exact tick path an order was checked against.

## Surgical diagnosis, not general reassurance

Replay takes one specific, suspicious trade and finds the exact mechanical reason behind it: a stop that triggered on a tick the strategy's own bar-level view wouldn't have shown, a limit order that technically never got touched despite the bar's close looking like it should have, a fill that happened at a worse price than expected because of how slippage or spread applied at that exact moment. None of these are visible from the aggregate result. All of them are visible in the per-order fill detail once the run producing that specific bar is reproduced.

## Why this matters more than it sounds like it should

Without the ability to go find that exact moment, a strange result either gets quietly trusted (because the summary number still looked fine overall) or quietly distrusted (because something felt off, with no way to confirm what). Replay removes the guessing from both of those outcomes. A specific concern about a specific trade gets a specific, checkable answer, from the same CLI and `reamer_py` workflow used to run the backtest in the first place.

---

Full reference: [docs](https://reamerlabs.com/docs)
