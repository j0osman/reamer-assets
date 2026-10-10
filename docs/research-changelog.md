---
title: Reamer Research changelog
description: Every customer-visible change to Reamer Research by release, including C ABI version bumps.
group: changelogs
order: 1
product: research
source: CHANGELOG.md
---

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Every
entry below describes a customer-visible change to the Reamer Research
product — the C ABI, the shipped reference material, or the kit
contents — not internal refactors. Updated at release, alongside the tag.

Version numbers here are the release version (`kReamerVersion`), which is
shared with Reamer Server and moves independently of `REAMER_ABI_VERSION`.
ABI bumps are called out explicitly where they occur.

## [4.5.0] - 2026-10-09

**ABI bump: `REAMER_ABI_VERSION` 6 → 7. Breaking.** Rebuild strategies
against the ABI 7 header.

### Added
- `on_bar` receives `position_states` (one `ReamerPositionState` per ticker:
  signed `qty`, `avg_entry_price`, `take_profit`, `stop_loss`,
  `unrealized_pnl`, `entry_ts`) and `open_orders`/`open_order_count` (every
  resting order as a `ReamerOpenOrder`, sorted by `order_id`). A strategy
  can now cancel its own resting order by the `order_id` it reads here.
- `on_bar` writes order and position actions to `out_actions`
  (`ReamerOrderAction`, up to 64 per call): `MODIFY_ORDER` re-prices a
  resting order and replaces its brackets, `MODIFY_POSITION` replaces a
  position's take-profit and stop-loss, `CLOSE_ALL` closes every position at
  market, `CANCEL_ALL` cancels every resting order. Modify and cancel-all
  take effect from the next bar; a stop may move to breakeven or into
  profit. Each action reports `applied` and `reject_reason`.
  `EXECUTION_SPEC.md` §6b has the timing and validation.
- Optional vtable slot `on_progress(user_data, steps_done, steps_total)`.
  The engine calls it between bars, at most once per 256 bars and 100 ms,
  plus a final call with `steps_done == steps_total`. Return nonzero to stop
  the run: `reamer_run_backtest()` and `reamer_run_backtest_files()` return
  `REAMER_RESEARCH_ERROR_CANCELLED` (11) and no handle. NULL means no calls.
  A run's result is the same with or without it.
- Result JSON `order_log` records each applied modification with status
  `"Modified"`: the order's snapshot under its own `id`, or `id` 0 for a
  position's brackets.
- reference-cpp passes the new arrays to the strategy function as spans,
  plus `std::span<ReamerOrderAction>` and a `size_t&` action count.
  reference-python exposes them as `data.position_state(ticker)`,
  `data.open_orders()` and `data.action_results()`; `engine/actions.py`
  builds the four actions, and a strategy's `on_progress(done, total)`
  method, if defined, receives progress calls and stops the run by
  returning a truthy value.
- Strategy relay to Reamer Server: `reamer_relay_open()`,
  `reamer_relay_send()`, `reamer_relay_poll()`,
  `reamer_relay_get_positions()`, `reamer_relay_is_connected()`,
  `reamer_relay_close()` and `ReamerRelayHandle`. Sends the same
  `ReamerOrderRequest` a strategy returns from `on_bar` to a running server
  over its local-mode strategy socket, with backtest order IDs and
  submission rules, and reads fills back as `ReamerPositionState`.
  `take_profit` and `stop_loss` travel as Reamer Server ABI 4 order fields;
  the server tracks the exits as `<order_id>:tp` and `<order_id>:sl`, your
  broker connector places them at the venue, and their fills move the
  reported position. Brackets on a close (`is_close`) are rejected: they
  attach to an entry. New error
  code `REAMER_RESEARCH_ERROR_RELAY` (12). Linux and macOS.
  `RESEARCH_TO_SERVER.md` has a one-order round trip against the reference
  gate. The gate and the broker connector remain yours to build.

### Changed
- `on_bar` takes three new parameters before `out_orders`:
  `const ReamerPositionState* position_states`,
  `const ReamerOpenOrder* open_orders`, `size_t open_order_count`, and three
  after `max_orders`: `ReamerOrderAction* out_actions`, `size_t max_actions`,
  `size_t* out_action_count`. The vtable becomes
  `{user_data, on_bar, on_progress}` (24 bytes).
- A cancel request (`is_cancel`) takes effect after the current bar replays,
  not before. In 4.4.0 a strategy that had seen bar N could cancel an order
  bar N would have filled. Runs that cancel orders can now differ from
  4.4.0: an order that fills on the bar the strategy saw stays filled.
  `EXECUTION_SPEC.md` §6.
