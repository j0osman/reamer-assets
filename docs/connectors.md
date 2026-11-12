---
title: Start from working code
description: What both kits ship beyond the core: a working FIX 4.4 connector, strategy relay, gate and event reader for Reamer Server, and a Python package, a C++ header and an offline trade viewer for Reamer Research.
group: start
order: 1
product: both
source: README.md, reference/README.md
---

Each product is a small core that does one job and is the same for
everyone: Reamer Research fills orders against bars, Reamer Server owns
order state and the pre-trade decision path. The parts that differ from
desk to desk, such as your venue, your risk rules and your data, stay in
your own code, on your own terms.

You do not start those parts from a blank file. Both kits ship working
source for them, and it runs on the day you download it. This page lists
it and says what it is and what it is not.

## Reamer Server: an order to a fill in one command

`reference/` in the Server kit is a complete integration of the layer
around the core, written in Go:

| Part | What it does |
| --- | --- |
| `cgo/fixsession/` | A FIX 4.4 session: Logon, heartbeats, sequence numbers, reconnect, NewOrderSingle, execution reports, RequestForPositions. |
| `cgo/` | A broker connector that turns execution reports into ABI structs, and a gate. |
| `relay/`, `cmd/relay` | A strategy relay: strategies speak a demonstration newline-JSON protocol on TCP port 9100, bridged to the core's strategy socket, and get acks and order updates back. Adapt it to your own transport. |
| `shmbus/`, `cmd/event-tail` | A reader for the ShmEventBus, the shared-memory event stream the core writes. |
| `cmd/fix-acceptor` | A simulated FIX venue that fills orders, so the whole path runs with no venue account. |
| `demo-trade.sh` | Starts all of the above and sends an order burst across three instruments. |

From the kit root, with Go, a C compiler, `nc` and an activated licence:

```bash
cd reference && ./demo-trade.sh
```

Cold, including the Go build, the run takes about twenty seconds. Five
orders fill across AAPL, SPY and QQQ, and the gate rejects a TSLA order
with `instrument not in approved list` before it reaches the venue. The
acceptor log shows the position book moving on each fill; the gate log
shows each decision. `go test ./...` in the same tree runs its 100 tests
in about eight seconds.

The fastest route to your own server is to read this tree, then replace
the simulated acceptor with your venue's session and the sample gate rule
with your own rules.

### The smallest thing that runs

`reference-skeleton/` (C++) and `reference-skeleton-rust/` (Rust) are an
accept-all gate and an in-memory paper broker linked against
`lib/libreamer_server_core.a`: about a hundred lines that confirm your
`reamer_server_run()` wiring before you write anything real. Its gate
accepts everything and its broker fills at a fixed price, so it is a
shape demo, not a connector.

### What the core already handles

Writing the connector does not mean writing an order manager. The core
tracks every order's state, applies fills, keeps positions, runs the gate
on each intent and publishes every step to the event stream. Server ABI 4
carries the full order set through it: nine order types, seven
time-in-force values, attached take-profit and stop-loss exits, OCO, OTO
and OUO links, and a 1024-byte payload of your own. Your connector maps
those to what your venue supports. The [extension protocol](/docs/extension-protocol.html)
is the complete contract.

## Reamer Research: a backtest in a few lines

| Path | What it is |
| --- | --- |
| `python/reamer_research-*.whl` | The [Python package](/docs/python-package.html), library and trade viewer inside: `pip install` it and `rr.backtest(on_bar, "AAPL.csv")` runs from any directory. Its source is `reference-python/`, with strategy templates and `fetchers/` for historical bars from Alpaca or Databento. |
| `include/reamer_research.hpp` | The [C++ wrapper](/docs/cpp-wrapper.html), one C++20 header: `rr::backtest(on_bar, "AAPL.csv")` with a lambda or an object strategy, found by `find_package(ReamerResearch)` and pkg-config. `reference-cpp/` holds five examples, `quickstart.cpp` first. |
| `viewer/reamer-viewer.html` | The offline [trade viewer](/docs/trade-viewer.html): each closed trade on its bars, beside the values your strategy printed. |
| `bin/reamer-csv-build` | Converts an OHLCV CSV into the `.bin` bar file the engine memory-maps, with a parse report. Needs no licence. |
| `bin/reamer-exo-build` | Converts a two-column CSV into the `.exo.bin` sidecar for non-price data. Needs no licence. |
| `bin/research-bench` | A prebuilt benchmark: a real throughput number on your hardware with no compiler. |

The Python quickstart, from the kit root, an ATR band reversion with a
bracket on every entry that opens in the trade viewer when it finishes:

```bash
pip install python/reamer_research-*.whl
python3 reference-python/examples/quickstart.py
```

### From research to live

From ABI 7 (4.5.0), `libreamer_research` includes a strategy relay:
`reamer_relay_open()`, `reamer_relay_send()` and `reamer_relay_poll()`
send the same `ReamerOrderRequest` a strategy returns from `on_bar()` to a running Reamer Server, or to any receiver you write that speaks `EXTENSION_PROTOCOL.md`, and `reamer_relay_get_positions()` reads the fills back. Brackets travel with the entry. The relay has limits: it sends new
orders and single cancels only, gate decisions are not echoed back to it,
and fills for orders sent before a reconnect do not reach it.
[Research to Server](/docs/research-to-server.html) lists every limit and
walks one order through both kits.

## Watch it done

An uncut screen recording shows Claude Code integrating Reamer Server from
a fresh kit, using only the shipped files, recorded against 4.2.1:
[Reamer Server in 19 minutes](/videos/reamer-server-integration-demo.html),
a C++ server with a gate and a broker connector, taking orders to fills.

For Reamer Research, the [product page](/products/reamer-research.html)
shows the whole quickstart in Python and C++ and the trade viewer it opens.

## What this code is, and is not

- **The C ABI is the contract.** `reamer_research_abi.h` and the Reamer
  Server headers in `include/` define it. The Reamer Research Python package
  and C++ header are built over it and ship in source: read them, change
  them, keep them in your own tree. Everything under `reference/` and
  `reference-skeleton*/` is reference material under the kit licence: you
  can read it, change it and ship it in your own work, but it is not a
  supported surface.
- **It is not an adapter for your venue.** The FIX session talks to the
  simulated acceptor in the kit. Session recovery is reconnect-only, with
  no ResendRequest and no persisted sequence numbers, so a production
  session for your venue needs that work.
- **Do not benchmark the core with it.** The cgo boundary costs about 1–2
  µs per call, against about 0.03 µs at P50 for the ABI call itself. Use
  `bin/server-bench` and `bin/research-bench` for numbers.
