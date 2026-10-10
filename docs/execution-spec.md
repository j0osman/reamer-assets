---
title: Reamer Research execution specification
description: The authoritative definition of how Reamer Research fills orders: bid/ask, slippage, spread, commission, margin, stops, brackets, roll and exogenous data.
group: integration
order: 2
product: research
source: EXECUTION_SPEC.md
---

**This document is the authoritative definition of Reamer Research execution behavior.**
Where any other document in this archive — a reference-implementation README, a
benchmark note, a deployment step — describes fill, slippage, spread, commission,
margin, or roll behavior differently, this document describes the shipped engine and
the other is wrong. Size positions against what is written here.

Behavior is defined here first and implemented to match, never defined implicitly by
what the code happens to do. Our internal conformance tests and golden regression
baselines are written against this specification, and a change to execution behavior
updates this document, those tests, and those baselines together. Those tests are
internal and do not ship in this archive; the claim above is ours, and you should
verify the behavior you depend on against your own backtests.

Every formula here is observable through the shipped library — set the relevant config
field, run a backtest, and read the result. §0a lists the fields and what their zero
value means.

---

## 0a. Execution Config Reference

The fields below are `ReamerBacktestConfig` in `include/reamer_research_abi.h`.
Zero-initialise the struct and set only the fields a run needs: an all-zero config is a
frictionless run on bid data.

| Field | Units | Zero value | Description |
|---|---|---|---|
| `commission_per_unit` | $ / unit | no commission | Fixed cost per unit traded. See §4. |
| `commission_mode` | enum | `ROUND_TRIP` | `REAMER_RESEARCH_COMMISSION_ROUND_TRIP` (open + close), `_OPEN_ONLY`, `_CLOSE_ONLY`. See §4. |
| `slippage` | price units | no slippage | Mean slippage per fill. Stochastic when `price_volatility > 0` — see §1. |
| `spread` | price units | no spread | Bid-ask spread in absolute price units. See §1. |
| `ohlcv_type` | enum | `BID` | `REAMER_RESEARCH_OHLCV_BID` (data = bid), `_ASK` (data = ask), `_MIDPOINT` (data = mid). See §1. |
| `price_volatility` | σ | deterministic | Enables stochastic spread and slippage noise. `0` = deterministic fills. See §1. |
| `rng_seed` | uint | seed `0` | Seeds the synthetic tick generator. Every value, `0` included, is a fixed seed. Engine-global; see §0. |
| `swap_per_unit[7]` | $ / unit / night | no swap | Overnight swap rate per weekday (Sun=0…Sat=6). See §9. |
| `roll_cycle_months[12]`, `roll_cycle_months_count` | months | rolls disabled | Months (1=Jan..12=Dec) a futures roll recurs in. See §12. |
| `roll_days_before_monthend` | business days | roll on the month's last day | Futures roll trigger offset. Set `5` for the common convention. See §12. |

Two run-level values sit beside the config, as arguments to `reamer_run_backtest()` and
`reamer_run_backtest_files()`:

| Argument | Units | Description |
|---|---|---|
| `initial_capital` | $ | Starting equity. |
| `leverage` | × | Max notional per entry: `abs(qty) × price ≤ equity × leverage`. See §10. |

`price_volatility` and `rng_seed` together drive every stochastic element of execution.
A run with `price_volatility = 0` is fully deterministic regardless of seed; a run with
`price_volatility > 0` is deterministic **for a fixed seed** and reproducible byte-for-byte
across repeats.

---

## 0b. Order Types and Brackets

The ABI defines six order types (`REAMER_RESEARCH_ORDER_TYPE_*`): `BUY`, `SELL`,
`BUY_LIMIT`, `SELL_LIMIT`, `BUY_STOP`, `SELL_STOP`. Fill semantics for each are in §2;
time-in-force is in §6.

Every order type accepts optional `take_profit` and `stop_loss` bracket levels.
Validation, applied at submission:

- **Long entry** — `take_profit` must be above the entry price; `stop_loss` below it.
- **Short entry** — `take_profit` must be below the entry price; `stop_loss` above it.

