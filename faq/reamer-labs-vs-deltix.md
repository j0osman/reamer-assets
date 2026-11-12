---
title: How do Reamer Research and Reamer Server compare with EPAM Deltix QuantOffice?
description: Deltix QuantOffice is an enterprise trading platform from EPAM that covers research, backtesting, paper trading and live execution on one code base, with the TimeBase data store and over 100 exchange and broker connectors, sold under a commercial licence to institutions. Reamer Research and Reamer Server cover the same span, research through live trading, as two smaller libraries that one person or a small team can buy self-serve and integrate themselves.
stage: 4
order: 35.5
product: both
next: deltix-alternatives, what-is-reamer-research, what-is-reamer-server, integration-time, who-its-for, pricing
date: 2026-10-08
topics: quantitative-analysis, backtesting, oms
---

Deltix QuantOffice is an enterprise trading platform from EPAM. It covers research, backtesting, paper trading and live execution on one code base, with its own data store and over 100 exchange and broker connectors. Reamer Research and Reamer Server cover the same span, from research to live trading. The difference is scale and the cost of getting started: they are two smaller libraries that one person or a small team can buy self-serve and integrate themselves.

## What Deltix is

- **A product family.** QuantOffice for strategy research and trading, TimeBase for market data, and execution and market-making products alongside them, all sold by EPAM.
- **One code set from research to production.** The same strategy runs in backtest, simulated trading and live trading.
- **TimeBase.** A time-series database and streaming server for market data and order events. Its community edition is open source (Apache 2.0) under FINOS.
- **Connectivity.** Over 100 connectors to exchanges and brokers, across asset classes.
- **Deployment.** Windows or Linux, on your own premises or in the cloud.
- **Built for institutions.** It suits firms with the engineering staff to integrate, configure and run a large platform. It has no public price list.

## How Reamer Labs differs

- **Two libraries, not a platform.** [Reamer Research](/products/reamer-research.html) is the research engine and [Reamer Server](/products/reamer-server.html) takes a strategy live. Each is a library you call from your own program: Reamer Research as a Python package or one C++ header, both through a stable C interface.
- **Self-serve start.** Checkout is online, with the key and kit by email within minutes. A first Reamer Research run takes a couple of minutes with the Python package or the C++ header; on an uncut recording, a working Reamer Server integration took about 19. See [How long does integration take?](/faq/integration-time.html)
- **Built for bars.** Systematic strategies on time, tick, volume, dollar or custom bars, held from minutes to days. No options, high-frequency or order-book strategies, and you bring your own data.
- **You supply the edges.** Reamer Server runs your own pre-trade gate on every order and sends accepted orders through a broker connector you write. The kit has a worked FIX 4.4 example. Deltix ships the connectors.
- **Research orders go live through a relay.** `reamer_relay_*` in Reamer Research sends the orders a strategy returns in a backtest to Reamer Server or to a receiver you write. Indicators move from a bar window to running state. See [What does it take to move from Reamer Research to Reamer Server?](/faq/research-to-server-move.html)
- **Published prices.** Reamer Research costs $20 per seat per month, cancel any time. Reamer Server costs $24,000 per seat per year, with a $2,000 30-day trial.

## Which to choose

- **Deltix** if you are an institution that needs many venues and asset classes, a managed market-data store, low-latency execution, and a vendor that can support a large deployment.
- **Reamer Labs** if you are an individual quant or a small firm trading systematic strategies on bars, already have data and a broker, and want research and order management without the cost of integrating an enterprise platform. See [Who are Reamer Research and Reamer Server for?](/faq/who-its-for.html)

See [pricing](/pricing.html).
