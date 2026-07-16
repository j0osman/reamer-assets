---
title: Why Monte Carlo Matters
description: A backtest happened with one specific set of trades. Monte Carlo asks a different question — if the strategy's real edge produced a slightly different draw of similar trades, how much would the result actually change?
date: 2026-07-17
---

A backtest's trade sequence is one specific outcome — these particular wins, these particular losses, in this particular count and mix. Monte Carlo exists to ask a question a single historical run can't answer on its own: if the strategy's real underlying edge had produced a slightly different draw of similar trades — a few more losses, a few fewer of the best winners, some other equally plausible mix — how much would the result actually change?

## What bootstrap resampling actually does

Reamer's Monte Carlo draws, with replacement, from the strategy's own set of observed trade returns — meaning a single resample can include some trades more than once and leave others out entirely, thousands of times over, each time producing a new synthetic equity path from that resampled mix. This isn't reordering the same fixed set of trades; it's treating the observed trades as a sample from the strategy's real behavior and asking what a range of equally plausible samples from that same behavior would look like.

## Why this matters beyond the one path that happened

A strategy's real edge, if it has one, doesn't guarantee any specific sequence of wins and losses — it guarantees a *tendency*, and the actual historical trades are one draw from whatever that tendency produces. A backtest that happened to draw an unusually favorable mix — slightly more of the big winners, slightly fewer of the ordinary losses — looks better than the underlying edge actually is, purely from the luck of which trades happened to occur. Resampling thousands of alternative, equally plausible draws from the same trade set is what reveals whether the one historical result sits near the middle of that distribution or out at its most fortunate edge.

## The question this answers that a single backtest can't

A backtest shows one outcome. Monte Carlo shows the spread of outcomes a similarly-behaved strategy could plausibly produce — risk of ruin, drawdown percentiles, a realistic range for terminal equity — built from resampling the very same trades that already happened. A strategy whose real risk only shows up when the mix of trades is slightly less fortunate than the one actually observed is exactly the kind of risk a single historical run can hide, without anything in that one result being fabricated or wrong.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
