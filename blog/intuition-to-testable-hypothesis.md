---
title: How to Turn an Intuition into a Testable Hypothesis
description: "This pattern tends to work" isn't testable yet. A hypothesis needs a specific rule, a specific market and timeframe, and a clear idea of what would prove it wrong — the test being whether it can actually be written as code.
date: 2026-07-16
---

"This pattern tends to work" is not a hypothesis. It's a feeling about a hypothesis that hasn't been written down yet. Turning it into something testable is a specific, mechanical step, and skipping it is how vague confidence gets mistaken for a real idea.

## What makes an intuition untestable

Usually it's missing one of a few specific things: an exact rule for when the signal fires, a specific market and timeframe it's claimed to apply to, or a precise definition of entry and exit. "Breakouts tend to continue" fails on all three — which breakout, on what, over what horizon, entered how and exited when. None of that is nitpicking. Each missing piece is a place where the eventual test could mean something different than what was actually noticed.

## The four things a hypothesis needs

**A specific, computable condition.** Not "price breaks out," but "the close crosses above the highest high of the last 20 bars." Something that can be checked against a bar of OHLCV data and gives a yes-or-no answer, not something that requires judgment to apply.

**A specific market and timeframe.** The pattern noticed on daily equity charts may have nothing to do with the same shape on hourly FX bars. Naming the actual scope up front prevents a result from quietly being generalized further than it was ever tested.

**Precise entry and exit logic.** Exactly what triggers a position, and exactly what closes it — a stop level, a target, a time limit, a reversal signal. If this can't be stated precisely enough to act on every single time the condition fires, it isn't finished yet.

**A clear idea of what would prove it wrong.** A hypothesis that can't fail isn't one. Before running anything, it's worth being able to say what result — a poor win rate, a specific drawdown, [a Monte Carlo distribution that looks too wide](https://reamerlabs.com/blog/why-monte-carlo-matters) — would mean the idea doesn't hold up. Deciding this after seeing the result invites finding a reason to explain away whatever came out.

## The actual test: can it be written as code?

A useful, blunt check for whether an idea has become specific enough: try to write it as `on_bar`. If the condition can be expressed against `tv.close`, `tv.high`, `tv.low` and a handful of parameters, it's a hypothesis. If writing it requires filling in gaps with judgment calls that weren't part of the original intuition, those gaps are exactly what still needs to be decided — and deciding them now, deliberately, is better than letting the implementation decide them by accident.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
