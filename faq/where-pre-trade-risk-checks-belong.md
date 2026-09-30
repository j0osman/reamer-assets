---
title: Where should pre-trade risk checks sit in a trading system?
description: Between the strategies and the broker, as one separate step that every order passes through after orders from all strategies are put into one sequence and before anything is sent. The check makes one pass or reject decision per order, including cancels and changes, using the order and the account's current state. It should live outside strategy code, so a bug in a strategy cannot switch it off, and the rules in it should be your own.
stage: 2
order: 13
product: server
next: self-hosted-order-management-engine, plug-own-risk-model-pre-trade, multiple-strategies-one-broker-connection, what-is-an-order-management-engine
date: 2026-09-30
---

Pre-trade risk checks belong between the strategies and the broker, as one separate step that every order passes through. Orders from all strategies are first put into one sequence, and then the check makes one decision per order, pass or reject, before anything is sent. It should not live inside a strategy, and it should not be left to the broker.

The position matters as much as the rules. A good limit in the wrong place can be skipped, can see the wrong account, or can act too late.

## Where it sits

In order, the path of an order is:

1. **The strategy** decides what it wants to trade.
2. **Order management** puts the order into one sequence with orders from every other strategy on the account.
3. **The pre-trade check** approves or rejects it.
4. **The broker connection** sends it, if it passed.

Two things follow from that position:

- **After sequencing,** so the check sees the account as it is, including the orders just before this one. If two strategies are checked at the same moment against the same limit, each can find room and together go over it.
- **Before the broker,** because an order that has been sent cannot be unsent. A check after the fact is monitoring, not prevention.

For what the order management step does, see [What is an order management engine, and does a small trading operation need one?](/faq/what-is-an-order-management-engine.html)

## Why not inside the strategy

Putting the limits in the strategy is the usual starting point, and it works for one strategy watched by its author. It stops working for four reasons:

- **A bug can skip it.** The code that makes the mistake is the same code that was meant to catch it. A wrong branch, a bad parameter or an exception in the wrong place can send an order around the check.
- **Each strategy sees only itself.** A strategy knows its own position, not the account's. Limits on total exposure or on one instrument across strategies cannot be checked from inside any one of them.
- **Changing a limit means changing a strategy.** Tightening a risk rule should not require touching, retesting and redeploying trading logic.
- **It is harder to test.** A check in its own place can be tested with a list of orders and expected decisions. A check spread through strategy code can only be tested by running the strategies.

A strategy can still refuse to send an order it knows is wrong. That is a useful first line, but not the line you rely on.

## Why not only at the broker

Brokers run their own checks, and they are worth having. They are not a substitute for yours:

- **They protect the broker.** Their limits are set for the broker's risk, usually margin and buying power, not for your strategy or your tolerance.
- **They are generic.** They cannot know that one strategy should never hold more than a set size, or that nothing should trade in the first five minutes.
- **They act late.** By the time a broker rejects an order, it has left your system. Some errors are only caught after a fill.

## What the check needs to see

One decision per order needs three things:

- **The order.** The instrument, side, quantity, order type, limit and stop prices, time in force, and which strategy sent it.
- **The account's current state.** Positions, open orders and whether the broker connection is up. The safest source is the broker connection, read just before the check, rather than a running total kept separately that can drift from the real account.
- **Your own reference data.** The limits themselves, trading hours, which instruments are allowed and, for price checks, a recent market price.

## What it usually checks

The rules are yours, but most setups start from a similar list:

- Maximum size of a single order.
- Maximum position in one instrument, for each strategy and for the account.
- Maximum total exposure across the account.
- Maximum number of open orders.
- Allowed instruments, and trading hours for each.
- Limit prices too far from the market, the classic mistyped price.
- A switch that stops all new orders at once.

## How it should behave

- **One decision per order.** Pass or reject, with the reason written down for every rejection.
- **Every order, including cancels and changes.** A change to a working order can raise risk as much as a new one. Cancels usually reduce risk, so rules that would stop a cancel need care: a check that rejects every order with a quantity of zero, for example, may reject every cancel.
- **Fast and deterministic.** It runs on every order, in the order's path. The same order and the same account state should always give the same answer.
- **Fail closed.** If the check cannot decide, because of an error, missing data or a service it depends on being down, the order is rejected. A check that lets orders through when it breaks is only a check when nothing is wrong.
- **No orders while disconnected.** While the broker connection is down, a new order has nowhere to go and the account's state is unknown. The order should be rejected at once, not queued and sent later.
- **Recorded.** Every decision, and every reason, goes into the record, so a rejection is something you can count and explain rather than an order that quietly did not happen.

## Layers of checks

Pre-trade checks are one layer among several:

1. The strategy's own sanity checks.
2. Your pre-trade check, which every order passes.
3. The broker's checks.
4. Monitoring after the fact: alerts on rejection rates, positions and profit.

The second is the only one that you control and that every order has to pass before it leaves. It is the one to get right.

## How Reamer Server places it

[Reamer Server](/products/reamer-server.html) is built around this position. Your check, called the gate, is a set of functions you write and link into the same program as the server core. The server supplies the position and the plumbing around it. It does not supply a risk model. Everything below is documented in the kit:

- **Between the sequence and the broker.** Orders from every connected strategy pass through one pipeline, and the gate is called for each one before your broker connector receives it. If the gate accepts, the order is sent in the same call. Nothing reaches the broker connector without the gate's decision.
- **Every intent.** The gate is called once for every new order, cancel and replace. Cancels arrive with quantity and prices at zero, so the kit warns against rejecting on quantity alone.
- **Fresh account state.** Just before each check, the server reads positions, the open order count and the connection state from your broker connector and passes them to the gate with the order. The gate is not given market prices, so price checks need your own source.
- **Disconnected means rejected.** If your broker connector reports it is disconnected, the order is rejected with the reason `venue disconnected` and the gate is not called. That rule is fixed.
- **Fails closed.** A gate that reports itself unavailable, or whose code throws an exception, produces a rejection, not an accepted order and not a crash.
- **Every reason recorded.** Each rejection carries your gate's reason text on the event stream, and the `/metrics` endpoint counts rejections by reason.
- **No timeout.** The gate runs in the order's path with no timeout, so a gate that hangs stops the order. `BENCHMARK.md` measures the cost of crossing into your gate and back at about 70 ns, before your own logic.

What stays yours is the gate itself: the rules, the limits and any data they need. The kit includes accept-all starting points in C++ and Rust and a worked Go reference whose demo gate passes some orders and rejects one, to replace with your own policy. `EXTENSION_PROTOCOL.md` gives the full contract.