- Result JSON `schema_version` 1 → 2: `order_log` `id` repeats on
  `"Modified"` records. `summary.total_orders` does not count them.
- A strategy that writes no actions produces the same result JSON as 4.4.0,
  byte for byte, apart from `abi_version` and `schema_version`.
- Cost: about 1-2%. `research-bench` P50 is 1.98–2.00 µs per `on_bar` call
  against 1.96 µs on 4.4.0 (same machine, same build flags, steady runs).

## [4.4.0] - 2026-10-09

**ABI bump: `REAMER_ABI_VERSION` 5 → 6. Breaking.** Rebuild against the ABI 6
header.

### Removed
- `reamer_research_run_backtest()` (v1) and `reamer_research_run_backtest_v2()`.
- The 7-field `ReamerBacktestSummary` and `reamer_research_get_summary()`.
- Vtable slots `on_bar` (v1), `on_bar_v2` and `on_bar_v3`.
- The `ticker_ids[]` parameter of `reamer_run_backtest()`. A ticker's
  `ticker_id` is its array position.
- `REAMER_RESEARCH_ERROR_INVALID_TICKER_ID` (3). Other codes keep their
  values.

### Changed
- **Every exported function is renamed from `reamer_research_*` to
  `reamer_*`.** `reamer_research_run_backtest_v3()` is now
  `reamer_run_backtest()`, and `reamer_research_get_summary_v2()` is now
  `reamer_get_summary()`.
- `ReamerBacktestSummaryV2` is renamed `ReamerBacktestSummary` (31 metrics,
  248 bytes).
- `reamer_get_summary(handle, periods_per_year, out)` takes the Sharpe and
  Sortino annualisation (252 daily, 52 weekly). `<= 0` infers it from
  closed-trade timestamps, as before. reference-cpp's `summary()` and
  reference-python's `periods_per_year` config key pass it through.
- Docs: `calmar_ratio` is `total_return_pct` over relative drawdown and is
  not annualised. The header and `RESULT_JSON_SCHEMA.md` said it was.
- `ReamerResearchStrategyVtable` is `{user_data, on_bar}` (16 bytes).
  `on_bar` takes the former `on_bar_v4` signature: `window`, `window_valid`,
  `window_len`, `ticker_count`, `positions`, `exogenous`, `exogenous_len`,
  `out_orders`, `max_orders`.
- All 11 exports sit in one version node, `REAMER_RESEARCH_2.0`. The 1.0-1.3
  nodes are gone.
- reference-cpp: `run_backtest<>()` takes a required `ticker_names` argument
  and no `ticker_ids`; the callable takes `(window, ticker_count, positions,
  window_valid, out_orders)`. `run_backtest_v2<>()` is removed.
- reference-python: the ctypes mirrors follow the new vtable and summary.
- `on_bar` documentation: a forward-filled row keeps the last real bar's
  `ts`, so per-ticker staleness is readable from the window.

### Added
- **`reamer_run_backtest_files()`**: run a backtest over memory-mapped `.bin`
  files, one per ticker, with an optional inclusive `start_ts`/`end_ts`
  range. Datasets larger than RAM run. Results are identical to
  `reamer_run_backtest()` on the same bars.
- **`reamer_write_bin()`**: write validated, sorted bars to a `.bin` file.
- **`REAMER_RESEARCH_ERROR_BAR_FILE` (10)**: a `.bin` file is missing,
  unreadable, lacks its header, is truncated, or is unsorted.
- **`bin/reamer-csv-build`**: converts an OHLCV CSV into a `.bin` file for
  `reamer_run_backtest_files()`. Detects delimiter, columns and timestamp
  format; prints a parse report (skipped rows, duplicate timestamps, first
  and last timestamp). Needs no license.
- **`reamer_run_monte_carlo()`**: resamples a completed run's per-trade
  returns with replacement and reports final-equity, loss, ruin and
  max-drawdown percentile stats in `ReamerMonteCarloStats`. Seeded, and
  bit-identical for any thread count. A run with fewer than two closed trades
  returns all zeros. License-gated.
- **Result JSON `equity_curve` and `equity_curve_ts`**: realised equity after
  each closed trade, with timestamps. It is the curve
  `equity_drawdown_maximal` is measured on, floored at 0. `schema_version`
  stays 1 (fields added).

### Fixed
- Result JSON `open_orders_end[]` was always empty. An order still pending
  at the end of a run now appears there as `Pending`, and in `order_log[]`
  as `Cancelled`.
