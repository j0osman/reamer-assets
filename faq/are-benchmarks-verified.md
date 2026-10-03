---
title: Have the benchmarks been independently verified?
description: No. No third party has audited the Reamer Research or Reamer Server benchmarks; Reamer Labs ran them itself. What you can check is the method, published as a whitepaper with a DOI, the raw output of every run, also published, and your own hardware, using the benchmark programs that ship in each kit.
stage: 5
order: 37
product: both
next: data-privacy, single-maintainer-risk, security-and-vendor-review, customer-reviews, performance
date: 2026-10-03
---

No. No third party has audited the Reamer Research or Reamer Server benchmarks; Reamer Labs ran them itself. What you can check is the method, published as a whitepaper with a DOI, the raw output of every run, also published, and your own hardware, using the benchmark programs that ship in each kit.

## Who ran them, and where

- **The laptop figures** come from a 2017 four-core laptop.
- **The many-core figures** come from a rented single-socket AMD EPYC 9575F server, run by Reamer Labs in September 2026.

Neither was observed or repeated by anyone else.

## What you can check

- **The method.** The [whitepaper](https://doi.org/10.6084/m9.figshare.33972466) sets out the hardware, settings, workloads and how each figure was measured.
- **The raw output.** The [raw benchmark data](https://doi.org/10.6084/m9.figshare.33877588) holds the unedited console and CSV output behind each published figure. Where a summary and a raw file differ, the raw file is the correct one.
- **Your own machine.** Each kit ships a ready-built benchmark: `research-bench` replays bars through Reamer Research, and `server-bench` sends orders from 1 to 64 strategies through Reamer Server. Both print throughput and latency percentiles, and each kit's `BENCHMARK.md` explains how to read them.
- **Determinism.** Run the same backtest twice with the same seed and compare the result files byte for byte. That claim needs no benchmark to test.

## What you cannot reproduce exactly

- **Not every figure.** The internal programs behind some figures, such as the cost of crossing the C interface and the largest stress tests, are not in the kits.
- **Not the absolute numbers.** The shipped benchmarks are built to run on any supported processor, not tuned to yours, so expect somewhat lower figures than the published ones. Compare the shape of the results, such as how latency changes as you add strategies, rather than the exact numbers.

Running the benchmarks needs an activated licence, so the 30-day trial is the way to test them. See [How fast is it?](/faq/performance.html) and [pricing](/pricing.html).
