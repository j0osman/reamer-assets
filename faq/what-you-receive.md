---
title: What exactly do I receive after buying?
description: An email with your licence key and a link to download the kit. The kit is the product itself, compiled libraries and command-line tools with the full documentation and reference code, and it holds no key. You activate the key once per machine with the reamer-license tool, and the term starts then. Nothing is shipped physically and nothing is hosted for you.
stage: 6
order: 49
product: both
next: support, seats, supported-platforms, integration-time, licence-expiry
date: 2026-10-03
---

An email with your licence key and a link to download the kit. The kit is the product itself, compiled libraries and command-line tools with the full documentation and reference code, and it holds no key. You activate the key once per machine with the reamer-license tool, and the term starts then. Nothing is shipped physically and nothing is hosted for you.

## The email

Once payment completes, the key and the kit download link go to the email address used at checkout. For an invoice, that is once the invoice is paid. The key carries the term you bought, 30 days or one year. If it does not arrive, write to support@reamerlabs.com.

## The Reamer Research kit

An archive for Linux x86-64 or macOS on Apple Silicon, containing:

- **The engine,** a shared library, with the C header that is its complete contract.
- **Tools:** the licence activator, a benchmark that measures throughput on your own machine, a converter for exogenous data files and a signature checker for the download.
- **Reference code:** a C++ wrapper with runnable examples, and a Python binding with strategy templates, a quickstart and 100 bars of sample data, so a first backtest runs before you source data.
- **Documents,** including the execution specification, the result schema, deployment and benchmark guides, and the guide to moving a strategy to Reamer Server.

## The Reamer Server kit

A zip for Linux x86-64, containing:

- **The core,** a static library you link into your own program, with its C header. There is no server program to run; you write a small `main()` around it.
- **Tools:** the licence activator, a config checker, a diagnostic bundle for support requests, a benchmark and a signature checker.
- **Reference code:** minimal starting points in C++ and Rust, and a worked FIX 4.4 integration that runs to a fill in one command.
- **Documents,** including the wire protocols, every config field, monitoring and deployment guides.

Both kits include the licence terms, the vendor questionnaire pack, a security hardening report, a software bill of materials and a checksum for every file.

## Getting started

1. Verify the download with the included signature checker.
2. Run `bin/reamer-license activate` with your key and accept the licence terms.
3. Run the bundled benchmark to see a first number on your own hardware.

See [How long does integration take?](/faq/integration-time.html) and [What support is included?](/faq/support.html)