An order violating these is rejected at submission. Bracket exits fill per §2's TP/SL
rows, resolve collisions per §7, and are accounted into the closed trade's fees and
slippage — they do not appear as separate live orders.

Closing a position is direction-resolved by the engine: a full close fills the entire
open quantity, and a partial close fills the requested quantity and leaves the remainder
open (§11). A close submitted while flat is rejected.

---

## 0. Execution Config Resolution (Per-Ticker Overrides)

Every parameter described in this document — spread, slippage, price_volatility, ohlcv_type,
commission_per_unit, commission_mode, swap_per_unit, the roll fields — is resolved per ticker, not
once for the whole run: a ticker whose `ticker_configs[i]` entry is non-NULL uses that config; a
ticker with a NULL entry, or every ticker when `ticker_configs` itself is NULL, uses `exec_config`
unchanged. This applies uniformly across every rule in sections 1-12 below — read "the config" in
what follows as "the resolved per-ticker config," whether that resolves to `exec_config` or to a
per-ticker entry.

The one exception is `rng_seed`: it is always engine-global, even inside a per-ticker entry. The
synthetic tick generator is constructed once per run from `exec_config.rng_seed`; per-ticker
trajectory decorrelation already comes from hashing the ticker's array position, not from a
distinct seed per ticker, so a per-ticker `rng_seed` is ignored.

A ticker that falls back to `exec_config` gets whatever that config holds — all zeros models no
execution costs. Set `exec_config` deliberately whenever `ticker_configs` is sparse.

---

## 1. Bid/Ask Derivation

Raw synthetic tick prices are generated per bar using a deterministic pseudo-random sequence seeded
by `rng_seed`, `bar_idx`, and a per-ticker hash. Raw ticks are clamped to `[bar.low, bar.high]`.

The ask spread and bid spread are sampled independently with different hash salts:

```
noise_std      = min(bar_range * price_volatility, base_spread * 4.0)
sampled_spread = max(0, base_spread + N(0,1) * noise_std)
```

where the Gaussian sample is deterministic (hash-based, per tick). Noise width scales with each
bar's own high-low range, not a fixed fraction of `base_spread` — a calm bar barely perturbs the
spread, a volatile bar can swing it substantially — capped at 4x `base_spread` (matching typical
real-world spread widening in volatile conditions) so the zero-floor clamp above stays a rare event
instead of dragging the realized mean upward.

Short-circuits, in order: `base_spread <= 0` returns `0`; `price_volatility <= 0` returns
`base_spread` exactly; `bar_range < 1e-12` (flat bar) also returns `base_spread` exactly.

Slippage is sampled independently, with its own hash salt and a different formula:

```
sampled_slippage = base_slippage + N(0,1) * bar_range * price_volatility
```

Unlike spread, slippage noise is uncapped and unclamped — there is no ask/bid-crossing invariant to
protect, since slippage is a fill-price adjustment rather than part of quote construction.
Occasional favorable (negative) slippage is realistic price improvement, and real-world slippage
during liquidity stress can spike far more than typical spread widening, so it is neither floored
at 0 nor capped. The same short-circuits apply: `base_slippage <= 0` returns `0`;
`price_volatility <= 0` or `bar_range < 1e-12` returns `base_slippage` exactly.

In addition, an intra-bar spread multiplier widens the spread at bar open and narrows it linearly
toward the base value at bar close (this multiplier applies only to spread, not slippage):

```
spread_mult = 1 + (bar_range / (bar_range + cfg.spread)) × (1 − progress)
effective_spread = sampled_spread × spread_mult
```

where `progress = tick_index / (tick_count − 1)`. This causes spreads to be wider near the open
and tighter near the close of each bar.

Bid/ask are derived from the raw tick depending on `OhlcvType`:

| OhlcvType  | tick_bid               | tick_ask               |
|------------|------------------------|------------------------|
| Bid        | raw_tick               | raw_tick + spread      |
| Ask        | raw_tick − spread      | raw_tick               |
| Midpoint   | raw_tick − spread/2    | raw_tick + spread/2    |

---

## 2. Fill Price Rules

All fills apply slippage on top of the prevailing bid or ask.

