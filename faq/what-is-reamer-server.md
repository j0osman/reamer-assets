---
title: What is Reamer Server?
description: Reamer Server is an order management engine that takes a strategy live. It is the core of your trading process, delivered as a library with a stable C interface. It holds order state and sequencing, runs your pre-trade check on every order and sends accepted orders to your broker through a connector you write. It costs $7,200 per seat per year, with a $900 30-day trial.
stage: 4
order: 25
product: server
next: who-its-for, broker-connections, supported-languages, paper-trading, what-is-reamer-research, pricing
date: 2026-10-02
---

Reamer Server is an order management engine that takes a strategy live. It is the core of your trading process, delivered as a library with a stable C interface. It holds order state and sequencing, runs your pre-trade check on every order and sends accepted orders to your broker through a connector you write.

It is a library, not an application. You write a short `main()` that links it, supplies two parts of your own and starts it. That program is your server.

## The core, and the two parts you build

- **The core (what you buy).** It takes order intents from your strategies, puts them into one sequence, calls your gate once for each, and passes accepted orders to your connector. It handles market, limit, stop and stop-limit orders. Fills and order updates go back to the strategy that sent the order.
- **Your gate.** Your pre-trade risk check. For each order it says pass or reject, with a reason, given the account's last known state. Its rules are yours; the core does not supply any. See [How can I plug my own risk model into every order before it reaches the broker?](/faq/plug-own-risk-model-pre-trade.html)
- **Your connector.** Your broker session. It sends orders, reports fills, and is the only source of positions and open orders. Nothing in the kit talks to a real broker for you.

Gate and connector can be written in C, C++, Rust or Go. Strategies connect over a socket protocol, specified byte for byte, from any language. See [How do I run several strategies through one broker connection?](/faq/multiple-strategies-one-broker-connection.html)

## What else is in the kit

- **Starting points.** An accept-all gate with an in-memory paper broker, in C++ and Rust, and a worked integration in Go with a FIX 4.4 session, a strategy relay and a simulated venue that runs an order to a fill in one command.
- **A live feed of every decision.** Each acceptance, rejection (with your gate's reason), fill and order update is published on a shared-memory event stream that any number of your own processes can read. It holds a fixed number of recent events, so keeping a permanent log is up to your reader.
- **Monitoring.** A Prometheus `/metrics` endpoint and a `/health` route.
- **A benchmark.** `server-bench` measures throughput and latency on your own hardware.

## What it is not

- **Not a broker or a broker connection.** You write the connector for your broker, and its certification is yours.
- **No stored state of its own.** On start, orders and positions are loaded from your connector; the broker is the only record. Persisting anything else is your job.
- **Not for high-frequency or order-book strategies,** and it supplies no market data or strategy framework.
- **Linux x86-64 only,** with glibc 2.39 or later. No macOS or Windows version.

## Price

$7,200 per seat per year, paid once, with no automatic renewal. The $900 30-day trial includes the full kit. See the [product page](/products/reamer-server.html), [What is an order management engine?](/faq/what-is-an-order-management-engine.html) and [pricing](/pricing.html).
