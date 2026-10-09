---
title: What is an order management engine, and does a small trading operation need one?
description: An order management engine sits between your strategies and your broker. It tracks the state of every order, puts orders from every strategy into one sequence, runs a pre-trade check on each one, sends it to the broker, and records every decision and fill. Every trading system already does these jobs somewhere, often scattered through strategy code. A small operation needs a dedicated engine once more than one strategy trades an account, once real money is at stake, or once the system runs without someone watching it.
stage: 2
order: 12
product: server
next: where-pre-trade-risk-checks-belong, self-hosted-order-management-engine, multiple-strategies-one-broker-connection, plug-own-risk-model-pre-trade
date: 2026-09-30
---

An order management engine is the part of a trading system between the strategies and the broker. A strategy decides what it wants to trade. The engine turns that decision into an order, checks it, sends it, follows it until it is finished, and keeps a record of all of it.

Every system that trades already does these jobs. In a single script they are a few lines scattered through the strategy: place the order, hope it fills, check the position later. The question for a small operation is not whether to have order management, but whether to keep it scattered or put it in one place.

## The five jobs

### 1. Track the state of every order

An order is not simply sent and then filled. It passes through states: waiting to be acknowledged, working at the broker, partly filled, filled, cancelled, rejected, waiting for a cancel or a change to be confirmed. The engine keeps each order's current state and applies each report from the broker to it.

Two rules keep that state trustworthy:

- **Finished is finished.** Once an order is filled, cancelled or rejected, nothing more is applied to it. A fill that arrives after a cancel was confirmed is an error to look into, not a fill to count.
- **No overfill.** An order can never be filled for more than its size. A report that would do that is rejected.

### 2. Put every order into one sequence

When several strategies trade one account, their orders have to pass through one place, one at a time, in a known order. Each check then sees the account as it really is, including the orders just before it. Without that, two strategies can each check the same limit at the same moment, each find room, and together go over it.

### 3. Check each order before it leaves

A pre-trade check approves or rejects every order: position limits, order size, exposure, trading hours, whatever your rules are. It should also see cancels and changes to existing orders, not only new ones. The engine's job is to make sure the check runs on every order and nothing reaches the broker without it. The rules themselves are yours, and are best kept apart from strategy code, so that a bug in a strategy cannot switch them off.

### 4. Send orders to the broker and handle what comes back

Sending is the easy part. What comes back is messier:

- **Repeated reports.** Brokers resend reports, especially after a reconnect. The same fill can arrive twice under a new message number. Without a check on each fill's own ID from the broker, the position doubles on paper while the real one does not.
- **Late reports.** A fill can arrive after the order was already marked finished.
- **Unknown orders.** A report can arrive for an order the system did not send or no longer tracks.
- **A dropped connection.** While the broker is disconnected, a new order has nowhere to go. The safe behaviour is to reject it at once, rather than queue it and send it later into a market that has moved.

### 5. Record every decision

For every order: what was asked for, whether it passed the check and why not if it did not, what the broker did, and when. That record answers "what happened?" after a bad day, and it is the raw material for monitoring while the system runs.

## What it is not

- **Not a strategy.** It does not decide what to trade.
- **Not an execution algorithm.** It sends the orders it is given. Splitting a large order over time or choosing between venues is a separate layer, and mid-frequency strategies trading on bars rarely need one.
- **Not your risk model.** It gives the risk model a place to sit and makes sure it runs. The limits and the logic are yours.
- **Not the record of your positions.** The broker is. After a restart, the safest engine takes positions and open orders from the broker rather than trusting its own saved copy.

## Does a small operation need one?

You can manage without a dedicated engine when:

- One strategy trades one account.
- The size is small enough that a mistake is a lesson, not a loss you cannot take.
- Someone watches the system whenever it trades.
- The orders are simple: market orders, few at a time.

You need one once any of these is true:

- **More than one strategy trades the same account.** Without one sequence and one check, their limits do not add up.
- **Real money is at stake.** A doubled fill, an order sent during a disconnect, or a check a strategy bypassed can each cost more than the engine.
- **The system runs unattended.** Overnight, or while you work on something else, the record and the rules are all that stand between a bug and the account.
- **Orders are more than simple.** Limits, stops, cancels and changes to working orders each add states that have to be tracked correctly.
- **Someone else needs to see what happened,** such as a partner, an allocator or an auditor.

For where this sits among the other layers of a trading setup, see [What software do independent quants use to go from research to live trading?](/faq/independent-quant-research-to-live-stack.html)

## Build it or use one

A minimal version for one strategy is a few hundred lines. The work grows with each item above: the order states, repeated and late reports, disconnects, several strategies, the record, the monitoring, and tests for all of it. As with a backtester, this is a layer that has to be right but is rarely an edge. See [Should I build my own backtester or buy one?](/faq/build-or-buy-backtester.html)

## How Reamer Server does it

[Reamer Server](/products/reamer-server.html) is an order management engine delivered as a C library you link into your own program on your own Linux machine. It does the five jobs above, and leaves the two parts that are yours to you: the pre-trade check and the broker connection. Everything below is documented in the kit:

- **Order state.** Every order is tracked through eight states, from pending to filled, cancelled or rejected, including pending cancel and pending replace. Finished is finished and no overfill are enforced on every report, and a report that breaks either is dropped and published as an event. `EXTENSION_PROTOCOL.md` gives the rules.
- **One sequence.** Strategies connect as separate processes, in any language, over a local socket or a remote relay. Their orders pass through one pipeline. In a sweep from 1 to 512 concurrent strategies, every order matched its accept or reject event exactly once at every point. See `BENCHMARK.md` and [Reamer Server on a 64-Core EPYC: The Tail Is the Measurement](/blog/reamer-server-findings-epyc.html).
- **Your check on every order.** Your gate is called once for every new order, cancel and replace, with the account's state fetched from your broker connection just before. If the broker connection reports it is disconnected, the order is rejected with the reason `venue disconnected` and the gate is not called. That rule is fixed.
- **Broker reports handled.** A fill repeated with the same execution ID is rejected before it adds quantity or commission. Late reports on finished orders are rejected, and reports for unknown orders are logged. Your connector has to supply the execution ID for the duplicate check to work.
- **The record.** Every intent, decision, fill and rejection, with the reason, is published on a shared-memory event stream that any number of your own processes can read. A Prometheus-format `/metrics` endpoint reports order counts, rejections by reason and connection state.

What stays yours: the gate's rules, the broker connection (the kit includes a worked FIX 4.4 reference in Go to adapt), durable storage for the event stream, which is held in memory, and alerting. The server does not cancel your open orders when it shuts down; they stay at the broker, and on the next start positions and open orders are read back from your broker connection. Its scope is mid-frequency order flow, not a high-frequency order-book system. Orders cover the standard types and time-in-force values, attached take-profit and stop-loss, iceberg, post-only and reduce-only, and linked orders, and each can carry up to 1,024 bytes of your own data; your connector maps them to your broker.
