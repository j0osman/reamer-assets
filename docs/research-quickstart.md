---
title: Reamer Research quickstart
description: What ships in the Reamer Research kit, where to start, how to load your own data, and how licence keys work.
group: start
order: 2
product: research
source: README.md
---

## What this is

Reamer Research is a deterministic research engine for mid-frequency
strategies. It is delivered as a **C ABI and a shared
library**, not as a program you run.

There is no `reamer-research` binary in this kit. You write a program that
builds a strategy vtable, hands the engine your bars and config, and calls
`reamer_run_backtest()`. That call is your backtest. Any language
that can call a C ABI can drive it — this kit ships worked examples in C++
and Python.

`reamer_run_backtest()` takes **per-ticker execution config**
(per-instrument commission, slippage and spread in one run), **mandatory
instrument names** (so results carry your own symbols rather than a
positional placeholder), and **exogenous data sidecars** (any non-OHLCV
series, resolved point-in-time and delivered to `on_bar`). Read the results
with `reamer_get_summary()` — 31 metrics including Sharpe and Sortino at the annualisation you choose, Calmar,
drawdown and holding-time statistics — or dump the whole run with
`reamer_write_result_json()`.

**Determinism is the property to check first.** A run with a fixed `rng_seed`
is byte-identical across repeats, including the stochastic slippage and
spread path. Run any backtest twice and compare. Determinism holds in
*timing* as well as output: on a 64-core EPYC 9575F the native engine ran
2000 independent backtests with per-run throughput CV 0.59% (1.04× min→max
spread) — the single-threaded engine has no scheduler in its path to
introduce run-to-run drift. See `BENCHMARK.md`, "Many-Core Verification."

**Scope.** OHLCV-bar mid-frequency strategies, intraday to multi-day. Not
a high-frequency order-book (L2/L3) system, not an options platform, and
no corporate-action adjustment for equities — your input data must already
be split- and dividend-adjusted. `EXECUTION_SPEC.md` states the boundaries
precisely.

**This is a batch research engine.** Backtests run over historical bars;
there is no live market-data feed. Live execution runs on Reamer Server:
`reamer_relay_*` sends the orders your strategy returns to a running server
over its local strategy socket. The gate and the broker connector are yours
to build — see `RESEARCH_TO_SERVER.md`.

## New in 4.5.0 — ABI 7

ABI 7 breaks ABI 6: `on_bar` and the strategy vtable change. Rebuild
against the shipped header. `CHANGELOG.md` lists every change.

| Addition | What it does |
|---|---|
| `on_bar` `position_states`, `open_orders` | Full position state per ticker (`ReamerPositionState`) and every resting order (`ReamerOpenOrder`), sorted by `order_id`. |
| `on_bar` `out_actions` | `MODIFY_ORDER`, `MODIFY_POSITION`, `CLOSE_ALL`, `CANCEL_ALL` (`ReamerOrderAction`). `EXECUTION_SPEC.md` §6b has the timing. |
| `on_progress` vtable slot | Progress calls between bars. Return nonzero to stop the run with `REAMER_RESEARCH_ERROR_CANCELLED` (11). |
| `reamer_relay_*` | Sends `ReamerOrderRequest`s, brackets included, to a running Reamer Server and reads fills back as `ReamerPositionState`. `REAMER_RESEARCH_ERROR_RELAY` (12). Linux and macOS. |

## In 4.4.0 — ABI 6

ABI 6 breaks ABI 5. Rebuild against the shipped header. `CHANGELOG.md`
lists every removal and rename.

| Addition | What it does |
|---|---|
| `reamer_run_backtest_files()` | Runs over memory-mapped `.bin` files, one per ticker, with an inclusive `start_ts`/`end_ts` range. Datasets larger than RAM run. |
| `reamer_write_bin()` | Writes validated, sorted bars to a `.bin` file. Needs no license. |
| `bin/reamer-csv-build` | Converts an OHLCV CSV into a `.bin` file and prints a parse report. Needs no license. |
| `reamer_run_monte_carlo()` | Resamples a run's per-trade returns. Reports final-equity, loss, ruin and max-drawdown percentiles. Seeded and reproducible. |
| `reamer_get_summary(handle, periods_per_year, out)` | Sets the Sharpe and Sortino annualisation. `<= 0` infers it. |
| Result JSON `equity_curve`, `equity_curve_ts` | Realised equity after each closed trade, with timestamps. |
| `REAMER_RESEARCH_ERROR_BAR_FILE` (10) | A `.bin` file is missing, unreadable, truncated or unsorted. |

