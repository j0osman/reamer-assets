---
title: Why OHLCV Execution Is Harder Than It Looks
description: Open, high, low, and close describe four points from a bar — not the path price actually took between them. Deciding which order filled first, or whether a level was touched at all, requires a real answer to a question OHLCV alone can't settle.
date: 2026-07-17
---

Four numbers — open, high, low, close — look like they describe what happened during a bar. They describe four points from it. The actual path price took to visit all four, and in what order, isn't in the data at all, and a surprising amount of what makes execution modeling hard comes down to that single gap.

## The ambiguity is real, not a technicality

A bar with a given open, high, low, and close is consistent with many different real price paths. Price could have run up to the high first and back down to the low before closing, or the reverse, or oscillated between the two several times — the same four numbers describe all of these equally well. Most of the time this doesn't matter. It matters a great deal the moment a bar contains more than one order that could plausibly have filled — a take-profit and a stop-loss both within range, or two separate limit orders on either side of the entry — because which one actually happened first depends on the path, and the path is exactly the thing OHLCV doesn't record.

## Guessing isn't good enough

Picking an answer arbitrarily — always assume the high comes first, or whichever level is closer to the open — produces a consistent-looking result, but a consistent guess is still a guess, and it can be wrong in a way that systematically favors one outcome over another across an entire backtest. The honest response to a genuinely unresolvable ambiguity isn't to hide it behind a confident-looking rule; it's to model the missing information explicitly.

## Making the ambiguity deterministic instead of invisible

Reamer generates a synthetic sequence of ticks within each bar — deterministic, seeded from the bar's own data and a run-level seed, so the same bar always produces the same internal sequence on every run. This doesn't recover the real path that actually happened; nothing can, from OHLCV alone. What it does is replace an arbitrary, hidden assumption with an explicit, reproducible one — order events get resolved against a specific, consistent sequence of prices instead of a silent rule that happens to be baked into whichever engine is running the test.

## Why this is worth understanding, not just trusting

None of this makes bar-based execution equivalent to having the real tick data — it can't be, since the information genuinely isn't there. What it means is that the uncertainty is handled openly, with a method that can be inspected and reasoned about, rather than resolved by an assumption nobody had to state out loud. That difference — a visible, deterministic model of the gap versus an invisible guess — is most of what separates careful OHLCV execution from careless OHLCV execution.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://github.com/reamerlabs/Reamer/blob/main/EXECUTION_SPEC.md) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
