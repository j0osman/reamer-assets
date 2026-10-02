---
title: How fast is it?
description: On a 64-core AMD EPYC 9575F, Reamer Research replays about 1.7 million bars a second in one backtest, and Reamer Server takes an order from a strategy to an accepted event in a median of 11.3 µs, with about 200,000 to 220,000 orders a second across many strategies. Each figure comes with the conditions it was measured under, and both kits include a benchmark to run on your own hardware.
stage: 4
order: 30
product: both
next: results-and-metrics, integration-time, paper-trading, supported-platforms
date: 2026-10-02
---

On a 64-core AMD EPYC 9575F, Reamer Research replays about 1.7 million bars a second in one backtest, and Reamer Server takes an order from a strategy to an accepted event in a median of 11.3 µs, with about 200,000 to 220,000 orders a second across many strategies.

Every figure below is from the kits' own benchmark documents or the whitepaper. Each one measures something specific, so read the conditions with the number.

## Reamer Research

- **About 1.7 million bars a second** on the EPYC, with a strategy that places no orders, called through the C interface. Across 2,000 runs, the slowest was within 4% of the fastest.
- **With real trading,** a 20-bar breakout strategy making 47,728 trades over 884,130 bars took about 1.9 seconds from C or C++ and about 65 seconds from Python. That was measured on a 2017 laptop processor, so a server will be faster.
- **Python costs about 33 times more per bar** than C++, because each bar crosses into Python.
- **Scale:** on the EPYC, 50 million daily bars across 10,000 instruments over 20 years, with 990,000 orders carrying stop-loss and take-profit brackets, ran in 29.1 seconds using about 1 GB of extra memory. The bars were generated for the test.

Each backtest runs on one thread. For a parameter sweep, run several backtests at once.

## Reamer Server

- **11.3 µs median and 12.4 µs at the 99th percentile,** from a strategy sending an order over the local socket to its accepted event, with one strategy, an accept-all gate and a stub connector.
- **About 200,000 to 220,000 orders a second** in total, with roughly 32 to 67 strategies on the one instance.
- **Latency stays steady up to about one strategy per physical core.** Past that, the slowest orders get slower first.

These figures leave out your gate's own logic, the network and the broker. A broker round trip typically takes 1 to 10 milliseconds or more, so it outweighs the server's share. Neither product is built for high-frequency trading.

## Measure it yourself

The Research kit ships `research-bench` and the Server kit ships `server-bench`. Each runs the benchmark above on your own machine in one command, during the [30-day trial](/pricing.html). The method and full results from the EPYC are in the [whitepaper](https://doi.org/10.6084/m9.figshare.33972466).
