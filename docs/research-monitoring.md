---
title: Monitoring Reamer Research runs
description: Running Reamer Research backtests unattended: what the library logs, which error codes to alert on, and what to send with a support request.
group: operations
order: 4
product: research
source: MONITORING.md
---

For operators running backtests unattended — on a grid, in CI, or behind a
scheduler. Covers what the library emits, what to alert on, and what to capture
when a result looks wrong.

Reamer Research is a batch library, not a service. It exposes no metrics
endpoint, no health check, and no diagnostic socket: a run is a process that
exits, and process exit status plus captured stderr is the whole operational
surface. That is deliberate. Anything a daemon would need to expose over a
socket, a batch job carries in its own exit and logs.

If you are looking for the daemon-shaped surface (`/metrics`, `/health`, an
event bus), that belongs to Reamer Server, which is a long-lived process. Do
not expect it here.

---

## 1. What the library writes

**One line format, stderr only:**

```
[ERROR] <code>: <message>
```

Written whenever the library records an error. Nothing else goes to stderr —
no progress, no per-bar output, no warnings, no info lines. Volume is bounded
by the number of errors, which for a run that completes is zero.

The library never writes to stdout. Anything on stdout is your own program.

`<code>` is a short stable string — `process.fatal`, `license.invalid`, and
`abi.*` for the rest — suitable for grepping. `<message>` is human-readable and
not stable across versions: match on the code, never on the message text.

Every stderr line has a programmatic counterpart. Call
`reamer_get_last_error(&code, buf, sizeof buf)` immediately after any
function returns `REAMER_RESEARCH_ERROR` to read the same code and message on
the calling thread. Prefer that in automation: it is structured, and it is
thread-local, so a parallel grid worker never reads another thread's error.

The numeric codes are declared in `reamer_research_abi.h`
(`ReamerResearchErrorCode`). The ones an operator sees in practice:

| stderr code | Enum (numeric) | Meaning | Operator action |
|---|---|---|---|
| `license.invalid` | `LICENSE_MISSING` (5) | No active license on this machine | Activate; see LICENSING.md |
| `abi.invalid_bar` | `INVALID_BAR` (7) | A bar failed validation at ingest | Fix the input data |
| `abi.exogenous_load` | `EXOGENOUS_LOAD` (9) | An exogenous data file failed to load | Fix the path or the file |
| `process.fatal` | `PROCESS_FATAL` (6) | An exception reached the ABI boundary | Capture and send to support |
| `abi.invalid_vtable` | `INVALID_VTABLE` (1) | Caller-side integration error | Fix the calling code |
| `abi.invalid_handle` | `INVALID_HANDLE` (4) | Caller-side integration error | Fix the calling code |
| `abi.invalid_ticker_name` | `INVALID_TICKER_NAME` (8) | A ticker name failed validation | Fix the ticker list |

---

## 2. Distinguishing a bad run from a bad strategy

**This is the distinction that matters most in automation, and it is not
visible from exit status alone.**

A backtest that rejects every order it was given still succeeds. It returns
`REAMER_RESEARCH_SUCCESS`, produces a valid handle, and reports a summary —
one showing zero trades. Rejections are a strategy outcome, not a failure, so
they are recorded in the result, not on stderr.

A run that produced no trades because every order was malformed and a run that
produced no trades because the strategy correctly saw no setup are
indistinguishable from the outside. Check the result:

1. `reamer_get_summary()` gives `trades` and `closed_trades_count`.
2. Zero trades on a strategy you expect to trade is the signal to look
   further, not to alert on directly.
3. The order log carries a rejection reason per order. A rejected order names
   its cause in text (`qty is not finite`, `side out of range`, `insufficient
   margin`, `missing ticker`). Surface those to whoever owns the strategy.

Alert on **unexpected zero-trade runs** in a grid where most cells trade. A
single cell producing nothing is usually a real result; a whole sweep
producing nothing is usually an integration break.

---

## 3. What to alert on

In priority order:

1. **Non-zero process exit.** The run did not complete. Nothing downstream
   should treat the result as usable.
2. **Any `[ERROR]` line on stderr, or any call returning
   `REAMER_RESEARCH_ERROR`.** These are the same events; catching either is
   enough. There is no severity below error, so any line at all is worth
   surfacing.
3. **`PROCESS_FATAL`, specifically.** Separate it from the rest. The others
   name something you can fix in configuration or input; this one means an
   exception reached the ABI boundary and the run aborted mid-flight. It is
   the only code that should page anyone.
4. **A run that stops producing output.** The engine does not bound callback
   execution time and cannot be interrupted (see DEPLOYMENT.md). A hung run
   hangs until killed, so enforce a wall-clock limit at the scheduler.
5. **Unexpected zero-trade runs**, per section 2.

Do not alert on rejected orders as such. They are ordinary, and a strategy
that rejects orders under some market conditions is behaving correctly.

---

## 4. Capturing a report for support

For a result that looks wrong — not one that failed outright — send:

1. **The library version actually loaded.** Not the version you intended to
   ship. Call `reamer_get_abi_version()` and record what it returns;
   see `reference-cpp/include/reamer_research.hpp` for a one-time startup
   check that refuses to run on a mismatch. A stale `.so` on one grid node is
   a common cause of a result that reproduces nowhere else.
2. **The full stderr capture**, not a grep of it.
3. **The exact input data and config** that produced the run, byte for byte.
   Runs are deterministic for a given input and `rng_seed`: the same inputs
   reproduce the same output exactly. A report without inputs cannot be
   reproduced, and a report with them almost always can.
4. **The result summary and, if the question is about trading behavior, the
   order log** including rejection reasons.
5. **The platform**: distribution and glibc version. See
   SUPPORTED_PLATFORMS.md.

Determinism is the strongest debugging tool available here. Before reporting,
run the same input twice and confirm you get the same result. If two runs of
identical input disagree, state that explicitly in the report — that is a different and more
serious class of problem than a result you disagree with.

---

## 5. Running a grid

- **Parallelism.** `reamer_run_backtest()` is safe to call
  concurrently from multiple threads, each with its own config, vtable, and
  resulting handle. A single handle is not safe to share across threads. See
  the thread-safety block at the top of `reamer_research_abi.h`.
- **Licensing.** Each machine needs its own activation. A grid that scales
  across nodes needs a license per node; check LICENSING.md before scaling
  out, not after.
- **Error state is per-thread.** `reamer_get_last_error()` reports
  the calling thread's most recent error. Read it on the thread that saw the
  failure, before that thread does anything else.
- **Exit status is the contract.** Have each worker exit non-zero on any
  `REAMER_RESEARCH_ERROR`, and let the scheduler handle retries and alerting.
  Do not parse stderr to decide whether a job succeeded.
