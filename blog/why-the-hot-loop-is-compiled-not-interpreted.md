---
title: "Why the Hot Loop Is Compiled, Not Interpreted"
description: Reamer isn't Python that happens to be fast. The hot path — order matching, fill resolution, tick generation — runs as compiled machine code with no GIL, and on_bar hands a strategy zero-copy numpy views instead of objects the interpreter has to build one at a time.
date: 2026-07-19
tier: Engineering
---

A backtesting engine spends almost all of its time in a small, repeated piece of work: given a bar, figure out what orders would fill, at what price, and update the book accordingly. Whether that piece of work runs as interpreted Python or compiled machine code determines the throughput ceiling of everything built on top of it — and it's a decision that has to be made at the architecture level, not tuned in later.

## Where Reamer draws the line

Reamer's execution core — order matching, fill resolution, synthetic tick generation, cost accounting — is written in C++ and compiled ahead of time. It is not a thin C extension wrapping logic that still runs in Python; the hot path never re-enters the Python interpreter per bar, and it isn't serialized by the GIL the way ordinary Python code is. `on_bar` hands the strategy zero-copy numpy array views of the result, rather than constructing Python objects the interpreter has to allocate and garbage-collect one at a time. The strategy itself stays in Python — that boundary is deliberate, not a compromise: research code should be Python, ordinary Python, the part a researcher actually wants to write and iterate on quickly. The part that has to run millions of times per backtest is the part that doesn't stay interpreted.

## What that boundary actually buys

This isn't an abstract architecture claim — it's [measured directly](https://reamerlabs.com/benchmark). A no-op workload that isolates pure per-bar overhead shows a 49–212x gap against Backtrader and QuantConnect LEAN, because that workload isolates exactly the cost a compiled core avoids and an interpreted or CLR-bridged loop can't. A realistic strategy with real order flow narrows that gap to 18–35x against both engines, because order matching and bookkeeping cost every engine something regardless of implementation language once there's real work happening on all sides — the loop overhead stops being the whole story, but it's still the majority of it.

## Why this matters more than a benchmark number

Throughput isn't the point on its own; it's what throughput buys back. A researcher running a fast iteration loop can afford to check an idea from more angles — more parameter ranges, more instruments, more robustness passes — in the same wall-clock time a slow loop spends on one. The compiled/interpreted boundary isn't a performance feature bolted onto a research tool. It's the reason the research loop can run at the pace research actually needs.

---

Full reference: [docs](https://reamerlabs.com/docs) · Real measurements: [benchmarks](https://reamerlabs.com/benchmark)
