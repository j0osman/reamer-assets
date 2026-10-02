---
title: Which brokers does Reamer Server connect to?
description: Any broker you write a connector for. Reamer Server ships no ready-made broker connection. You write the connector against its C interface, in C, C++, Rust or Go, over whatever API your broker offers. The kit includes an in-memory paper broker and a worked FIX 4.4 example run against a simulated venue.
stage: 4
order: 31
product: server
next: integration-time, paper-trading, supported-languages, research-to-server-move
date: 2026-10-02
---

Any broker you write a connector for. Reamer Server ships no ready-made broker connection. You write the connector against its C interface, in C, C++, Rust or Go, over whatever API your broker offers. The kit includes an in-memory paper broker and a worked FIX 4.4 example run against a simulated venue.

## What a connector does

The connector is a set of functions the server calls. Yours must:

- **Send orders, cancels and replaces** to the broker.
- **Report back** fills and order updates, as they arrive.
- **Supply the account's state,** meaning positions and open orders, on request. The broker is the only record of these; the server keeps none of its own.
- **Say whether its trading session is up.** While it is down, the server rejects new orders with the reason "venue disconnected" rather than sending them.

Inside those functions, the broker's API is your choice: FIX, a REST or WebSocket API, or the broker's own SDK, as long as your language can call it.

## What ships in the kit

- **A paper broker,** in C++ and Rust, that fills orders in memory. It confirms your server is wired up before you touch a real account.
- **A worked FIX 4.4 integration in Go.** Its session logs on, keeps sequence numbers, sends heartbeats and reconnects, and its connector turns execution reports into the server's format. One script runs an order to a fill against a simulated venue.

## What it is not

- **Not certified for any broker.** The FIX example shows what a connector needs; it is not your broker's dialect, credentials or certification.
- **Not production session handling.** On a sequence gap it reconnects and resets rather than requesting the missing messages, and it does not save sequence numbers across restarts. A real broker integration adds those.

Writing and certifying the connector for your broker is your work, or a contractor's. See [What is Reamer Server?](/faq/what-is-reamer-server.html) and [How do I run several strategies through one broker connection?](/faq/multiple-strategies-one-broker-connection.html)
