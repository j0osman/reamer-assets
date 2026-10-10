---
title: Result JSON schema
description: Field-by-field schema of the Reamer Research result document: summary metrics, closed trades, fills, order log, equity curve and the returns series.
group: integration
order: 3
product: research
source: RESULT_JSON_SCHEMA.md
---

The complete result of a backtest, as emitted by
`reamer_get_result_json()` and `reamer_write_result_json()`.

Available from **REAMER_ABI_VERSION 5**. Current **`schema_version`: 2**.

---

## 1. Getting the document

```c
size_t need = 0;
reamer_get_result_json(handle, NULL, 0, &need);   /* size, incl. NUL */

char* buf = malloc(need);
reamer_get_result_json(handle, buf, need, &need); /* fill */
```

Sizing is two-call: passing `buf == NULL` reports the required size
**including the terminating NUL**, so the reported value can be passed straight
back as the capacity. An undersized buffer is rejected rather than truncated.

`reamer_write_result_json(handle, path)` writes the same bytes to a
file, minus the terminating NUL.

The document is serialized on one line. Pretty-print it downstream if you want
it readable — the bytes are stable either way.

**Locale independence.** Doubles are formatted by the serializer's own Grisu2
implementation, never the C locale's `printf`, so the decimal separator is a
`.` regardless of `LC_NUMERIC`. This is enforced by a test that runs under
`de_DE.UTF-8`.

---

## 2. Versioning

Two independent version fields:

| Field | Meaning |
|---|---|
| `schema_version` | Layout of **this document**. Bumped when fields are added, removed, or their meaning changes. |
| `abi_version` | `REAMER_ABI_VERSION` of the library that produced the run. |

They move independently on purpose: a document layout can change without an ABI
change, and vice versa. **Branch on `schema_version`**, not `abi_version`, when
parsing.

---

## 3. Top level

Thirteen keys, always present. Arrays are empty rather than omitted when a run
produced no such records.

| Key | Type | Notes |
|---|---|---|
| `schema_version` | int | Document layout version (currently `2`) |
| `abi_version` | int | Producing library's ABI version |
| `initial_capital` | double | Starting equity for the run |
| `summary` | object | §4 — every metric `reamer_get_summary()` reports |
| `closed_trades` | array | §5 — completed round trips |
| `trade_log` | array | §6 — individual fills |
| `order_log` | array | §7 — every order, whatever its outcome, plus one record per modification |
| `open_orders_end` | array | §7 — orders still live at the final bar |
| `scale_in_fills` | array | §8 — incremental adds to an existing position |
| `roll_log` | array | §9 — futures roll events |
| `returns` | array of double | §10 — **read this section before computing risk metrics** |
| `equity_curve` | array of double | §11 — realised equity after each closed trade |
| `equity_curve_ts` | array of string | §11 — timestamp of each `equity_curve` element |

**Deliberately excluded:** `orders_by_step`, `tick_orders_by_ticker`, and
`prints_by_step`. These are replay scaffolding — per-step indices the engine
uses internally. They carry no information not already in `order_log` and would
multiply the document size for no consumer benefit.

**Ticker names.** Every `ticker` field anywhere in the document carries the
caller's own instrument name from `ticker_names[]`, which every run entry point
requires.

---

## 4. `summary`

31 fields — the same set `reamer_get_summary()` returns, so a
consumer reading the JSON never has to call back into the ABI.

**Core P&L** (identical to the frozen `ReamerBacktestSummary`):

| Field | Type | Notes |
|---|---|---|
| `gross_pnl` | double | Before costs |
| `net_pnl` | double | After fees, slippage, swap |
| `total_fees` | double | |
| `total_slippage_cost` | double | |
| `total_swap_cost` | double | |
| `trades` | int | Completed round trips. The denominator behind `win_rate` and `net_ev_per_trade`. **Not** a count of fills — see `trade_log` for those |
| `closed_trades_count` | int | Completed round trips — matches `closed_trades.length` |

**Returns and ratios:**

| Field | Type | Notes |
|---|---|---|
| `total_return_pct` | double | Against `initial_capital` |
| `win_rate` | double | Fraction in `[0,1]` |
| `gross_profit` | double | Sum of winning trades |
| `gross_loss` | double | Sum of losing trades |
| `net_profit` | double | |
| `profit_factor` | double | `gross_profit / |gross_loss|` |
| `net_ev_per_trade` | double | Expected net P&L per closed trade |
| `recovery_factor` | double | |
| `sharpe_ratio` | double | Annualised — see §12 |
| `sortino_ratio` | double | Annualised — see §12 |
| `calmar_ratio` | double | `total_return_pct` / (`equity_drawdown_relative` × 100); not annualised |

**Order outcomes:**

| Field | Type | Notes |
|---|---|---|
| `total_orders` | int | Orders submitted; `Modified` records in `order_log` are not counted |
| `filled_orders` | int | |
| `expired_orders` | int | |
| `cancelled_orders` | int | |
| `rejected_orders` | int | |

**Streaks, holding time, drawdown, rolls:**

