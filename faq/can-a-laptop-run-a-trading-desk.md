---
title: Can one laptop run serious quant research and live trading?
description: For mid-frequency strategies on bars, yes. Research on a 2017 four-core laptop can process hundreds of thousands of bars per second, and a live order pipeline on the same laptop can keep latency in the tens of microseconds with one strategy per core. The limits are cores, uptime and the network, not raw speed. Keep live strategies on their own cores and plan for the machine to restart.
stage: 1
order: 6
product: both
next: what-is-institutional-grade-trading-infrastructure, institutional-grade-infrastructure-for-independent-quants, performance, supported-platforms
date: 2026-09-29
---

A modern laptop has more computing power than a trading desk's servers had twenty years ago. For strategies that trade on bars, from minutes to days, speed is rarely the constraint. What decides whether one machine is enough is how the work is shared across its cores, and whether it stays on and connected while the market is open.

## What "serious" needs from the machine

Two different jobs run on a quant's machine, and they want different things.

- **Research** wants throughput: as many bars through as many variations as possible, so a parameter sweep finishes while you are still thinking about it. It uses every core it is given, and nobody is hurt if it slows down.
- **Live trading** wants steady latency: each order handled promptly and consistently, every time. It uses few cores, and it must never wait for one.

A laptop can do both. The mistake is letting them compete.

## Where a laptop is enough

- **Mid-frequency strategies.** Signals on bars, from a minute upward, with orders measured per minute or per hour, not per microsecond. Here, a few hundred microseconds of processing is nothing next to the network trip to a broker.
- **A handful of live strategies.** One strategy per physical core keeps latency steady. A 8-core laptop runs 8 live strategies comfortably.
- **Years of intraday history.** Millions of bars per study fit in memory and run in seconds to minutes.

## Where it is not

- **High-frequency trading.** Strategies that compete on microseconds of network distance to an exchange need hardware placed next to the exchange, not a laptop anywhere.
- **More live strategies than cores.** Past one busy strategy per physical core, the typical order still processes quickly, but the slowest orders wait for a core and get much slower.
- **Unattended trading that must not stop.** A laptop sleeps, loses power, updates itself and changes networks. If a strategy must run all day without you, a small always-on server or a cloud machine is the safer home for the live side.

## How to share one machine between research and live trading

1. **Give live strategies their own cores.** Pin each live strategy to a physical core, and leave those cores alone.
2. **Run research on what is left.** Sweeps use the remaining cores, or run outside market hours.
3. **Turn off sleep and automatic updates** while live strategies run, and use a wired connection where you can.
4. **Plan for a restart.** Know what happens to open orders and positions if the machine or process stops, and make sure the live system can pick up from the broker's records rather than from its own memory.
5. **Measure on your own machine.** Published benchmarks tell you what is possible. Your hardware, operating system and strategy decide what you get.

## How Reamer Research and Reamer Server use one machine

Reamer Labs was built around this question: one engine for research and one for live execution, fast enough that a single ordinary laptop could do the work of a trading desk's infrastructure. The figures below come from `BENCHMARK.md` in each kit, measured on a 2017 laptop-class Intel i5-8250U with four cores. The same documents also report runs on a 64-core server processor.

**[Reamer Research](/products/reamer-research.html)**

- The native engine processed about 467,000 bars per second on that laptop with realistic fills. A Python strategy, which pays for the language on every bar, ran 884,130 bars in about 65 seconds.
- Each backtest uses one thread, and one seat covers any number of concurrent backtests on the licensed machine, so a sweep can use every core not reserved for live trading.

**[Reamer Server](/products/reamer-server.html)**

- On the same laptop, with four strategies connected at once, one per physical core, the middle order took 56 microseconds and 99% of orders finished within 139 microseconds.
- At eight strategies on four cores, the middle order barely moved, to 100 microseconds, but the slowest 1% rose to 4.2 milliseconds. No orders were lost at any strategy count tested. That is the one-strategy-per-core rule measured, and the 64-core run showed the same limit at around 64 strategies.
- The server keeps no state of its own between runs. On start, order and position state is rebuilt from your broker connector, so a restart does not leave two versions of the truth to reconcile.

Two limits to check before choosing a laptop. Reamer Server runs on Linux x86-64 only, with glibc 2.39 or newer. Reamer Research also runs on Apple Silicon Macs, so a Mac can do the research but not the live side. And each product's seat covers one machine: research and live trading on the same laptop use one seat of each.

The engines make one machine enough on speed. Keeping that machine on, connected and uncontended while it trades is still your part of the job.
