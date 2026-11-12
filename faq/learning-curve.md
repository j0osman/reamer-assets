---
title: Is there a steep learning curve?
description: Not for the integration. Reamer Research installs as a Python package or one C++ header, and a strategy is a plain function, so a first backtest runs in a couple of minutes. For Reamer Server, the kit documents are enough for a coding agent: in an uncut recording, Claude Code working only from the kit integrated it in about 19 minutes. What still takes your judgement is the trading itself: your strategy, your data, your gate's rules and your broker.
stage: 4
order: 31.5
product: both
next: integration-time, supported-languages, build-around-a-core, broker-connections
date: 2026-10-08
topics: abi, sdk
---

Not for the integration. Reamer Research installs as a Python package or one C++ header, and a strategy is a plain function, so a first backtest runs in a couple of minutes. For Reamer Server, the kit documents are enough for a coding agent: in an uncut recording, Claude Code working only from the kit integrated it in about 19 minutes. What still takes your judgement is the trading itself: your strategy, your data, your gate's rules and your broker.

## Why it looks steep, and why it isn't

Both products are engines with a C interface that you link into your own program. Traditionally that meant reading a header, a specification and example code, then writing the glue yourself. That is days of work for someone new to it.

Reamer Research now ships that glue. The Python package and the C++ header wrap the C interface, so you install one, write a function that receives a window of bars and returns an order, and call `backtest`. The run prints a summary, writes the JSON report and opens the trade viewer.

For Reamer Server, and for Reamer Research from any other language, the glue is exactly what a coding agent is good at. Each kit ships everything the agent needs to read:

- **The C header,** the full interface, with every call and structure.
- **The specifications:** for Reamer Research, how orders fill and what the result report contains; for Reamer Server, the configuration, the event bus and the strategy socket protocol.
- **Reference code:** a Python binding and a C++ wrapper for Reamer Research; a working server in Go, with C, C++ and Rust starting points, for Reamer Server.

You describe what you want. The agent reads the kit and writes the wiring.

## What the Reamer Server recording shows

The recording starts from a fresh copy of the kit, with no source code and no earlier exposure, and a wall clock in the frame. In about 19 minutes ([watch](/videos/reamer-server-integration-demo.html)), Claude Code wrote a trading server in C++ with its own FIX 4.4 broker connector and a gate with an instrument allowlist and a position limit. Against the kit's simulated venue it sent 6 orders: 5 filled, and the gate rejected 1 with its exact reason. It was recorded against version 4.2.1.

For Reamer Research, the [product page](/products/reamer-research.html) shows the whole quickstart in Python and C++ and the trade viewer it opens. See [How long does integration take?](/faq/integration-time.html)

## What you still decide

An agent removes the learning curve of the interface, not of trading. These stay yours:

- **Your strategy and your data.** The engine tests what you give it.
- **Your gate's rules,** in place of a demonstration policy. See [How can I plug my own risk model into every order before it reaches the broker?](/faq/plug-own-risk-model-pre-trade.html)
- **Your broker,** its own protocol details, session recovery and certification. See [Which brokers does Reamer Server connect to?](/faq/broker-connections.html)

You should still read what the agent writes, as you would any code that trades your money.
