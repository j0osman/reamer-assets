# Should you build your own backtester? | Bar by Bar #13

Build the parts where your edge lives. Take the rest from an engine you can check.

Split a research setup into layers and ask which ones a competitor could copy without hurting you:

1. **Signals.** Usually the edge.
2. **Data.** What you feed in and how it's cleaned. Often part of the edge.
3. **Sizing and portfolio rules.** Often part of the edge.
4. **Execution simulation.** Fills, costs, margin. Has to be right, but rarely an edge.
5. **Accounting and metrics.** Has to be right, never an edge.
6. **Plumbing.** Loading, aligning, reporting, sweeps. Never an edge.

A loop over bars takes a day to write. The cost is in what comes after: a written rule for a stop that gaps through, for a stop and a target in the same bar, for spread that widens, for partial closes and reversals, for margin, for instruments with different hours. Then tests for every rule, because a backtest has no answer key. Each wrong rule moves the result without warning.

Building the engine is right when:

- The fill model is your edge.
- Nothing available models what you trade, such as the order book or options across strikes.
- You must own every line of code.
- You're learning how a backtester works.

Before you adopt one, read its execution rules first. If they aren't written down, you're adopting code you'll have to test yourself. Then run a strategy whose answer you already know, twice, and check that the output matches.

Most setups end up in the middle: an adopted engine, with your own data, strategies, sweeps and reports around it.

The layers, the costs of each path, and how to decide: https://reamerlabs.com/faq/build-or-buy-backtester