- `reamer_run_backtest()`: a ticker passed with zero bars no longer shifts
  every later ticker down one asset index. Before the fix, `on_bar`'s
  `ticker_id`, `positions[]` and closed-trade names for later tickers pointed
  at the wrong instrument.

## [4.3.2] - 2026-10-03

No change to the C ABI or the Reamer Research binary behaviour. Released
alongside a Reamer Server fix to the `license.expiring` warning schedule.

### Added
- **`LICENSING.md` "Renew on the same machine"**: deactivate the old key,
  then activate the new one.

## [4.3.1] - 2026-09-17

No change to the C ABI. `REAMER_ABI_VERSION` stays at 5, every entry point and
struct is untouched, and a program compiled against an earlier header links
and runs against this build unmodified. The version moves with the shared
release number (`kReamerVersion`, one number across both products); the
substantive documentation change this release is on the server side (see the
Reamer Server changelog).

### Changed
- **The written ABI versioning policy is clarified.** The advance-notice and
  deprecation sections are resolved to the actual policy: support the latest
  version only, no sunsetting because the surface is additive-only, and a
  version bump is communicated through the changelog at release. The document
  is also rewritten in plain technical english with no em or en dashes.

## [4.3.0] - 2026-09-13

No change to the C ABI. `REAMER_ABI_VERSION` stays at 5, every entry point and
struct is untouched, and a program compiled against an earlier header links
and runs against this build unmodified. This release carries new benchmark
evidence and the documentation built on it; the version moves with the shared
release number (`kReamerVersion`, one number across both products).

### Changed
- **`BENCHMARK.md` now includes a many-core verification section** measured on
  a 64-core AMD EPYC 9575F. The pure-ABI native engine held a per-call latency
  tail essentially flat across 2000 independent backtests (P99.9 within ~3% of
  P50) with per-run throughput CV 0.59% (1.04× min→max spread) — the
  quantitative form of the determinism claim. A stress/capacity pass on the
  same box reported no bottlenecks, ingesting a 50-million-bar universe
  (10,000 tickers × 20 years) without failure.

## [4.2.1] - 2026-09-12

No change to the C ABI. `REAMER_ABI_VERSION` stays at 5, every entry point and
struct is untouched, and a program compiled against 4.1.15 links and runs
against this build unmodified. The fix is entirely inside the shipped Python
reference binding; the library and every other reference tree are unchanged.

### Fixed
- **Multi-ticker strategies in the Python reference binding now read each
  ticker's own bars instead of aliasing every ticker to the first.** The
  ctypes binding hands `on_bar` one `_TickerView` per instrument over the
  flat, row-major `(ticker_count, window_len)` window the ABI delivers. Every
  view was constructed on the same base pointer, so all tickers reported
  ticker 0's OHLCV; a strategy reading two instruments saw the first one
  twice. The binding now offsets each view by `ticker_id * window_len` into
  the window before constructing it, so each ticker reads its own row. Single
  ticker runs were unaffected (offset 0), which is why this survived to now.

  This is a read-side view bug only. The engine's execution path — tick
  simulation between bars, the synthetic-tick PRNG, spread and slippage
  randomization, and per-ticker wall-clock alignment of asynchronous bars —
  was never involved and is unchanged. Fills, results, and RNG draws for any
  run that did not read a multi-ticker window are byte-identical to 4.2.0.

No change to the C ABI. `REAMER_ABI_VERSION` stays at 5, every entry point and
struct is untouched, and a program compiled against 4.1.15 links and runs
against this build unmodified.

### Fixed
- **Malformed orders are now rejected at the ABI boundary instead of reaching
  the engine.** Order requests are validated in `ToEngineOrder` before any
  field is cast: the four enums (`order_type`, `kind`, `side`, `tif`) are
  range-checked, and `qty`, `limit_price`, `stop_price`, `take_profit` and
  `stop_loss` must be finite. A failing order is rejected with a reason
  naming the field — `qty is not finite`, `side out of range` — through the
  existing rejection path.

  This closes two paths that previously produced wrong answers rather than
  errors. A non-finite quantity on a **position reversal** bypassed the
  margin check entirely and filled: the run returned success with a NaN
  `net_pnl` and no indication anything had gone wrong. An out-of-range enum
  matched no comparison in the engine, so the order silently never became
  fill-eligible and sat pending for the life of the run.

  Inbound bars were already checked for finiteness at ingest; orders crossing
  the other way were not. Runs remain byte-identical for identical input:
  rejection consumes no RNG.

