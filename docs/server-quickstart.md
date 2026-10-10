---
title: Reamer Server quickstart
description: What ships in the Reamer Server kit, the order set it supports, where to start, and how licence keys work.
group: start
order: 3
product: server
source: README.md
---

## What this is

Reamer Server is the order core of a mid-frequency trading stack: it owns
order state, sequencing, and the pre-trade decision path between a
strategy and a venue. It is delivered as a **C ABI and a static library**,
not as a program you run.

There is no `reamer-server` binary in this kit and none is coming. You
write a `main()` that implements two vtables — a **gate** (your pre-trade
check, with rules you define) and a **broker connector** (your venue
session) — links `lib/libreamer_server_core.a`, and calls
`reamer_server_run()`. That call is your server process.

That split is deliberate. The core is the part that has to be right and is
the same for everyone; the gate and the venue connector are yours, because
your risk rules and your venue relationship are yours. You do not start
from nothing: `reference/` ships a working FIX 4.4 connector, a strategy
relay, an event reader and a gate that take an order to a fill in one
command (see [Start from working code](/docs/connectors.html)). Nothing in
the kit talks to a real venue on your behalf.

**Scope.** Mid-frequency order execution on any instrument you name:
market, limit, stop, stop-limit, trailing stop, trailing stop-limit,
market-if-touched, limit-if-touched and market-to-limit orders; day, GTC,
IOC, FOK, GTD, at-the-open and at-the-close time in force; attached
take-profit and stop-loss exits, OCO/OTO/OUO links, iceberg display size,
minimum quantity, post-only, reduce-only, hidden and all-or-none flags,
account and destination routing, and a user-defined payload of up to 1024
bytes that core passes to your gate and connector untouched. Your
connector decides which of these your venue supports. Built around
per-intent decisions at the pace of a pre-trade check, on a core sized for
that latency class rather than a full L2/L3 order book.

## What ships

| Path | What it is |
|---|---|
| `include/reamer_server_abi.h` | The entire integration contract. Every struct, vtable, and entry point, with the threading and lifetime rules stated inline. Read this first. |
| `lib/libreamer_server_core.a` | The execution core you link. |
| `lib/libreamer_license.a` | License verification, linked by your binary. |
| `bin/reamer-license` | Activates this machine's license (`activate` / `status` / `deactivate`). |
| `bin/reamer-config-check` | Validates a config file offline, without starting a server. |
| `bin/reamer-diag-bundle` | Point-in-time diagnostic bundle for a support request. |
| `bin/release-sign` | Verifies the detached signature over this kit's checksum. |
| `bin/server-bench` | Runs the concurrent-strategy sweep on your own hardware after activation, reproducing `BENCHMARK.md`'s sizing curve — whose core-count-scaling shape is verified on a 64-core EPYC 9575F (`BENCHMARK.md`, "Many-Core Verification"). See "Measuring on your own hardware" there. |
| `EXTENSION_PROTOCOL.md` | Both extension points and both strategy wire protocols, in full. With the header, this is everything needed to integrate. |
| `reference-skeleton/`, `reference-skeleton-rust/` | An accept-all gate and in-memory paper broker, in C++ and Rust. The smallest thing that runs: confirms your `reamer_server_run()` wiring before you write anything real. |
| `reference/` | A worked integration: FIX 4.4 session, broker connector, strategy relay, event-bus reader, and the 100-test suite that exercises them. `demo-trade.sh` runs the whole path to a fill in one command — **start here**. Reference material, not a supported surface — see its README, including why it must not be used to benchmark the core. |
| `VENDOR_QUESTIONNAIRE_PACK.md` | The standing answer to an infra/risk/procurement team's security questionnaire, support model, vulnerability disclosure policy, data-handling statement, and business-continuity/escrow note — hand it to your own security review. Published as the [vendor questionnaire pack](/docs/vendor-questionnaire.html). |
| `SECURITY_HARDENING.md` | The internal adversarial security pass already performed against the ABI/IPC and license boundaries: the defect classes found and closed, and the sanitizer + regression-test verification each fix carries. |
| `THIRD_PARTY_NOTICES`, `sbom.cdx.json` | Every open-source dependency linked into the libraries, with version and license. |
| `LICENSE.txt` | The licence terms governing this kit. `bin/reamer-license activate` reads this file from the kit root and will not activate until you accept it, so do not move `bin/` out of the extracted tree. |
| `SHA256SUMS` | Manifest covering every file above, for post-extraction verification. |
| `SUPPORTED_PLATFORMS.md` | OS, arch, and glibc floor for both products — this server's floor is the higher of the two. |

