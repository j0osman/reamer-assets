---
title: What does it take to move from Reamer Research to Reamer Server?
description: Your trading logic and your order structure carry over. The relay functions in the Reamer Research library send the same orders your strategy returns in a backtest to a running Reamer Server and reads the fills back. Around it, you build a pre-trade gate and a broker connector once per deployment, feed live bars, and keep indicators as running state. Cost settings stay in research: live costs are what your broker charges.
stage: 4
order: 35
product: both
next: backtest-to-live-without-rewrite, paper-trading, broker-connections, integration-time
date: 2026-10-02
---

Your trading logic and your order structure carry over. The relay functions in the Reamer Research library send the same orders your strategy returns in a backtest to a running Reamer Server and reads the fills back. Around it, you build a pre-trade gate and a broker connector once per deployment, feed live bars, and keep indicators as running state. Cost settings stay in research: live costs are what your broker charges.

Both kits ship the same guide to the move, `RESEARCH_TO_SERVER.md`, with a one-order round trip you can run. This is a plain summary of it.

## What carries over

- **The decision logic:** entry and exit rules, position sizing, stop and target levels, and risk thresholds.
- **The order.** `reamer_relay_send()` takes the same `ReamerOrderRequest` your `on_bar` returns. Order IDs are numbered as in a backtest, so a cancel names the same order in both places, and the submission rules for closing and reducing are the same.
- **The instrument names.** `reamer_relay_open()` takes the same `ticker_names[]` array you pass to `reamer_run_backtest()` and sends each order under its name. Backtest results carry those names too, so you can check them against your live instrument table as a build step.
- **The language.** Reamer Server hosts no strategy. Your strategy runs in its own process, so it can stay in Python, C++ or anything else that reaches the server's socket.

## What changes

- **Where bars come from.** In Reamer Research the engine calls your strategy with a window of past bars. Live, your process reads bars from your feed, so an indicator that needs history, such as a moving average or ATR, is kept as running state. This is the one change to the strategy's own code.
- **Exits.** The live socket carries each order on its own, so a stop and a target are sent as their own orders rather than as a bracket attached to the entry.
- **Costs.** Backtest spread, slippage and commission are a simulation. Live costs are what your broker charges.

## What you build

- **A gate** that accepts or rejects each order against your own rules. Its decisions are published on the server's event stream.
- **A broker connector** for your broker. See [Which brokers does Reamer Server connect to?](/faq/broker-connections.html)

Both are built once and shared by every strategy. The Server kit has worked examples to start from, including a full FIX 4.4 integration in Go, and the round trip in `RESEARCH_TO_SERVER.md` runs one research order through its reference gate to a simulated fill.

## Licences

Each product is licensed separately, per machine. A machine running both holds one seat of each. See [pricing](/pricing.html).

For how to write a strategy so the move is easy, and how to prove the live version matches the backtest, see [How do I take a strategy from backtest to live trading without a rewrite?](/faq/backtest-to-live-without-rewrite.html)
