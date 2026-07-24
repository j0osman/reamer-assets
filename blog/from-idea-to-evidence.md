---
title: From Idea to Evidence
description: A walk through what actually happens, stage by stage, the first time a trading idea goes through Reamer — from a half-formed hunch to a result that's either trustworthy or worth confidently discarding.
date: 2026-07-16
---

"Turn ideas into evidence" isn't a slogan sitting on top of Reamer — it's a literal description of what happens, stage by stage, every time an idea goes through it. Here's what those stages actually are.

## The idea

It rarely starts as a formal hypothesis. More often it's [a half-formed pattern-notice a researcher brings to the desk — something in a chart, a conversation, someone else's writeup](https://reamerlabs.com/blog/where-trading-ideas-come-from) — that hasn't earned a decision yet. The question underneath it is simple and a little uncomfortable: *is that actually real, or does it just look real?* Everything that follows exists to answer that question honestly.

## Implementation

[The idea becomes code](https://reamerlabs.com/blog/intuition-to-testable-hypothesis). A Python class with one method, `on_bar`, called once per bar of data. Whatever the idea actually is — a breakout, a mean-reversion signal, a reaction to an earnings surprise — gets expressed as ordinary Python: read the data, decide, return an order. Nothing about this stage requires learning a proprietary strategy language; if it can be written as a function of the data in front of it, it can be implemented.

## Execution

`reamer_py.run_backtest(...)` runs the strategy against historical data, bar by bar, [filling orders against an execution model](https://reamerlabs.com/blog/why-execution-modeling-matters) that charges commission, slippage, spread, and swap the way a real broker would. This is the stage most tools stop at — a number comes out, and it's tempting to treat that number as the answer. It isn't, yet. It's a claim that hasn't been checked.

## Inspection

Before trusting the number, look at what actually produced it. [The free Reamer GUI opens the saved result and replays it tick by tick](https://reamerlabs.com/blog/why-replay-matters) — the exact moment an order filled, at the exact price, against the exact tick that crossed it. This is where a result stops being a black box. If a trade did something surprising, the replay shows why, instead of leaving that question to a guess about the log file.

## Understanding

Every fill, every rejected order, every partial close is part of the result, not just the summary statistics on top of it. Reading the order log and the closed-trade history is what turns "this made money" into "this made money because of X, and lost it back because of Y" — the difference between having a number and understanding where it came from.

## Iteration

Understanding usually points at something to change — a filter that was missing, an entry condition that was too loose, a parameter that was never really tested, just guessed at. [Iteration is refining the idea against its own results](https://reamerlabs.com/blog/validation-vs-iteration) and running it again. It's easy to mistake this stage for the whole process, since a strategy can be tweaked indefinitely and keep looking a little better each time. It isn't the whole process. It's preparation for the next stage, which asks a different question entirely.

## Validation

A single backtest is one draw from a distribution of possible outcomes. [Monte Carlo bootstrap-resamples the trade sequence](https://reamerlabs.com/blog/why-monte-carlo-matters) to ask the harder question: across a spread of plausible variations, does this idea hold up, or did it just get lucky on the one run that got kept? This is the stage that separates a strategy that looks good from one that's actually earned some trust.

## Decision

Kept or discarded. Both are a real outcome, not just one of them. An idea that gets confidently thrown out after failing validation isn't a failed research session — it's exactly what the process is for. The alternative, quietly believing in an idea that was never actually tested, is the worse outcome by far.

## Knowledge

Whatever the decision, something persists past it. The `.reamer` result file is a complete, reproducible, inspectable record — the same execution config, the same seed, the same fills, available to revisit or to hand to someone else exactly as it happened. [Even a discarded idea leaves behind a specific, concrete reason it didn't work](https://reamerlabs.com/blog/from-decision-to-knowledge), which is worth more going into the next idea than a vague memory of "we tried something like that once and it didn't pan out."

## What this adds up to

Nine stages, one pass, and a result at the end that's either trustworthy or honestly rejected — not a guess dressed up as a number. That's what "idea to evidence" means in practice: not a slogan, a specific sequence, repeatable on the next idea and the one after that.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
