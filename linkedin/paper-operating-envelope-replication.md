A benchmark from one run on one date is one data point. So I ran mine again.

In September I published a whitepaper on the operating envelope of Reamer Server and Reamer Research on an AMD EPYC 9575F. Six weeks later I rented the same CPU model and measured everything again, on the same rental that ran our comparison with Backtrader, NautilusTrader and QuantConnect LEAN.

Between the two runs we shipped 43 commits and seven releases, 4.2.1 to 4.6.0. The OS moved from Ubuntu 24.04 to 26.04.1 and the compiler from GCC 13.3 to 15.2.

Before the run, I wrote five findings from the first paper as claims the new run could refute. All five held:
→ Throughput plateaus at about one strategy per physical core. The peak reproduced within 0.6%.
→ Thread placement governs run-to-run variance, and pinning recovers it. Pinned spread stayed within 1.03×.
→ The ring buffer does best when it is sized to reader lag and kept in cache. The curve reproduced within 3%, and the default is again at the peak.
→ The research engine is exactly reproducible. Its full trade ledger was bit-identical over 20 runs, a stricter test than the first paper's.
→ The order path got faster: 10.3 µs at the median end to end, against 11.3 µs before.

Two results got worse, both outside the envelope. Past one thread per core, throughput rose 15 to 19% but the latency tail widened. Above the L3 size, large rings cost 10 to 11% more.

The largest change surprised me. In the first run, an unpinned 32-strategy run landed in one of two throughput bands about 1.24× apart, a coin flip. This time all 36 runs landed in one band, and the coefficient of variation fell from 10.6% to 1.9%.

Three things changed together, so the run cannot say which one caused any difference. The paper says so. Reproduce the figures on your own hardware before you rely on one of them.

Paper: https://doi.org/10.6084/m9.figshare.34787757
Raw outputs: https://doi.org/10.6084/m9.figshare.34785762

#latency #tradingsystems #performanceengineering #benchmarking #reproducibility #algotrading
