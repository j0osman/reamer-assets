---
title: Deploying Reamer Research
description: Verify, license, build and run Reamer Research: the C ABI entry points, linking on Linux and macOS, and the strategy callback contract.
group: integration
order: 1
product: research
source: DEPLOYMENT.md
---

Sequences everything needed to go from "I have the ABI header and
library" to "my backtest is running and licensed." Each step points at
the doc that owns that surface rather than restating it.

**Most integrators run a first backtest within an hour of extracting the
release tarball,** using the `reference-python` or `reference-cpp` wrapper
against one of the template strategies in `reference-python/templates/`
(e.g. `buy_and_hold.py`).

## Step 1 — Get the header and library

Download the versioned tarball from a tagged release
(`research-kit-<version>-linux_x86_64.tar.gz`, with a macOS arm64
variant).

**Verify the download before extracting it** — see "Verify your download"
below. Verifying first is both the correct order and the only order in
which the commands work as printed: the `.tar.gz`, `.sha256`,
`.sha256.sig`, and `.tar.gz.sig` are downloaded as siblings, and
extraction creates the `research-kit-4.5.0-linux_x86_64/` directory
*next to* them, not around them.

Once verification passes, extract — see "Extract the archive" at the end
of this step for the commands, which must run after the verification
above.

Contents: `reamer_research_abi.h` (the public C ABI header), the compiled
shared library (`lib/libreamer_research.so`; `.dylib` on macOS), plus this
doc set. The library is
everything needed to link and call `reamer_run_backtest()` —
there is no separate process to start. Linux and macOS only — Windows is
not a supported build target. See `SUPPORTED_PLATFORMS.md` for the exact
supported OS versions, CPU architectures, and glibc floor.

**Shared libraries `libreamer_research.so` needs at runtime** (Linux):

| Library | Why |
|---|---|
| `libc.so.6`, `libm.so.6` | glibc, per the floor in `SUPPORTED_PLATFORMS.md`. |
| `libstdc++.so.6` | C++ standard library. |
| `libgcc_s.so.1` | GCC support runtime (unwinding). |

Nothing else. Confirm with `ldd lib/libreamer_research.so` on your own
host. This matters on a minimal or distroless base image: those images
routinely omit `libstdc++`/`libgcc_s`, and the load then fails with a
loader error naming the missing object rather than anything of ours.
Install your distribution's C++ runtime package (`libstdc++6` on
Debian/Ubuntu) or use a base image that includes it.

### Verify your download

Every release publishes a `.sha256` checksum, a detached Ed25519
signature over that checksum (`.sha256.sig`), and a detached Ed25519
signature over the tarball's own bytes (`.tar.gz.sig`), alongside the
tarball, and the tarball itself contains a `SHA256SUMS` manifest covering
every file inside it.

**Run every command in this section from the directory holding the four
downloaded files** — the `.tar.gz`, the `.sha256`, the `.sha256.sig`, and
the `.tar.gz.sig`. Do not `cd` into the extracted archive first; the four
files are siblings of that directory, not members of it.

1. Check the tarball wasn't corrupted or tampered with in transit:

   ```
   sha256sum -c research-kit-4.5.0-linux_x86_64.tar.gz.sha256
   ```

