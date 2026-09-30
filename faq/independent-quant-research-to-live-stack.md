---
title: What software do independent quants use to go from research to live trading?
description: A small stack of five layers: market data, a research engine, an execution pipeline that holds order state and runs pre-trade risk checks, a broker connection, and monitoring with a durable record. Few people buy all five from one place. The usual pattern is to buy the data and the brokerage, buy or adopt the engines, and write the strategy, the risk rules and the glue between the layers yourself.
stage: 2
order: 11
product: both
next: what-is-an-order-management-engine, where-pre-trade-risk-checks-belong, research-engine-for-mid-frequency-strategies, backtest-to-live-without-rewrite
date: 2026-09-30
---

Most independent quants who trade systematically end up with the same five layers: market data, a research engine, an execution pipeline, a broker connection, and monitoring with a record of what happened. The tools differ from person to person. The layers do not, because each one does a job the others cannot.

What matters as much as the tools is which layers you own. Some are best bought, some can be adopted, and a few have to be yours whatever you buy.

## The five layers

### 1. Market data

History for research and a live feed for trading, from a data vendor, an exchange or the broker.

- **Clean and adjust it before research.** Splits, dividends, missing bars, bad prints and time zones all have to be handled before the data reaches an engine. Most engines check the shape of the data, not whether it is right.
- **Build live bars the same way as historical ones.** A strategy researched on 15-minute bars built one way and traded on bars built another way is trading a different strategy. The session times, the time zone and the rule for when a bar closes should match.
- **Keep a fixed copy of every dataset a study used,** so the study can be rerun.

### 2. A research engine

The program that tests, diagnoses, sweeps and reports on a strategy against history, with one set of fill, cost and metric rules for every study. It turns ideas into strategies worth trading. See [What is a quant research engine, and how is it different from a backtesting script?](/faq/research-engine-vs-backtest-script.html)

### 3. An execution pipeline

Everything between a strategy's decision and the broker:

- **Strategy processes.** The live version of each strategy. It reads live data, keeps its indicators up to date bar by bar, and decides what it wants to trade.
- **Order management.** One place that holds the state of every order, from sent to filled, cancelled or rejected, and puts orders from every strategy into one sequence.
- **Pre-trade risk checks.** A separate step that approves or rejects each order before it leaves: position limits, order size, exposure, trading hours. Kept apart from strategy code, so a bug in one strategy cannot switch off the limits.

With one strategy and small size, this can be a few hundred lines. With several strategies on one account, it is the layer that stops two strategies from each believing they are within limits while together they are not.

### 4. A broker connection

The session with the broker: logging in, sending orders, receiving fills and rejections, reconnecting after a drop, and reporting the account's real positions. It is reached through the broker's own programming interface or through FIX, the standard message protocol most brokers and venues support.

The broker is the one true record of your positions. After a crash or restart, the safest system takes its positions and open orders from the broker rather than trusting its own saved copy.

The broker's own risk checks protect the broker. They are not a substitute for yours.

### 5. Monitoring and records

- **Live metrics and alerts:** orders sent, accepted and rejected, the reasons for rejections, connection state, and whether the system is keeping up. An alert when any of them stops.
- **A durable record** of every order, decision and fill, written to storage that survives a restart, so any question about what happened has an answer.

## The gap between research and live

The move from layer 2 to layer 3 is where most of the unplanned work sits. Four things rarely carry over as they are:

1. **Indicators.** In research, a strategy can look back over a window of past bars on every step. Live, it gets one new bar at a time, so any indicator that needs history (a moving average, an ATR) has to be kept as a running value.
2. **Instrument names.** Research tools often refer to instruments by position in a list. Brokers use symbols. The mapping between them has to be built and checked, because an index that quietly points to a different symbol is the worst bug this move can produce.
3. **Configuration and costs.** Backtest cost settings are a simulation. Live, costs are whatever the broker and the market actually charge. The research configuration does not configure anything live.
4. **Data.** The research data format and the live feed are usually different, and the bars have to match (layer 1).

The strategy's decision logic, its signals, entries, exits and sizing, should carry over unchanged. If it has to be rewritten, the research and live systems are testing different things. See [Why don't my backtest results match live trading?](/faq/backtest-vs-live-results.html)

Run the whole stack against a paper account before real money, and compare its fills and costs with what the research assumed.

## Which layers you own

- **Always yours:** the strategies, the risk rules, the choice of data and broker, and the glue between the layers. Nobody can sell you these, because they are your edge and your risk tolerance.
- **Usually bought:** market data and brokerage.
- **Bought, adopted or built:** the research engine and the core of the execution pipeline. These have to be right but are rarely an edge, which is the case for not building them. See [Should I build my own backtester or buy one?](/faq/build-or-buy-backtester.html)
- **Yours to run:** monitoring, alerting and storage for the record, using whatever tools you already use.

## Three common shapes

1. **An all-in-one platform from a broker or vendor.** Quickest to start. Research and live trading are usually tied to that one platform, and its fill and cost rules may not be written down.
2. **Scripts and a broker's programming interface.** Cheapest, and fine for one strategy. It tends to drift as strategies are added, and risk checks often end up inside strategy code.
3. **Separate components.** A research engine, an execution core, and your own strategies, risk rules and broker connection around them. More assembly, and each layer can be checked and replaced on its own.

For bar strategies that hold positions for minutes or days, all five layers usually fit on one well-configured machine. See [Can one laptop run serious quant research and live trading?](/faq/can-a-laptop-run-a-trading-desk.html)

## Where Reamer Research and Reamer Server fit

Reamer Labs makes the third shape's two engines. Each is a precompiled library with a stable C interface that runs on your own machine. Everything below is documented in the kits:

- **Layer 2: [Reamer Research](/products/reamer-research.html).** A research engine for mid-frequency strategies on OHLCV bars: every fill and cost rule written in `EXECUTION_SPEC.md`, byte-identical output for a fixed seed, and a versioned JSON result for every run. Strategies are written in Python or C++ through the reference code, or in any language that can call C. It runs on Linux x86-64 and macOS on Apple silicon.
- **Layer 3: [Reamer Server](/products/reamer-server.html).** The core of the execution pipeline. It holds every order's state, sequences orders from every strategy through one path, and calls your pre-trade check on every order before it reaches your broker. Strategies connect as separate processes over a local socket or a remote relay, in any language. On a 64-core EPYC machine it measured 11.3 µs at the median from strategy to a local acceptor, before the broker's own round trip. It runs on Linux x86-64.
- **The gap between them.** `RESEARCH_TO_SERVER.md` ships in both kits and lists what carries over and what does not: the four points above, in more detail.

What you build, and what the kits give you to start from:

- **The risk check.** It is yours: one pass or reject per order, given the order and the account's last known state.
- **The broker connection.** The Server kit includes a worked reference in Go with a FIX 4.4 session, a broker connector and a strategy relay, driven from strategy to fill in one command, and an in-memory paper broker to confirm your wiring before you connect to anything real. Adapting it to your own broker is your work.
- **Monitoring and records.** Every order event is published on a shared-memory event stream, and a Prometheus-format `/metrics` endpoint reports order counts, rejection reasons and connection state. The server keeps no state of its own between runs: on start, positions and open orders come from your broker connection. Durable storage for the event stream and your alerting are yours to set up.

Neither product supplies market data or brokerage. You bring your own data, historical and live, and your own broker account.
