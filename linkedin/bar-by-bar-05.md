# How realistic do slippage and spread need to be in a backtest? | Bar by Bar #5

Realistic enough that the result survives them. Costs don't need to be perfect, but they decide more strategies than any parameter does.

Take a strategy that earns 0.06% per trade before costs and trades 1,000 times a year. A round-trip cost of 0.02% leaves two thirds of the edge. At 0.05% it leaves one sixth. At 0.07% it loses money. None of those costs is unusual.

What "realistic" means in practice:

- **Per instrument.** One cost for a whole universe undercharges thin instruments, so the backtest drifts toward trading them.
- **From your own fills.** Your broker's quotes and statements, not a round number. With no live fills yet, err high.
- **Wider when the market moves.** Spread and slippage widen in fast markets and just after a bar opens, exactly when breakout and stop entries fill.
- **Set before you tune.** A sweep with no costs selects the settings that trade most, which costs hurt most.

Then stress-test: rerun at 1.5x and 2x every cost. If the result degrades gently, the edge is bigger than your uncertainty. If it collapses at 1.5x, any error in the estimate decides the outcome.

The comparison only works if the runs differ in nothing but the costs.

The four costs to model, and the strategies most exposed: https://reamerlabs.com/faq/how-realistic-slippage-and-spread
