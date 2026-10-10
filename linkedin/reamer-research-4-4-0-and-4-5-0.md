# Reamer Research 4.4.0 and 4.5.0 are out

I've just shipped two releases of Reamer Research.

4.4.0 widens what the engine runs on and what it reports:

- **Datasets larger than RAM.** Runs read memory-mapped bar files, and a bundled converter builds them from your CSVs with a parse report.
- **Monte Carlo resampling.** It resamples a run's per-trade returns, seeded and bit-identical for any thread count.
- **An equity curve in every result,** the same curve the drawdown is measured on.
- **One plainly named function for each job.**

4.5.0 widens what a strategy can do. On every bar, it now:

- **Sees its whole book.** Position, average entry, the stop and target in force, unrealised P&L, and every resting order.
- **Manages what is open.** It can re-price an order, move a stop to breakeven or into profit, close all, or cancel all. Each change applies from the next bar, so nothing it has already seen gets re-settled.
- **Can be stopped mid-run.** A progress callback reports how far a long sweep has got and can stop it. The result is the same with or without the callback.
- **Sends the same orders live.** A relay in the library sends the orders a strategy returns in a backtest to a running Reamer Server and reads the fills back as the same position state. Your logic, order structure and instrument names carry over unchanged.

Every stop or target move is recorded in the result's order log, so you can trace the trade history from entry to exit.

This is C ABI version 7. From here on, ABI versions are additive: a program built against ABI 7 runs unmodified on newer libraries.

Both releases, and how to move to ABI 7: https://reamerlabs.com/blog/reamer-research-4-4-0-and-4-5-0
