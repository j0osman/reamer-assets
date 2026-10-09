---
title: Why use Reamer Labs if I want to build my own trading system?
description: You still build your own system. Reamer Labs supplies only its core, the part that has to be right and works the same for everyone. Reamer Research is the research engine, with fill and cost rules already written and tested. Reamer Server is the order core, with order state, sequencing and a record of every decision. You build everything around them your own way, including strategies, data, risk rules, the broker connection, storage and monitoring.
stage: 4
order: 35.7
product: both
next: what-is-reamer-research, what-is-reamer-server, self-hosted-order-management-engine, research-to-server-move, integration-time
date: 2026-10-08
---

You still build your own system. Reamer Labs supplies only its core: the part that has to be right, works the same for everyone, and makes no one money by being written again. You build everything around it, your own way, in your own language and on your own machine.

That sits between the two usual choices. Building everything means months spent on plumbing before any strategy is tested. Buying a finished platform means taking its data, its brokers, its risk rules and its way of working. A core leaves the parts that are yours in your hands.

## What Reamer Labs supplies

**Reamer Research, the research core.** A research engine delivered as a library with a stable C interface:

- Fill and cost rules, written down in an execution specification that ships with the kit, with the engine tested against it. They cover market, limit and stop orders, gaps, stops and targets in the same bar, bid and ask, spread, slippage, commission, margin, overnight swap and futures rolls.
- Byte-identical output for a fixed `rng_seed`, so a change in results is always your change.
- About 1.7 million bars a second per backtest, fast enough to sweep.
- 31 summary metrics plus every order, fill and trade, in a versioned JSON report.
- Full position state and every resting order on each bar, with actions that re-price an order, move a position's stop and target, close all or cancel all.
- A relay, `reamer_relay_*`, that sends the strategy's orders to Reamer Server and reads the fills back.

**Reamer Server, the live core.** An order management engine delivered as a library you link into your own program:

- Order state and sequencing: eight order states, no overfills, and duplicate fills rejected.
- A call to your pre-trade gate for every new order, cancel and replace.
- Account state read from your broker connection, so there is no second ledger to drift.
- Every decision published on an event stream, with Prometheus metrics and a health check.

## What you build

- **With Reamer Research:** your strategies, in Python, C++ or any language that can call C. Your data pipeline and your sweep, analysis and reporting scripts.
- **With Reamer Server:** the gate's rules, which are your risk policy. The connector to your broker, plus your strategy processes, storage for the event stream, and dashboards and alerts.

Each product comes with starting points rather than a blank file: Python and C++ example strategies and templates for Reamer Research, and for Reamer Server an accept-all gate and paper broker in C++ and Rust, plus a full FIX 4.4 integration in Go. On uncut recordings, a working integration took about 13 minutes for Reamer Research and 19 for Reamer Server. See [How long does integration take?](/faq/integration-time.html)

## Why the line sits there

The core is where a subtle bug costs money without anyone noticing: a stop filled at the wrong price, a fill counted twice, an order changed after it finished. It is also identical from one trader to the next, so writing it yourself adds nothing. The parts outside it are where traders differ: what you trade, how you size, what you will and will not risk, and which broker you use. Those stay yours.

## When a core is not the right fit

- **The core itself is your edge,** for example a fill or microstructure model better than anyone else's.
- **Your strategy is outside its scope:** order-book or high-frequency trading, options, or Windows.
- **You want everything done for you,** including data, brokers and hosting. A full platform fits better.

## Price

Reamer Research costs $1,800 per seat per year and Reamer Server $7,200, each paid once with no automatic renewal. The 30-day trials cost $225 and $900. See [pricing](/pricing.html).
