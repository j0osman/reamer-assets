---
title: How do Reamer Research and Reamer Server compare with EPAM Deltix QuantOffice?
description: Deltix QuantOffice is an enterprise trading platform from EPAM that covers research, backtesting, paper trading and live execution on one code base, with the TimeBase data store and over 100 exchange and broker connectors, sold under a commercial licence to institutions. Reamer Research and Reamer Server cover the same span, research through live trading, as two smaller libraries that one person or a small team can buy self-serve and run without an integration project.
stage: 4
order: 35.5
product: both
next: what-is-reamer-research, what-is-reamer-server, integration-time, who-its-for, pricing
date: 2026-10-08
---

Deltix QuantOffice is an enterprise trading platform from EPAM. It covers research, backtesting, paper trading and live execution on one code base, with its own data store and over 100 exchange and broker connectors. Reamer Research and Reamer Server cover the same span, from research to live trading. The difference is scale and the cost of getting started: they are two smaller libraries that one person or a small team can buy self-serve and run without an integration project.

## What Deltix is

- **A product family.** QuantOffice for strategy research and trading, TimeBase for market data, and execution and market-making products alongside them, all sold by EPAM.
- **One code set from research to production.** The same strategy runs in backtest, simulated trading and live trading.
- **TimeBase.** A time-series database and streaming server for market data and order events. Its community edition is open source (Apache 2.0) under FINOS.
- **Connectivity.** Over 100 connectors to exchanges and brokers, across asset classes.
- **Deployment.** Windows or Linux, on your own premises or in the cloud.
- **Built for institutions.** It suits firms with the engineering staff to integrate, configure and run a large platform. It has no public price list.

## How Reamer Labs differs

- **Two libraries, not a platform.** [Reamer Research](/products/reamer-research.html) backtests and [Reamer Server](/products/reamer-server.html) takes a strategy live. Each has a stable C interface, and you write a short program around it.
- **Bought and running in an afternoon.** Self-serve checkout, with the key and kit by email within minutes. On uncut recordings, a working integration took about 13 minutes for Reamer Research and 19 for Reamer Server. See [How long does integration take?](/faq/integration-time.html)
- **Narrower scope.** Systematic strategies on OHLCV bars, held from minutes to days. No options, no high-frequency or order-book strategies, no bundled data store or market data.
- **You supply the edges.** Reamer Server runs your own pre-trade gate on every order and sends accepted orders through a broker connector you write. The kit has a worked FIX 4.4 example. Deltix ships the connectors.
- **Moving to live is a port.** Your trading logic carries over, but you rewrite the code around it once. See [What does it take to move from Reamer Research to Reamer Server?](/faq/research-to-server-move.html)
- **Published prices.** Reamer Research costs $1,800 per seat per year and Reamer Server $7,200, each with a 30-day paid trial.

## Which to choose

- **Deltix** if you are an institution that needs many venues and asset classes, a managed market-data store, low-latency execution, and a vendor that can support a large deployment.
- **Reamer Labs** if you are an individual quant or a small firm trading systematic strategies on bars, already have data and a broker, and want institutional-grade research and order management without the cost of integrating an enterprise platform. See [Who are Reamer Research and Reamer Server for?](/faq/who-its-for.html)

See [pricing](/pricing.html).
