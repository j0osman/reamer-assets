---
title: How do I run several strategies through one broker connection?
description: Put one process between the strategies and the broker. Each strategy connects to it, it puts every strategy's orders into one sequence, checks each against your limits and sends it over the one connection, then routes each fill back to the strategy that sent the order. Track which strategy owns each order and each position yourself, because the broker sees one account. Reamer Server is that process; it was measured with up to 512 strategies connected at once.
stage: 3
order: 21
product: server
next: self-hosted-order-management-engine, plug-own-risk-model-pre-trade, where-pre-trade-risk-checks-belong, what-is-reamer-server
date: 2026-10-01
---

Put one process between the strategies and the broker. Each strategy connects to it, it puts every strategy's orders into one sequence, checks each against your limits and sends it over the one connection, then routes each fill back to the strategy that sent the order. Track which strategy owns each order and each position yourself, because the broker sees one account. [Reamer Server](/products/reamer-server.html) is that process; it was measured with up to 512 strategies connected at once.

## Why one connection

Most brokers give an account one trading session, or a small number, and some charge per session. Even where several are allowed, one shared path is usually better:

- **One place to check risk.** Limits on the whole account, such as total exposure or open orders, can only be enforced where every strategy's orders meet.
- **One order of events.** When two strategies trade the same instrument at the same moment, one sequence decides which order went first, and the record shows it.
- **One session to keep healthy.** Logon, heartbeats, reconnection and sequence numbers are handled once, not once per strategy.

The alternative, each strategy holding its own broker session, puts every one of those jobs into every strategy, and no single place sees the whole account.

## The shape

1. **Strategies** run as separate processes. They decide; they do not talk to the broker.
2. **The hub** accepts their connections, puts orders into one sequence, runs the pre-trade check, and sends accepted orders to the broker.
3. **The broker connection** is one session, owned by the hub.
4. **Fills and order updates** come back through the hub, which routes each one to the strategy that sent the order.

Running strategies as separate processes also contains failures. A strategy that crashes or hangs takes only itself down; the others and the broker session keep running.

## What the hub has to get right

- **Order ids that cannot collide.** Two strategies that both number their orders from 1 will both send an order called `1`. Prefix every id with the strategy's name, or have the hub assign ids.
- **Routing back.** Every fill, partial fill, rejection and cancel confirmation must reach the strategy that owns the order, and only that strategy.
- **Who may cancel what.** A strategy should be able to cancel or change only its own orders. Enforce it in the hub's check; do not rely on strategies behaving.
- **Fair order.** Decide what happens when several strategies send at once: arrival order, or priority. With strict priority, a busy high-priority strategy can hold back a quieter one indefinitely.
- **Disconnects.** Decide what happens to a strategy's working orders when it disconnects: leave them at the broker, or cancel them. Either can be right; it should be a decision, not an accident.

## The account is shared; positions are not

The broker reports one net position per instrument for the account. If one strategy is long 100 AAPL and another is short 60, the broker shows long 40. Neither strategy's view is the broker's.

So:

- **Each strategy keeps its own position** from its own fills, which the hub routes to it.
- **Account limits use the broker's net figure,** which is what you are actually exposed to.
- **Per-strategy limits need per-strategy positions,** kept in the hub from the fills it routes. They will not come from the broker.
- **Reconcile regularly.** The sum of the strategies' positions should equal the broker's net position. When it does not, a fill was missed or misrouted.
- **Two strategies on one instrument can trade against each other's intentions,** one buying as the other sells. Some setups block that in the pre-trade check; at minimum, make it visible.

## How many strategies one machine can take

Strategies that only wait for bars use little CPU, so dozens can share a machine. Strategies that compute or send orders constantly each want a core of their own. The useful sizing rule is to measure on your own hardware: latency stays flat while every busy strategy has a core, and the slowest orders get much slower once they start sharing.

## How Reamer Server does it

Everything below is documented in the kit:

- **Strategies connect over a socket.** By default each strategy opens a local Unix socket to the server; one connection is one strategy, and the server identifies it from the connection. Both wire protocols are specified byte for byte in `EXTENSION_PROTOCOL.md`, so a strategy can be written in any language.
- **Or through a relay.** In remote mode, your own relay process holds many strategy connections, over any transport you choose, and forwards them to the server on one link. The kit's reference relay accepts newline-delimited JSON over TCP, so a Python strategy needs only the standard library.
- **One sequence, one gate, one connector.** Every strategy's orders pass through one pipeline, your gate is called for each one, and accepted orders go to your one broker connector. Each order carries the strategy's id, so your gate can apply per-strategy rules.
- **Updates go back to the sender.** Over a local socket, the server writes order updates only to the connection that sent the order. Through a relay, each update carries the id of the strategy connection, for your relay to route.
- **Every decision on the event stream.** Acceptances and rejections, with your gate's reasons, are published on a shared-memory event stream that any number of your own processes can read.
- **Ordering is stated.** Over local sockets, orders in each cycle are taken in order of arrival. Through a relay you can set a priority per strategy; it is strict, and the kit warns that a busy high-priority strategy can starve a lower one.
- **Measured.** On an AMD EPYC 9575F with 64 cores, the kit's benchmark ran from 1 to 512 strategies, each a separate connection sending orders back to back. There was no correctness failure at any count. Throughput peaked at about 220,000 orders a second around 34 strategies; latency stayed tight up to about one busy strategy per core, and past that the slowest orders slowed sharply. `server-bench` in the kit reruns that sweep on your hardware.

## Limits to know

- **No per-strategy positions.** The account state your gate receives is the broker's net position per instrument. Positions per strategy, and limits on them, are yours to keep from the fills.
- **Strategy clients.** The protocols are specified. Reamer Research ships a client for the local socket, `reamer_relay_*`, so a research strategy sends its orders directly. For remote mode, the client is yours, or adapted from the reference relay.
- **One broker connector per server.** Several brokers, or several accounts with separate limits, means running several servers or a connector that handles them itself.
- **Order ids and cancel ownership are your rules.** Make ids unique across strategies, and have your gate decide which strategy may cancel which order.
- **No allocation across accounts.** Splitting one order's fills between accounts is not part of it.
- **Linux on x86-64 only,** with glibc 2.39 or later.

$7,200 per seat per year, paid once, with no automatic renewal. The $900 30-day trial includes the full kit, so you can connect several strategies through the reference relay to a simulated venue and run `server-bench` on your own machine before buying a licence. See [pricing](/pricing.html).
