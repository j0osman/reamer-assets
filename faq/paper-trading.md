---
title: Can I paper trade before going live?
description: Yes, by pointing your Reamer Server connector at your broker's paper or demo account. The server handles paper and live orders the same way, and the licence allows both. The kit's in-memory paper broker only checks that your server is wired up correctly; it fills every order instantly at a made-up price, so it is not a market simulation.
stage: 4
order: 34
product: server
next: research-to-server-move, broker-connections, integration-time, results-and-metrics
date: 2026-10-02
---

Yes, by pointing your Reamer Server connector at your broker's paper or demo account. The server handles paper and live orders the same way, and the licence allows both. The kit's in-memory paper broker only checks that your server is wired up correctly; it fills every order instantly at a made-up price, so it is not a market simulation.

## Three stages of testing

1. **Wiring check, with the kit's paper broker.** An accept-all gate and a broker that fills in memory, in C++ and Rust. It confirms your program starts, takes orders and returns fills. It tells you nothing about how your strategy would trade.
2. **Session check, with the kit's simulated FIX venue.** It lets you test your FIX session and connector, logon through to fills and rejections, before you connect to anyone else.
3. **Paper trading, at your broker.** Point your connector at the broker's paper or demo account, if it offers one. Prices, fills and rejections then come from the broker. Nothing in Reamer Server changes when you later switch to the live account. Brokers usually simulate paper fills themselves, often more kindly than the real market, so this stage tests your system more than your strategy's profits.

## Running paper and live side by side

A second instance on the same machine can run against a paper account, with its own socket, event bus and monitoring port. The two cannot share them. Reamer Server has no built-in switch for this; you set it up in configuration.

## Licensing

The licence does not restrict paper or live trading, and a 30-day trial key is a full paid licence on the same terms. A seat is one machine, so an instance on another machine needs its own seat. See [pricing](/pricing.html).

Reamer Research has no paper or live mode; it runs backtests only. See [Which brokers does Reamer Server connect to?](/faq/broker-connections.html) and [What is Reamer Server?](/faq/what-is-reamer-server.html)