| Field | Type | Notes |
|---|---|---|
| `max_consecutive_wins` | int | |
| `max_consecutive_losses` | int | |
| `avg_holding_time_seconds` | double | |
| `max_holding_time_seconds` | double | |
| `min_holding_time_seconds` | double | |
| `equity_drawdown_maximal` | double | Absolute, against `initial_capital`; max drop of `equity_curve` (§11) |
| `equity_drawdown_relative` | double | Fractional |
| `roll_event_count` | int | Matches `roll_log.length` |

---

## 5. `closed_trades[]`

One completed round trip each.

| Field | Type | Notes |
|---|---|---|
| `ticker` | string | Caller's instrument name |
| `side` | string | `"long"` or `"short"` — **position direction**, not the entry order's side |
| `open_timestamp` | string | `"YYYY-MM-DD HH:MM:SS"` UTC |
| `close_timestamp` | string | Same form |
| `entry_price` | double | |
| `exit_price` | double | |
| `qty` | double | |
| `leverage` | double | |
| `margin_used` | double | |
| `fees` | double | |
| `slippage` | double | |
| `take_profit` | double | `0.0` when unset |
| `stop_loss` | double | `0.0` when unset |
| `gross_pnl` | double | |
| `net_pnl` | double | |
| `return_pct` | double | Percent against this trade's own margin — **not** the `returns[]` element |
| `open_step` | int | Bar index of entry |
| `close_step` | int | Bar index of exit |
| `open_tick_i` | int | Intrabar tick index of entry |
| `close_tick_i` | int | Intrabar tick index of exit |

---

## 6. `trade_log[]`

Individual fills. A round trip appears here as two entries and in
`closed_trades` as one.

| Field | Type | Notes |
|---|---|---|
| `timestamp` | string | UTC |
| `ticker` | string | |
| `side` | string | `"long"` / `"short"` |
| `price` | double | Fill price |
| `qty` | double | |
| `fees` | double | |
| `slippage_cost` | double | |

---

## 7. `order_log[]` and `open_orders_end[]`

Same record shape. `order_log` holds every order the run produced.
`open_orders_end` holds the orders still pending when the run ended, as they
stood before the end-of-run cancel: `status` `"Pending"`, `closed_timestamp`
`""`. Each also appears in `order_log` with `status` `"Cancelled"`.

**Modifications.** Each applied `MODIFY_ORDER` and `MODIFY_POSITION` action
(`reamer_research_abi.h`, `ReamerOrderAction`) adds one `order_log` record with
`status` `"Modified"`, in the order applied, and `closed_timestamp` set to the
time of the bar whose `on_bar` call returned the action:

- `MODIFY_ORDER`: a snapshot of the order after the change, under the order's
  own `id`. One order can therefore appear several times: once per
  modification, then once with its final status.
- `MODIFY_POSITION`: `id` `0`; `order_type` and `side` give the position's
  direction (`buy`/`long` for a long), `qty` its size, `take_profit` and
  `stop_loss` the new levels, `created_timestamp` equal to `closed_timestamp`.

`CLOSE_ALL` adds no `Modified` record; each position it closes produces an
ordinary market close order. `CANCEL_ALL` produces one `Cancelled` record per
order it cancels.

| Field | Type | Notes |
|---|---|---|
| `id` | int | Unique per order within the run; `Modified` records repeat their order's `id`, or carry `0` for a position |
| `created_timestamp` | string | UTC |
| `created_ts` | int | Epoch seconds |
| `expiry_timestamp` | string | `""` when not GTD |
| `expiry_ts` | int | `0` when not GTD |
| `closed_timestamp` | string | Fill, cancel, expiry, or modification time |
| `ticker` | string | |
| `from_tick` | bool | Whether it was triggered intrabar |
| `order_type` | string | `buy`, `sell`, `buy_limit`, `sell_limit`, `buy_stop`, `sell_stop` |
| `side` | string | `"long"` / `"short"` |
| `tif` | string | `GTC`, `GTD`, `IOC` |
| `status` | string | `Pending`, `Filled`, `Cancelled`, `Expired`, `Rejected`, `Modified` |
| `reject_reason` | string | `""` unless `status == "Rejected"` |
| `qty` | double | |
| `limit_price` | double | `0.0` when not applicable |
| `stop_price` | double | `0.0` when not applicable |
| `take_profit` | double | |
| `stop_loss` | double | |
| `fill_price` | double | `0.0` unless filled |
| `fees` | double | |
| `slippage_cost` | double | |

---

## 8. `scale_in_fills[]`

Incremental additions to a position already open.

| Field | Type |
|---|---|
| `ticker` | string |
| `timestamp` | string |
| `side` | string |
| `price` | double |
| `qty` | double |
| `step` | int |
| `tick_i` | int |

---

## 9. `roll_log[]`

Futures roll events.

| Field | Type | Notes |
|---|---|---|
| `ticker` | string | |
| `timestamp` | string | UTC |
| `ratio` | double | Price adjustment ratio applied at the roll |

