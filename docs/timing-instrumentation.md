---
title: Reamer Server timing instrumentation
description: How to read Reamer Server's REAMER_TIMING_DEBUG lines: input-to-poll latency, dispatch outliers and frame decode timing.
group: operations
order: 6
product: server
source: TIMING_INSTRUMENTATION.md
---

## Overview

Reamer Server core ships with timing instrumentation built into its main
request-processing loop, inert unless you enable it. Set
`REAMER_TIMING_DEBUG=1` and core emits timing lines to stderr.

Use it to answer "is the loop keeping up, and did any dispatch stall" —
whether you are sizing a deployment, tuning your own gate policy, or
chasing a tail you cannot explain. This document defines every line the
shipped 4.1.0 binary emits and how to read them.

**Scope of what this gives you, stated plainly.** As of 4.1.0 the
instrumentation reports **poll latency, batch size, and outlier dispatch
totals** — not a per-stage breakdown of each order. Earlier versions
emitted a per-intent `pullState`/`gate_check`/`sendAck` split; that was
removed when the dispatch path was restructured, because keeping it meant
timing every stage of every request on the hot path. If you need to
attribute latency to your own `pull_state()` or `check()`, instrument
those functions inside your own vtable implementations — you own that
code, and a timer there costs nothing when you are not measuring.

`BENCHMARK.md`'s "Measuring this kit" section is the short version: how
to turn this on against the reference gate and collect the output. This
document is the field reference behind it.

## Instrumentation Points

### 1. Input->Poll Overhead
Emits:
```
[TIMING] input->poll(1): X.XXXms  batch_size=N
```

Measures the complete poll(1) call duration, including:
- Socket I/O wait time
- Message queuing/dequeuing
- Any internal scheduling overhead

Emitted once per poll that returned at least one request. A poll that
returns nothing emits nothing — which is why an idle server produces no
timing output at all (see "Getting output at all", below).

### 2. Outlier Dispatch
Emits:
```
[TIMING] dispatch: total=X.XXXms [OUTLIER_DETECTED]
```

The wall time core spent dispatching **one** strategy request — the whole
of it, covering your broker connector's `pull_state()`, your gate's
`check()`, and core's ack send back to the strategy.

**This line is emitted only when that total exceeds 200ms.** It is an
outlier alarm, not a per-request trace: a healthy server under load emits
`input->poll(1)` lines continuously and `dispatch:` lines never. Seeing
one means a single request took longer than two-tenths of a second, which
on this path is pathological and worth chasing.

The line does not attribute the time to a stage. To find which of the
three your time went to, time them inside your own vtable functions.

### 3. Frame Decode
Emits:
```
[TIMING_DETAIL] intent=<intent_id> frame_decode=X.XXXms
```

Wire-frame decode duration for one inbound intent. Emitted on both
strategy-input paths — the UDS relay path (`strategy_input_mode:
"remote"`, `strategy_relay_endpoint`) and the direct socket path
(`strategy_input_mode: "local"`, `socket_path`) — for new-order, cancel,
and replace frames.

`intent=(empty)` means the client sent the frame with an empty intent id.
That is a fact about the client's frame, not a gap in the trace: the
server does not synthesize an id. Frames of other types emit no
`[TIMING_DETAIL]` line at all.

### The complete list

Those three are **every** timing line the 4.1.0 binary can emit. You can
confirm that against the shipped artifact yourself, without source:

```
strings -a lib/libreamer_server_core.a | grep '\[TIMING'
```

If you are writing a log parser, write it against that output.

## How to Interpret the Logs

### Healthy server under load
```
[TIMING] input->poll(1): 0.045ms  batch_size=1
[TIMING] input->poll(1): 0.038ms  batch_size=3
[TIMING] input->poll(1): 0.041ms  batch_size=1
```

Poll lines only. No `dispatch:` line means no single request crossed
200ms — which is the normal state. Do not wait for a `dispatch:` line to
conclude the server is working; its absence is the good outcome.

`batch_size` is the useful signal here. It is how many requests that poll
returned, so a steadily climbing `batch_size` means requests are arriving
faster than the loop drains them — the queue is building. A `batch_size`
that stays low while poll latency stays low is a loop keeping up.

### Outlier
```
[TIMING] input->poll(1): 0.045ms  batch_size=1
[TIMING] dispatch: total=3199.020ms [OUTLIER_DETECTED]
```

One request took 3.2 seconds. The line tells you that and nothing more —
it does not say which of `pull_state()`, `check()`, or the ack send
absorbed it.

