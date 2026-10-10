---
title: Reamer Server capacity and limits
description: The measured operating envelope of Reamer Server and Reamer Research: event-bus wraps, order tracking, memory and file-descriptor stability.
group: operations
order: 5
product: server
source: CAPACITY_AND_LIMITS.md
---

Single authoritative reference for the operating envelope of both
products, for capacity-planning review against a proposed
deployment's expected load. Every number below traces to the
internal load-test-thresholds reference (a measured, re-runnable load
test kept in the product source tree) or to an existing config default
documented in this kit's `CONFIG.md` — nothing here is invented.

> **Version these describe: 4.1.0.** The measurements were taken
> 2026-08-23 on the 4.1.0 line. Later releases have not been re-measured
> against this envelope; `CHANGELOG.md` lists what each one changed.
>
> What you cannot do is **reproduce** it: the benchmarks that
> produced these numbers are internal and do not ship, for the reasons in
> "How these limits were derived" below. Provenance is not the open
> question here; independent verification is.

## Reamer Server

**Max sustained order rate / event throughput.** The shared-memory event
bus (`ShmEventBus`) survives 10 full wraps of its 4096-slot ring (~10,240
orders submitted back-to-back, 4+ events/order) with zero `readBatch()`
gap-detection triggers (measured threshold, Dimension 1 of the
internal load-test-thresholds reference above).

**`max_tracked_orders`** — default `5000`, evicts the oldest tracked order
(FIFO by insertion order) once exceeded, logging an `order_tracking.evicted`
warning naming the evicted order id and the active cap; see `CONFIG.md`
for full behavior (cancel/replace against an evicted order id). Eviction
correctness beyond a configured cap (a fixed `max_tracked_orders=50` for
reproducibility, submitting 6,000 orders back-to-back) is verified by
Dimension 3 of the internal load-test-thresholds reference above:
zero crashes, all acks accepted, oldest order's cancel request lands in
"unknown order" rejection after eviction.

**`event_buffer_capacity`** — default `4096` (event count, not a time
window), overwrites the oldest event once the ring fills; see `CONFIG.md`
for full behavior (readers attaching read-only via `since()` and hitting a
gap once evicted).

**Memory and file-descriptor stability over sustained operation.** Resident
memory does not grow with cumulative event volume: over 500,000 orders
submitted back-to-back at saturation, post-warm-up RSS slope measured
**0.0259 KB per 1,000 events** against a 2.0 KB/1,000 tolerance — flat
within measurement noise, on a resting set of roughly 39 MB. Open file
descriptors do not grow across relay reconnects: fd count held at **14**,
its cycle-0 baseline, through 80 consecutive disconnect/reconnect cycles of
the strategy-relay path (tolerance ±2). Both are Dimension 4 of the
internal load-test-thresholds reference above, measured 2026-08-23 on
Linux x86-64 in a Release build.

Plan capacity against event and reconnect counts, not elapsed time: these
are the axes the measurements accelerate, and neither figure degrades with
how long the process has been up. The runs do not characterize a leak keyed
to wall-clock time itself (a timer callback, a daily rollover, a TTL cache
eviction); no such trigger is known in the current design.

## Reamer Research

**Max practical portfolio / bar count.** A 100-ticker, union-aligned
portfolio with 5 years of daily bars (~1,260 bars/ticker, 126,000 bars
total — the "100-ticker portfolio, 5yr history" scenario
Reamer Research is designed around)
sustains at least 3000 steps/sec, the floor enforced by our CI performance
gate (Dimension 2 of the internal load-test-thresholds reference).

## How these limits were derived

Every threshold above is defined, with full rationale and measured
baselines, in an internal load-test-thresholds reference. Each is backed
by a re-runnable benchmark rather than a one-off measurement:

- Dimensions 1 and 2 (event throughput, backtest replay throughput): a
  stress benchmark, with the threshold enforced as a CI performance gate.
- Dimension 3 (order tracking eviction under load): a dedicated load
  benchmark driving the order-tracking path past its configured cap.
- Dimension 4 (RSS slope under saturation, fd count across relay
  reconnects): a resource-ceiling benchmark. A loose smoke variant runs on
  every CI pass; the numbers above come from the full-length variant.

**None of these benchmarks ship in this kit, and none is something you
could build** — the product is closed-source and its test suite is not
part of the delivery. They are described here so the basis for each number
is traceable, not so you can reproduce them. Every figure above is
enforced in our CI on each release: a regression fails the release rather
than reaching you.

**What you can do about that.** Taking capacity on trust is a reasonable
thing to refuse. Two options, in order of preference:

1. **Ask for the results.** We provide the measured outputs of the runs
   behind every number above, including the run conditions and hardware,
   to any licensee. Request them from support@reamerlabs.com — this is a
   normal request, not an exception we make.
2. **Measure the dimension that matters to you, on your own hardware.**
   The shipped kit can be driven with real order flow end to end; see
   `BENCHMARK.md`'s "Measuring this kit" for the working setup. That gives
   you your own numbers on your own box, which for sizing decisions is
   worth more than ours.

To validate the envelope against your own hardware and order flow, use
the shipped path in `BENCHMARK.md`'s "Measuring this kit": run your
`cgo-gate` with `REAMER_TIMING_DEBUG=1` and watch the
`order_tracking.evicted` and event-bus gap events described in
`MONITORING.md`.