### Added
- **`MONITORING.md` ships in the kit.** For running backtests unattended:
  what the library writes to stderr and at what volume, the error codes worth
  alerting on, how to tell a run that failed from a run whose orders were all
  rejected (a distinction exit status does not make), what to capture for a
  support report, and what changes when running a grid.
- **`reference-cpp/examples/rejected_orders.cpp`.** A worked example of
  reading rejections out of a result. `summary()` and `closed_trades()`
  report what a strategy achieved; only the order log says what the engine
  refused, and it reaches callers through the result JSON.
- **`BacktestResult::result_json()`** in the reference C++ wrapper, exposing
  `reamer_research_get_result_json()`. The full result document — order log,
  rejection reasons, the `returns[]` series — was previously unreachable
  through the wrapper.

### Changed
- **The vtable's callback contract is written down.** `reamer_research_abi.h`
  now states what happens when a callback throws (it is caught at the ABI
  entry point and reported as `REAMER_RESEARCH_ERROR_PROCESS_FATAL`, but the
  run ends — there is no per-bar recovery), that callback execution time is
  deliberately unbounded and uninterruptible, and that `user_data` is passed
  through untouched. `DEPLOYMENT.md` covers the same ground for operators.

## [4.1.13] - 2026-09-05

No change to the C ABI. `REAMER_ABI_VERSION` stays at 5, every entry point
and struct is untouched, and a program compiled against 4.1.12 links and
runs against this build unmodified.

### Added
- **`LICENSE.txt` ships in the kit.** The archive previously carried no
  licence terms at all. The file sits in the kit root and is covered by
  `SHA256SUMS`.
- **Activation requires accepting the terms.** `reamer-license activate`
  now displays `LICENSE.txt` and requires an explicit `accept` before it
  contacts the licence server, converting activation from browsewrap to
  clickwrap. Declining consumes no seat, and the acceptance record — key,
  machine fingerprint, UTC timestamp, licence version, and the SHA-256 of
  the accepted text — is written next to the activation record only after
  activation succeeds. Amended terms hash differently and prompt again.

  For unattended provisioning set **`REAMER_ACCEPT_LICENSE=accept`**.
  Activation with no terminal and no such variable **fails** rather than
  defaulting to acceptance, so existing non-interactive provisioning that
  activates a licence must set it.

  `reamer-license` reads `LICENSE.txt` from the kit root, one level above
  `bin/`. Moving `bin/` out of the extracted tree makes activation fail
  with a message naming the cause.

## [4.1.12] - 2026-09-02

**`REAMER_ABI_VERSION` 4 → 5.** Purely additive. Four new exported
functions and one new
optional vtable slot; the library now exports eleven symbols across three
version-script nodes (`REAMER_RESEARCH_1.0` / `1.1` / `1.2`). Every
pre-existing entry point and struct is untouched, so a program compiled
against ABI 1-4 links and runs against this build **unmodified**. No
recompilation is forced by this release.

This bump exposes three capabilities that were already implemented in the
engine but unreachable through the ABI, which is the product.

### Added
- **`reamer_research_run_backtest_v3()`** — the entry point to write new
  integrations against. Adds per-ticker execution config, mandatory
  instrument names, and exogenous data sidecars in one signature.

  **Per-ticker execution config.** A sparse `ticker_configs[]` array
  applies distinct commission, slippage, spread, and swap terms per
  instrument within a single run. A NULL entry falls through to the
  global config. Resolution order is specified in `EXECUTION_SPEC.md`
  §0. **`rng_seed` is deliberately excluded from override**: it is
  forced from the global config when building each resolved entry, so
  differing costs between instruments can never introduce
  cost-correlated randomness into the synthetic tick path. Enforced in
  code, not only documented.

  **Mandatory `ticker_names[]`.** A NULL array is rejected at this entry
  point. Consequently every instrument name in a v3 run's results —
  `ReamerClosedTrade.ticker` and every `ticker` field in the result JSON
  — is the caller's own symbol, never the synthetic `"T%09zu"`
  placeholder the older entry points fall back to. This makes the
  research→server instrument-identity mapping *checkable* rather than
  merely required; see `RESEARCH_TO_SERVER.md` §2.

  **Exogenous data sidecars.** An optional `exogenous_paths[]` array
  attaches a non-OHLCV series per instrument. Delivered to `on_bar_v4`.