| Order type     | Trigger condition         | Fill price                        |
|----------------|---------------------------|-----------------------------------|
| Market buy     | always                    | `tick_ask + slippage`             |
| Market sell    | always                    | `max(0, tick_bid − slippage)`     |
| Buy limit      | `tick_ask <= limit_price` | `tick_ask + slippage`             |
| Sell limit     | `tick_bid >= limit_price` | `max(0, tick_bid − slippage)`     |
| Buy stop       | `tick_ask >= stop_price`  | `tick_ask + slippage`             |
| Sell stop      | `tick_bid <= stop_price`  | `max(0, tick_bid − slippage)`     |
| Long TP exit   | `tick_bid >= take_profit` | `max(0, tick_bid − slippage)`     |
| Long SL exit   | `tick_bid <= stop_loss`   | `max(0, tick_bid − slippage)`     |
| Short TP exit  | `tick_ask <= take_profit` | `tick_ask + slippage`             |
| Short SL exit  | `tick_ask >= stop_loss`   | `tick_ask + slippage`             |

**Consequences:**
- Buy limit fill price `≤ limit_price + slippage` (since `tick_ask ≤ limit_price` at trigger)
- Sell limit fill price `≥ limit_price − slippage` (since `tick_bid ≥ limit_price` at trigger)
- Fill prices can exceed raw bar OHLCV range by up to `sampled_spread + slippage` in each direction

---

## 3. Slippage Cost

`slippage_cost = |fill_price − reference_price| × qty`

where `reference_price` is the bid or ask at the fill tick (the price that would be obtained without
slippage). For a market buy: `reference_price = tick_ask`; for a sell: `reference_price = tick_bid`.

Note: `slippage_cost` captures only the slippage component (`slippage × qty`). The bid/ask spread
cost is embedded in `fill_price` relative to raw bar prices but is not separately tracked in
`slippage_cost`.

---

## 4. Commission

Commission is applied per fill based on `CommissionMode`:

| Mode        | Open leg      | Close leg     |
|-------------|---------------|---------------|
| RoundTrip   | `qty × rate`  | `qty × rate`  |
| OpenOnly    | `qty × rate`  | 0             |
| CloseOnly   | 0             | `qty × rate`  |

---

## 5. Accounting Identities

```
gross_pnl  = (exit_fill − entry_fill) × qty × sign   (sign = +1 long, −1 short)
net_pnl    = gross_pnl − entry_fees − exit_fees
```

At end of backtest the engine force-closes every open position and cancels every pending
order. Then, in `ReamerBacktestSummary` and the result JSON:
```
summary.net_pnl             = sum(closed_trades[k].net_pnl)
summary.total_fees          = sum(closed_trades[k].fees)
summary.total_slippage_cost = sum(closed_trades[k].slippage)
```

Tolerance: 1e-6.

---

## 6. Order Time-in-Force

| TIF | Behavior                                                                             |
|-----|--------------------------------------------------------------------------------------|
| GTC | Stays open until filled, cancelled, or end of backtest                              |
| IOC | Must fill on its one eligible bar (see below); cancelled at that bar's seal if not   |
| GTD | Expires when `bar_epoch >= expiry_epoch`; status = Expired                          |

Fill eligibility is origin-dependent:

- **Bar-origin orders** — every order a strategy writes to `out_orders` in `on_bar(N)`.
  `on_bar(N)` only ever runs after bar N is fully sealed (the newest window row is bar N's
  real close), so an order it returns is only eligible starting
  the ticker's **next** bar (N+1) — `created_ts = bar_N.epoch`, and the order becomes
  eligible once `now_ts > created_ts`, i.e. bar N+1 tick 1 onward. It is never matched
  against any tick of bar N itself: a decision made from bar N's close cannot fill at a
  price from earlier in bar N's own (already fully-determined) tick path. Submitting a
  bar-origin order on a ticker's last available bar is rejected outright
  (`"no next bar available to fill against (final bar of dataset)"`) since no next bar
  will ever arrive to make it eligible.
- **Tick-origin orders** — submitted by the engine's internal intrabar path in reaction
  to a genuinely observed intrabar tick (§6a). These remain eligible
  starting the same tick/bar they were submitted in, `now_ts >= created_ts` — no backdating
  concern, since the tick that triggered them and the tick they might fill at both move
  forward in real time.

