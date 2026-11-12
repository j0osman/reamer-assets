---
title: Is there a backtesting engine I can call from Python that runs locally and never uploads my code?
description: Yes. Look for an engine delivered as a library you run on your own machine, with a licence that is checked offline, and confirm it by running it with the network cut off. Reamer Research is one: a compiled engine in a Python package whose Python code ships in source, which runs without a connection after activation and sends no strategy code, data or results to Reamer Labs.
stage: 3
order: 17
product: research
next: what-is-reamer-research, research-engine-for-mid-frequency-strategies, deterministic-backtesting-with-slippage-and-spread, research-engine-vs-backtest-script
date: 2026-10-01
topics: backtesting, abi
---

Yes. Look for an engine delivered as a library you run on your own machine, with a licence that is checked offline, and confirm it by running it with the network cut off. [Reamer Research](/products/reamer-research.html) is one: a compiled engine in a Python package whose Python code ships in source, which runs without a connection after activation and sends no strategy code, data or results to Reamer Labs.

"Local" is easy to claim, so the useful part of the answer is how to check it.

## Why it matters where your code runs

A strategy's code, parameters and backtest results are the edge itself. Anything that leaves your machine is outside your control: it can be stored, logged, read by staff, exposed in a breach, or kept after you leave. Even without bad intent, you cannot audit what you cannot see.

There are also practical reasons. A firm's compliance may forbid sending strategy logic to a third party. A tool that needs a connection stops when the connection does. And research that depends on someone else's servers depends on their prices and their continued existence.

## The three kinds of tool

- **Hosted platforms.** You write the strategy in a browser or upload it, and it runs on the vendor's machines, usually with the vendor's data. Convenient, but your code lives on their servers by design.
- **Open-source libraries.** Run locally and you can read every line, so you can check what they send. You also take on their fill model, their gaps and their maintenance.
- **Commercial libraries.** Run locally, with a written specification and support, but closed source. Here the questions below matter most, because you cannot read the code to check.

A desktop application can fall in either camp. Running on your machine is not the same as keeping everything on it; some send telemetry, crash reports or results to the vendor.

## What to ask a commercial vendor

1. **What does the licence check send, and when?** The best answer is a one-time activation, then nothing during normal use.
2. **Is there telemetry or crash reporting, and can it be turned off?** Ideally there is none.
3. **What data is collected at all?** Ask for it in writing, with what is explicitly not collected.
4. **Does it need a connection to run?** A tool that refuses to start offline is phoning home.
5. **Where does the data come from?** If market data comes only from the vendor, your symbols and date ranges go to them.

## How to verify it yourself

Do not rely on the answers alone. During your first month:

1. **Activate, then cut the network.** Disconnect the machine, or on Linux run the backtest in a namespace with no network (`unshare -rn python3 your_backtest.py`). It should run exactly as before.
2. **Watch the connections.** Run it with a network monitor or under `strace -f -e trace=network` and check that no connection is opened during a backtest, other than the licence refresh in the two days before a renewal.
3. **Firewall it.** Block the process's outbound traffic entirely and confirm nothing breaks.

If the engine runs normally with no network at all, nothing during that run can have been uploaded.

## Python's cost, and how to live with it

An engine with a compiled core called from Python is usually the best of both: the fill and cost logic runs at native speed, and only your strategy callback runs in Python. That callback is the cost. Calling into Python once per bar is far slower than the engine itself, so:

- **Keep the per-bar code small.** Compute indicators with numpy over the lookback window, not in Python loops.
- **Prototype in Python, sweep where it pays.** For thousands of runs, either run backtests in parallel or port the finished strategy to a compiled language.

## How Reamer Research does it

Everything below is stated in the documents that ship with the kit:

- **A library on your machine.** The engine is a precompiled library with a stable C interface. Your program loads it and runs backtests in-process.
- **Runs with the network cut off.** The licence is machine-locked and checked locally against a signed grant. The licence server is contacted when you activate or deactivate, and near each monthly renewal to re-sign the end date, sending only the key, fingerprint and seat id. `REAMER_LICENSE_NO_REFRESH=1` turns the renewal call off.
- **Nothing about your trading leaves.** The vendor questionnaire pack in the kit states that strategy code, backtest configurations and trading data are never sent to Reamer Labs, and that there is no telemetry, feature analytics or crash reporting. Reamer Labs keeps only your email address, your licence key and its activation time.
- **No data from us.** You load your own bars, from CSV, a database or anything numpy or pandas can read, and pass them in. No market data is included, so nothing about what you test is requested from anyone. The bundled Alpaca and Databento fetchers are optional and contact only the vendor you call.
- **A Python package, ready to run.** `pip install` the wheel in the kit, with the engine inside it, and `rr.backtest(on_bar, "AAPL.csv")` runs from any directory on Python 3.9 or later with numpy. Nothing is compiled. Bars come from CSV, `.bin` files, pandas DataFrames or numpy arrays. Your `on_bar` function or class sees each ticker's lookback window as numpy arrays and returns orders: market, limit and stop entries, optional stop-loss and take-profit, cancels, and GTC, IOC or GTD lifetimes. `data.position_state(ticker)` and `data.open_orders()` give the full position and every resting order, and actions re-price an order, move a position's stop and target, close all or cancel all. Costs can be set per ticker, and non-price data attaches to a ticker. `open_viewer=True` opens the finished run in an offline trade viewer, a single HTML file that uploads nothing.
- **Templates and examples.** Templates for buy-and-hold, mean reversion, an ATR bracket and a multi-asset strategy, longer example strategies, and a quickstart that runs the bundled sample data end to end.
- **Repeatable from Python too.** A fixed `rng_seed` gives byte-identical output, whichever language drives the engine.
- **Platforms.** Linux on x86-64, macOS on Apple Silicon, and Windows through WSL2.

## Limits to know

- **Python is much slower per bar than C++.** In the kit's benchmark, on release 4.1.2, a realistic strategy cost about 74 µs per bar from Python against about 2.2 µs from C++, roughly 33 times slower. An 884,130-bar backtest took about 65 seconds from Python.
- **The package is sample code under the licence.** It works out of the box, and you may use and change it, in production too. But the licence (Section 4) treats it as unsupported, without warranty and liable to change between releases. The supported contract is the C interface.
- **Closed source.** You can verify what the engine sends, as above, but not read its code.
- **Activation needs a connection once.** And deactivating, to move the licence to another machine, needs one again.
- **No native Windows build.** On Windows it runs through WSL2.

A $20 month includes the full kit and the Python package, so you can activate, cut the network and run your own strategy, and cancel if it does not fit.
