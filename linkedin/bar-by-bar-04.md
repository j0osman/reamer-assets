# Why do I get different backtest results each time I run it? | Bar by Bar #4

Because something in the run isn't fixed. Until you find it, no result can be compared with any other: you can't tell whether an improvement came from your edit or from the run.

The usual causes:

- **Randomness without a fixed seed.** Simulated slippage, sampled spreads, a model's starting weights. Cost noise is useful; noise you can't replay isn't.
- **Data that changed.** Vendors revise history, a last bar is still forming, or a time zone or daylight-saving change moves bars.
- **Order that isn't guaranteed.** Instruments looped from a hash map, threads finishing in a different order, floating-point sums added in a different sequence. One flipped signal changes every trade after it.
- **An environment that drifted.** A library upgrade, a new machine, a changed config default.
- **The clock or the machine.** Reading the current time, calling a network service, or timing out a slow step.
- **State left from the last run.** Caches, globals, indicators that weren't reset.

To find it, run twice with nothing changed, compare the full trade logs rather than the final return, and look at the first trade that differs. The cause is at or just before that bar.

The target is identical output, byte for byte. "Close enough" hides the next change in the same noise.

Each cause in detail: https://reamerlabs.com/faq/backtest-results-change-between-runs