## What ships

| Path | What it is |
|---|---|
| `include/reamer_research_abi.h` | The entire integration contract. Every struct, vtable, and entry point, with lifetime rules stated inline. Read this first. |
| `lib/libreamer_research.so` | The engine you link against (`.dylib` on macOS). |
| `bin/reamer-license` | Activates this machine's license (`activate` / `status` / `deactivate`). |
| `bin/release-sign` | Verifies the detached Ed25519 signature over this archive's `.sha256`. See `DEPLOYMENT.md` Step 1. |
| `bin/reamer-exo-build` | Converts a two-column CSV into the `.exo.bin` sidecar `reamer_run_backtest()` reads. Offline converter — needs no license. See `EXOGENOUS_FORMAT.md`. |
| `bin/reamer-csv-build` | Converts an OHLCV CSV into the `.bin` bar file `reamer_run_backtest_files()` memory-maps. Offline converter — needs no license. See "Your own data" below. |
| `bin/research-bench` | Runs the pure-ABI backtest throughput benchmark on your own hardware after activation, reproducing `BENCHMARK.md`'s bars/sec figure. See "Measuring on your own hardware" there. |
| `reference-cpp/` | An RAII wrapper over the ABI plus four runnable examples. Source, not binaries — you are meant to read and adapt it. |
| `reference-python/` | A ctypes binding, strategy templates, and a runnable end-to-end quickstart. Same status: reference material, not a supported surface. |
| `reference-python/sample_data/` | A 100-bar daily OHLCV CSV, so the quickstart runs without you sourcing data first. |
| `VENDOR_QUESTIONNAIRE_PACK.md` | The standing answer to an infra/risk/procurement team's security questionnaire, support model, vulnerability disclosure policy, data-handling statement, and business-continuity/escrow note — hand it to your own security review. Published as the [vendor questionnaire pack](/docs/vendor-questionnaire.html). |
| `SECURITY_HARDENING.md` | The internal adversarial security pass already performed against the ABI/IPC and license boundaries: the defect classes found and closed, and the sanitizer + regression-test verification each fix carries. |
| `LICENSE.txt` | The licence terms governing this kit. `bin/reamer-license activate` reads this file from the kit root and will not activate until you accept it, so do not move `bin/` out of the extracted tree. |
| `SHA256SUMS` | Manifest covering every file above, for post-extraction verification. |
| `sbom.cdx.json`, `THIRD_PARTY_NOTICES` | CycloneDX SBOM and license text for every third-party component in the shipped library. |

Every binary answers `--version`, so you can confirm which build you have:

```bash
./bin/reamer-license --version    # reamer-license 4.5.0
./bin/release-sign --version      # release-sign 4.5.0
./bin/reamer-exo-build --version  # reamer-exo-build 4.5.0
./bin/reamer-csv-build --version  # reamer-csv-build 4.5.0
```

Docs: `EXECUTION_SPEC.md` (**authoritative** on fills, slippage, spread,
commission, margin, and roll), `RESULT_JSON_SCHEMA.md` (field-by-field
schema for the result document, including how `returns[]` is defined — read
it before recomputing any risk metric yourself), `EXOGENOUS_FORMAT.md`
(attaching non-OHLCV data to a run), `DEPLOYMENT.md` (quickstart and licensing),
`LICENSING.md` (seat policy, activation failure modes),
`SUPPORTED_PLATFORMS.md` (OS, arch, and glibc floor), `BENCHMARK.md`
(relative cost of each integration path), `MONITORING.md` (running backtests
unattended: what the library logs, what to alert on, what to send to
support), `RESEARCH_TO_SERVER.md` (for
teams moving a strategy to live execution), and
`VENDOR_QUESTIONNAIRE_PACK.md`/`SECURITY_HARDENING.md` (procurement and
the security-hardening record).

## Where to start

1. **See a number in about a minute — `bin/research-bench`.** The cheapest
   thing to try, and the one to do first. It is a prebuilt binary: no
   compiler, no toolchain, no environment setup. Activate a key (next
   section), then, from this directory:

   ```bash
   bin/reamer-license activate <license-key>   # once per machine
   bin/research-bench                          # a real number on your hardware
   ```

   It prints a bars/sec throughput and latency table for this box, directly
   comparable to `BENCHMARK.md`. That is the whole ask to get a first result
   — everything below is there when you want to go deeper. See "Measuring on
   your own hardware" in `BENCHMARK.md` for how to read the shape.