2. Check the checksum's detached signature, using the `release-sign`
   binary shipped in this archive.

   `release-sign` ships *inside* the archive, so before extracting there
   is no copy on disk to run. Extract just that one file next to the
   downloads:

   ```
   tar xzf research-kit-4.5.0-linux_x86_64.tar.gz \
     --strip-components=2 \
     research-kit-4.5.0-linux_x86_64/bin/release-sign
   ```

   Then verify, from the directory holding the downloads:

   ```
   ./release-sign verify \
     research-kit-4.5.0-linux_x86_64.tar.gz.sha256 \
     research-kit-4.5.0-linux_x86_64.tar.gz.sha256.sig \
     b0ff9dcdcbd767ec78375629e10afec2baf18319df27e000ed6d8307cb93e68c
   ```

   Expected output: `signature verification OK`. A modified file reports
   `signature verification FAILED` and exits non-zero.

   Extracting the verifier from the archive it verifies is not a
   circularity this step can escape. Step 1's checksum is what binds the
   extracted `release-sign` to the archive you are about to trust, and
   the note on out-of-band key confirmation below is what turns internal
   consistency into provenance.

   Expected output: `signature verification OK`.

   The release-signing public key is:

   ```text
   b0ff9dcdcbd767ec78375629e10afec2baf18319df27e000ed6d8307cb93e68c
   ```

   It is the same key published in the Reamer Server kit's
   `DEPLOYMENT.md` — one signing keypair covers both products' release
   archives. This key is a separate trust domain from your license key:
   it attests that Reamer Labs built this archive, not that any
   particular license grant is valid.

   **Confirm the key out of band before trusting it.** A public key that
   arrives with the archive it authenticates proves internal consistency,
   not provenance — anyone able to replace the archive could replace the
   key printed beside it. Obtain the fingerprint above from a second,
   independent channel and compare: this page on reamerlabs.com is one
   such channel, **team@reamerlabs.com** confirms it on request, and a
   Reamer Server kit you received separately carries the same key. Then pin
   it in your own configuration management and verify future releases
   against your stored copy rather than against whatever the next archive
   contains.

3. Also check the detached signature over the tarball's own bytes, not
   only its checksum — a party who could replace both the tarball and
   its `.sha256` would produce a self-consistent pair that steps 1-2
   alone cannot catch:

   ```
   ./release-sign verify \
     research-kit-4.5.0-linux_x86_64.tar.gz \
     research-kit-4.5.0-linux_x86_64.tar.gz.sig \
     b0ff9dcdcbd767ec78375629e10afec2baf18319df27e000ed6d8307cb93e68c
   ```

   Expected output: `signature verification OK`. Same key, same
   circularity caveat, same out-of-band confirmation as step 2 — this is
   an additional check, not a replacement for it.

4. After extracting — see "Extract the archive" below, which must run
   first — verify every individual file in the tarball against the
   manifest shipped inside it, from the archive root:

   ```text
   sha256sum -c SHA256SUMS
   ```

   This catches a partial or selectively-tampered extraction that step 1's
   whole-archive checksum would also catch, but lets you re-verify
   individual files later without re-downloading the tarball.

### Extract the archive

Only after both verification commands above have passed, still from the
directory holding the downloads:

```
tar xzf research-kit-4.5.0-linux_x86_64.tar.gz
cd research-kit-4.5.0-linux_x86_64
ls include/reamer_research_abi.h lib/libreamer_research.so
```

Every later step in this document runs from that
`research-kit-4.5.0-linux_x86_64/` directory unless it says otherwise.

## Step 2 — Activate the license

