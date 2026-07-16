---
title: When to Trust an Edge
description: Monte Carlo, stress testing, and a check for curve fitting each answer a different question. Put together, they're a reasonable way to decide how much trust an edge has earned — not a pass/fail gate, but a more informed basis for a judgment call.
date: 2026-07-17
---

Monte Carlo, stress testing, and a check for curve fitting each answer a different question about an edge. None of them, on their own, says "trust this" or "don't." Put together, though, they give a research process something more useful than a gut feeling to base that judgment on.

## What each check actually contributes

A robustness pass gives a sense of how the result sits relative to the range of outcomes its own trades could plausibly have produced — a low risk of ruin and probability of loss is reassuring, though what counts as "low enough" is a judgment call worth deciding in advance rather than in the moment. A stress test suggests how much room for error exists if costs turn out somewhat worse than assumed. A look for curve-fitting signs — parameter count, whether tuning happened in direct response to watching results improve — is a sanity check on how much to trust the setup in the first place. None of these are a single pass/fail switch; together, they paint a picture worth weighing rather than a verdict to accept or reject mechanically.

## What trusting an edge doesn't mean

Even a strategy that looks favorable across all three of these isn't guaranteed to be real, and treating it as certain would overstate what any of this can actually promise. It means the available checks have been run and considered, rather than skipped or assumed — a meaningfully more informed position than "this made money in a backtest," even if it stops well short of certainty.

## Worth deciding deliberately, not by default

It's easy to end up trusting an edge simply because nothing has definitively ruled it out yet, which is a different thing from actually having weighed what these checks showed. Treating this as a considered judgment — what did each check actually suggest, and does that add up to something worth acting on — tends to hold up better than letting "I've tested this a lot" quietly stand in for an actual decision.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
