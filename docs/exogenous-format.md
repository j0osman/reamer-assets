---
title: Exogenous data format
description: How to attach non-OHLCV data to a Reamer Research run: the CSV format, the .exo.bin layout and the point-in-time visibility rule.
group: integration
order: 4
product: research
source: EXOGENOUS_FORMAT.md
---

Attach arbitrary non-OHLCV data to a ticker and read it from a strategy at the
right point in time. The engine never parses the values — the format inside
them is yours.

Available from **REAMER_ABI_VERSION 5**.

---

## 1. What this is for

An OHLCV bar carries price and volume. It does not carry an earnings date, a
credit rating, a VIX level, a positioning survey, a sentiment score, or a
regime label. Exogenous sidecars attach that data to a ticker's timeline and
deliver it to `on_bar` under the same point-in-time discipline the bars
themselves follow.

The engine treats every value as opaque bytes. It does not parse, validate,
interpret, or NUL-terminate them. Store JSON, CSV fragments, a single float in
ASCII, packed binary — anything your strategy can read.

---

## 2. Authoring format (CSV input)

Two columns, one record per line:

```
2024-01-02,{"vix": 13.4, "regime": "calm"}
2024-01-03,{"vix": 21.9, "regime": "stress"}
2024-01-04,{"vix": 15.0, "regime": "calm"}
```

**Column 1 — timestamp.** One of:

| Form | Example |
|---|---|
| `YYYY-MM-DD HH:MM[:SS]` | `2024-01-02 09:30:00` |
| `YYYYMMDD HH:MM[:SS]` | `20240102 09:30` |
| `YYYY-MM-DD` | `2024-01-02` |
| `YYYYMMDD` | `20240102` |

All timestamps are UTC. A date-only form means 00:00:00 UTC.

**Column 2 — value.** Everything after the **first** delimiter to end of line,
taken verbatim. A JSON object containing commas needs no quoting or escaping,
because the split happens once.

**Delimiter.** Comma by default; a tab is detected automatically from the first
non-empty line. Leading and trailing whitespace is trimmed from both fields.

**Constraints.**

- One record per line. Multi-line values are not supported.
- Blank lines are skipped.
- A line with no delimiter is an error, not a skipped row.
- An unparseable timestamp is an error.
- Empty input is an error — an empty sidecar would silently deliver nothing.

---

## 3. Building a sidecar

```
reamer-exo-build <input.csv> <output.exo.bin>
reamer-exo-build --version
```

Ships in the kit at `bin/reamer-exo-build`. It requires **no license** — it is
an offline converter that links no engine code, so it works before activation.

```
$ reamer-exo-build vix.csv vix.exo.bin
entries=3 duplicates_collapsed=0 bytes=191
```

Exit status is 0 on success, 1 on any input error (with a message on stderr),
2 on a usage error.

Records are sorted by timestamp. **On duplicate timestamps the last record in
file order wins**, and `duplicates_collapsed` reports how many were dropped —
check it if you did not expect any.

---

## 4. On-disk layout (`.exo.bin`)

Little-endian, naturally aligned, no packing. Read zero-copy via `mmap`.

```
ExoFileHeader      24 bytes
ExoEntry[]         24 bytes × entry_count, sorted ascending by ts, no duplicates
blob               blob_size raw bytes, values concatenated
```

**ExoFileHeader**

| Offset | Size | Field | Notes |
|---|---|---|---|
| 0 | 8 | `magic` | `"RMREXOG1"` |
| 8 | 4 | `entry_count` | `uint32` |
| 12 | 4 | `_pad0` | zero |
| 16 | 8 | `blob_size` | `uint64` |

**ExoEntry**

| Offset | Size | Field | Notes |
|---|---|---|---|
| 0 | 8 | `ts` | `int64`, epoch seconds UTC, strictly ascending |
| 8 | 8 | `offset` | `uint64`, byte offset into the blob |
| 16 | 4 | `length` | `uint32`, byte length of this value |
| 20 | 4 | `_pad0` | zero |

Length is explicit per entry, so values need no delimiters and may contain any
byte — including NUL.

