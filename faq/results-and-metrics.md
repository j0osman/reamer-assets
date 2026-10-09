---
title: What results and metrics does a run produce?
description: A Reamer Research run produces 31 summary metrics, every closed trade, every fill and every order with its outcome, as one schema-versioned JSON document. From Python the binding returns less, the main profit and cost totals and the closed trades, so the full set comes through the C interface.
stage: 4
order: 33
product: research
next: paper-trading, research-to-server-move, performance, integration-time
date: 2026-10-02
---

A Reamer Research run produces 31 summary metrics, every closed trade, every fill and every order with its outcome, as one schema-versioned JSON document. From Python the binding returns less, the main profit and cost totals and the closed trades, so the full set comes through the C interface.

## The 31 summary metrics

- **Profit and cost:** gross and net profit, with fees, slippage and overnight swap each reported separately.
- **Returns and ratios:** total return, win rate, profit factor, expected profit per trade, recovery factor, and the Sharpe, Sortino and Calmar ratios.
- **Orders:** how many were placed, filled, expired, cancelled and rejected.
- **Trades:** longest winning and losing streaks, and the average, longest and shortest holding times.
- **Drawdown:** the largest fall in equity, in money and as a fraction.

## The records behind them

- **Closed trades,** each with entry and exit time and price, size, costs, profit and the stop and target it carried.
- **Fills,** one per execution.
- **Orders,** every one the strategy placed, with its type, lifetime, outcome, fill price and, if rejected, the reason. Orders still open at the end are listed separately.
- **Futures rolls,** if you set them up.

## The JSON report

The whole result is one JSON file, with a schema document in the kit. It carries a schema version, and within a version fields are only ever added, so your parsing code keeps working on newer releases. The same inputs and seed give the same bytes, so two reports can be compared with a plain file diff.

## Limits worth knowing

- **Equity per closed trade, not per bar.** The report carries the realised equity curve after each closed trade, with timestamps, and the ratios are computed from per-trade returns. Build a daily curve yourself from the trade records if you need one.
- **Annualisation is yours to set.** Pass the number of periods per year and the Sharpe and Sortino ratios use it. Leave it unset and it is inferred from the typical gap between trade exits, which may not match your convention for a strategy that trades irregularly.
- **The Python binding** returns the main totals and the closed trades, up to 65,536 by default. For the full report, call the C interface.

See [How do I know if my backtest is overfit?](/faq/is-my-backtest-overfit.html) and [What is Reamer Research?](/faq/what-is-reamer-research.html)