Run the license management CLI (included in the same release tarball as
your product's corresponding ABI library):

```
./bin/reamer-license activate <license-key>
./bin/reamer-license status
```

**Running in a container?** Pin a stable hostname before you activate —
`LICENSING.md`'s "Self-Service Activation Failure Modes" explains why an
unpinned container looks like a new machine on every restart and gets
reactivation refused.

Activation is offline, per machine and per product, not per binary — it
writes this product's grant into `~/.config/reamer/activation.json`, keyed
to this machine's fingerprint. Run this kit's `reamer-license` once and
every binary on this machine that links Reamer Research passes its license
check afterward (a C/C++ backtest runner, a Python ctypes wrapper, a Go cgo
program — the language does not matter). Reamer Server is licensed
separately and activates with its own key.

One license key activates one machine at a time; see `LICENSING.md` for
the full seat and machine-migration policy.

`reamer_run_backtest()` refuses to run any backtest without an
active license, checked internally before anything else.

**Mid-backtest license expiry.** License validity is checked once, at
the start of the call, and not re-checked during the replay loop —
unlike `reamer-server`, which re-checks periodically while running (see
`reamer-server/DEPLOYMENT.md`'s "License expiry" note) because
a server process can stay up for days. This is a deliberate, evaluated
difference, not an oversight: a backtest call returns in well under a
second even at a 100-ticker portfolio scale, so there is no realistic
window for expiry to land mid-run against a term of 30 days or 1 year.

Internal timing of the replay path, with a no-op strategy callback to
isolate engine and ABI overhead from strategy cost, completed 100 tickers
× 5 years of daily bars in ~0.02s, and 500 tickers × 40 years in ~0.75s.
Those figures were measured on the hardware described in `BENCHMARK.md`
and are indicative, not a specification — your own strategy callback, not
the engine, will dominate wall-clock time in any real backtest.

## Step 3 — Link and call the library

Include `reamer_research_abi.h` and link against the compiled library:

```bash
gcc -c my_backtest.c -I./include
gcc my_backtest.o -L./lib -lreamer_research \
    -Wl,-rpath,'$ORIGIN/lib' -o my_backtest
./my_backtest
```

`libreamer_research.so` is a shared library, so the produced binary must be
able to find it at run time. `-Wl,-rpath,'$ORIGIN/lib'` records that lookup
relative to the binary itself, which is why the `./my_backtest` line above
works from the archive root. Without it the binary builds but fails at
launch with `error while loading shared libraries:
libreamer_research.so`.

**`$ORIGIN` resolves relative to wherever `-o my_backtest` is actually
written** — not to your current directory, and not to where `lib/` lives.
Run the `gcc` command above from the archive root, as shown, so the
produced binary lands next to `lib/`. Build it into any other directory
(a normal instinct, e.g. `-o build/my_backtest`) and it compiles clean but
fails at launch with the same `error while loading shared libraries`
message, because `$ORIGIN/lib` now points at a `lib/` that doesn't exist
next to the binary. Either keep the `-o` target in the archive root, or
replace `$ORIGIN/lib` with an absolute path to `lib/`. The alternative is
to set the path in the environment on every run:

```bash
export LD_LIBRARY_PATH="$PWD/lib:$LD_LIBRARY_PATH"
```

The Python binding resolves the library the same way. It searches `lib/`
relative to the archive root and then the system loader path; if neither
has it, it raises `OSError: Could not find libreamer_research. Set
REAMER_RESEARCH_LIB`. Point it at the shipped library explicitly:

```bash
export REAMER_RESEARCH_LIB="$PWD/lib/libreamer_research.so"
```

**Bars to run against.** You supply market data, as in-memory
`ReamerOhlcvBar` arrays to `reamer_run_backtest()` or as `.bin` files to
`reamer_run_backtest_files()` (see "Run from `.bin` files" below). So you can run
immediately, this kit ships a 100-bar daily fixture at
`reference-python/sample_data/AAPL.csv`
(`Date,Time,Open,High,Low,Close,Volume`). Parse those columns into a
`ReamerOhlcvBar[]` — `ts` as UTC epoch seconds, `notional` 0.0 and
`tick_count` -1 when unknown — and pass it as a one-ticker portfolio.

For the Python path the whole sequence already ships as a runnable
script — CSV to summary, using only files in this archive:

```bash
cd reference-python
export REAMER_RESEARCH_LIB="$PWD/../lib/libreamer_research.so"
python3 examples/quickstart_run.py
```

Run it first to confirm the path works end to end, then point the engine
at your own data.

Your program builds a `ReamerResearchStrategyVtable` (the strategy callback
interface), populates your backtest config and input data, and calls:

```c
ReamerResearchStrategyVtable strategy_vtable = {
    .user_data = NULL,
    .on_bar = my_strategy_on_bar
};

size_t lookback = 20;  // number of historical bars on_bar should receive
ReamerBacktestHandle result = {0};
ReamerResearchStatus status = reamer_run_backtest(
    bars_by_ticker, bar_counts, ticker_names,
    NULL /* ticker_configs */, NULL /* exogenous_paths */, ticker_count,
    &strategy_vtable, lookback, &exec_config,
    initial_capital, leverage, &result);

if (status == REAMER_RESEARCH_SUCCESS) {
    ReamerBacktestSummary summary;
    reamer_get_summary(result, 0.0, &summary);  /* 0.0: infer annualisation */
    printf("Net PnL: %.2f\n", summary.net_pnl);
    reamer_free_result(result);
}
```

A ticker's `ticker_id` is its array position in `bars_by_ticker[]`.

### ABI 7 additions (4.5.0)

ABI 7 breaks ABI 6. Rebuild against the shipped header. `CHANGELOG.md` lists
every change.

1. `on_bar` receives `position_states` and `open_orders`, and writes
   `out_actions`. See "Strategy callback" below and `EXECUTION_SPEC.md` §6b.
2. Optional vtable slot `on_progress` — progress calls between bars; return
   nonzero to stop the run with `REAMER_RESEARCH_ERROR_CANCELLED` (11).
3. `reamer_relay_*` — send orders to a running Reamer Server and read fills
   back. `REAMER_RESEARCH_ERROR_RELAY` (12). See `RESEARCH_TO_SERVER.md`.

### ABI 6 additions (4.4.0)

ABI 6 breaks ABI 5. Rebuild against the shipped header. `CHANGELOG.md` lists
every removal and rename.

1. `reamer_run_backtest_files()` — run over memory-mapped `.bin` files. See
   "Run from `.bin` files" below.
2. `reamer_write_bin()` and `bin/reamer-csv-build` — produce `.bin` files.
   Neither needs a license.
3. `reamer_run_monte_carlo()` — resample per-trade returns. See "Result
   access" below.
4. `reamer_get_summary()` takes `periods_per_year` — the Sharpe and Sortino
   annualisation.
5. Result JSON `equity_curve` and `equity_curve_ts` — realised equity after
   each closed trade. See `RESULT_JSON_SCHEMA.md`.
6. `REAMER_RESEARCH_ERROR_BAR_FILE` (10) — a bad `.bin` file.

### Strategy callback: `on_bar()`

`reamer_get_abi_version()` returns `7`, and `REAMER_ABI_VERSION` is `7` in
the shipped header. Implement one callback:

```c
typedef struct {
    void* user_data;
    size_t (*on_bar)(
        void* user_data,
        const ReamerOhlcvBar* window,       // borrowed: rolling-window data
        const uint8_t* window_valid,        // 1 = real bar, 0 = forward-filled
        size_t window_len,                  // bars per ticker (including current)
        size_t ticker_count,                // total tickers in the run
        const double* positions,            // signed position per ticker
        const char* const* exogenous,       // per-ticker sidecar value or NULL
        const size_t* exogenous_len,        // byte length of each exogenous[i]
        const ReamerPositionState* position_states, // full position per ticker
        const ReamerOpenOrder* open_orders, // resting orders, by order_id
        size_t open_order_count,            // entries in open_orders
        ReamerOrderRequest* out_orders,     // caller-allocated: up to max_orders
        size_t max_orders,                  // maximum orders to write
        ReamerOrderAction* out_actions,     // caller-allocated: up to max_actions
        size_t max_actions,                 // maximum actions to write
        size_t* out_action_count);          // set to the actions written (0 on entry)
    int (*on_progress)(                     // optional: NULL for no progress calls
        void* user_data,
        size_t steps_done,                  // bars replayed so far
        size_t steps_total);                // bars in the run; nonzero return stops it
} ReamerResearchStrategyVtable;
```

On each step, `on_bar` receives borrowed arrays (never freed or stored by
you — valid only for this one call), writes orders into the caller-provided
buffer, and returns the count actually written. No dynamic allocation
crosses the boundary in either direction. `on_bar` is required: a NULL slot
fails the run with `REAMER_RESEARCH_ERROR_INVALID_VTABLE`.

**on_progress.** Optional. The engine calls it between bars, on the same
thread as `on_bar`, at most once per 256 bars and per 100 ms, plus a final
call with `steps_done == steps_total`. `steps_done` never decreases. The
number of calls varies with machine speed; the result does not. Return 0 to
continue. Return nonzero to stop: the run returns `REAMER_RESEARCH_ERROR`
with code `REAMER_RESEARCH_ERROR_CANCELLED` and sets no handle. A stopped
run produces no result.

**window.** Row-major, `ticker_count * window_len` bars:
`window[i * window_len + j]` is ticker `i`'s bar `j`.
`window[i * window_len + window_len - 1]` is the current bar;
earlier entries are prior bars (up to your `lookback`).

**window_valid.** Same layout as `window`:

- **1** — a real bar for that ticker on that step.
- **0** — a forward-filled placeholder: either no bar existed for that
  ticker on that step, or it is a warmup row before the ticker's first
  real bar.

The bar's fields are still populated when the flag is 0 — the whole bar
is the ticker's last real one, carried forward, **including its original
`ts`**. Without the flag, a stale bar is indistinguishable from a fresh one
that happened to print the same price. This matters most in a multi-ticker
run, where tickers that did not trade on a given step are forward-filled to
keep the window rectangular.

Observed directly: a 2-ticker union run where ticker B has no bar on
step 2 delivers, on that step, `valid=0` with B's day-1 close and day-1
timestamp still in the slot, while ticker A shows `valid=1` with its
fresh day-2 bar. The stale `ts` lagging the step is a second way to
detect this, but `window_valid` states it directly and is the supported
signal.

**positions.** Length `ticker_count`: the engine's signed position per
ticker entering this bar (positive=long, negative=short, 0.0=flat).

**exogenous / exogenous_len.** Length `ticker_count`. NULL/0 when no
sidecar paths were supplied or no entry is visible yet. See
`EXOGENOUS_FORMAT.md`.

**position_states.** Length `ticker_count`: each ticker's position entering
this bar — signed `qty`, `avg_entry_price`, bracket `take_profit` and
`stop_loss`, `unrealized_pnl` and `entry_ts`. All fields are 0 for a flat
ticker.

**open_orders / open_order_count.** Every order still resting entering this
bar, sorted by `order_id`. To cancel one, write a `ReamerOrderRequest` with
`is_cancel = true` and `cancel_target_id` set to its `order_id`. The
cancel takes effect after the current bar replays: an order that fills on
the bar `on_bar` has just seen stays filled.
`open_orders` is NULL when `open_order_count` is 0.

**out_orders.** The order buffer format is `ReamerOrderRequest`, the
engine's internal order struct flattened for the C ABI: `ticker_id` (an
index into your ticker list, not a string), `order_type`/`kind`/`side`/`tif`
enums, doubles for `qty`/`limit_price`/`stop_price`, boolean flags for
`is_cancel`/`is_close`, etc. See `reamer_research_abi.h`'s
`ReamerOrderRequest` struct definition for the exact field layout and
comments.

**out_actions / max_actions / out_action_count.** Changes to existing orders
and positions, as `ReamerOrderAction` entries (64 per call):
`MODIFY_ORDER` re-prices a resting order and replaces its brackets,
`MODIFY_POSITION` replaces a position's take-profit and stop-loss,
`CLOSE_ALL` closes every position at market, `CANCEL_ALL` cancels every
order in `open_orders`. Set `*out_action_count` to the number written; it is
0 on entry. Modify and cancel-all actions take effect from the next bar;
`EXECUTION_SPEC.md` §6b has the timing and validation. The engine writes
`applied` and `reject_reason` into each entry after the bar replays and
never clears the buffer, so read the previous call's results at the start
of the next call.

### Result access

After a successful backtest, retrieve results via:

- `reamer_get_summary(handle, periods_per_year, &out_summary)` — 31 scalar
  summary metrics written into caller-allocated `ReamerBacktestSummary`: PnL
  and cost totals, trade and order counts, holding times, risk ratios and
  drawdowns. `periods_per_year` sets the Sharpe/Sortino annualisation (252
  daily, 52 weekly); `<= 0` infers it from closed-trade timestamps. See the
  struct in `reamer_research_abi.h` for every field.
- `reamer_get_closed_trades(handle, trades_buffer, max_trades, &out_count)`
  — poll-style accessor matching the ABI's "caller allocates, library fills"
  pattern: you provide a buffer and max count, the library fills it with up to
  that many closed trades and writes the actual count into `out_count`. This
  function has **no offset/cursor parameter**: callers must allocate a
  large-enough buffer up front. To detect if more trades exist than fit, compare
  `out_count` against `closed_trades_count` from `reamer_get_summary`.
- `reamer_run_monte_carlo(handle, &config, &out_stats)` — resamples the run's
  per-trade returns and fills `ReamerMonteCarloStats`: final-equity
  distribution, probability of loss, probability of ending below half of
  initial capital, max-drawdown percentiles.
  Pass `config` NULL for 10000 simulations on an entropy seed; read
  `effective_seed` back to reproduce a run.
- `reamer_get_last_error(&out_code, out_message, message_capacity)` —
  call immediately after any function returns `REAMER_RESEARCH_ERROR` to retrieve
  a structured error code and optional human-readable message on the calling
  thread (thread-local state, safe to use from multiple threads concurrently).
- `reamer_free_result(handle)` — release the result handle's
  backing memory (owned entirely on the library side).

For full reference and field definitions, see `reamer_research_abi.h`.

### Run from `.bin` files (datasets larger than RAM)

`reamer_run_backtest_files()` (ABI 6) memory-maps one `.bin` file
per ticker. The OS pages bars in as the run reaches them. The whole dataset
never sits in memory.

1. Write each ticker's bars once: convert a CSV with
   `bin/reamer-csv-build AAPL.csv AAPL.bin`, or call
   `reamer_write_bin(path, bars, count)` from code. Bars must be sorted
   strictly ascending by `ts`.
2. Run over the files. Pass `ticker_names[]` (required) and an inclusive
   `start_ts`/`end_ts` range in epoch seconds UTC. Pass
   `REAMER_RESEARCH_TS_UNBOUNDED_START` / `REAMER_RESEARCH_TS_UNBOUNDED_END`
   for every bar.

```c
const char* paths[] = {"aapl.bin", "spy.bin"};
const char* names[] = {"AAPL", "SPY"};
ReamerBacktestHandle h = {0};
reamer_run_backtest_files(paths, names, NULL, NULL, 2,
    REAMER_RESEARCH_TS_UNBOUNDED_START, REAMER_RESEARCH_TS_UNBOUNDED_END,
    &vtable, 20, &cfg, 100000.0, 1.0, &h);
```

Given the same bars, the result is identical to
`reamer_run_backtest()`. A bad file returns
`REAMER_RESEARCH_ERROR_BAR_FILE`. **Do not modify a file while a run maps it.**

### Callback execution time is not bounded

The engine does not limit how long your `on_bar` callback may take, and will
not interrupt it. A callback that never returns hangs the run.

This is deliberate. A wall-clock timeout would make identical inputs produce
different results under different machine load, which the byte-identical-replay
guarantee forbids. The engine will not trade determinism for a liveness check
it cannot make deterministic.

Stop a run between bars with `on_progress`. Track a deadline, or a stop
flag another thread sets, and return nonzero from `on_progress` once it
trips. The run ends at the next progress call with
`REAMER_RESEARCH_ERROR_CANCELLED` and no result, so no partial result ever
exists to differ between machines.

`on_progress` cannot stop a callback that never returns. To bound a single
`on_bar` call, supervise the process: a grid scheduler that already enforces
a wall-clock limit per job needs nothing further. Do not call into the
library from another thread to interrupt a run; no function in the header may
be called against an in-progress run. The only way to stop a hung callback is
to kill the process.

## Reference Implementations

Reference implementations in compiled and scripting languages are included in the
release archive alongside this doc:

- **`reference-cpp/`** — C++ RAII wrapper over the raw C ABI, providing ergonomic
  handle lifetime management and familiar `std::span` interfaces. Reference
  material, not a supported product surface — the same tier as
  `reference-python/`: a worked example meant to be read and adapted, with no
  test suite of its own. Validate any adaptation against your own tests before
  relying on it.

- **`reference-python/`** — Python ctypes binding wrapping the C ABI with
  numpy-backed arrays and Python-idiomatic callbacks. Real, tested implementation
  for the buy-and-hold / ATR breakout-bracket strategy scope shown in `templates/`.

- **`BENCHMARK.md`** — Performance comparison across pure-ABI (C/C++), C++ wrapper,
  and Python binding approaches on the same workload, showing overhead of each
  language boundary and wrapper layer.

- **`RESEARCH_TO_SERVER.md`** — After backtesting, this guide covers moving
  the same strategy logic to live execution via Reamer Server: what changes in
  the callback model, what stays the same, and where the server-side ABI docs live.
