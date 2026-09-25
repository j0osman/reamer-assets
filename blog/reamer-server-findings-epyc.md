---
title: "Reamer Server on a 64-Core EPYC: The Tail Is the Measurement"
description: A trading engine is usually sold on one number, a peak throughput or a best-case latency. Neither tells a team what happens under concurrent load. We measured the integration boundary, the end-to-end order path, and the full 1-to-512 strategy sweep on an AMD EPYC 9575F. The boundary costs 53 nanoseconds, the engine-only P99 sits at 12.4 microseconds, and throughput holds a plateau to about 67 strategies.
date: 2026-09-19
tier: Findings
---

Vendors and internal teams usually describe an execution engine with a single headline number: a peak throughput measured once under ideal placement, or a best-case latency. That number tells a team almost nothing about the question that actually matters, which is what happens to the distribution when many strategies run at once. We measured Reamer Server, the order-management engine that takes a proven strategy live, across that whole range on a single-socket AMD EPYC 9575F, and the theme that runs through every result is the same: the tail carries the information, not the median.

## What Reamer Server is

Reamer Server is the order-management engine delivered as a stable C ABI and a static library. It holds order state and sequencing and runs the pre-trade decision path between a strategy and a venue. A deploying team links the engine into its own process, writes its own pre-trade risk gate and venue connector around it, and connects strategies out of process. What we measured is the engine itself, with no venue, no network path, and no language binding in the latency-critical path, so the numbers reflect the engine rather than the harness around it.

## The integration boundary costs tens of nanoseconds

Crossing the ABI boundary costs something: the engine marshals an order intent in and an acknowledgement out. We measured that full boundary check at a raw median of about 73 nanoseconds, and a net median of 53 nanoseconds once the timer overhead is corrected out. Marshaling drives that cost more than dispatch does, as you would expect for a boundary crossing. The practical point is that the boundary sits at tens of nanoseconds, which is negligible against any real venue round trip. Linking the engine into your process does not cost you a meaningful slice of your latency budget.

## End-to-end, the tail is flat

The end-to-end order path, engine only, ran a P50 of 11.3 microseconds and a P99 of 12.4 microseconds across 100,000 orders. Every order matched its accept-or-reject event exactly once.

The shape is the finding. On a laptop the same path showed a P99 at about 3 times the median and a P99.9 at about 10 times it. On the EPYC, P99 is 1.1 times the median and P99.9 is 1.2 times it. A tail that sits almost flat against the median is what a team building a risk deadline actually needs, because the deadline is set by the tail, not the median. A prior note had described this path as occasionally exceeding 100 microseconds under single-strategy burst load; that does not reproduce here, where P99 sits at about 12 microseconds.

## Throughput holds a plateau, then the tail moves first

We swept from 1 to 512 concurrent strategies, 5,000 orders per strategy, stepping one strategy at a time. Aggregate throughput holds a broad plateau of roughly 200,000 to 220,000 orders per second that runs from a handful of strategies out to about 67 of them on this hardware. Inside that band the latency distribution stays stable.

Past the plateau, the change shows up in the tail before it shows up anywhere else. The median can still look healthy while the P99 has already started to move, which is exactly why a single headline throughput number is misleading: it is measured at one point, under ideal conditions, and it says nothing about where the distribution starts to degrade. Watching the tail across the sweep tells a capacity-planning team the one thing the peak number cannot.

## Placement governs variance, not the median

Run-to-run variance on the concurrent path was governed by thread placement rather than by the median latency. Unpinned, the engine's run-to-run coefficient of variation was 10.6%; pinning the strategy clients tightened it. The takeaway for a deployment is that the median tells you the typical cost while placement tells you the consistency, and consistency is a scheduling decision you control.

## The caveats, stated plainly

Every figure here comes from one machine on one date, and characterizes an operating envelope rather than a guaranteed service level. Reproduce on your own target hardware before you rely on any specific number. Two benchmark-harness fixes were needed to run on current builds; neither is a defect in the measured software, and both are described in the paper. The raw console and CSV outputs behind every number are published, and where a raw file and this note appear to differ, the raw file is authoritative.

The full method, the environment table, and the complete 1-to-512 curve are in the preprint, [Predictable Trading Infrastructure on Many-Core CPUs](https://doi.org/10.6084/m9.figshare.33972466). The unedited benchmark artifacts are archived openly at [doi.org/10.6084/m9.figshare.33877588](https://doi.org/10.6084/m9.figshare.33877588).