Tick count per bar defaults to the time delta in seconds to the next bar timestamp
(`bars[i+1].ts − bars[i].ts`) — a 1-minute bar has 60 ticks, a daily bar has 86,400 —
*unless* the bar itself supplies a `tick_count` (any value other than the `-1`
"not supplied" sentinel), in which case that value is used verbatim instead of the
gap-derived estimate. This is what makes non-time-interval bars (tick/volume/dollar/
imbalance bars, or any other pre-aggregated activity bar — see §14) simulate
correctly: a dollar bar that took 40 real trades to fill gets 40 synthetic ticks,
regardless of how many wall-clock seconds its own timestamp gap happens to span.

Market orders have no price condition and fill unconditionally at the first eligible tick
(bar N+1 tick 1, for a bar-origin order). Limit/stop orders fill at the first eligible tick
where their price condition is satisfied — the crossing search itself is unchanged and
purely price-driven; only the earliest tick it may search from moves to N+1.

Cancellation:

- A cancel request (`is_cancel = true`, `cancel_target_id`) takes effect after the current
  bar replays, as `CANCEL_ALL` does (§6b): an order that fills on the bar the strategy has
  already seen stays filled. If the order is still Pending then, it is cancelled at the
  current bar's timestamp. An unknown or no-longer-pending ID is a silent no-op. Cancel requests consume no order ID; every other order written to
  `out_orders`, rejected or not, takes the next ID in submission order, starting at 1.
- A full position close — strategy order or bracket exit — cancels every Pending order on
  that ticker, entry orders included. A partial close cancels none.
- At end of run, every order still Pending is recorded in `open_orders_end[]`, then
  cancelled.

---

## 6b. Order and Position Actions

`on_bar` may write `ReamerOrderAction` entries to `out_actions` (64 per call), alongside
its orders. Each kind's fields are in `reamer_research_abi.h`.

Timing, for an action returned at bar N:

| Action | Takes effect |
|---|---|
| `CLOSE_ALL` | One `is_close` market order per ticker with a nonzero `position_states` qty, submitted after that call's `out_orders` and taking IDs after them. Fills at bar N+1 tick 1, as any close. |
| `MODIFY_ORDER` | After bar N replays. The new price and brackets apply from bar N+1. |
| `MODIFY_POSITION` | After bar N replays. The new levels apply from bar N+1. |
| `CANCEL_ALL` | After bar N replays. Cancels each order in that call's `open_orders` still Pending, at bar N's timestamp, as a cancel request does (§6). Orders submitted in the same call are not in that list and stay live. |

Bar N has already reached the strategy when it writes the action, so a fill on bar N
stands at the levels the strategy saw it with. An order that fills, or a position that
closes, during bar N is no longer there to modify: its action reports `applied = false`.
Actions apply in array order, so a later action on the same order or ticker overrides an
earlier one.

Validation:

- A malformed action (kind out of range, non-finite price or bracket, `ticker_id` out of
  range for `MODIFY_POSITION`) reports `applied = false` and has no effect.
- `MODIFY_ORDER` needs an order ID that is still Pending. While its ticker is flat, the new
  brackets must sit on the correct side of the order's price, as at submission (§0b).
- `MODIFY_POSITION` needs an open position. Each new level must sit on the correct side of
  the ticker's latest close in the window: a long's take-profit above it and stop-loss
  below it, reversed for a short. Entry price is not a bound: a stop may move to breakeven
  or into profit.
- A rejected action leaves the order or position unchanged and the run continues.

A changed bracket fires by the rules of §2 and §7 from bar N+1: at the first eligible tick
through the new level.

---

## 6a. Tick-Origin Fill Semantics (internal engine path — not exposed by the ABI)

The C ABI exposes no tick-level callback: strategies receive `on_bar` only. This section
documents tick-origin semantics because they affect the fill rules above, not because
the path is reachable from your integration.

Tick-origin orders are processed at the tick that triggered them and fill against that
tick's bid/ask prices:

- Market orders fill unconditionally at the triggering tick, not at tick 1.
- Limit/stop orders are eligible from the triggering tick onward within the same bar.
- IOC orders not filled at the triggering tick are cancelled at end of that tick.
- All normal commission, slippage, and margin rules apply.

