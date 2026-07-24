---
title: What Is Reamer?
description: Reamer is a local-first experimentation environment that turns a trading idea into validated strategy logic — and stops the moment that proof is complete.
date: 2026-07-16
---

A trading idea is cheap to have. Almost anyone can look at a chart, notice a pattern, and feel confident about it. What's expensive — in time, in money, in the compounding cost of trusting a result that turns out not to hold — is finding out whether that confidence is deserved.

Reamer exists to close that gap.

## The short answer

Reamer is a local-first trading experimentation environment: a Python SDK and C++ execution core that takes a trading idea, runs it against realistic execution costs, and tells you — with evidence, not a vibe — whether it survives contact with the way real fills actually happen.

You write a strategy in ordinary Python. Reamer runs it bar by bar, fills orders using a formally specified execution model (commission, slippage, spread, swap — the costs that separate a real result from a spreadsheet fantasy), and hands back a result you can [replay tick by tick](https://reamerlabs.com/blog/why-replay-matters), [stress-test with Monte Carlo](https://reamerlabs.com/blog/why-monte-carlo-matters), and either trust or discard.

Both outcomes count as success. An idea that gets confidently killed in an afternoon is not a wasted afternoon — it's one fewer idea you're still carrying around, half-believing, without evidence either way.

## What it actually does

Concretely, working with Reamer looks like this:

- **Write a strategy.** A Python class with one method, `on_bar`, called once per bar. You read OHLCV data, optionally attach arbitrary side information to any timestamp (earnings surprises, macro releases, anything JSON-serializable), and return orders.
- **Run it.** `reamer_py.run_backtest(...)` executes the strategy against your data, using an execution model you configure — commission, slippage, spread, swap, all in absolute units that map onto how a real broker actually charges a desk.
- **Look at what happened, not just the summary number.** Every fill, every rejected order, every partial close is in the result. The free Reamer GUI opens the saved result file and replays it tick by tick, so you can watch the exact moment a trade entered and ask why, instead of reverse-engineering it from a log file.
- **Stress-test it before you believe it.** Monte Carlo bootstrap-resamples the trade sequence to ask a harder question than "did this make money": how much of that result is luck, and what does the distribution of plausible outcomes actually look like.

None of this requires a live account, a broker connection, or any infrastructure beyond your own machine. Local-first isn't a footnote here — it's the reason the whole loop can run in seconds instead of waiting on a server somewhere.

## What Reamer is not

Reamer does not currently place trades or connect to a broker — it's not an OMS, an execution platform, or a live trading platform today. Validated strategy logic is the finished thing Reamer produces. What happens to it after that — wiring it into a broker, an OMS, a platform like QuantConnect or NautilusTrader, or a stack you've already built — is a separate decision, made with separate tools, downstream of anything Reamer does.

Proving the idea is Reamer's job today. Deploying it is a later, separate one.

It's also not built for everything. No L2/L3 order-book data, so it's not a fit for high-frequency or order-book-dependent strategies. No implied volatility or Greeks, so it's not built for options. It targets mid-frequency OHLCV strategies — intraday to multi-day holding periods — across forex/CFD, futures, and equities, and it's honest about where that scope ends.

## Why "experimentation environment" and not "backtesting engine"

Backtesting is one stage of what Reamer does — the part where a strategy meets historical data and produces fills. But a single backtest, on its own, proves very little; a strategy can look excellent on one run and be indistinguishable from noise. The actual job is the full loop: form an idea, implement it, run it, inspect exactly what happened, decide whether the result would survive data it hasn't seen, and carry what you learned into the next idea — whether this one got kept or thrown away.

"Backtesting engine" names the middle step. ["Experimentation environment"](https://reamerlabs.com/blog/the-experimentation-environment) names the whole loop, which is the thing Reamer is actually built around.

## Who this is for

The trigger for reaching for Reamer isn't "we've decided to formally backtest this." It's earlier than that — a half-formed pattern-notice a researcher brings to the desk, from a chart, a conversation, someone else's writeup, crystallizing into "wait, is that actually real?" If a team is already at the point of writing a formal spec for a strategy, they've usually already done a chunk of the work Reamer is meant to shortcut. It's built for the moment right before that, when an idea is still cheap enough for a desk to kill quickly if it deserves it.

---

Full technical reference: [docs](https://reamerlabs.com/docs). The execution model this all rests on is public and testable: [execution specification](https://reamerlabs.com/spec). Real throughput numbers, not marketing claims: [benchmarks](https://reamerlabs.com/benchmark). The workflow this loop maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow).