**Invariants.** Entries are sorted ascending by `ts` with no duplicates; every
`[offset, offset + length)` lies inside the blob. A file that violates these,
or carries the wrong magic, is rejected at load.

---

## 5. Attaching a sidecar to a run

`exogenous_paths` is an array of length `ticker_count`, parallel to
`bars_by_ticker[]`. A NULL entry means that ticker has no exogenous series; a
NULL array means none do.

```c
const char* names[3] = {"AAPL", "MSFT", "SPY"};
const char* exo[3]   = {"aapl_vix.exo.bin", NULL, "spy_regime.exo.bin"};

reamer_run_backtest(
    bars_by_ticker, bar_counts, names,
    /*ticker_configs=*/NULL, exo,
    3, &vtable, lookback, &config, 100000.0, 1.0, &handle);
```

Every supplied path is opened and validated **before the run starts**. An
unreadable or malformed sidecar fails the call with
`REAMER_RESEARCH_ERROR_EXOGENOUS_LOAD` and runs no backtest — it never surfaces
mid-replay as a missing value the strategy would trade through.

---

## 6. Reading values in a strategy

Values arrive through the `on_bar` vtable slot:

```c
size_t on_bar(void* user_data,
              const ReamerOhlcvBar* window, const uint8_t* window_valid,
              size_t window_len, size_t ticker_count,
              const double* positions,
              const char* const* exogenous, const size_t* exogenous_len,
              const ReamerPositionState* position_states,
              const ReamerOpenOrder* open_orders, size_t open_order_count,
              ReamerOrderRequest* out_orders, size_t max_orders,
              ReamerOrderAction* out_actions, size_t max_actions,
              size_t* out_action_count) {
    for (size_t t = 0; t < ticker_count; ++t) {
        if (!exogenous[t]) continue;              /* nothing visible yet */
        parse_my_format(exogenous[t], exogenous_len[t]);
    }
    return 0;
}
```

**Rules.**

- `exogenous[i]` and `exogenous_len[i]` are indexed by **array position**, the
  same way `positions[]` is.
- The bytes are **engine-owned and valid only for the duration of the call**.
  Copy anything you need to retain.
- They are **not NUL-terminated**. `exogenous_len[i]` is the only authority on
  length. Never call `strlen()` on `exogenous[i]`.
- `exogenous[i] == NULL` and `exogenous_len[i] == 0` always agree.
- A run that supplies no paths receives NULL throughout and pays no per-bar
  cost for it.

---

## 7. Point-in-time semantics

On each bar, a ticker's visible value is **the most recent entry whose
timestamp is at or before the current bar's timestamp**.

- An entry becomes visible **exactly on the bar carrying its own timestamp**,
  never one bar earlier.
- Before the first entry's timestamp, `exogenous[i]` is NULL.
- A value persists across later bars until a newer entry supersedes it, and the
  switch happens exactly on the newer entry's bar.

Given a sidecar with entries on 2020-01-02 (`"first"`) and 2020-01-05
(`"second"`), against daily bars:

| Bar | Visible |
|---|---|
| 2020-01-01 | NULL |
| 2020-01-02 | `first` |
| 2020-01-03 | `first` |
| 2020-01-04 | `first` |
| 2020-01-05 | `second` |
| 2020-01-06 | `second` |

This is the whole point of the feature. A backtest that could read a value one
bar early would be trading on data the live system did not yet have, and the
resulting edge would not survive deployment. **Timestamp your records at the
moment the data was actually available to you, not the moment it describes** —
a figure published on the 5th about the 1st belongs at the 5th.

Resolution is a forward-only cursor per ticker: O(1) amortised per bar, no
per-bar search, and no re-reading of the file after load.

---

## 8. Determinism

Exogenous data does not affect determinism. A run with a fixed `rng_seed`
repeated with the same bars, configs, and sidecars produces byte-identical
results.

---

## 9. Related

- `include/reamer_research_abi.h` — `on_bar` and `reamer_run_backtest` contracts
- `EXECUTION_SPEC.md` — fill, slippage, commission, and roll methodology
- `RESULT_JSON_SCHEMA.md` — result document field reference
