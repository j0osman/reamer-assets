---
title: "What 2,000 Runs on a 64-Core EPYC Show About Reamer Research"
description: A single-threaded research engine has one job the rest of research depends on: run the same input twice and get the same answer, at a speed that does not wander. We measured both across 2,000 runs on an AMD EPYC 9575F. Timing held to a 0.59% coefficient of variation and every hashed output came back byte-identical.
date: 2026-09-19
tier: Findings
---

A research result is only worth acting on if two things hold. The same input has to give the same output, exactly, every time. And the time it takes to produce that output has to stay steady, so a sweep of a thousand variants is a stable measurement rather than a shifting one. We measured both on Reamer Research, the deterministic research engine, on a single-socket AMD EPYC 9575F, and this note reports what came back. The raw artifacts behind every number are published, so none of it has to be taken on trust.

## The engine and the machine

Reamer Research is a single-threaded research engine delivered as a stable C ABI and a shared library. A strategy is a user-supplied callback that the engine calls once per bar as it replays historical bars through it. Single-threaded is a deliberate choice here, not a limitation: nothing schedules against the hot loop, so run-to-run variance has no operating-system placement noise to pick up.

The machine is an AMD EPYC 9575F, one socket, 64 cores and 128 threads, one NUMA node, with the CPU governor set to performance before every run. The research build is compiled `-O3 -funroll-loops -march=native` for the Zen 5 target. Every figure below comes from runs captured on 2026-09-13.

## Timing that does not wander

We measured per-callback latency across 2,000 independent runs, each replaying 884,130 bars through a buy-and-hold strategy over a direct ABI path with no wrapper. Per-call latency sat at roughly 0.56 to 0.59 microseconds across the runs. Aggregate throughput averaged 1,721,199 bars per second.

The number that matters for research is not the mean, it is the spread around it. Across 2,000 runs the coefficient of variation was 0.59%. That is the property a parameter sweep rests on: when you run the same workload a thousand times with different parameters, the timing floor is steady enough that a difference you see is a difference in the work, not in the machine's mood that second. A companion measurement on the concurrent execution path, where placement does come into play, showed a coefficient of variation of 10.6% unpinned. The research engine's 0.59% against that 10.6% is the whole point of keeping the research engine single-threaded and off the scheduler's contended path.

## Byte-identical, including the stochastic path

Consistent timing is one property. Exact reproducibility is a separate one, and the two do not imply each other. To measure reproducibility we ran the full path from CSV to result 20 times and hashed each output. All 20 came back byte-identical, including the stochastic path where a naive implementation would let a random seed drift. Twenty of twenty identical hashes means the result is fixed by its inputs and nothing else: not run order, not wall-clock time, not which core it landed on.

That is what makes the rest of a research workflow trustworthy. Determinism is not a headline speed; it is the precondition that lets any other check, a validation, a robustness sweep, a regression comparison, mean something. If the same input can give two answers, none of those checks can be run at all.

## Headroom under load

The engine also has room to spare on the data-volume axis. Parsing about 4.7 million CSV rows down to 884,130 bars ran at 2,223,465 bars per second. The hot loop stays compiled, not interpreted, so the cost of a larger dataset scales with the data rather than with per-bar overhead.

## The caveats, stated plainly

These figures come from one machine on one date. They characterize an operating envelope, not a guaranteed service level. Reproduce them on your own target hardware before you rely on any specific number. The point of publishing the raw console and CSV outputs is exactly that: a reader can check every figure here against the files, and where a raw file and this note appear to differ, the raw file is authoritative.

The full method, the environment table, and the rest of the measurements are in the preprint, [Predictable Trading Infrastructure on Many-Core CPUs](https://doi.org/10.6084/m9.figshare.33972466). The unedited benchmark artifacts are archived openly at [doi.org/10.6084/m9.figshare.33877588](https://doi.org/10.6084/m9.figshare.33877588).
