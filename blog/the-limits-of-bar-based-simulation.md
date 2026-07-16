---
title: The Limits of Bar-Based Simulation
description: Deterministic synthetic ticks make bar-based execution honest about what it doesn't know — they don't turn it into something it isn't. A bar-based simulation is not a substitute for real tick data, and strategies that depend on one shouldn't reach for the other.
date: 2026-07-17
---

A careful, deterministic model of what might have happened inside a bar is still a model of what might have happened inside a bar. It's worth being precise about what that does and doesn't solve, because the honest version of bar-based execution can start to feel more capable than it actually is.

## What synthetic ticks actually fix

Generating a deterministic sequence of prices within a bar replaces an arbitrary, hidden assumption about intra-bar order with an explicit, reproducible one. That's a real improvement — it turns a silent guess into something inspectable and consistent. It does not reconstruct the actual path price took in the real market. The genuine tick-by-tick history of a bar isn't recoverable from its open, high, low, and close, no matter how carefully the substitute is built.

## What this means for scope

A strategy whose edge depends on real order-book depth, or on sub-second price movements smaller than a single bar can meaningfully represent, isn't well served by any amount of care applied to bar-based simulation — the information that strategy needs simply isn't present in OHLCV data to begin with. This isn't a gap waiting to be closed with a better synthetic-tick model. It's a boundary on what kind of question bar-based execution can honestly answer at all. High-frequency and order-book-dependent strategies sit on the other side of that boundary, not because the modeling isn't good enough yet, but because the input data itself doesn't contain what they need.

## Honest scope beats false precision

It would be easy to describe deterministic intra-bar modeling in a way that implies it closes this gap — technical-sounding language can make a bar-based approximation sound like a genuine microstructure model if nobody looks closely. It doesn't, and describing it as though it does would be exactly the kind of overclaim this whole approach to execution modeling is trying to avoid. Mid-frequency, OHLCV-driven strategies — intraday to multi-day holding periods — are the fit. Strategies that need to see what happened between the ticks that make up a bar are not, and no amount of careful bar-based modeling changes that.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://github.com/reamerlabs/Reamer/blob/main/EXECUTION_SPEC.md) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