Tick-origin and bar-origin orders coexist but follow different eligibility rules (§6):
bar-origin orders are only eligible starting the ticker's *next* bar (bar N+1 tick 1+,
never bar N); tick-origin orders are eligible from their own submission tick onward,
same bar included.

Both paths read the same synthetic tick sequence, seeded by `rng_seed`.

---

## 7. Bracket Exit Collision

When both TP and SL are set and both are touched by the same bar's tick sequence, the first one
encountered in tick order wins. The tick sequence is deterministic given `rng_seed` and bar data.
The winning exit is the one whose trigger condition is first satisfied. Both TP and SL being
touched means whichever happens at a lower tick index is the exit.

---

## 8. Gap Execution

When a bar opens beyond a pending order's trigger level, tick 1 of that bar will trigger
and fill the order. Tick 1 is the first interpolated price after `bar.open` — very close to
`bar.open` but not exactly equal to it. There is no additional slippage adjustment for gap
magnitude beyond `cfg.slippage`.

---

## 9. Overnight Swap

Computed once per calendar date boundary in the bar sequence.
Long positions: equity decreases by `qty × swap_per_unit[weekday]`.
Short positions: equity increases by the same amount.

---

## 10. Margin Model

Order rejected if: `new_notional + existing_notional > equity × leverage`

where `new_notional = qty × fill_price` and `existing_notional` is the sum of all open position
notionals. Rejected orders have a non-empty `reject_reason`.

---

## 11. Partial Closes

A market sell order with `0 < qty < position.qty` triggers a partial close:

- A `ClosedTrade` is recorded for `qty` units at the fill price and the original entry price.
- `position.qty` is reduced by `qty`; the position remains open with its original `entry_price`.
- Commission and slippage are computed on `qty` only.
- A subsequent full-close (`qty=0`) or another partial close may follow.

Rejection rules:
- `qty <= 0` on a close-side order (`is_close = true`, or an order opposite the open position) means "close the full position" — not a partial close.

Netting reversal:
- `qty > position.qty` on any close-side order (including `is_close = true` with `qty = N`) triggers a **netting reversal**: the existing position is fully closed and the excess qty opens a new position in the opposite direction. Example: long 10, sell 15 → close long 10, open short 5. Commission and slippage are split proportionally between the close leg and the open leg.

Same-side orders:
- Same-side orders (buy while long, sell while short) **scale into** the existing position: `position.qty` increases by the filled qty, `entry_fill_price` is updated to the weighted average of the previous entry and the new fill (`(prev_price × prev_qty + fill_price × fill_qty) / (prev_qty + fill_qty)`). Margin check applies to the incremental notional.

The PnL identity holds across partial closes: the sum of `net_pnl` over all partial `ClosedTrade`s for a position equals the `net_pnl` that a single full-close would have produced.

---

## 12. Futures Roll-Cycle Rebasing

Disabled by default (`roll_cycle_months` empty) — behavior is then byte-identical to a build
without this feature. Enabling it is per-ticker, via the same `ticker_configs` resolution
described in Section 0: `roll_cycle_months` (up to 12 months, 1=Jan..12=Dec, count in
`roll_cycle_months_count`) and `roll_days_before_monthend` (5 is the common convention) declare a raw/naive continuous-contract series' recurring
roll dates so the engine can rebase execution-model bookkeeping across each boundary instead of
treating the raw price splice as real price movement.

**Roll date rule.** For each `month` in `roll_cycle_months`, in each candidate year: take that
month's last calendar day, then step backward one calendar day at a time — counting only Mon–Fri,
skipping Sat/Sun without counting them, and never counting the starting day itself — until
`roll_days_before_monthend` business days have been counted. The roll boundary is the epoch
(UTC midnight) reached at that count; `0` makes it the month's last calendar day. With
`roll_days_before_monthend=5` and a cycle
month whose last calendar day is a Friday, the boundary is the Friday of the preceding week.

