---
title: Is there a self-hosted order management engine I can build my own OMS around?
description: Yes. Look for an engine you link into your own program, which keeps order state, puts every strategy's orders into one sequence and publishes every decision, and leaves the pre-trade check, the broker connection and storage to you. Reamer Server is built that way. It is a C library for Linux at $7,200 per seat per year, with a $900 30-day trial.
stage: 3
order: 19
product: server
next: what-is-reamer-server, what-is-an-order-management-engine, where-pre-trade-risk-checks-belong, backtest-to-live-without-rewrite, broker-connections, integration-time
date: 2026-10-01
---

Yes. Look for an engine you link into your own program, which keeps order state, puts every strategy's orders into one sequence and publishes every decision, and leaves the pre-trade check, the broker connection and storage to you. [Reamer Server](/products/reamer-server.html) is built that way: a C library for Linux that you link into a program you write.

Building around a core is a middle path between writing an order management system from scratch and buying a finished one. The core is the part that is hard to get right and the same for everyone. The parts around it are the parts that are yours anyway: your risk rules, your broker, your records.

What an order management engine does, and when a small operation needs one, is covered in [What is an order management engine, and does a small trading operation need one?](/faq/what-is-an-order-management-engine.html) This page is about building one around a core.

## What to buy and what to build

An order management system in a small operation has six parts. They divide cleanly:

- **Order state and sequencing: buy the core.** It is the same problem for everyone, and the easiest place for a subtle bug: duplicate fills, overfills, a cancel that races a fill.
- **Pre-trade check: build.** Your risk rules are your policy. A vendor's fixed rules rarely fit, and cannot be audited as your own.
- **Broker connection: build or adapt.** It is specific to your broker, your account and your credentials.
- **Strategy connection: build, thinly.** The client your strategies use to send orders and receive updates.
- **Durable record: build.** Where events are stored, for how long, and in what database is your choice.
- **Monitoring and alerts: build on what the core exposes.** Your dashboards and your pager.

The core should be the one part you would not want to write, and should be easy to wrap with the rest.

## What to look for in a core

1. **It runs in your process, on your machine.** A library you link, not a hosted service. Orders, positions and credentials go from your machine to your broker, never through the vendor.
2. **A small, written contract.** A header and a protocol document that say exactly what the core calls, when, from which thread, and what it expects back.
3. **Your pre-trade check on every order,** including cancels and changes, with no way for a strategy to go around it.
4. **One source of account state.** The core should read positions and open orders from your broker connection, not keep a second copy that can drift from the broker's.
5. **Strict handling of broker reports.** Duplicate fills, reports for finished orders, and quantities beyond the order should be rejected and reported, not applied.
6. **Strategies outside the core,** in separate processes and any language, so a crashing strategy cannot take the order path down with it.
7. **Every decision published,** so your storage, dashboards and alerts can read them without slowing the order path.
8. **Defined behaviour at the edges:** what happens on shutdown, on restart, when the broker disconnects, and when a limit is reached.
9. **Measured performance,** with the method and the raw data, and a way to rerun it on your own hardware.

## The work you are signing up for

Building around a core is less work than building the core, but it is not none:

- **The broker connection is the largest part.** A session that logs on and sends an order is quick. One that recovers after a dropped connection, handles a gap in message sequence numbers, and survives a restart without losing fills takes real time, and your broker may require certification.
- **The pre-trade check is small but important.** It should be simple enough to read in one sitting, and tested against every order type, cancels and changes.
- **Storage and alerting are routine** once the core publishes everything, but they are still yours to run.
- **Test it all against a simulated broker** before a paper account, and against a paper account before money.

## How Reamer Server is built to be wrapped

Reamer Server is a static C library. You write `main()`, implement two sets of callbacks, and call `reamer_server_run()`. That call is your server process. Everything below is documented in the kit:

- **Two callbacks you implement.** A gate, which is your pre-trade check, called for every new order, cancel and replace with the account's current state. And a broker connector, which submits orders, returns fills, and reports positions and connection state. If the connector reports it is disconnected, orders are rejected without calling the gate.
- **One copy of account state, from your broker.** The core keeps no position ledger of its own. It uses what your connector last reported, so there is nothing to reconcile and nothing to restore on restart.
- **Order state enforced.** Eight order states, with no overfill, no change to a finished order, and duplicate fills rejected by execution ID. Market, limit, stop, stop-limit, trailing, if-touched and market-to-limit orders, auction time in force, attached take-profit and stop-loss tracked as orders of their own, iceberg, post-only and reduce-only, OCO and OTO links, and up to 1,024 bytes of your own data per order, on any instrument you name. Your connector maps them to your broker.
- **Strategies as separate processes.** They connect over a local socket or through a relay, in any language. Both wire protocols are specified byte for byte in `EXTENSION_PROTOCOL.md`.
- **Every decision on an event stream.** Intents, gate decisions, fills, rejections and connection changes go to a shared-memory ring that any number of your own processes can read, for storage, dashboards or audit. A `/metrics` endpoint in Prometheus format and a `/health` probe cover monitoring.
- **Defined edges.** On shutdown, the core stops taking new orders and drains your connector for up to five seconds by default, then publishes a final snapshot from the broker. It never cancels your open orders; they stay at the broker and are read back on the next start. Configuration is a JSON file plus environment variables, checked offline by `reamer-config-check`; a change needs a restart.
- **Starting points, not a blank file.** An accept-all gate and an in-memory paper broker in C++ and Rust, about a hundred lines against the header. And a full worked integration in Go: a FIX 4.4 session, broker connector, strategy relay and event reader, with 100 tests and one script that drives an order from strategy to fill against a simulated venue in about twenty seconds.
- **Measured.** On an AMD EPYC 9575F, 53 ns per order to cross into your own code and back, 11.3 µs at P50 and 12.4 µs at P99 end to end, and no correctness failure at any point from 1 to 512 concurrent strategies. Memory stayed flat across 500,000 orders. `server-bench` in the kit reruns the strategy sweep on your hardware. See [Reamer Server on a 64-Core EPYC: The Tail Is the Measurement](/blog/reamer-server-findings-epyc.html).

## Limits to know

- **No adapter for your broker.** The FIX reference runs against a simulated venue. Fitting it to your broker, including sequence-gap recovery and certification, is your work.
- **No risk rules.** The gate is the decision point; what it checks is entirely yours.
- **No durable storage.** The event stream is a ring of 4,096 events by default. A reader that falls further behind than that loses the events in between, so your storage process has to keep up.
- **A cap on tracked orders.** 5,000 by default; past that, the oldest is dropped from tracking and a later cancel of it is rejected as unknown. Raise it to suit your volume.
- **Not a full trading platform.** No allocation across accounts, no compliance layer, and no order-book data. It is built for mid-frequency order flow, not high-frequency trading.
- **Linux only, on a recent base.** x86-64 with glibc 2.39 or later: Ubuntu 24.04 or Debian 13 and later. RHEL 9 does not meet it.
- **Closed source.** You link the library and read the header and protocol; you do not get the core's source.

$7,200 per seat per year, paid once, with no automatic renewal. The $900 30-day trial includes the full kit, so you can run the reference integration to a fill and write your own gate and connector before buying a licence. See [pricing](/pricing.html).
