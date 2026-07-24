---
title: Trusting a Backtest
description: Given one specific backtest result, four concrete questions decide whether it deserves trust — is it actually reproducible, was the execution model realistic, has it been validated rather than just run once, and has anyone actually looked at the trades.
date: 2026-07-17
---

The properties that make research trustworthy in general are one thing. Applying them to one specific backtest a desk is deciding whether to trust is another — it means asking a short list of concrete questions about that result, not just agreeing with the principle in the abstract.

## Can it actually be reproduced?

[Run the same strategy against the same data with the same seed again](https://reamerlabs.com/blog/deterministic-research). If the result changes, nothing else about it can be trusted yet — a result that can't be regenerated exactly hasn't earned any of the other questions being asked about it, because there's no way to know whether that number is the strategy's real behavior or an artifact of one particular run.

## Was the execution model actually realistic for this?

[A strategy tested with zero commission, zero slippage, and no spread](https://reamerlabs.com/blog/why-execution-modeling-matters) is a different, easier strategy than the one that would actually be traded. Before trusting a specific number, it's worth checking what assumptions produced it — realistic costs for the instrument and frequency involved, or a forgiving default that happened to be left in place.

## Has this been validated, or just run once?

A single backtest is a single measurement, not a test of whether it would generalize. Before trusting it, it's worth asking specifically whether it's been checked against a spread of plausible variations — [a real Monte Carlo pass](https://reamerlabs.com/blog/why-monte-carlo-matters), not just a single clean run that nobody has stress-tested yet.

## Has anyone actually looked at the trades?

A strong summary number can still be hiding something a closer look would catch — a handful of outsized trades doing most of the work, an entry that only makes sense because of a subtle look-ahead, a pattern that only appears in one narrow slice of the data. [Replaying the result](https://reamerlabs.com/blog/why-replay-matters), at least enough to spot-check a few trades, is what turns "the number looks good" into "we've actually seen what produced it."

## None of these are optional extras

A result that hasn't cleared these four — reproducible, realistically costed, validated, actually inspected — isn't a slightly-less-certain version of a trustworthy backtest. It's an unverified one, regardless of how good the headline number looks.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
