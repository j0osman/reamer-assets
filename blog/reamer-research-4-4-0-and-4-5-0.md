---
title: "Reamer Research 4.4.0 and 4.5.0: Larger-Than-RAM Data, Monte Carlo, Full Position State and a Relay to Reamer Server"
description: Reamer Research 4.4.0 runs datasets larger than RAM from memory-mapped bar files, adds Monte Carlo resampling and an equity curve, and gives every function a plain name. 4.5.0 gives a strategy its full position state and every resting order on each bar, lets it re-price orders, move stops and targets, close all or cancel all, adds a progress callback that can stop a run, and sends the same orders a backtest produces to a running Reamer Server. C ABI version 7, additive from here on.
date: 2026-10-09
tier: Release
---

Reamer Research 4.4.0 and 4.5.0 are out. 4.4.0 widens what the research engine runs on and what it reports: datasets larger than RAM, Monte Carlo resampling, an equity curve, and one plainly named function for each job. 4.5.0 widens what a strategy can do: it sees everything it holds, manages it bar by bar, and sends its orders to Reamer Server unchanged. This post covers both releases and what they mean for your code.

## 4.4.0: data, reports and one clean interface

### Datasets larger than RAM

`reamer_run_backtest_files()` runs over memory-mapped `.bin` files, one per ticker, with an optional inclusive start and end timestamp. The operating system pages bars in as the run reaches them, so a dataset larger than RAM runs on an ordinary machine. Results are identical to `reamer_run_backtest()` on the same bars.

Two ways to build the files ship with the kit, and neither needs a licence:

- `bin/reamer-csv-build` converts an OHLCV CSV. It detects the delimiter, columns and timestamp format, and prints a parse report: skipped rows, duplicate timestamps, and the first and last timestamp.
- `reamer_write_bin()` writes validated, sorted bars from your own program.

A missing, truncated or unsorted file returns `REAMER_RESEARCH_ERROR_BAR_FILE` before the run starts.

### Monte Carlo and the equity curve

`reamer_run_monte_carlo()` resamples a completed run's per-trade returns with replacement and reports final-equity, loss, ruin and maximum-drawdown percentiles. It is seeded, and bit-identical for any thread count, so a robustness check reproduces as exactly as the run it checks.

The result JSON now carries `equity_curve` and `equity_curve_ts`: realised equity after each closed trade, with timestamps. It is the curve the maximum drawdown is measured on, so a chart and the summary metric always agree.

`reamer_get_summary()` takes a `periods_per_year` argument for Sharpe and Sortino annualisation, 252 for daily bars or 52 for weekly. Zero or below infers it from the trade timestamps.

### One function per job, with plain names

Every export is now `reamer_*`: `reamer_run_backtest()`, `reamer_get_summary()` and so on, all in one version node. Each job has one entry point, `on_bar` has one signature, and the summary is one 31-metric struct. A ticker's ID is its position in the array you pass. A forward-filled row keeps the last real bar's timestamp, so a strategy can read how stale each ticker is from its window.

At the end of a run, `open_orders_end` lists every order still pending, and `order_log` records it as cancelled. A ticker passed with zero bars keeps every later ticker on its own index, name and position.

## 4.5.0: the strategy manages its own book

### The strategy sees its whole book

On every bar, `on_bar` now receives two things it previously had to track itself:

- **Position state for each ticker** (`ReamerPositionState`): signed quantity, average entry price, the take-profit and stop-loss in force, unrealised P&L and the entry timestamp.
- **Every resting order** (`ReamerOpenOrder`), sorted by `order_id`. A strategy can cancel one of its own orders by the ID it reads here.

The engine's own accounting is the source of these numbers, so a strategy reads the same state the fill rules act on. Python strategies get the same through `data.position_state(ticker)` and `data.open_orders()`.

### Four actions to manage what is open

A strategy can now return up to 64 actions per bar alongside its orders:

- `MODIFY_ORDER` re-prices a resting order and replaces its brackets.
- `MODIFY_POSITION` replaces a position's take-profit and stop-loss. A stop may move to breakeven or into profit, so trailing stops and break-even rules are a few lines in the strategy.
- `CLOSE_ALL` closes every position at market.
- `CANCEL_ALL` cancels every resting order.

Modify and cancel-all take effect from the next bar. A bar the strategy has already seen is settled against the levels it had when it saw it, which keeps the run free of look-ahead. Each action reports whether it was applied and, if not, why. A rejected action leaves the order or position as it was and the run continues. [`EXECUTION_SPEC.md` §6b](/docs/execution-spec.html#6b-order-and-position-actions) sets out the timing and validation rule by rule.

Every applied modification appears in the result JSON `order_log` as a `Modified` record, so a trade-by-trade diagnosis shows when each stop moved and to where. The result schema is now version 2.

### A run you can stop

An optional `on_progress` callback receives steps done and steps total between bars, at most once per 256 bars and 100 ms, with a final call at completion. Returning nonzero stops the run with `REAMER_RESEARCH_ERROR_CANCELLED`. A dashboard can show progress on a long sweep and stop a run that is no longer needed. A run's result is the same with or without the callback.

### From research to Reamer Server through the same orders

The Reamer Research library now ships a strategy relay: `reamer_relay_open()`, `reamer_relay_send()`, `reamer_relay_poll()`, `reamer_relay_get_positions()`, `reamer_relay_is_connected()` and `reamer_relay_close()`.

The relay sends the same `ReamerOrderRequest` a strategy returns from `on_bar` in a backtest to a running Reamer Server over its local strategy socket, with the backtest's order IDs and submission rules. Fills come back as the same `ReamerPositionState` the strategy reads in research. Your logic, your order structure and your instrument names carry over. A stop and target travel with the entry order, and your broker connector places them at the venue. `RESEARCH_TO_SERVER.md` in the kit runs one order through the reference gate to a simulated fill on Linux and macOS.

The pre-trade gate and the broker connector in Reamer Server stay yours to build. That is where your risk model and your venue live.

## Moving to ABI 7

ABI 6 and ABI 7 are both breaking changes. Rebuild strategies against the ABI 7 header:

- Rename calls from `reamer_research_*` to `reamer_*`, and drop the `ticker_ids` argument.
- Implement the single `on_bar` signature, with the 4.5.0 state and action parameters, and the vtable `{user_data, on_bar, on_progress}`. Pass NULL for `on_progress` if you do not need it.

Two behaviours to note:

- A cancel request now takes effect after the current bar replays. An order that fills on the bar the strategy saw stays filled.
- A strategy that writes no actions produces the same result JSON as 4.4.0, byte for byte, apart from `abi_version` and `schema_version`.

From ABI 7 on, interface versions are additive: a program built against ABI 7 runs unmodified against a newer library. The added state costs about 1 to 2% per `on_bar` call on the same machine and build flags.

The 4.5.0 kits include everything in 4.4.0 and are available now, and a 30-day trial key is a full licence on the same terms. See [pricing](/pricing.html) and [What is Reamer Research?](/faq/what-is-reamer-research.html)
