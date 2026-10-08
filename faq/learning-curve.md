---
title: Is there a steep learning curve?
description: Not for the integration. The kit documents are enough for a coding agent to do it. In uncut recordings, Claude Code working only from the kit integrated Reamer Research in about 13 minutes and Reamer Server in about 19, so you don't have to learn the C interface by hand. What still takes your judgement is the trading itself: your strategy, your data, your gate's rules and your broker.
stage: 4
order: 31.5
product: both
next: integration-time, supported-languages, build-around-a-core, broker-connections
date: 2026-10-08
---

Not for the integration. The kit documents are enough for a coding agent to do it. In uncut recordings, Claude Code working only from the kit integrated Reamer Research in about 13 minutes and Reamer Server in about 19, so you don't have to learn the C interface by hand. What still takes your judgement is the trading itself: your strategy, your data, your gate's rules and your broker.

## Why it looks steep, and why it isn't

Both products are engines with a C interface that you link into your own program. Traditionally that meant reading a header, a specification and example code, then writing the glue yourself. That is days of work for someone new to it.

That glue is exactly what a coding agent is good at. Each kit ships everything the agent needs to read:

- **The C header,** the full interface, with every call and structure.
- **The specifications:** for Reamer Research, how orders fill and what the result report contains; for Reamer Server, the configuration, the event bus and the strategy socket protocol.
- **Reference code:** a Python binding and a C++ wrapper for Reamer Research; a working server in Go, with C, C++ and Rust starting points, for Reamer Server.

You describe what you want. The agent reads the kit and writes the wiring.

## What the recordings show

Each recording starts from a fresh copy of the kit, with no source code and no earlier exposure, and a wall clock in the frame.

- **Reamer Research, about 13 minutes** ([watch](https://www.youtube.com/watch?v=-GT1v4EXzA0)). Claude Code wrote a multi-instrument strategy from scratch, showed that a second run gives byte-identical results, swept it across signal and cost settings, traced the worst trade, and wrote the full JSON report.
- **Reamer Server, about 19 minutes** ([watch](https://www.youtube.com/watch?v=C0Q27j8m4TM)). Claude Code wrote a trading server in C++ with its own FIX 4.4 broker connector and a gate with an instrument allowlist and a position limit. Against the kit's simulated venue it sent 6 orders: 5 filled, and the gate rejected 1 with its exact reason.

Both were recorded against version 4.2.1. See [How long does integration take?](/faq/integration-time.html)

## What you still decide

An agent removes the learning curve of the interface, not of trading. These stay yours:

- **Your strategy and your data.** The engine tests what you give it.
- **Your gate's rules,** in place of a demonstration policy. See [How can I plug my own risk model into every order before it reaches the broker?](/faq/plug-own-risk-model-pre-trade.html)
- **Your broker,** its own protocol details, session recovery and certification. See [Which brokers does Reamer Server connect to?](/faq/broker-connections.html)

You should still read what the agent writes, as you would any code that trades your money.
