---
title: What does professional-grade trading infrastructure actually include?
description: Less hardware than people expect and more discipline. Research that gives the same result every run, fill and cost rules written down, risk checks kept separate from strategy code, one ordered path for every order, a record of every decision, live monitoring, safe restarts, and performance claims backed by published data. None of it requires a large team or a data centre.
stage: 1
order: 7
product: both
next: institutional-grade-infrastructure-for-independent-quants, what-is-an-order-management-engine, where-pre-trade-risk-checks-belong, what-makes-a-backtest-deterministic
date: 2026-09-29
---

"Institutional-grade" is often used to mean expensive: colocated servers, a team of engineers, data contracts with long names. For strategies that trade on bars, from minutes to days, most of that is not what separates professional infrastructure from a collection of scripts. What separates them is a set of properties, and each can be checked.

## In research

### 1. The same inputs give the same result

A backtest run twice with the same strategy, data and settings should produce identical output. Without that, no comparison between two results can be trusted, because a difference might come from the run rather than the change you made. See [Why do I get different backtest results each time I run it?](/faq/backtest-results-change-between-runs.html)

### 2. Fill and cost rules are written down

How a stop fills through a gap, which price a buy pays, how commission is charged, which of two bracket exits wins inside one bar. Professional tools state these rules in a document you can read and check against the results, rather than leaving them to whatever the code happens to do. See [What Is an Execution Specification?](/notes/what-is-an-execution-specification.html)

### 3. Every study uses the same engine

One set of fill, cost and metric rules for every study, so results from different studies are measured with the same ruler. See [How do I manage many research scripts without results drifting apart?](/faq/research-scripts-drifting-apart.html)

## In live trading

### 4. Risk checks sit outside strategy code

A strategy decides what it wants to trade. A separate check decides whether each order is allowed: position limits, exposure, order size, trading hours. Keeping the two apart means a bug in one strategy cannot switch off the limits, and one set of rules covers every strategy.

### 5. Every order takes one ordered path

When several strategies trade through one account, their orders need to pass through one place, in a known order, with each order's state tracked from submission to fill or rejection. Without that, two strategies can each believe they are within limits while together they are not.

### 6. Every decision is recorded

For every order: what was asked for, whether it was accepted or rejected, why, and what the broker did. When something goes wrong, the record answers what happened without anyone reconstructing it from memory. The record has to be written somewhere that survives a restart.

### 7. The system can be watched

Order counts, rejection counts and their reasons, connection state, and how quickly the system is keeping up, visible while it runs, with alerts when something stops. A problem noticed in a minute costs far less than one found at the end of the day.

### 8. A restart is safe

Processes crash and machines reboot. After a restart, the live system should know its real positions and open orders. The safest design takes them from the broker, which is the one true record, rather than trusting its own saved copy, which can be out of date.

## In the software itself

### 9. Performance claims come with evidence

Latency and throughput figures should come with the hardware, the method and the raw output behind them, and with a way to measure on your own machine. A number without its conditions is a claim, not evidence.

### 10. You can check what you installed, and it will not break under you

Releases that can be verified before they are run, interfaces that stay compatible across upgrades, and a plan for what happens if the vendor stops.

## What it does not require

- **A large team.** The properties above are design choices. Once built in, they do not need people to maintain them day to day.
- **A data centre.** For mid-frequency strategies, one well-configured machine is usually enough. See [Can one laptop run serious quant research and live trading?](/faq/can-a-laptop-run-a-trading-desk.html)
- **Microsecond networking.** That matters for high-frequency trading, not for strategies that hold positions for minutes or days.

## How Reamer Research and Reamer Server cover the list

Each point below is documented in the kits:

- **1 to 3: [Reamer Research](/products/reamer-research.html).** With a fixed seed, runs are byte-identical. Every fill, cost and accounting rule is written in `EXECUTION_SPEC.md`. Every study runs through the same engine and writes the same versioned result format.
- **4 and 5: [Reamer Server](/products/reamer-server.html).** Strategies connect as separate processes, and every order from every strategy passes through one sequenced pipeline and one gate before it reaches the broker. The gate is yours: you write the risk rules, and the server calls them on every order.
- **6 and 7.** Every order's outcome, including the gate's own reason for a rejection, is published on a shared event stream. A Prometheus-format metrics endpoint reports submitted, accepted and rejected counts, rejection reasons, connection state and loop latency. By default the server refuses to start rather than run without its event stream. The event stream is held in memory, so writing it to durable storage is part of your deployment.
- **8.** The server keeps no state of its own between runs. On start, orders and positions are rebuilt from your broker connector.
- **9.** Both kits ship a `BENCHMARK.md` with the hardware and method, the raw benchmark output is published, and the method is written up in a whitepaper with a DOI. See [About](/about.html).
- **10.** Every release is published with checksums and signatures to verify before installing. In Reamer Research, interface versions are additive from ABI 7 on: a program built against ABI 7 runs unmodified against a newer library. Each kit includes a vendor questionnaire pack covering support, security and business continuity, and a source escrow is available so a single-maintainer vendor does not become a single point of failure.

Your own work stays yours: the risk rules in the gate, the broker connector, durable storage for the record, and alerting on the metrics. The products provide the structure those fit into.
