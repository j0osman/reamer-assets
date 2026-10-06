---
title: How long does integration take?
description: In uncut recordings, Claude Code working only from the kit documents integrated Reamer Research in about 13 minutes and Reamer Server in about 19, through to a simulated fill for Reamer Server. The Reamer Server kit puts a person's first simulated trade at a few hours. Going live at a real broker takes longer, because the broker's protocol details, session recovery and certification are your work.
stage: 4
order: 32
product: both
next: paper-trading, results-and-metrics, research-to-server-move, broker-connections
date: 2026-10-02
---

In uncut recordings, Claude Code working only from the kit documents integrated Reamer Research in about 13 minutes and Reamer Server in about 19, through to a simulated fill for Reamer Server. The Reamer Server kit puts a person's first simulated trade at a few hours. Going live at a real broker takes longer, because the broker's protocol details, session recovery and certification are your work.

## What the recordings show

Each recording starts from a fresh copy of the kit, with no source code and no earlier exposure. A wall clock runs throughout, and the elapsed time is the measurement. They were recorded against version 4.2.1 and are sped up, returning to normal speed at the end.

- **Reamer Research, about 13 minutes.** The agent wrote a multi-instrument strategy from scratch and ran a backtest. It then confirmed that a second run gave identical results, swept the parameters and cost settings, traced the worst trade, and wrote the JSON report.
- **Reamer Server, about 19 minutes.** The agent built its own gate and broker connector in C++, modelled on the kit's Go example, including a FIX 4.4 session. It ran orders through to a fill against the kit's simulated venue.

The recordings are on the [Reamer Research](/products/reamer-research.html) and [Reamer Server](/products/reamer-server.html) product pages.

## What comes after, for Reamer Server

A fill against a simulated venue is not a live trading system. Before trading real money, you still need:

- **Your broker's own version of its protocol,** and your credentials.
- **Session recovery that survives restarts,** including saving sequence numbers and requesting missed messages.
- **Your gate's real rules,** in place of a demonstration policy.
- **Your broker's certification,** and your own compliance sign-off.

How long those take depends on your broker and your firm more than on Reamer Server. See [Which brokers does Reamer Server connect to?](/faq/broker-connections.html)

For Reamer Research, the remaining work is mostly your own data. See [Does it include market data?](/faq/market-data.html)
