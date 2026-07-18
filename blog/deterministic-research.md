---
title: Deterministic Research
description: "Run it again, get the same answer" sounds like a low bar. Most ad-hoc research setups don't actually clear it — and without it, none of the other checks that make research trustworthy are even possible to run.
date: 2026-07-17
---

"Run it again and get the same answer" sounds like the most basic requirement research could have. A surprising amount of ad-hoc research doesn't actually clear it — an unseeded random number somewhere, a live data pull that returns slightly different values on a second call, a notebook cell run out of order. Determinism isn't a nice-to-have layered on top of good research. It's the property that makes the other checks possible at all.

## What determinism actually requires

Full determinism means every source of randomness in the backtest itself traces back to a single, explicit seed — including things that don't look obviously random, like [the synthetic tick sequence used to resolve intra-bar ambiguity](https://reamerlabs.com/blog/why-ohlcv-execution-is-harder-than-it-looks). One seed controls the entire run, engine-wide, so that the same strategy against the same data produces the exact same fills, the exact same trade sequence, the exact same result — on any machine, at any time.

[This deliberately doesn't extend to Monte Carlo](https://reamerlabs.com/blog/why-monte-carlo-matters). Monte Carlo is a randomized stress test by design — its whole job is to resample many different plausible variations of a result, and reseeding it to reproduce one fixed resampling every time would defeat that purpose, not strengthen it. Reamer's Monte Carlo draws from real hardware entropy rather than a fixed seed for exactly this reason: repeatability isn't the goal there, and forcing it in would just mean re-examining one arbitrary sample of the distribution forever instead of exploring it. Determinism belongs to the backtest, which is a single, specific, reproducible measurement. It doesn't belong to Monte Carlo, which is intentionally the opposite of that on purpose.

## Why this is harder to get right than it sounds

It's tempting to assume a script is deterministic just because nothing in it looks random. Hidden non-determinism creeps in from unexpected places — a dictionary whose iteration order isn't guaranteed, a timestamp read from the system clock, a random seed set in one place but not propagated to a library call somewhere else. A system has to be built deliberately around a single source of randomness to actually guarantee reproducible output, not just assumed to be deterministic because no explicit `random()` call is visible in the strategy code.

## Why everything else depends on this

A conformance test that checks whether a fill matches a specification only means something if running the same scenario twice produces the same fill both times — otherwise the test is checking one arbitrary outcome among many possible ones, not verifying actual behavior. Comparing two runs to see whether a code change altered execution behavior only works if the only thing that changed between them was the code. Even handing a result to someone else to verify only works if they can regenerate the exact same output from the same inputs. Determinism isn't one item on a list of trust properties — it's the property that has to be true first, before any of the others can be checked at all.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
