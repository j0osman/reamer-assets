---
title: Is there institutional-grade trading infrastructure an independent quant can afford?
description: Yes, for systematic strategies on bars. What makes infrastructure institutional is a set of properties you can check, not a budget. Reamer Labs sells both halves at public prices, per seat per year, with no sales call. Reamer Research, the research engine, is $1,800. Reamer Server, the order management engine, is $7,200. Each has a paid 30-day trial.
stage: 3
order: 15
product: both
next: what-is-reamer-research, what-is-reamer-server, who-its-for, research-engine-for-mid-frequency-strategies, self-hosted-order-management-engine
date: 2026-10-01
---

Yes, for systematic strategies that trade on bars. What makes infrastructure institutional is a set of properties you can check, not a budget: results that repeat exactly, fill and cost rules written down, risk checks separate from strategy code, one ordered path for every order, a full record, and performance claims backed by published data. Reamer Labs sells both halves at public prices, per seat per year: Reamer Research for research at $1,800 and Reamer Server for live order management at $7,200.

The harder part of the question is telling a real claim from a label. "Institutional-grade" costs nothing to write on a web page.

## What the word should mean

The full list is in [What does professional-grade trading infrastructure actually include?](/faq/what-is-institutional-grade-trading-infrastructure.html) In short:

- **In research:** the same inputs and seed give the same output, every fill and cost follows written rules, and every study runs on the same engine.
- **In live trading:** orders from every strategy go through one sequence and one pre-trade check before the broker, and every decision is recorded and monitored.
- **In the software:** published benchmarks with the method and raw data, verifiable releases, and stated limits.

None of these needs colocated hardware or a team. They need software built to those standards.

## Where the money usually goes

An independent quant usually meets three options, each with a cost that is not always the price:

- **Build it yourself.** No licence, but months of engineering before the first trustworthy result, and the maintenance after. The real cost is the time not spent on strategies. See [Should I build my own backtester or buy one?](/faq/build-or-buy-backtester.html)
- **Enterprise vendors.** Built for funds and banks. Prices are usually on request, contracts are annual with minimums, and the sales process can take longer than a trial should.
- **Hosted platforms.** Cheap or free to start, but your strategy runs on someone else's machine, with that platform's execution model and its data. Leaving means rewriting.

What an independent quant needs is the property list from the enterprise option at a price one person can pay, running on their own machine.

## How to check an "institutional" claim

Before paying for any product sold with the word, look for evidence you can check yourself:

1. **A specification you can read.** The exact rules for fills, costs and order handling, not a feature list.
2. **Benchmarks with the hardware, method and raw output,** and a way to rerun them on your own machine.
3. **Repeatability you can test.** Run the same thing twice and compare the output byte for byte.
4. **Verifiable releases.** Checksums and signatures to check before installing.
5. **Security and procurement answers on paper.** A security record and a standing answer to the questions a fund's due diligence would ask.
6. **Stated limits.** What the product does not do, said plainly. A vendor that claims to do everything has not tested much.
7. **A public price.** If the price needs a call, the product was not priced for you.

## What Reamer Labs offers

**[Reamer Research](/products/reamer-research.html): $1,800 per seat per year, 30-day trial $225.** A backtesting engine delivered as a library with a stable C interface, driven from Python, C++ or any language that can call C:

- Byte-identical output for a fixed seed, randomised spread and slippage included; in the published test, 20 of 20 runs matched by SHA-256.
- A written execution specification for fills, costs, margin, order lifetimes, swap and futures rolls, shipped with the kit.
- About 1.72 million bars a second per backtest on an AMD EPYC 9575F, with speed varying by 0.59% across 2,000 runs.

**[Reamer Server](/products/reamer-server.html): $7,200 per seat per year, 30-day trial $900.** An order management engine you link into your own program. Strategies connect over a socket; every order from every strategy goes through one sequence, then your own pre-trade check, then your broker connector:

- 53 ns per order to cross into your own code and back, and 11.3 µs at P50 and 12.4 µs at P99 end to end, on an AMD EPYC 9575F.
- A peak of 219,652 orders a second, and no correctness failure at any point up to 512 concurrent strategies.
- Every decision on an event stream, and counts, rejections and latency on a metrics endpoint.

**For both:**

- **One seat is one machine.** Any number of processes and cores on that machine run under one seat.
- **Your code stays on your machine.** After activation they run offline; only activation and deactivation contact the licence server.
- **Evidence.** The benchmark method is in a whitepaper with a DOI, and the raw output is published. See [Reamer Research findings](/blog/reamer-research-findings-epyc.html) and [Reamer Server findings](/blog/reamer-server-findings-epyc.html).
- **Verifiable releases.** Checksums and a signature for every release, and a software bill of materials for every third-party component.
- **Procurement papers.** Each kit includes a vendor questionnaire pack covering security, support, data handling and business continuity, and a record of the security review already done.

Research alone is $1,800 a year. Both products are $9,000 a year for one person on one machine. Prices are on the [pricing page](/pricing.html), checkout is by card, and the key and kit arrive by email within minutes.

## What you still own

- **Data.** Neither product includes market data.
- **Your risk rules and broker connection.** Reamer Server is the core of an order management system, not the whole of one. You write the pre-trade check and the broker connector. Starting points ship in the kit, including a worked FIX 4.4 integration in Go.
- **Durable storage and alerting.** The server publishes the record and the metrics; storing and alerting on them is part of your deployment.
- **The port from research to live.** Strategy logic carries over from Reamer Research to Reamer Server; the code is adapted, not copied.

## The trade-offs of a small vendor

The price is low partly because Reamer Labs is one person. That has consequences you should weigh:

- **Support is best-effort.** Email to one person on weekdays, with a target of acknowledging within two business days, and no contractual service level. Issues that stop trading come first.
- **Continuity.** There is no standing source escrow. A customer can set one up at their own cost, released only if Reamer Labs closes or ends the product. `LICENSING.md` in each kit gives the terms.
- **Closed source.** You link the library and read the specification; you do not get the engine's source.

If those are acceptable, the trial is the test: run your own strategy on your own data and check each claim above before buying a year.