Every binary in `bin/` answers `--version`, so you can confirm which build
you have:

```bash
bin/reamer-license --version    # reamer-license 4.5.0
```

Docs: `DEPLOYMENT.md` (quickstart and operations), `CONFIG.md` (every
config field), `EXTENSION_PROTOCOL.md` (the extension points in detail),
`LICENSING.md` (seat policy, term anchoring, activation failure modes,
business-continuity/escrow), `MONITORING.md`, `CAPACITY_AND_LIMITS.md`,
`BENCHMARK.md`, `TIMING_INSTRUMENTATION.md`, `RESEARCH_TO_SERVER.md` (for
teams coming from Reamer Research), `VENDOR_QUESTIONNAIRE_PACK.md` and
`SECURITY_HARDENING.md` (procurement and the security-hardening
record), and `CHANGELOG.md`.

## Where to start

1. **See a number in about a minute — `bin/server-bench`.** The cheapest
   thing to try, and the one to do first. It is a prebuilt binary: no
   compiler, no Go toolchain, no open ports, no setup. Activate a key (next
   section), then, from the kit root:

   ```bash
   bin/reamer-license activate <license-key>   # once per machine
   bin/server-bench                            # a real number on your hardware
   ```

   It runs the concurrent-strategy sweep against `reamer_server_run()` and
   prints the throughput/latency curve for this box, directly comparable to
   `BENCHMARK.md`. The curve's shape — clean latency up to roughly one strategy
   per physical core, tail (not median) degrading past that — is verified on
   hardware from a 4-core laptop to a 64-core EPYC 9575F, so expect your own
   elbow near your own core count. That is the whole ask to get a first
   result — everything below is there when you want to go deeper. For
   run-to-run repeatability, pin each strategy to its own core: it collapses
   the unpinned scheduler-driven variance (`BENCHMARK.md`, "pinning ... is a
   repeatability lever"). See "Measuring on your own hardware" in
   `BENCHMARK.md` for how to read the shape.
2. **Close the whole path: `reference/demo-trade.sh`.** When you want to see
   a live order rather than a benchmark, run it. It starts a simulated FIX
   venue, a strategy relay and a gate, waits for FIX Logon, and drives an
   order burst across three instruments through the full path — gate
   decision, venue fill, position book, and the updates back to the strategy:

   ```text
   $ cd reference && ./demo-trade.sh
   {"type":"order_update","intent_id":"aapl-open","instrument":"AAPL","status":2,"filled_quantity":100,"last_price":190,"venue_order_id":"aapl-open"}
   {"type":"ack","request_id":"aapl-open","accepted":true}
   {"type":"order_update","intent_id":"spy-open","instrument":"SPY","status":2,"filled_quantity":250,"last_price":530,"venue_order_id":"spy-open"}
   {"type":"ack","request_id":"spy-open","accepted":true}
   {"type":"order_update","intent_id":"qqq-short","instrument":"QQQ","status":2,"filled_quantity":50,"last_price":470,"venue_order_id":"qqq-short"}
   {"type":"ack","request_id":"qqq-short","accepted":true}
   {"type":"ack","request_id":"tsla-blocked","accepted":false,"reason":"instrument not in approved list"}
   ...
   ```

   Five orders fill across AAPL/SPY/QQQ on both sides; the TSLA order is
   rejected by the gate's policy before it reaches the venue
   (`accepted:false`). The acceptor log shows the position book moving on
   each fill; the gate log shows the accept/reject decision per intent.
   Cold, including the Go build, the whole run is about twenty seconds. It
   needs a Go toolchain, a C compiler, `nc`, and ports 4000/9100/8090
   free. `go test ./...` in the same tree runs the 100-test suite in ~8s.
   You now have a working reference deployment exercising real order flow
   to read against, rather than a specification to imagine against.
3. **`include/reamer_server_abi.h`.** Twenty minutes in this file answers
   most integration questions — it is the whole contract, and it is
   written to be read. Everything else is either a way to run what this
   file describes or a worked example of it.
4. **`reference/`, to see what a real integration looks like.** A FIX
   session that logs on, sequences, heartbeats and reconnects, a connector
   that turns execution reports into ABI structs, a relay, and an event-bus
   reader. Read its README first, particularly the latency section — **do
   not benchmark the core with this tree** (`bin/server-bench` at step 1 is
   the tool for that); `BENCHMARK.md` states the conditions for every number
   the product claims.
5. **Write your own two vtables against the header.** A gate that accepts
   and a broker connector that fills is about a hundred lines of C or
   C++, linked against `lib/libreamer_server_core.a`, with your own
   `main()` calling `reamer_server_run()`. That is the whole integration,
   and no reference code is required to reach a running server —
   `reference-skeleton/` (C++ and Rust) is the smallest version of it.
6. **`EXTENSION_PROTOCOL.md`, "Strategy socket protocol".** How a
   strategy talks to core in the default `local` mode, byte for byte, so
   you can drive your server from any language. Read the framing table
   and the two message tables; that is enough to write a client.
7. **`DEPLOYMENT.md`.** Steps 1–2 verify the download and licence the
   machine; Steps 3–7 are config, your own gate and connector, and the
   operational checks around a live server.

## Your license key

**This archive contains no license key, deliberately.** Your key arrives by
email on payment, with this kit's download link. It carries the term you
bought: a 30-day trial or 1 year. Nothing in this zip will activate on its
own.

`reamer_server_run()` refuses to enter its loop without an active license,
so **Step 4 of `DEPLOYMENT.md` will not run until a key is activated.** If
your key did not arrive, write to:

> **support@reamerlabs.com** — missing or failed keys.

Activation is per-machine and keyed to a host fingerprint, so a
containerized deployment needs a word of setup — `LICENSING.md`'s "Self-Service Activation Failure
Modes" covers this, and it is worth reading *before* you activate rather
than after.

Keys are also per-product: this kit's `bin/reamer-license` activates
Reamer Server licences only. A Reamer Research key will not activate it,
and a machine running both products activates each separately.

`reamer-license activate` presents `LICENSE.txt` and requires you to type
`accept` before it contacts the licence server. Declining consumes no seat.
The acceptance is recorded next to the activation record and is not
re-prompted for the same terms. For unattended provisioning, set
**`REAMER_ACCEPT_LICENSE=accept`** in the environment — activation without a
terminal and without that variable fails rather than defaulting to
acceptance. Amended terms carry a different hash and prompt again.

Other contacts:

| For | Address |
|---|---|
| Purchases, invoices, commercial terms | hello@reamerlabs.com |
| Licensing and technical support | support@reamerlabs.com |
| Security correspondence | team@reamerlabs.com |

## What is not in this kit

Stated plainly, so you can scope the work rather than discover it:

- **No venue adapter for your venue.** A reference FIX 4.4 session and
  broker connector ship in `reference/`, exercised end to end against a
  simulated acceptor. They demonstrate what
  `ReamerBrokerConnectorVtable` requires; they are not an adapter for your
  venue's dialect, your certification, or your credentials. That work is
  yours to build or to source.
- **No gate policy.** `ReamerGateVtable` defines the decision point. The
  rules behind it are yours.
- **No strategy client.** `EXTENSION_PROTOCOL.md` specifies both wire
  protocols byte for byte; the client that speaks them is yours.
- **No OEMS, no allocation, no compliance layer.**
- **No product source.** The core is closed-source and stays that way.
  The header and `EXTENSION_PROTOCOL.md` remain the complete integration
  contract, and nothing in `reference/` is required to reach a running
  server. Reference code is sample material under this kit's licence —
  read it, copy from it, adapt it. It is not the core, and it carries no
  support commitment.
- **No key.** See above.
