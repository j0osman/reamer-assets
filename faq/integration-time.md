---
title: How long does integration take?
description: For Reamer Research, a couple of minutes to a first run: install the Python package or include the C++ header, write a strategy as a plain function and call `backtest`. For Reamer Server, an uncut recording shows Claude Code, working only from the kit documents, integrating it through to a simulated fill in about 19 minutes; the kit puts a person's first simulated trade at a few hours. Going live at a real broker takes longer, because the broker's protocol details, session recovery and certification are your work.
stage: 4
order: 32
product: both
next: paper-trading, results-and-metrics, research-to-server-move, broker-connections
date: 2026-10-02
topics: sdk, fix
---

For Reamer Research, a couple of minutes to a first run: install the Python package or include the C++ header, write a strategy as a plain function and call `backtest`. For Reamer Server, an uncut recording shows Claude Code, working only from the kit documents, integrating it through to a simulated fill in about 19 minutes; the kit puts a person's first simulated trade at a few hours. Going live at a real broker takes longer, because the broker's protocol details, session recovery and certification are your work.

## Reamer Research: a first run

1. **Install.** `pip install` the wheel from the kit, or add `include/reamer_research.hpp` with CMake `find_package`, pkg-config or by hand.
2. **Write a function.** It receives a window of bars and returns an order, or nothing. Brackets attach in the same call.
3. **Run and inspect.** `backtest` prints the summary, writes the JSON report and opens the trade viewer, with every closed trade charted on its own bars.

The whole quickstart, an ATR band reversion with a bracket on every entry, is 14 lines of Python. It is on the [Reamer Research](/products/reamer-research.html) page in Python and C++.

## Reamer Server: the recording

The recording starts from a fresh copy of the kit, with no source code and no earlier exposure. A wall clock runs throughout, and the elapsed time is the measurement. It was recorded against version 4.2.1 and is sped up, returning to normal speed at the end. In about 19 minutes the agent built its own gate and broker connector in C++, modelled on the kit's Go example, including a FIX 4.4 session, and ran orders through to a fill against the kit's simulated venue. It is on the [Reamer Server](/products/reamer-server.html) page.

## What comes after, for Reamer Server

A fill against a simulated venue is not a live trading system. Before trading real money, you still need:

- **Your broker's own version of its protocol,** and your credentials.
- **Session recovery that survives restarts,** including saving sequence numbers and requesting missed messages.
- **Your gate's real rules,** in place of a demonstration policy.
- **Your broker's certification,** and your own compliance sign-off.

How long those take depends on your broker and your firm more than on Reamer Server. See [Which brokers does Reamer Server connect to?](/faq/broker-connections.html)

For Reamer Research, the remaining work is mostly your own data. See [Does it include market data?](/faq/market-data.html)
