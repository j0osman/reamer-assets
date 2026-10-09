---
title: Which languages can I write strategies in?
description: For Reamer Research, Python or C++ through the code that ships in the kit, or any language that can call a C library. For Reamer Server, the gate and the broker connector are written in C, C++, Rust or Go, and strategies connect over a socket from any language, Python included.
stage: 4
order: 27
product: both
next: supported-platforms, performance, integration-time, research-to-server-move
date: 2026-10-02
---

For Reamer Research, Python or C++ through the code that ships in the kit, or any language that can call a C library. For Reamer Server, the gate and the broker connector are written in C, C++, Rust or Go, and strategies connect over a socket from any language, Python included.

Both products are compiled libraries with a stable C interface. The C interface is the supported contract; the language code in the kits ships in source, ready to use or adapt.

## Reamer Research

- **Python.** A pure-Python binding using `ctypes` and numpy, for Python 3.8 or later, with no build step. You write an `on_bar` method that sees each instrument's recent bars as numpy arrays, its positions and resting orders, and returns orders and order actions. Templates and example strategies are included.
- **C++.** A C++20 wrapper around the C interface, with example strategies.
- **Anything else that can call C,** such as Rust, Go, Julia or C#, against the header directly.

Two things to know about Python:

- **It is slower per bar.** In the kit's benchmark, a strategy cost about 74 µs per bar from Python against about 2.2 µs from C++. An 884,130-bar backtest took about 65 seconds.
- **The binding covers less than the C interface.** It applies one cost setting to every instrument in a run, and passes no non-price data such as earnings dates. For those, call the C interface's newest entry point from C++ or extend the binding.

See [Is there a backtesting engine I can call from Python that runs locally and never uploads my code?](/faq/local-python-backtesting-engine-no-upload.html)

## Reamer Server

- **The gate and the connector** are compiled into your server program, which links the core library. The kit has starting points in C++ and Rust (an accept-all gate with a paper broker) and a worked integration in Go, including a FIX 4.4 session.
- **Strategies are separate processes,** so they can be in any language. They connect over a local Unix socket, with a byte-for-byte specified protocol, or through a relay. The kit's reference relay accepts one JSON message per line over TCP, so a Python strategy needs only the standard library. See [How do I run several strategies through one broker connection?](/faq/multiple-strategies-one-broker-connection.html)

A strategy researched in Python can stay in Python when it goes live; the change is from a backtest loop to a live event loop. See [How do I take a strategy from backtest to live trading without a rewrite?](/faq/backtest-to-live-without-rewrite.html)
