---
title: Why Reamer Exists
description: The bottleneck in systematic trading isn't a shortage of ideas or capital — it's that most backtesting was never strict enough to tell a real edge from noise. Reamer exists to close that gap.
date: 2026-07-16
---

Ask most systematic traders what's holding them back, and the honest answer is rarely "I don't have enough ideas." Ideas are cheap. Charts suggest patterns constantly, conversations plant hypotheses, someone else's writeup makes you wonder if the same thing works in a different market. The actual bottleneck sits one step later: almost none of those ideas get tested under conditions strict enough to trust the answer.

That's the gap Reamer exists to close.

## The failure mode isn't the idea — it's the test

A trading idea's worth isn't determined by how promising it feels, how elegant the logic sounds, or how confidently someone describes it. It's determined by how the idea behaves against realistic costs, real market friction, and the uncomfortable possibility that the apparent edge is noise that happened to look like signal on one particular run. That behavior can only be known by testing it under conditions rigorous enough to actually trust the result.

Most of what passes for backtesting doesn't meet that bar. Not because the idea inside it is necessarily wrong — because the test wrapped around it was never strict enough to tell wrong from right. A loop over a CSV that fills every order at the exact closing price, with no slippage, no spread, no accounting for whether the position could realistically have been sized that way — that setup will tell you a strategy is profitable whether or not it actually is. The number at the end looks like an answer. It isn't one.

## The cost of getting this wrong

The obvious cost is capital: a strategy that looked good in a flattering simulation and doesn't survive contact with real fills. But there's a quieter cost that shows up earlier — the specific anxiety of half-trusting your own backtest. Did that result hold up because the idea is real, or because the test was too forgiving to say no? Without a rigorous test, there's no way to tell the difference, which means every result carries a small, permanent asterisk.

There's a professional version of the same problem. A backtest that can't be explained or defended — to yourself, to a boss, to an investor, to a risk committee — isn't really evidence, no matter how good the return number looks. Being able to show *how* a result was produced, not just what it says, is part of what makes it usable at all.

## What actually closes the gap

Closing it isn't one feature — it's a handful of things that have to be true at once:

- **Execution costs that resemble reality.** Commission, slippage, spread, swap — modeled to a [published specification](https://github.com/reamerlabs/Reamer/blob/main/EXECUTION_SPEC.md), not assumed away, so a result reflects what would have actually happened to an order, not what a spreadsheet wishes had happened.
- **Determinism.** The same inputs producing the same fills, every time, on any machine — so a result is something you can hand to someone else and have them reproduce, not something that only makes sense on the one run you happened to keep.
- **A way to check whether the edge is real or lucky.** One backtest, however careful, is one draw from a distribution of possible outcomes. Testing the strategy against a spread of plausible ones is the difference between "this made money" and "this made money for a reason."
- **The ability to actually look at what happened.** Not just a summary statistic, but the specific fill, at the specific tick, that explains why a trade did what it did.

[What Is Reamer?](https://reamerlabs.com/blog/what-is-reamer) covers how those pieces fit together into one loop. This article is about why that loop needs to exist at all.

## The trigger is earlier than "let's backtest this"

By the time someone sits down to formally test an idea, they've usually already decided it's worth an afternoon. The actual moment Reamer is built for is earlier and vaguer than that — a half-formed pattern-notice that hasn't earned a decision yet, crystallizing into "wait, is that actually real?" The point of a fast, rigorous loop is that this question can get answered before the idea has cost anything beyond the time it took to ask it.

## The belief underneath all of this

This gap — not a shortage of ideas, not a shortage of capital — is the actual bottleneck in systematic trading. Closing it, quickly and honestly, one idea at a time, is the entire reason Reamer exists.

---

Read next: [What Is Reamer?](https://reamerlabs.com/blog/what-is-reamer) · [Research, Not Deployment](https://reamerlabs.com/blog/research-not-deployment) · Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
