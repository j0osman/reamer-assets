---
title: What exactly do I receive after buying?
description: An email with your licence key and a link to download the kit. The kit is the product itself, compiled libraries, ready-to-run Python, C++ and Go code in source, command-line tools and the full documentation, and it holds no key. You activate the key once per machine with the reamer-license tool, and the term starts then. Nothing is shipped physically and nothing is hosted for you.
stage: 6
order: 49
product: both
next: support, seats, supported-platforms, integration-time, licence-expiry
date: 2026-10-03
topics: sdk, software-license
---

An email with your licence key and a link to download the kit. The kit is the product itself, compiled libraries, ready-to-run Python, C++ and Go code in source, command-line tools and the full documentation, and it holds no key. You activate the key once per machine with the reamer-license tool, and the term starts then. Nothing is shipped physically and nothing is hosted for you.

## The email

Once payment completes, the key and the kit download link go to the email address used at checkout. For an invoice, that is once the invoice is paid. The key carries the term you paid for: a month for Reamer Research, extended each time the subscription renews, or 30 days or one year for Reamer Server. If it does not arrive, write to support@reamerlabs.com.

## The Reamer Research kit

An archive for Linux x86-64 or macOS on Apple Silicon (on Windows, the Linux archive runs through WSL2), containing:

- **The engine,** a shared library, with the C header that is its complete contract.
- **Ready-to-run code, in source:** a Python package you `pip install` (helpers for every order type, bracket, cancel, partial close and modify), a one-header C++ wrapper, an offline trade viewer, 6 strategy templates, 17 Python and 5 C++ examples, Alpaca and Databento fetchers that pull historical bars from your own account into a backtest, and a one-command quickstart with 100 bars of sample data, so a first backtest runs before you source data.
- **Tools:** the licence activator, a converter from your CSV bars to the engine's memory-mapped files, a converter for exogenous data files, a benchmark that measures throughput on your own machine and a signature checker for the download.
- **Documents,** including the execution specification, the result schema, deployment and benchmark guides, and the guide to moving a strategy to Reamer Server.

## The Reamer Server kit

A zip for Linux x86-64, containing:

- **The core,** a static library you link into your own program, with its C header. There is no server program to run; you write a small `main()` around it.
- **Ready-to-run code, in source:** a worked FIX 4.4 integration in Go (session, broker connector, gate, strategy relay and event-bus reader, with a 100-test suite) that runs to a fill in one command, and starting points in C++ and Rust.
- **Tools:** the licence activator, a config checker, a diagnostic bundle for support requests, a benchmark and a signature checker.
- **Documents,** including the wire protocols, every config field, monitoring and deployment guides.

Both kits include the licence terms, the vendor questionnaire pack, a security hardening report, a software bill of materials and a checksum for every file.

## Getting started

1. Verify the download with the included signature checker.
2. Run `bin/reamer-license activate` with your key and accept the licence terms.
3. Run the bundled benchmark to see a first number on your own hardware.

See [How long does integration take?](/faq/integration-time.html) and [What support is included?](/faq/support.html)
