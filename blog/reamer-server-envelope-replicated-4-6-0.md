---
title: "Reamer Server's Operating Envelope, Measured Again on 4.6.0"
description: Six weeks after the first whitepaper, I measured Reamer Server and Reamer Research again on the same CPU model, after a new release, a new kernel and a new compiler. I stated five earlier findings as claims the new run could refute. All five held, and two results outside the envelope moved.
date: 2026-10-26
tier: Findings
topics: benchmark, manycore, latency, reproducibility, determinism, oms
cites: replication, whitepaper
---

In September I published a [whitepaper](https://doi.org/10.6084/m9.figshare.33972466) that maps the operating envelope of Reamer Server and Reamer Research on an AMD EPYC 9575F. A single run on a single date is one data point. So six weeks later I rented the same CPU model and measured again. That paper is now public: [Does the Operating Envelope Survive an Upgrade?](https://doi.org/10.6084/m9.figshare.34787757)

## What changed in between

Three things, all at once:

- the release, from 4.2.1 to 4.6.0, which is 43 commits and seven releases
- the operating system, from Ubuntu 24.04 to 26.04.1
- the compiler, from GCC 13.3 to 15.2.0

Because they changed together, no measurement here can say which one caused a difference. Every delta below is a delta of the combined change.

## Five claims that could have failed

Before the run, I wrote five findings of the first paper as claims that the new run could refute:

1. The clean-latency region and the throughput plateau extend to about one strategy per physical core, about 64.
2. Thread placement, not the median, governs run-to-run variance, and pinning recovers it.
3. A ring buffer does best when it is sized to reader lag and kept in cache.
4. The research engine is exactly reproducible, even with seeded randomness.
5. The ABI boundary costs about 53 ns net at the median, and the order path takes 11.3 µs at the median and 12.4 µs at P99 end to end.

## All five held

- **The plateau.** Its peak reproduced within 0.6% and its level within 2%, in two independent sweeps.
- **Placement.** Pinned run-to-run spread stayed within 1.03×.
- **The ring.** The ring-size curve reproduced within 3% up to the 256 MiB L3, with the product default again at its peak.
- **Reproducibility.** The research engine's full closed-trade ledger was bit-identical over 20 runs, and the C++ and Python paths agreed. This is a stricter test than the first paper's.
- **The order path.** End to end, it was 1.3 to 1.6 µs (11 to 15%) faster at the median, and lower at every percentile through P99.9. The ABI boundary read 80 ns against 73 ns raw. That difference is below the machine's effective timer resolution of 10 ns, so it held within resolution.

The research engine also ran 6.2% more bars per second than before.

## What moved

Two results moved, and both are outside the operating envelope:

- Between 97 and 128 strategies, beyond one thread per physical core, throughput was 15 to 19% higher than before, with a worse latency tail.
- Above the L3 size, large rings cost 10 to 11% more than before.

Neither changes what an operator should do. Inside the envelope, the numbers are the same or better.

## What is new in this run

The run also measured things the first paper did not: a concurrency sweep in steps of one strategy from 1 to 128, a ring-size sweep with hardware counters, pinning of the client threads, and the measured effect of 4.6.0's position-state fix on a strategy that reverses.

## The caveats

One machine per run, one date per run, and three changes at once. Both servers were the same CPU model from the same provider, but I cannot confirm they were the same physical machine. Reproduce the figures on your own hardware before you rely on any one of them.

The full method, the scorecard and every table are in the preprint, [Does the Operating Envelope Survive an Upgrade?](https://doi.org/10.6084/m9.figshare.34787757). The raw outputs are archived openly at [doi.org/10.6084/m9.figshare.34785762](https://doi.org/10.6084/m9.figshare.34785762).