- **`on_bar_v4` vtable slot.** Extends `on_bar_v3` with per-ticker
  exogenous bytes and lengths. The vtable grows 32 → 40 bytes, which is
  safe *only* because the caller passes it by pointer, the library reads
  no slot it does not know about, and the header has mandated
  zero-initialization since ABI 2. Dispatch falls back
  `on_bar_v4 → v3 → v2 → on_bar`, so a strategy that sets only `on_bar`
  gets exactly v1 behavior.

  **Resolution is strictly point-in-time.** At each bar the engine
  resolves the latest entry with `ts <= current_bar_ts` and never a
  later one; a per-ticker cursor advances forward only, keeping this
  O(1) amortised rather than a per-bar search. Where no sidecar is
  attached or no entry qualifies yet, the strategy receives NULL and
  length 0. The bytes are engine-owned, valid only for the duration of
  the call, and are never parsed or validated by the engine.

- **`reamer_research_get_summary_v2()`** and `ReamerBacktestSummaryV2` —
  31 metrics where `get_summary()` reported 7: Sharpe, Sortino, Calmar,
  profit factor, recovery factor, expectancy, win rate, consecutive
  win/loss streaks, holding-time statistics, absolute and relative
  drawdown, and full order-outcome counts.

  `ReamerBacktestSummary` is **frozen at 56 bytes** and `get_summary()`
  returns byte-identical values to previous releases. The new metrics
  went into a separate struct reached through a separate entry point
  rather than growing the existing one, because growing a
  caller-allocated struct overruns a buffer sized by an older compiled
  header — silently.

  Sharpe, Sortino, and Calmar are annualised by `sqrt(periods_per_year)`,
  where `periods_per_year` is inferred from the median gap between
  consecutive closed trades' `close_timestamp` values, falling back to
  252 below two closed trades or on unparseable timestamps.

- **`reamer_research_get_result_json()`** and
  **`reamer_research_write_result_json()`** — the complete run as a
  single JSON document: summary, closed trades, fills, every order and
  its outcome, scale-in fills, roll events, and the per-trade return
  series. Two-call sizing (`buf = NULL` reports the required size
  including the NUL); an undersized buffer is rejected rather than
  truncated. Output is locale-independent by construction — doubles are
  formatted by the serializer's own Grisu2 path, never the C locale's
  `printf` — and this is enforced by a test running under `de_DE.UTF-8`.

  Replay scaffolding (`orders_by_step`, `tick_orders_by_ticker`,
  `prints_by_step`) is deliberately excluded: it carries no information
  not already in `order_log` and would multiply the document size.

- **`bin/reamer-exo-build`** — converts a two-column CSV into the
  `.exo.bin` sidecar `run_backtest_v3()` reads. Reports entry count,
  duplicates collapsed, and byte size; exits non-zero with a stderr
  message on unreadable, malformed, or empty input. **No license check**
  — it is an offline converter that links no engine code, so gating it
  would add an activation failure mode ahead of evaluation.

- **`EXOGENOUS_FORMAT.md`** — CSV authoring format, the `.exo.bin`
  binary layout, the point-in-time visibility rule, and CLI usage.
- **`RESULT_JSON_SCHEMA.md`** — field-by-field schema for the result
  document, with `schema_version` versioned independently of
  `REAMER_ABI_VERSION`.

  It documents `returns[]` explicitly, because the definition is not the
  one most consumers assume: `returns[i] = net_pnl / equity_at_entry`,
  clamped to `[-1, 1]`. The denominator is **account equity at entry,
  not the trade's own margin**, so it is not
  `closed_trades[i].return_pct / 100`. This is the exact series the
  Sharpe, Sortino and Calmar figures are computed from — recomputing
  risk metrics from `closed_trades[].net_pnl` will not reconcile with
  `summary`.

Both new documents ship in the kit and are enforced by the archive
verifier, not merely copied by the build script.

### Changed
- **`EXECUTION_SPEC.md`** gains §0 (per-ticker execution config
  resolution, including the `rng_seed` exclusion) and §13 (exogenous
  data resolution).
- **`RESEARCH_TO_SERVER.md` §2** — instrument-identity mapping is still
  required and still yours to build, but a `run_backtest_v3()` run's
  results now carry the same string identifiers Server's
  `intent.Instrument` uses, so the mapping can be asserted as a build
  step instead of trusted.
- The internal ABI versioning policy records the v5 row and the
  struct-growth hazard that motivated putting the new metrics in a
  separate struct.

## [4.1.11] - 2026-09-02

### Changed
- **Shipped archive filename renamed for clarity**: this kit is now
  `research-kit-<version>-<platform>.tar.gz` (was
  `reamer-research-<version>-<platform>.tar.gz`). Only the
  customer-facing archive filename moved; the extracted layout is
  unchanged.
