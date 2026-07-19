---
title: Reamer vs vectorbt PRO
description: vectorbt PRO is genuinely fast at what it's built for — sweeping thousands of parameter combinations across a strategy shape you've already fixed. That's a different job than carrying an unformed idea through the whole research loop, which is what Reamer is built around.
date: 2026-07-18
tier: Comparisons
---

vectorbt PRO is not a slow tool pretending to be fast. Built on NumPy, pandas, and Numba, it vectorizes backtests in a way that genuinely outruns anything looping bar-by-bar in Python — sweeping thousands of parameter combinations in the time a naive loop would take to run one. If the question is "which of these 500 moving-average lengths works best on this dataset," vectorbt PRO answers it about as fast as that question can be answered.

The question worth asking is what happens before that sweep, and after it.

## What a parameter sweep assumes

Sweeping 500 parameter combinations assumes the strategy's shape is already fixed — you know it's a moving-average crossover, you know roughly what you're optimizing, and the open question is which specific numbers to plug in. That's a real and common need. It's also a late-stage one. [Most of a trading idea's actual risk lives earlier](https://reamerlabs.com/blog/from-idea-to-evidence) — in whether the strategy shape itself survives realistic costs at all, whether the fills underneath it reflect how orders actually get filled, whether the resulting edge is real or [an artifact of tuning against the same data being evaluated on](https://reamerlabs.com/blog/curve-fitting-vs-evidence).

A fast sweep over a shape that was never validated just finds the best-fitting version of an idea that might not have been real to begin with — speed doesn't fix that, it just gets you to the wrong answer faster.

## A loop, not a single fast stage

Reamer isn't optimized to be the fastest tool at any one stage. It's built around [the full loop](https://reamerlabs.com/blog/the-research-loop) — idea, implementation, execution, inspection, validation, iteration, decision, knowledge — with every stage held to the same standard: [deterministic, seeded execution](https://reamerlabs.com/blog/deterministic-research), fills modeled to [a published specification](https://reamerlabs.com/spec), [tick-level replay](https://reamerlabs.com/blog/why-replay-matters) to see exactly what happened, and [Monte Carlo robustness testing](https://reamerlabs.com/blog/why-monte-carlo-matters) before anything is trusted. Parameter optimization is one stage in that loop, not the reason the loop exists.

Reamer's C++ execution core is [11.4×–16.2× faster than Backtrader on identical realistic-strategy workloads](https://reamerlabs.com/benchmark). Raw throughput on a fixed-shape sweep was never the bottleneck this product targets — the bottleneck is earlier: whether an idea deserves to reach the sweep stage at all.

## Cross-sectional access and side-channel data

Reamer's `on_bar` fires once per aligned timestep with every ticker's data available inside that same call, as zero-copy numpy views — a genuinely different shape than vectorbt PRO's whole-history array operations. Reamer also supports attaching arbitrary, schema-free exogenous data — earnings surprises, macro prints, anything JSON-serializable — to any ticker, auto-resolved to the latest-known-as-of-this-bar value inside `on_bar`. Both exist to express and inspect a strategy's logic, not to sweep a fixed shape across a parameter grid.

## Where vectorbt PRO is genuinely the better tool

If a strategy's shape is already validated and the actual task is exploring a large parameter space fast, vectorbt PRO is the right tool for that stage. Some researchers use both: validate a shape's realism and robustness in one tool, then hand a confirmed shape to a vectorized sweep for fine-tuning. Reamer doesn't try to be the fastest possible parameter-sweep engine; it tries to make sure the thing being swept was worth sweeping in the first place.

---

Full reference: [docs](https://reamerlabs.com/docs) · Benchmarks: [reamerlabs.com/benchmark](https://reamerlabs.com/benchmark) · Free tier: 10,000 processed bars per machine, permanently, no signup.
