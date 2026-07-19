---
title: "Designing a Deterministic Simulation Engine"
description: A backtest that gives a different answer on a different run, or a different machine, isn't measuring a strategy — it's measuring noise. Determinism was a hard constraint from the first line of the engine, not a property added once something else broke.
date: 2026-07-19
tier: Engineering
---

Most of what a simulation engine has to get right is invisible until it's wrong. Determinism is the one property where "wrong" is easy to define: run the same strategy against the same data with the same seed, and the fills, the costs, and the final result should be bit-identical — every time, on any machine, forever. Not "usually." Not "unless something's under load." Every time.

## Why this had to be a constraint, not a feature

A simulation engine that's merely *usually* reproducible is worse than one that's honestly non-deterministic, because it fails exactly when someone trusts it least — when a result looks surprising enough to double-check, and checking it produces a different number. Reamer treats determinism as an architectural constraint that shapes everything built on top of it, not a checkbox added after the fact: no shared mutable random state carried between calls, no dependency on wall-clock time, thread scheduling, or floating-point summation order, and no code path whose behavior depends on anything other than the strategy, the data, and the seed.

## What this rules out, on purpose

That constraint is stricter than it sounds. It rules out any source of randomness that isn't explicitly seeded and explicitly reproducible — including the parts of the simulation that are supposed to look random, like intra-bar price movement. It rules out parallelism inside a single run wherever that parallelism could reorder floating-point operations and change a result by a rounding error. It rules out anything cached across runs in a way that could let a stale value leak into a fresh one. None of these are exotic failure modes — they're the ordinary ways real systems quietly stop being reproducible without anyone deciding that they should.

## The one deliberate exception

Not every part of the engine is seeded, and that's itself a design decision rather than an inconsistency: Monte Carlo robustness testing is intentionally left non-deterministic, because its job is statistical convergence over many resampled draws, not bit-for-bit reproducibility of one. Determinism is the default because almost everything in the loop needs it — order matching, fill resolution, cost accounting, replay. The one place that doesn't need it is the one place where seeding it would defeat the actual point.

## Determinism is a prerequisite, not the whole story

None of this, by itself, makes a result *correct* — only reproducible. A deterministic engine can still model a fill price wrong, or ignore a real cost. Reproducibility is what makes the rest of the checking possible: a specification worth publishing, a conformance suite worth running, a replay worth trusting. Every one of those depends on the same run producing the same answer twice. Get that wrong, and nothing built on top of it means anything.

---

Full reference: [docs](https://reamerlabs.com/docs) · The specification this enables: [execution specification](https://reamerlabs.com/spec)
