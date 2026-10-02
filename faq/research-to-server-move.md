---
title: What does it take to move from Reamer Research to Reamer Server?
description: Your trading logic carries over; the code around it does not. You build a pre-trade gate, a broker connector and a strategy process once per deployment, map instrument indexes to names, and keep indicators as running state. No configuration, data file or cost setting carries across, and nothing converts the strategy for you.
stage: 4
order: 35
product: both
next: backtest-to-live-without-rewrite, paper-trading, broker-connections, integration-time
date: 2026-10-02
---

Your trading logic carries over; the code around it does not. You build a pre-trade gate, a broker connector and a strategy process once per deployment, map instrument indexes to names, and keep indicators as running state. No configuration, data file or cost setting carries across, and nothing converts the strategy for you.

Both kits ship the same guide to the move, `RESEARCH_TO_SERVER.md`. This is a plain summary of it.

## What carries over

- **The decision logic:** entry and exit rules, position sizing, stop and target levels, and risk thresholds.
- **The language.** Reamer Server hosts no strategy. Your strategy runs in its own process and connects over a socket, so it can stay in Python, C++ or anything else.

## What does not

- **The per-bar callback.** In Reamer Research the engine calls your strategy with a window of past bars. Live, nothing hands you a window, so any indicator that needs history, such as a moving average or ATR, must be kept as running state. This is the one change to the strategy's own code.
- **Instrument identity.** Reamer Research identifies instruments by their position in the run; Reamer Server by name. You build the mapping. Backtest results carry your own instrument names when you supply them, as the Python binding does, so you can check the two match as a build step.
- **Configuration, data format and cost settings.** Backtest costs are a simulation; live costs are what your broker charges.

## What you build

- **A gate** that accepts or rejects each order against your own rules.
- **A broker connector** for your broker. See [Which brokers does Reamer Server connect to?](/faq/broker-connections.html)
- **A strategy process** that sends orders to the server.

The gate and connector are built once and shared by every strategy. The Server kit has worked examples to start from, including a full FIX 4.4 integration in Go.

## Licences

Each product is licensed separately, per machine. A machine running both holds one seat of each. See [pricing](/pricing.html).

For how to write a strategy so the move is easy, and how to prove the port matches the backtest, see [How do I take a strategy from backtest to live trading without a rewrite?](/faq/backtest-to-live-without-rewrite.html)