Attributing it is your next step, and the instrumentation cannot do it
for you. The three candidates are all reachable from code you own:

| Suspect | Why it stalls | How to confirm |
|---|---|---|
| Your gate's `check()` | Policy doing real work, a lock, or an I/O call that does not belong on the hot path. | Time the body of your own `check()`. |
| Your connector's `pull_state()` | Lock contention, or a blocking venue call. `pull_state()` must return cached state and never do I/O. | Time the body of your own `pull_state()`. |
| Core's ack send | Socket write stall — the strategy's receive buffer is full because its reader is slow. | Check your strategy client's read loop for backpressure. |

Rule out the host first. An unpinned process on a contended box produces
multi-second stalls that belong to the scheduler, not to any of the
three above — check CPU contention, noisy neighbors, and the power
governor before instrumenting anything. A debug-built gate policy will
also dominate in a way a release build does not.

## Diagnostic workflow

1. **Start your server with instrumentation on, capturing stderr:**
   ```bash
   REAMER_TIMING_DEBUG=1 ./your-server <your-config.json> 2>timing.log
   ```

2. **Drive it with order flow.** This is not optional and it is the step
   most often skipped: an idle server emits **no timing lines at all**,
   because every line above is emitted per non-empty poll. See "Getting
   output at all" below for the shortest working driver.

3. **Find the outliers.** Core marks anything over 200ms for you:
   ```bash
   grep OUTLIER_DETECTED timing.log
   ```

4. **Check whether the loop is keeping up**, independently of outliers:
   ```bash
   grep -o 'batch_size=[0-9]*' timing.log | sort -t= -k2 -n | tail
   ```
   Large batches mean the queue is building.

5. **Attribute any outlier** using the table above — by timing your own
   vtable functions, since core reports only the total.

## Enabling instrumentation

No special build is needed, and none is possible — the shipped
`libreamer_server_core.a` is precompiled with this instrumentation
already in it, gated entirely by the `REAMER_TIMING_DEBUG` environment
variable at runtime.

Set it when starting your server:

```bash
REAMER_TIMING_DEBUG=1 ./your-server <your-config.json>
```

Unset (the default), the instrumentation is inert.

## Getting output at all

**An idle server emits nothing.** Every line above is emitted per
non-empty poll — no order flow, no output. Starting the server you built
in `DEPLOYMENT.md` Step 4 and waiting produces an empty `timing.log`: the
stub connector is a venue, not a strategy, and nothing is submitting.

To get output you must connect a strategy and send it orders. In the
default `local` mode that is one extra process, not three:

1. Start your server with instrumentation on, and confirm the config path
   resolves:

   ```bash
   bin/reamer-config-check --config <your-config.json>
   REAMER_TIMING_DEBUG=1 ./your-server <your-config.json> 2>timing.log
   ```

   A config path that does not resolve starts core silently on defaults,
   so check it first rather than debugging an empty log.

2. Connect a strategy client to `strategy_socket_path` and submit orders.
   The `local`-mode framing is specified byte for byte in
   `EXTENSION_PROTOCOL.md` under "Strategy socket protocol (`local`
   mode)" — a `NewOrder` is a 5-byte frame header followed by the
   documented field layout, which is a few dozen lines in any language.

   Each order is one trip through the full path — decode, dispatch, your
   gate, your broker connector — and each produces timing output.

   Whether an order is accepted depends on the gate *you* wrote. A
   rejected order still crosses the decode and dispatch path, so it still
   produces timing lines — but it never reaches the venue, so do not use
   rejections to measure the full path.

3. Read the log:

   ```bash
   grep '\[TIMING\]' timing.log
   ```

Loop step 2 to generate enough samples to be worth aggregating. A
hand-driven client is a functional check, not a load generator: for
capacity numbers you want your own driver submitting in a tight loop,
which is the same plumbing your real strategies will need anyway.

## Reporting a latency problem to us

If the outliers point at core itself — `dispatch:` lines on a quiet,
pinned host, with your own `check()` and `pull_state()` timers showing
they were not the cost — send us the evidence rather than the symptom:

1. The `timing.log` excerpt covering the slow requests, including the
   surrounding `input->poll(1)` lines and any `[TIMING_DETAIL]` lines.
2. Your own stage timings from inside `check()` and `pull_state()`, which
   are what rule those two out.
3. A `bin/reamer-diag-bundle` capture from the same run (`MONITORING.md`).
4. Your host's CPU model, core count, and whether the process was pinned.

support@reamerlabs.com.
