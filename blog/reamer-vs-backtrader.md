---
title: Reamer vs Backtrader
description: Backtrader and Reamer overlap on the surface — both are Python, both are event-driven, both give you an on_bar-style callback. The difference is what happens underneath that callback, and whether the tool was built to make execution realism a discipline or an option.
date: 2026-07-18
tier: Comparisons
---

Almost everyone doing systematic trading research in Python has used Backtrader, or has a script that's clearly descended from one. It's free, it's been around long enough to have a decade of Stack Overflow answers, and its `next()`-per-bar shape is simple enough to learn in an afternoon. For a first backtest, it's a completely reasonable place to start.

The question worth asking isn't whether Backtrader works. It's what happens once a strategy stops being a toy.

## Where they actually overlap

The surface similarity is real, not superficial. Both are Python-first. Both give you a callback invoked once per bar, with access to a rolling window of OHLCV data and a way to submit orders. Someone who's written a handful of Backtrader strategies would recognize Reamer's `on_bar` shape immediately — that's intentional. The problem the two tools are aimed at is the same problem: take a trading idea, run it against historical data, get fills back.

Where they diverge is everything underneath that callback.

## Execution realism as a discipline, not an option

Backtrader lets you configure commission and slippage. It does not make configuring them correctly, or checking that the resulting fills match a specific, written standard, part of using the tool. It's possible to build a Backtrader strategy that fills every order at the exact close price, with a broker model left at its permissive defaults, and get a result that looks like an answer without ever being checked against [what a realistic fill actually requires](https://reamerlabs.com/blog/why-execution-modeling-matters). Nothing in the tool stops that, or even flags it.

Reamer's execution model works from a [published, testable specification](https://reamerlabs.com/spec) — commission, slippage, spread, and swap aren't optional configuration, they're the thing a 282-check conformance suite verifies every fill against. The difference isn't that Reamer supports more realistic execution. It's that realistic execution is the default the tool is built around, not a setting a user has to know to reach for.

## Where the speed actually comes from

Reamer's execution core is written in C++. It isn't "Python that happens to be fast" — the hot path (order matching, fill resolution, synthetic tick generation) runs as compiled machine code, and `on_bar` hands the strategy zero-copy numpy array views of the result rather than Python objects the interpreter has to construct and garbage-collect one at a time. Backtrader's entire engine runs as interpreted Python, one bytecode operation at a time, serialized by the GIL like any other pure-Python program.

This isn't an abstract architecture claim — it's [measured directly](https://reamerlabs.com/benchmark). A no-op workload that isolates pure per-bar overhead (read one price, do nothing) shows a 76–212x gap, because that workload isolates exactly the cost a compiled core avoids and a pure-Python interpreter can't. A realistic Donchian breakout strategy with real order flow narrows that gap to 24–34x, because order matching and bookkeeping cost both engines something regardless of implementation language — the interpreter overhead stops being the whole story once there's real work happening on both sides.

## Multi-ticker access and exogenous data

Reamer's `on_bar` fires once per aligned timestep, with every ticker's data available inside that same call as zero-copy numpy views — a cross-sectional strategy reads across instruments directly, rather than reconstructing state across N separate per-instrument callbacks the way a strict per-bar event loop requires. Reamer also supports attaching arbitrary, schema-free timeseries data — earnings surprises, macro prints, anything JSON-serializable — to any ticker, auto-resolved to the latest-known-as-of-this-bar value inside the same `on_bar` call as the OHLCV data itself. Backtrader has neither primitive built in; both would need to be hand-rolled by the strategy author.

## Determinism and maintenance

Backtrader's development has slowed considerably in recent years — issues sit open, and the project doesn't have the kind of active maintenance that catches subtle execution bugs before they quietly bias a result. For a tool whose entire output is "trust this number," an unmaintained core is a real cost, not a cosmetic one.

[Reamer's execution is deterministic by construction](https://reamerlabs.com/blog/deterministic-research) — the same strategy against the same data with the same seed produces the exact same fills, on any machine, forever. That property, plus active maintenance and a conformance suite that runs against every change, is what makes a result something a desk can hand to someone else and have them reproduce, rather than something that only makes sense on the one run that happened to get kept.

## What Backtrader is still fine for

None of this makes Backtrader a bad tool for what it's actually good at: a first pass at an idea, a teaching environment, a lightweight sanity check before deciding whether something is worth taking further. If the question is "does this idea have any shape at all," Backtrader answers it well and cheaply.

The trigger for reaching past it is [earlier than "we've decided to formally validate this"](https://reamerlabs.com/blog/why-reamer-exists) — it's the moment an idea has survived a first look and now needs to be checked against realistic costs, replayed tick by tick, and stress-tested with Monte Carlo before anyone would trust it with real capital. That's a different job, and it's the one Reamer is built around.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/spec) · Real measurements: [benchmarks](https://reamerlabs.com/benchmark) · License: GUI free, SDK requires an active license — [contact us](https://reamerlabs.com/#contact) for a free, time-limited test license.