**Trigger condition.** A ticker's next roll boundary is resolved once and cached; the rebase for
a given step fires when that step's timestamp reaches or passes the cached boundary
(`RollBoundaryCrossed`). If more than one boundary falls within a single step's gap (only possible
with an unusually coarse bar interval relative to `roll_days_before_monthend`), only one ratio is
ever applied for that step — the cache advances past every such boundary, but the ratio itself is
computed once, from that step's own bar pair.

**Ratio measurement and application.**

```
ratio = curr_bar.open / prev_bar.close
```

where `prev_bar` and `curr_bar` are that ticker's bars immediately before and at the step
containing the roll boundary. If `prev_bar.close <= 0`, the rebase is skipped entirely for that
boundary — no fields are modified and no `RollEvent` is logged. Otherwise, `ratio` is applied by
multiplication to:

- the open position, if any: `entry_fill_price`, `take_profit` (if set), `stop_loss` (if set)
- every resting order on that ticker: `limit_price`, `stop_price`, `take_profit`, `stop_loss`
  (each only if already set / nonzero)

No other position or order field is touched. Commission, slippage, and margin calculations on any
*subsequent* fill use the rebased levels exactly as they would any other configured price level —
Sections 2–4 and 10 apply unchanged.

**What is not rebased.** The raw bars a strategy reads from `on_bar()`'s `window` are never
modified — this is forward-only bookkeeping rebase, not a historical rewrite. A strategy
computing its own indicators directly from the window's closes still sees the raw splice jump at
the roll boundary; only the engine's own position/order price levels are corrected.

**Bracket re-scheduling.** Take-profit/stop-loss crossing ticks are pre-scheduled at bar-seal time
using whatever `take_profit`/`stop_loss` stood at that moment. Because the rebase for a given step
runs after that step's own bar has already sealed, its bracket schedule was decided against the
stale, pre-rebase levels — the engine re-schedules immediately after rebasing so the already-current
bar is re-evaluated against the corrected levels. The single exception: the roll-boundary step's own
bracket schedule was locked in before the rebase ran and is not itself revisited — a narrow,
deliberate edge case, not an oversight.

**Logging.** Every applied rebase appends one entry (`ticker`, `timestamp` of the boundary,
`ratio`) to the result JSON's `roll_log[]` and counts toward `summary.roll_event_count`,
unconditionally — there is no separate mode to opt into this visibility. A ticker with
`roll_cycle_months` configured that never actually crosses a boundary during the run produces no
entries; `roll_log` is empty whenever no ticker rolled.

---

## 13. Exogenous Data Resolution

Attaches an arbitrary, freeform timeseries to a ticker via the run's `exogenous_paths[]` argument
(parallel to the ticker arrays, one `.exo.bin` sidecar path per ticker, NULL for none), independent
of `ticker_configs`. A ticker with no sidecar behaves as if the feature does not exist for it: its
`exogenous[i]` is always NULL and `exogenous_len[i]` always `0`.

**Storage.** Each `.exo.bin` sidecar holds one ticker's entries, sorted strictly ascending by epoch
second with no duplicate timestamps (`bin/reamer-exo-build` enforces this at write time: on duplicate
timestamps in the source CSV, the last line wins). Each entry's value is stored as raw bytes at a
recorded byte offset and length — the engine never parses, validates, or imposes a schema on the
content. EXOGENOUS_FORMAT.md defines the CSV rules and the file layout.

**Resolution.** A per-ticker cursor advances forward only, once per step, using the same "advance
forward, never re-search" technique used for bar indexing elsewhere in the engine:

```
while next_entry.ts <= current_step_ts: advance cursor to next_entry
```

Once at least one entry's timestamp is `<=` the current step's timestamp, `exogenous[i]` points at
the *latest* such entry and `exogenous_len[i]` holds its byte length, on every step for the rest of
the run. The bytes are the CSV value verbatim, engine-owned, valid only for that `on_bar()` call,
and not NUL-terminated: copy them before returning, and use `exogenous_len[i]`, never `strlen()`.
Parsing them is the strategy's responsibility; a malformed value is the strategy's error to handle,
not an engine-level failure.

**Before the first entry.** For every step strictly before a ticker's first exogenous timestamp,
and for the entire run on any ticker with no sidecar, `exogenous[i]` is NULL and `exogenous_len[i]`
is `0` — indistinguishable at the API level; check for NULL before use.