2. **`DEPLOYMENT.md`, Step 1.** Verify the download, then read Step 2 about
   licensing — see the next section first, because you need a key.
3. **`include/reamer_research_abi.h`.** Twenty minutes here answers most
   integration questions. It is the contract, and it is written to be read.
4. **Run something.** The fastest path to a *backtest* result, from this
   directory (keep the `-o` target here too — `'$ORIGIN/lib'` resolves
   relative to wherever the binary lands, so building it into another
   directory compiles clean but fails at launch; see `DEPLOYMENT.md` Step 3):

   ```bash
   g++ -std=c++20 -Iinclude -Ireference-cpp/include \
     reference-cpp/examples/buy_and_hold.cpp \
     -Llib -lreamer_research -Wl,-rpath,'$ORIGIN/lib' -o buy_and_hold
   ./buy_and_hold
   ```

   Or, in Python:

   ```bash
   export REAMER_RESEARCH_LIB="$PWD/lib/libreamer_research.so"
   cd reference-python && python3 examples/quickstart_run.py
   ```

   Both require an activated license. Run either twice — the output is
   byte-identical.
5. **`EXECUTION_SPEC.md`.** Before you size a position against any backtest
   result, read §1 (bid/ask and the slippage model), §2 (fill prices), and
   §10 (margin). Slippage is **not floored at zero** — negative slippage,
   meaning price improvement, occurs by design, and aggregate slippage cost
   can be negative. Model costs accordingly.

## Your own data

1. Convert each ticker's CSV to a `.bin` file:

   ```bash
   bin/reamer-csv-build AAPL.csv AAPL.bin
   ```

   The tool detects the delimiter, the column names and the timestamp
   format. It prints what it detected, the rows it skipped, the duplicate
   timestamps it dropped, and the first and last timestamp written. Check
   the report before you run.
2. Pass the `.bin` paths to `reamer_run_backtest_files()`. The files are
   memory-mapped, so a dataset larger than RAM runs. `start_ts` and
   `end_ts` select a date range without rewriting the files.

## Your license key

**This archive contains no license key, deliberately.** Your key arrives by
email on payment, with this kit's download link. It carries the term you
bought: a 30-day trial or 1 year. Nothing in this archive will activate on
its own.

`reamer_run_backtest()` refuses to run without an active license,
checked before anything else, so **nothing in "Where to start" step 3 will
run until a key is activated.** If your key did not arrive, write to:

> **support@reamerlabs.com** — missing or failed keys.

Activation is per-machine and keyed to a host fingerprint, so a
containerized deployment needs a word of setup — `LICENSING.md`'s "Self-Service Activation Failure
Modes" covers this, and is worth reading *before* you activate.

Activation is per machine and per product: this kit's `reamer-license`
activates Reamer Research keys only, and a machine running Reamer Server
too activates each product with its own key. See `LICENSING.md` for the
seat and machine-migration policy.

`reamer-license activate` presents `LICENSE.txt` and requires you to type
`accept` before it contacts the licence server. Declining consumes no seat.
The acceptance is recorded next to the activation record and is not
re-prompted for the same terms. For unattended provisioning, set
**`REAMER_ACCEPT_LICENSE=accept`** in the environment — activation without a
terminal and without that variable fails rather than defaulting to
acceptance. Amended terms carry a different hash and prompt again.

| For | Address |
|---|---|
| Purchases, invoices, commercial terms | hello@reamerlabs.com |
| Licensing and technical support | support@reamerlabs.com |
| Security correspondence | team@reamerlabs.com |

## What is not in this kit

Stated plainly, so you can scope the work rather than discover it:

- **No data.** Beyond the 100-bar sample that makes the quickstart run,
  sourcing, cleaning, and adjusting market data is yours. The engine
  validates bar structure, not data quality, and feeding it unadjusted
  equity data produces wrong results with no warning.
- **No source.** Reamer Research is closed-source. The ABI header is the
  contract; nothing here asks you to compile the product, and no document
  in this archive should tell you to. If one does, that is a defect —
  report it to support@reamerlabs.com.
- **No test suite.** The reference trees ship without tests. Validate any
  adaptation against your own.
- **No live market data.** The relay sends orders to Reamer Server; it
  does not feed bars to `on_bar`. See "What this is" above.

## Platform

Linux x86-64 and macOS arm64. Windows is not a supported target. See
`SUPPORTED_PLATFORMS.md` for exact OS versions and the glibc floor —
and note that if you are deploying Reamer Server on the same image, the
server's floor is higher and governs the combined image.
