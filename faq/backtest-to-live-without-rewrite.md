---
title: How do I take a strategy from backtest to live trading without a rewrite?
description: Keep the decision logic, and port everything around it. Write the strategy so its signals depend only on state it updates one bar at a time, keep instrument names and costs out of it, then prove the live version matches by replaying the backtest's bars through it and comparing every signal. What changes is the plumbing, not the strategy, but the plumbing is real work.
stage: 3
order: 18
product: both
next: research-to-server-move, independent-quant-research-to-live-stack, backtest-vs-live-results, where-pre-trade-risk-checks-belong, paper-trading
date: 2026-10-01
---

Keep the decision logic, and port everything around it. Write the strategy so its signals depend only on state it updates one bar at a time, keep instrument names and costs out of it, then prove the live version matches by replaying the backtest's bars through it and comparing every signal. What changes is the plumbing, not the strategy, but the plumbing is real work.

"Without a rewrite" is achievable for the logic. It is not achievable for the code around it, because a backtest and a live system are built differently. A backtest calls your strategy with history and takes its orders back. A live system gets one update at a time and sends orders to a broker that may refuse them, fill them partly, or fill them late.

## What carries over, and what does not

**Carries over:** entry and exit signals, position sizing, stop and target levels, and your risk limits. These are the strategy. If any of them changes in the move, the live system is trading something you did not test.

**Does not carry over:**

- **How indicators are computed.** A backtest can read a window of past bars on every step. Live, nothing hands you a window.
- **How instruments are named.** Research tools often use an index into a list; brokers use symbols.
- **The cost model.** Backtest spread, slippage and commission are a simulation. Live costs are whatever the broker charges.
- **Configuration and data formats.** The research configuration configures nothing live, and the live feed is rarely in the backtest's file format.
- **How orders leave.** In a backtest, orders go back to the engine. Live, they go through a pre-trade check to a broker, and the answer comes back later.

The causes behind each are covered in [Why don't my backtest results match live trading?](/faq/backtest-vs-live-results.html) This page is about how to do the move.

## 1. Write the strategy so it can move

Most of the porting work can be avoided while the strategy is still in research:

- **Separate the decision from the data.** Put the logic in one function or class that takes the latest bar and the current position and returns what to do. Keep loading data, sending orders and logging outside it.
- **Keep indicators as running state, even in the backtest.** Instead of recomputing an average from a window on every bar, update it from the newest bar and keep it between calls. A running average, an exponential average, a running true range for an ATR: each needs only the previous value and the new bar. Code written this way runs unchanged in both places.
- **Plan the warm-up.** A running indicator is not valid until it has seen enough bars. In the backtest, skip signals until then. Live, seed it from recent history before the first trading bar, using the same bars the backtest would have used.
- **Name instruments by symbol.** Use the broker's symbol inside the strategy, not a position in a list.
- **Keep costs out of the strategy.** The strategy decides; the engine or broker charges. If the logic depends on a cost, such as skipping trades whose expected edge is below the spread, pass it in as a parameter.

## 2. Prove the port with a parity test

The single most useful check: run the live code on the historical bars and compare its decisions with the backtest's.

1. Take a period the backtest covered, with the same bars, in order.
2. Feed them one at a time through the live strategy code, with orders captured instead of sent.
3. Compare every signal and order, bar by bar, with the backtest's order log.
4. Find the first difference. It is almost always warm-up, an off-by-one in when an indicator updates, or a symbol mapped wrongly.

Repeat it whenever the strategy changes. This only works if the backtest repeats exactly, so its order log is a fixed reference to compare against. See [What makes a backtest deterministic, and why does it matter?](/faq/what-makes-a-backtest-deterministic.html)

## 3. Build the live plumbing once

The rest is infrastructure, built once and shared by every strategy:

- **One path for every order,** with a sequence and a pre-trade check between the strategies and the broker. See [Where should pre-trade risk checks sit in a trading system?](/faq/where-pre-trade-risk-checks-belong.html)
- **A broker connection** that handles disconnects, rejections and partial fills.
- **An instrument table** that maps your symbols to the broker's, kept in one place and checked.
- **A record** of every order and decision, so a live result can be traced as a backtest can.

## 4. Go live in steps

1. **Paper.** Run the whole live path against a paper account or simulated venue. Confirm orders reach it and fills come back.
2. **Compare costs.** After enough fills, compare live spread and slippage with what the backtest assumed, and update the backtest's model, not the strategy.
3. **Small size.** Start with the smallest position the broker allows. A size that cannot hurt is the last test of the plumbing.
4. **Then scale,** watching whether live results stay inside the range the backtest's stress tests suggested.

## How Reamer Research and Reamer Server handle it

[Reamer Research](/products/reamer-research.html) and [Reamer Server](/products/reamer-server.html) are separate products built for this move. Both kits ship the same guide to it, `RESEARCH_TO_SERVER.md`, and what follows is from it:

- **The decision logic does not change.** Entry signals, exit rules, sizing, stop and target levels and risk thresholds carry over. The guide calls the move a port from batch to event-driven code, not a rewrite.
- **The order does not change.** `reamer_relay_send()` in the Reamer Research library takes the same `ReamerOrderRequest` a strategy returns from `on_bar` and sends it to a running Reamer Server over its local strategy socket. Order IDs are numbered as in a backtest, the closing and reducing rules are the same, and `reamer_relay_get_positions()` reads the fills back as the same position structure the backtest passes to `on_bar`.
- **Strategies stay in your own processes.** Reamer Server hosts no strategy. Strategies connect to it over a socket, in any language; a Python strategy can submit a live order with nothing beyond the standard library's `socket` and `json`.
- **Exactly one item touches strategy code: indicator state.** In Reamer Research, `on_bar` receives a lookback window every call. Live, an indicator that needs history is kept as running state. Writing it that way in the backtest, as above, removes even that.
- **Instrument names travel with the order.** `reamer_relay_open()` takes the same `ticker_names[]` array as `reamer_run_backtest()`, and backtest results carry those names, so the backtest's instruments can be checked against your live instrument table as a build step.
- **The plumbing is built once.** The pre-trade check and the broker connector are yours to write, once per deployment, not per strategy. You start from worked examples, not a blank file: a minimal accept-all check and an in-memory paper broker in C++ and Rust, and a full FIX 4.4 integration in Go. `RESEARCH_TO_SERVER.md` runs one research order through the Go reference gate to a simulated fill in five steps.
- **Byte-identical backtests make the parity test exact.** A fixed `rng_seed` gives the same order log every run, so a difference in the parity test is the port, not the engine.

## Limits to know

- **Costs stay in research.** The research configuration, data format and cost settings describe the simulation. Live costs are what your broker charges, and the server reads its own configuration.
- **Exits travel as their own orders.** The live socket has no bracket fields, so the relay rejects an order with a stop or target attached. Send the stop and the target as separate orders.
- **The relay speaks the local socket.** It connects to a server on the same machine. A strategy on another host uses the server's remote mode through a relay process you run.
- **Your venue adapter.** The FIX reference runs against a simulated venue. Making a connection fit for your broker in production, including recovery after sequence gaps and venue certification, is your work.
- **Your market data.** The live feed and the bars it is compared against are yours to source and match.

The $225 Reamer Research trial includes the full execution specification and the kit, so you can write a strategy with running indicators and check it repeats exactly before buying a licence. The $900 Reamer Server trial includes both reference integrations, so you can send that strategy's orders through to a simulated fill.