**No execution-model interaction.** Exogenous data is read-only context for strategy code. It has
no effect on fill price, slippage, commission, margin, or any other rule in this document, and
attaching a series to a ticker does not itself submit, modify, or reject any order — Sections 1–12
apply identically whether or not a ticker has exogenous data attached.

---

## 14. Bar-Type-Agnostic Ingestion and Multi-Asset Alignment

**Any OHLCV aggregation is accepted, not just fixed time intervals.** `OhlcvBar` is
`(ts, open, high, low, close, volume, notional, tick_count)` — a shape that every
sampling scheme reduces to identically, whether a bar's boundaries were decided by a
fixed time interval, a fixed tick count, a fixed volume, a fixed notional (dollar)
value, or an adaptive tick/volume/dollar imbalance threshold. The engine never
distinguishes between them: it doesn't implement tick bars, dollar bars, or
imbalance bars as distinct concepts, only the one generic record shape every one of
them can be built into upstream, before the data ever reaches Reamer Research. `volume` is the
share/contract count aggregated into the bar; `notional` is the dollar/currency value
aggregated into it (0.0 if unused — kept as a separate field from `volume` rather
than overloading one to mean either, depending on bar type); `tick_count` is the real
tick/trade count that built the bar, or the `-1` sentinel meaning "not supplied" (see
§6 for how it drives intrabar tick simulation when present).

**Bar construction itself is out of scope.** Reamer Research executes correctly against
whatever bar sequence a `.bin` file contains — it does not, and is not intended to,
turn raw tick data into tick/volume/dollar/imbalance bars. That aggregation step
(buy/sell classification, imbalance threshold estimation, bucketing) is entirely the
producer's responsibility (a user's own pipeline, or a vendor's tooling), same as it
already is for anyone building custom time-bar OHLCV today.

**Multi-asset alignment is union-only, timestamp-driven, and origin-agnostic.**
The engine emits one portfolio step per distinct timestamp seen
across any ticker's bars, including only the ticker(s) whose bar actually lands
there. This requires no change to support
independently-sampled activity bars: a step is defined purely by "some ticker
produced a bar here," never by exact-timestamp coincidence across tickers.
(`AlignmentMode::Intersection` — a step only when *every* ticker shared a bar —
existed prior to this and was removed: it required exact timestamp coincidence
across every ticker, which independently-sampled activity bars essentially never
give you, producing an empty timeline.)

**At any given step, only some tickers are fresh — the rest carry forward their
last-known state.** `on_bar()` exposes this per ticker through `window_valid` and the
window rows' own timestamps. With `last = i * window_len + window_len - 1` (ticker
`i`'s newest row):

- **Fresh this step** — `window_valid[last] == 1`: ticker `i`'s bar is one that
  just landed. `0` on every other step, including before the ticker's first bar and
  on every step where a *different* ticker's bar was the one driving that step. For
  tickers sharing a common time grid (ordinary time bars), every step where a ticker
  is present is fresh.
- **Has ever had data** — any row of ticker `i` with `window_valid == 1`, or a
  nonzero `window[last].ts`. A ticker before its first bar holds zeroed placeholder
  rows (`ts == 0`).
- **Staleness** — `current_ts − window[last].ts`, where `current_ts` is the largest
  `window[k * window_len + window_len - 1].ts` across all tickers (at least one ticker
  is fresh on every step). A forward-filled row keeps the `ts` of the last real bar it
  repeats.

A forward-filled row repeats the last known `open/high/low/close` (a price *level*
remains the best estimate) and sets `volume`/`notional` to zero rather than carrying the
prior period's total forward — both are period *flow* quantities, and zero is the honest
statement that nothing traded during a step where this ticker produced no bar at all.

An order for a ticker that is not fresh this step is rejected (`"no market data"`):
submit orders only for tickers whose newest row is real.

A strategy reading multiple asynchronously-sampled tickers together should always check
`window_valid` before treating a read as fresh information, and may use staleness to decide
how to treat a ticker that hasn't updated this step — e.g. skip it entirely (no data yet),
or use its last-known value if it's still within a staleness bound the strategy defines.
