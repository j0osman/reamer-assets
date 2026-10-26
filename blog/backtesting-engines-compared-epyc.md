---
title: "Four Backtesting Engines on a 64-Core EPYC: What One Backtest Costs"
description: I ran Backtrader, NautilusTrader, LEAN and Reamer Research on the same data and strategies on an AMD EPYC 9575F, and measured how wall time and memory grow with bars, tickers and parallel copies. Reamer Research had the highest throughput and the lowest memory per run. Two of the four outcomes I stated against it in advance occurred, and both are reported here.
date: 2026-10-26
tier: Findings
topics: backtesting, benchmark, manycore, reproducibility, algorithmic-trading
cites: fourway
---

When you research a strategy, the limit is turnaround: how many backtests one machine finishes per hour, and how much memory each one needs. A single headline speed does not answer that, because the cost of a backtest grows in three directions: with the length of the data, with the number of instruments, and with the number of copies that run at once. I measured all three for four engines. The paper with the full method is public: [What a Backtest Costs](https://doi.org/10.6084/m9.figshare.34787745).

A disclosure first. I build and sell Reamer Research, one of the four engines. So the study is set up so that you do not have to trust me. The other engines ran at their defaults. The one patch to an engine is disclosed, with both the stock and the patched results. The runners and the raw results are [public](https://doi.org/10.6084/m9.figshare.34785729). And before I saw any result, I wrote down four outcomes that would count against Reamer Research. Two of them occurred. They are below, not in a footnote.

## The setup

- **Engines:** Backtrader 1.9.78.123, NautilusTrader 1.231.0, QuantConnect LEAN at commit f3d841e, and Reamer Research 4.6.0 through its C++ and Python paths. The Python results include one binding fix that ships in 4.6.1.
- **Machine:** one AMD EPYC 9575F, 64 cores, on 2026-10-25.
- **Data:** 58,942 real GBP/USD 15-minute bars, repeated to build the longer tiers.
- **Strategies:** two, each written once per engine in that engine's own idiom. The minimal strategy reads the close on each bar and does nothing else, which measures the cost of delivering one bar. The realistic strategy is a 20-bar Donchian channel breakout that holds +1,000 or −1,000 units and reverses in one order.

## One backtest

On 500,000 bars of the realistic strategy, one backtest took 0.36 s on Reamer Research's C++ path and 3.4 s on its Python path. The other engines took 13.6 to 22.1 s, and stock NautilusTrader took 76.9 s, for a reason given below. In bars per second, the C++ path processed 1.39 million and the Python path 148,000, against 6,500 to 37,000 for the other engines.

Breadth told the same story. At 100 tickers, Reamer Research's Python path finished in 14.9 s, against 94.8 s for the next fastest engine.

## Many copies at once

A parameter sweep runs many copies side by side, so I ran 1 to 64 copies of the 500,000-bar realistic backtest. Every engine that completed gained throughput up to one copy per physical core. At 64 copies, Reamer Research's C++ path completed 332,249 backtests per hour, against 11,242 for patched NautilusTrader and 11,125 for Backtrader. LEAN completed at 1 and 8 copies only: at 32 copies, every copy stalled at start-up.

The machine was rented at €2.83 an hour. At each engine's best measured rate, the hardware cost of 1,000 of these backtests was about €0.0085 on the C++ path, €0.067 on the Python path, €0.25 for Backtrader and for patched NautilusTrader, and €2.19 for LEAN at 8 copies. That counts hardware only. It leaves out your time, your data and licences, and three of these engines cost nothing to license.

## What went against Reamer Research

- **Memory on long single-ticker runs.** Above about 270,000 bars of the minimal strategy, Backtrader used less memory than Reamer Research's Python path: 209 MB against 309 MB at 500,000 bars. The C++ path still used the least memory of any engine at every bar count it ran.
- **Parallel efficiency.** At 64 copies, the C++ path had the lowest parallel efficiency of the engines, 0.53 against 0.63 to 0.72, although its throughput kept rising. I did not measure the cause. Memory bandwidth and the shared L3 cache are candidates.

## What users of the other engines should know

- **NautilusTrader 1.231.0** records PnL at a cost that grows with the square of the trade count. That is why the stock engine took 76.9 s. The cost was already reported upstream as issue #4434, and the maintainers say 2.0 removes it. The paper reports both the stock and the patched results. NautilusTrader 1.231.0 also pays an extra cost per instrument in its position snapshots.
- **LEAN** has the largest fixed cost, 3.3 to 3.9 s per start. It hung in 14 of 22 multi-ticker attempts on this machine, and at 32 parallel copies every copy stalled at start-up.

## Reproducibility

Every completed run of every engine repeated its result. Reamer Research's full closed-trade ledger was bit-identical across 20 runs on one build.

## Beyond speed

Speed is not the whole choice, and the paper also compares, with sources, what a benchmark cannot capture. LEAN has the broadest asset, data and order model, and a hosted path to live trading. NautilusTrader has a rich order and data model and built-in live adapters on one code path. Backtrader is pure Python and easy to read and change, but has had no release since 2023. Reamer Research has a small native order set that strategies extend in code: market, limit and stop orders, brackets on orders and on the open position, GTC, IOC and GTD, and modify, cancel and close-all actions. It runs on bars with a simulated intrabar tick path, not on tick or order-book data, and it is closed source and paid.

## The caveats

One machine, one date, one instrument series and two strategy shapes, with each engine at its defaults plus the listed changes. A strategy with heavy indicators or a model in the loop spends more of its time in its own code, and the engine differences then matter less. The runners are published so that anyone can repeat this with tuned configurations.

The full method, every table and the sourced comparison are in the preprint, [What a Backtest Costs](https://doi.org/10.6084/m9.figshare.34787745). The raw results, runners and dataset builders are archived openly at [doi.org/10.6084/m9.figshare.34785729](https://doi.org/10.6084/m9.figshare.34785729).
