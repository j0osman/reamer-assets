# Why don't my backtest results match live trading? | Bar by Bar #1

Usually because the backtest assumed cheaper costs and easier fills than the market gives, used information that wasn't available at the time, or tested a slightly different strategy from the one that went live.

The gap rarely has a single cause. It's several small ones pushing in the same direction:

- **Costs modelled too kindly.** Zero spread and slippage, or one fixed cost per trade, understates exactly the moments breakout strategies trade in: near the open and in fast markets.
- **Fills the market wouldn't give.** Trading on the bar that produced the signal, stops filling at their level through a gap, or assuming the take-profit always wins when both bracket exits sit inside one bar.
- **Information from the future.** Earnings dates and fundamentals stored as revised values, and universes built from today's index members.
- **Luck mistaken for an edge.** One good parameter set, or a tool that gives different answers on different runs.
- **A different strategy live.** Indicators computed differently once they become running state, or symbols mapped to the wrong instrument.

Each one can be checked before real money finds it. No backtest closes the gap entirely; a careful one makes it small and known in advance.

The full breakdown, with a note on each cause: https://reamerlabs.com/faq/backtest-vs-live-results
