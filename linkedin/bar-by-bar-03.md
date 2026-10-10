# How should a backtest fill a stop and a target that both fall inside one bar? | Bar by Bar #3

The bar alone can't tell you which was hit first, so no answer is certain. A backtest should settle it by one stated rule, apply that rule the same way on every run, and let you check how much the result depends on it.

A bar that opened at 100, reached 103 and 97, and closed at 101 could have gone up first or down first. With a target at 102 and a stop at 98, one path is a win and the other a loss, from identical data.

The common rules:

- **Stop first:** the worst case. The safer error, but it can reject a strategy that works.
- **Target first:** the best case. A bracket strategy tested this way can look profitable when it isn't.
- **Bar shape, or the level closer to the open:** rules of thumb, right only part of the time.
- **Shorter bars or real ticks:** the most accurate, and the costliest to get.
- **A simulated path through the bar:** not the real path, but repeatable and checkable.

Two checks show how much it matters for your strategy: count the trades whose exit bar touched both levels, then run once stop-first and once target-first. If it only works target-first, the edge is in data the bars don't contain.

More on each rule, plus gaps and spread at the open: https://reamerlabs.com/faq/stop-and-target-in-same-bar
