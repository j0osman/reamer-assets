---
title: "Why Synthetic Ticks Instead of Stored Ticks"
description: An OHLCV bar gives four prices, not the path between them. Reamer answers the question of what happened inside a bar with a deterministic, seeded synthetic tick path — not with stored tick data most researchers can't afford or don't need.
date: 2026-07-19
tier: Engineering
---

An OHLCV bar is a summary, not a record. Open, high, low, close — four numbers describing what happened over an interval, with no information about the order any of them occurred in. That's fine for most questions a bar-based strategy asks. It stops being fine the moment two things are true in the same bar: a stop-loss and a take-profit are both within range, or a limit order's trigger price sits between the open and one of the extremes. At that point, "which happened first" is a real question, and OHLCV alone cannot answer it.

## The two ways to answer it

There are exactly two honest ways to fill that gap. One is to actually have real tick data and replay the true path. The other is to construct a plausible one and be honest that it's constructed. Real tick data for every instrument, every timeframe, at the volume a research loop needs to run cheaply and often, is either unavailable or expensive enough to change what kind of research is affordable to do at all — and most of the value it would add over a well-built synthetic path, for the questions bar-based research actually asks, is not worth that cost.

## What "deterministic" has to mean here

Reamer generates a synthetic intra-bar tick path instead — and the property that makes it usable for research, not just decoration, is that it's fully deterministic: the same seed produces the exact same tick path, on the same bar, every time, on any machine, forever. That's what turns "which order fired first" from an arbitrary implementation detail into a specified, reproducible, testable answer. A bracket collision isn't resolved by chance at run time; it's resolved by a path that was fixed the moment the seed was, and replaying that exact bar shows exactly what happened and why.

## What stays undisclosed, and why

The specific construction of that path — how it's shaped, how directional bias is derived, exactly how spread and slippage are sampled within it — isn't published here. That's not because it's arbitrary; it's because the value of publishing it is close to zero for a researcher (the guarantee that matters is determinism and specification-tested fill behavior, not the internal shape of the generator) while the cost of publishing it is real: it's one of the harder pieces of this engine to get right, and there's no reason to hand a competitor the finished version of that work for free. What's public, and load-bearing, is the guarantee: same seed, same path, every time — and a [published fill-price specification](https://reamerlabs.com/spec) that the resulting fills are tested against regardless of how the path underneath them was generated.

## The honest limit

None of this makes a bar-based simulation equivalent to one built on real tick data. It makes the bar-based version honest about what it doesn't know, instead of silently pretending an arbitrary tie-break is a real answer. A strategy whose edge genuinely depends on true sub-bar microstructure needs real tick data — that's a different tool for a different question, not something synthetic ticks are trying to replace.

---

Full reference: [docs](https://reamerlabs.com/docs) · Fill-price behavior: [execution specification](https://reamerlabs.com/spec)
