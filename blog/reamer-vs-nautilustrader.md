---
title: Reamer vs NautilusTrader
description: NautilusTrader closes the gap between backtest and live by running both through the same Rust engine — genuinely fast, not a competitor by design. Reamer targets a different, earlier problem, and one specific, narrow finding from our own benchmark suite explains why we don't publish a head-to-head number against it.
date: 2026-07-18
tier: Comparisons
---

NautilusTrader's core idea is a good one: run the exact same event-driven engine in backtest and in live trading, so a strategy that worked in simulation can't quietly behave differently once it's live because of some implementation gap between two separate codebases. That's a real, well-known failure mode in this space, and building one engine that does both is a legitimate way to close it.

It's also an answer to a narrower question than the one Reamer is built around.

## Two different kinds of drift

"Backtest and live behave differently" usually gets called one problem, but it's actually two, and they have different fixes.

**Sandbox-to-live drift** is what NautilusTrader targets directly: the same strategy, run once in simulation and once against a real account, diverging because the two paths went through different code. Sharing one engine for both is a real, working answer to that specific problem.

**Infra-drift in research** is a different, earlier problem — whether the sandbox's own execution assumptions (fill price, slippage, spread, which of a stop and a take-profit resolves first) [actually reflect how a real market behaves](https://reamerlabs.com/blog/why-execution-modeling-matters), independent of whether live matches it exactly. Sharing an engine between backtest and live doesn't make that engine's own assumptions any more realistic — it guarantees both runs are wrong in the same consistent way, if the underlying execution model was never held to a rigorous, published standard to begin with. [Reamer's execution model is checked against exactly that standard](https://reamerlabs.com/spec), by a 282-check conformance suite, independent of what happens to a strategy after it leaves the research stage.

Reamer treats these as genuinely separate problems, solved in that order: get the research infrastructure right first. Deployment is a later, separate decision.

## Where the compiled cores actually differ

NautilusTrader's core is written in Rust, and it's genuinely fast — this isn't a case where a compiled competitor turns out to be secretly slow underneath. When we benchmarked it directly, one specific, narrow finding is worth naming precisely rather than generalizing from: NautilusTrader's `Portfolio` unconditionally instantiates a `PortfolioAnalyzer` that records every trade via a `pandas.concat()` call appending a row to a growing Series on each position close — an operation that isn't gated by the engine's own `run_analysis=False` configuration flag, and that scales quadratically with trade count. At a measured 40,000 bars / 2,638 trades, that single method accounted for 55% of total wall time, and throughput degraded from roughly 512 trades/s to 124 trades/s as trades accumulated — a run with real order flow at our benchmark's scale effectively never finishes without bypassing it directly.

That's a specific implementation detail in an analytics subsystem, not a statement about the Rust execution core itself — which is exactly why [we don't publish a head-to-head throughput number against NautilusTrader](https://reamerlabs.com/benchmark): a fair comparison would need to either patch around something a real user wouldn't know to patch around, or publish a number dominated by a bug unrelated to the actual execution engine, and neither is honest.

## Multi-ticker access and exogenous data

This is where the two engines' shapes diverge most concretely, and it's directly visible in how each is actually called: Reamer's `on_bar` fires once per aligned timestep with every ticker's data available inside that same call, as zero-copy numpy views. NautilusTrader's `on_bar` fires once per bar *per subscribed instrument* — a strategy trading N instruments tracks per-instrument state across N separate calls itself, rather than reading across instruments directly in one place. Reamer also supports attaching arbitrary, schema-free exogenous data — earnings surprises, macro prints, anything JSON-serializable — to any ticker, auto-resolved to the latest-known-as-of-this-bar value inside that same call. Neither is a speed claim; both are a difference in what the strategy author has to build themselves versus what the engine hands them directly.

## Why decoupling, not merging

Reamer's answer to sandbox-to-live drift specifically isn't to unify backtest and live into one engine — it's to treat the problem as a boundary question. Once a strategy's decision logic has actually survived validation, the belief underneath [Reamer's boundary](https://reamerlabs.com/blog/research-not-deployment) — research owns experimentation, other systems own execution — is that very little should need to change to run that logic live: mostly how an order gets dispatched to a specific broker or platform, not the reasoning that decided to place it. Keeping that line firm, logic on one side and integration on the other, never touching, is a more durable fix than merging both environments into a single system — it doesn't just prevent two runs from diverging, it prevents the strategy's own decision logic from ever being reshaped by deployment-specific concerns in the first place.

That's not a claim that unifying backtest and live is the wrong approach — it's a real, working answer to a real problem. It's a claim that it addresses the symptom, two code paths that can diverge, more directly than the cause: research logic and deployment concerns sharing space at all.

## Not a competitor — a downstream option

Same boundary as everywhere else in this loop. A strategy that's actually survived validation in Reamer could reasonably go on to run in NautilusTrader's live engine, exactly as it could go through a broker directly or through QuantConnect — that decision happens after Reamer's job is already done, not instead of it. NautilusTrader solves "how do I run this live without drift from my backtest" well. Reamer solves the earlier problem of whether there's anything worth running live in the first place.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/spec) · Real measurements: [benchmarks](https://reamerlabs.com/benchmark) · Free tier: 10,000 processed bars per machine, permanently, no signup.
