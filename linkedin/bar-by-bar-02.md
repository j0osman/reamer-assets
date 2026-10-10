# How do I know if my backtest is overfit? | Bar by Bar #2

You can't prove a backtest isn't overfit, but you can test for the signs. An overfit result holds at one parameter set and collapses at its neighbours, fails on data it wasn't tuned on, rests on a handful of trades, or disappears once realistic costs are applied.

Six checks:

1. **Record every variation you tried.** The best of two hundred is weaker evidence than a strategy that worked first time.
2. **Look for a plateau, not a spike.** If a 20-bar lookback works and 18 and 23 work nearly as well, it's responding to something real. If 19 and 21 lose money, the backtest found a coincidence.
3. **Hold data back.** Keep a period untouched while you develop, and test on it once at the end.
4. **Break the profit down** by trade, period and instrument. A strategy that depends on three trades has a sample size of three.
5. **Set realistic costs before sweeping.** Parameter searches drift toward settings that trade more often, and those are the settings costs hurt most.
6. **Confirm identical inputs give identical output.** If two runs of the same backtest differ, every comparison above is unreliable.

A strategy that passes all six can still fail live. One that fails any of them likely will.

The full checklist, including walk-forward testing: https://reamerlabs.com/faq/is-my-backtest-overfit