See `EXECUTION_SPEC.md` for roll methodology.

---

## 10. `returns[]` — read this before recomputing risk

A flat array of **per-trade fractional returns**, one element per closed trade,
in close order:

```
returns[i] = closed_trades[i].net_pnl / equity_at_entry   (clamped to [-1, 1])
```

Three things follow, and all three catch people out:

1. **The denominator is account equity at entry, not the trade's margin.** It is
   therefore *not* `closed_trades[i].return_pct / 100`. Those two fields answer
   different questions: `return_pct` is the return on the position, `returns[i]`
   is the return on the account.
2. **It is clamped to `[-1, 1]`.** A leveraged loss exceeding starting equity
   contributes `-1.0`, not its raw ratio.
3. **This is the exact series `sharpe_ratio` and `sortino_ratio` are
   computed from.** Recomputing risk metrics from
   `closed_trades[].net_pnl` will **not** reconcile with `summary`.

Worked example — the single-trade run in §13: `net_pnl` 28.7958645855241 on
`initial_capital` 100000.0 gives `returns[0] = 0.000287958645855241`.

---

## 11. `equity_curve[]` and `equity_curve_ts[]`

Realised account equity, one point per closed trade, in close order:

```
equity_curve[0]   = initial_capital
equity_curve[i+1] = equity_curve[i] + closed_trades[i].net_pnl   (floored at 0)
```

1. **Granularity is per closed trade, not per bar.** Open positions are not
   marked to market. A curve point moves only when a round trip closes.
2. **It floors at 0 and stops there.** A loss larger than the remaining equity
   takes the curve to `0.0`, and no later point is emitted. After ruin,
   `equity_curve` is shorter than `closed_trades` + 1.
3. **Its maximum peak-to-trough drop equals `summary.equity_drawdown_maximal`.**
   This is the curve the drawdown metrics are measured on.

`equity_curve_ts` has the same length as `equity_curve`. Element 0 is the first
closed trade's `open_timestamp`; element `i+1` is `closed_trades[i].close_timestamp`.
With no closed trades, both arrays have one element: `initial_capital` and `""`.

Worked example — the single-trade run in §13:
`equity_curve = [100000.0, 100028.79586458553]`,
`equity_curve_ts = ["2020-01-02 00:00:00", "2020-01-05 00:00:00"]`.

---

## 12. Annualisation

`sharpe_ratio` and `sortino_ratio` are annualised by `sqrt(periods_per_year)`.

The `summary` object in this document always uses the **inferred**
`periods_per_year`: the median gap between consecutive closed trades'
`close_timestamp` values:

```
periods_per_year = seconds_per_year / median_gap_seconds
```

It falls back to **252** (daily) when inference is impossible: fewer than two
closed trades, unparseable timestamps, or a median gap of zero.

The value is not reported in the document. To get the ratios on a fixed
convention, call `reamer_get_summary(handle, 252.0, &summary)` (or 52, 12, …).
A value `<= 0` gives the inferred figure shown here. A strategy trading at
irregular intervals gets a median-based figure that may not match the calendar
convention you expect.

---

## 13. Example

A single-asset, single-trade run, pretty-printed and abbreviated:

```json
{
  "schema_version": 2,
  "abi_version": 7,
  "initial_capital": 100000.0,
  "summary": {
    "gross_pnl": 28.7958645855241,
    "net_pnl": 28.7958645855241,
    "total_fees": 0.0,
    "trades": 1,
    "closed_trades_count": 1,
    "sharpe_ratio": 0.0,
    "win_rate": 1.0
  },
  "closed_trades": [
    {
      "ticker": "AAPL",
      "side": "long",
      "open_timestamp": "2020-01-02 00:00:00",
      "close_timestamp": "2020-01-05 00:00:00",
      "entry_price": 101.20097077334702,
      "exit_price": 104.08055723189943,
      "qty": 10.0,
      "net_pnl": 28.7958645855241,
      "return_pct": 2.8454138695977784,
      "open_step": 1,
      "close_step": 4
    }
  ],
  "trade_log": [ /* 2 fills */ ],
  "order_log": [ /* 2 orders */ ],
  "open_orders_end": [],
  "scale_in_fills": [],
  "roll_log": [],
  "returns": [0.000287958645855241],
  "equity_curve": [100000.0, 100028.79586458553],
  "equity_curve_ts": ["2020-01-02 00:00:00", "2020-01-05 00:00:00"]
}
```

---

## 14. Compatibility

Within a `schema_version`, fields are only **added**, never removed or
repurposed. Parse permissively — ignore unknown keys — and a document from a
newer library keeps working.

A `schema_version` bump means a field was removed or changed meaning. Check it
before parsing, and fail loudly on an unexpected value rather than reading
fields that may no longer mean what you assume.

---

## 15. Related

- `include/reamer_research_abi.h` — `get_result_json` / `write_result_json` contracts
- `EXECUTION_SPEC.md` — fill, slippage, commission, and roll methodology
- `EXOGENOUS_FORMAT.md` — attaching non-OHLCV data to a run
