---
title: How can I plug my own risk model into every order before it reaches the broker?
description: Turn the model into one function that takes an order and the account's current state and returns pass or reject with a reason, then put that function where every order has to pass it. Keep any state the model needs that the order path does not supply, such as prices, daily counts or losses, in your own code, updated as fills arrive. Reamer Server calls a function you write, the gate, for every new order, cancel and replace.
stage: 3
order: 20
product: server
next: where-pre-trade-risk-checks-belong, self-hosted-order-management-engine, multiple-strategies-one-broker-connection, what-is-reamer-server
date: 2026-10-01
---

Turn the model into one function that takes an order and the account's current state and returns pass or reject with a reason, then put that function where every order has to pass it. Keep any state the model needs that the order path does not supply, such as prices, daily counts or losses, in your own code, updated as fills arrive. [Reamer Server](/products/reamer-server.html) calls a function you write, the gate, for every new order, cancel and replace.

Where that check should sit, and why it should be outside strategy code, is covered in [Where should pre-trade risk checks sit in a trading system?](/faq/where-pre-trade-risk-checks-belong.html) This page is about writing your own model into it.

## From a risk model to a check

A risk model in a spreadsheet or a policy document is a list of limits. A pre-trade check is the same limits written as a function:

- **Input:** the order, and the account's state just before it.
- **Output:** pass or reject, and for a reject, a reason a person can read.
- **Nothing else.** It does not change the order, send anything, or wait for anything.

Write each limit as a separate rule with its own reason, and run them in a fixed order. The first rule that fails decides the reason. A rejection that says `max position AAPL 500 exceeded (would be 650)` can be acted on; one that says `rejected` cannot.

## The three kinds of rule

Most risk models mix three kinds of rule, and they need different data:

- **Rules on the order alone.** Allowed instruments, maximum order size, allowed order types, limit prices inside a band. These need only the order and your own reference data.
- **Rules on the order and current state.** Maximum position per instrument, maximum open orders, total exposure. These need the account's positions and open orders as they are now, which the order path should hand you on every check.
- **Rules on history.** Orders per minute, a daily loss limit, a maximum number of trades per day. These need state built up over time, which no single order or snapshot carries.

The first two kinds are simple once the check is in the right place. The third is where most of the work is.

## State the order path does not give you

A check that is handed the order and a positions snapshot still lacks several things a real model uses:

- **Market prices.** Exposure in money, and the band for a mistyped limit price, both need a recent price. A market order carries no price at all. The check needs its own source, and a rule for what to do when that price is stale: reject, not guess.
- **Counts and rates.** Orders per strategy per minute, or per day. Keep a counter that the check reads and updates.
- **Profit and loss.** A daily loss limit needs fills and prices. Update it from fills as they arrive, not from a separate system queried in the order's path.
- **Your own switches.** A flag that stops new orders, for one strategy or for everything.

Keep this state in the same program as the check, updated by the code that already sees fills and prices. A check that calls another service on every order adds that service's latency and failures to every order.

## Things that go wrong

- **Treating every order the same.** A cancel reduces risk; a rule that rejects orders with no quantity will reject every cancel. A change to a working order can add risk, so check the new quantity and price, not just the fact of a change.
- **A kill switch that blocks cancels.** When you stop trading in a hurry, you usually want to cancel what is open. Stop new orders and increases, and let cancels through.
- **Mixed units.** Quantities as decimals in one place and as integers scaled by a power of ten in another will pass every test with round numbers and fail on the first fractional fill.
- **Failing open.** If a price is missing, a counter is unreadable or the code throws, the order should be rejected.
- **Slow checks.** The check runs in the order's path, once per order. Keep it to reading memory and comparing numbers.

## Test it before it trades

1. **Unit tests per rule.** One passing and one failing case for every limit, plus cancels and changes, plus an order arriving while disconnected.
2. **Replay real order flow.** Run a day of your strategies' orders through the check offline and read every rejection. An unexpected one is a rule written wrongly; an expected one that is missing is worse.
3. **Run it end to end on paper.** Confirm a rejected order never reaches the broker and that the reason appears in your records.
4. **Version it.** Every change to a limit is a change to how you trade. Keep the rules and their limits under version control, and record which version was running when.

## How Reamer Server takes your model

In Reamer Server, the gate is your model. Everything below is documented in `EXTENSION_PROTOCOL.md` in the kit:

- **One function.** `check(intent, venue_state, out_ack)` returns pass or reject and writes a reason string. The server writes the order's id into the answer for you. A second function, `is_available`, tells the server whether your gate can make decisions at all; if it says no, orders are rejected.
- **Every intent, one shape.** New orders, cancels and replaces all arrive as the same intent struct. The kit's table lists which fields each carries: a cancel has quantity and prices at zero and no side or order type, so the kit warns against rejecting on quantity alone.
- **Fresh state on every call.** Immediately before each check, the server reads net positions per instrument, the open order count and the connection state from your broker connector, and hands them to the gate. There is no second copy inside the server to drift from your broker's.
- **No prices, by design.** The gate is not given market prices, fills or profit and loss. State like that lives in your own code. Your gate and your broker connector are linked into the same program, and the server calls every one of their functions in turn on one thread, so the connector can update counters and profit and loss as fills arrive and the gate can read them, with no locks.
- **Units to watch.** Order quantities and prices reach the gate as decimals; positions in the account state are integers scaled by 10⁹. Scale before you compare.
- **Fails closed.** A gate whose code throws an exception is treated as a rejection, logged, and the server keeps running. If the broker connector reports it is disconnected, the order is rejected with `venue disconnected` before your gate is called.
- **Every reason counted.** Your reason text is published on the event stream with each rejection, and the `/metrics` endpoint counts rejections by reason, so a shift in why orders are refused shows up on a dashboard.
- **Cheap to call.** The kit's benchmark measures crossing into your code and back at 53 ns on an AMD EPYC 9575F, before your own logic. The check has no timeout, so a gate that hangs holds up the order.
- **Starting points.** An accept-all gate in C++ and Rust, and a Go reference gate that rejects instruments outside an approved list and orders while disconnected, with tests for its fail-closed behaviour. The kit's end-to-end demo shows one order rejected by that gate before it reaches the simulated venue.

## Limits to know

- **No risk rules ship.** The gate is the place for your model, not a model. Every limit is yours to write and test.
- **No market data.** Price-based rules need a feed you supply.
- **Changing the gate's code needs a restart.** The gate is linked into your program, and server configuration is read only at start. If you want limits you can change during the day, have your own gate read them from a source you control, and treat each change with the same care as a release.
- **256 positions per account snapshot.** The account state carries at most 256 instrument positions; past that, the order is rejected.
- **Linux on x86-64 only,** with glibc 2.39 or later.

$7,200 per seat per year, paid once, with no automatic renewal. The $900 30-day trial includes the full kit, so you can write your own gate, run the reference demo with it, and watch your rejections and their reasons before buying a licence. See [pricing](/pricing.html).
