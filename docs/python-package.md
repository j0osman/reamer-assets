---
title: Reamer Research for Python
description: Install the Reamer Research Python package and run a backtest with rr.backtest(): strategies, bar input, costs, orders, brackets, results and the trade viewer.
group: start
order: 4
product: research
source: reference-python/README.md
---

`import reamer_research` runs the Reamer Research engine from Python. The library ships prebuilt inside the package, so there is nothing to compile.

```python
import numpy as np
import reamer_research as rr

def on_bar(d):
    c, h, l = d.close, d.high[1:], d.low[1:]
    if c[0] == 0: return               # warm-up: the window is still filling
    atr = (np.maximum(h, c[:-1]) - np.minimum(l, c[:-1])).mean()
    mid, px = c.mean(), c[-1]
    if d.position() == 0 and px < mid - 2 * atr:
        return rr.buy_market(10, take_profit=mid, stop_loss=px - 2 * atr)
    if d.position() == 0 and px > mid + 2 * atr:
        return rr.sell_market(10, take_profit=mid, stop_loss=px + 2 * atr)

print(rr.backtest(on_bar, 'AAPL.csv', lookback=20, open_viewer=True))
```

That is a complete script, an ATR band reversion: it fades a close more than two ATRs from its 20-bar mean, with a bracket that targets the mean and stops two ATRs further out. It reads the CSV, runs the strategy, prints the summary and opens the run in the trade viewer (`viewer/README.md`). It is `examples/quickstart.py`.

**`EXECUTION_SPEC.md` (kit root) is authoritative** on fill, slippage, spread, commission and margin behaviour. The package ships in source (LICENSE.txt, Section 4): read it, change it, keep it in your own tree.

## Install

From the kit root, either:

```bash
pip install python/reamer_research-*.whl     # the prebuilt wheel
pip install ./reference-python               # or from source, which copies in lib/ and the viewer
```

Both install the same package, with the library and the viewer page inside it, so `import reamer_research` works from any directory and any virtualenv. It needs Python 3.9+ and numpy; pandas is optional (`pip install "./reference-python[pandas]"`). To use the package without installing it, put `reference-python/` on `PYTHONPATH`, and it finds the library in the kit's `lib/`. `REAMER_RESEARCH_LIB` overrides the search either way.

## Running a backtest

```python
result = rr.backtest(strategy, bars, lookback=30, commission_per_unit=0.005, spread=0.01)
```

**`strategy`** is a function `on_bar(data)`, an object with an `on_bar(data)` method, or a class (instantiated once). An object or class may set `lookback`, and `on_progress(done, total)`, which stops the run by returning `True`. An exception raised in either stops the run and is raised from `backtest()`.

**`bars`** is any of:

- a path to a `.csv` or `.bin` file, or a list of paths. Each ticker is named after its file (`AAPL.csv` is `'AAPL'`).
- `{ticker: bars}`, where bars is a pandas DataFrame, a numpy `rr.BAR_DTYPE` array, a list of `(ts, open, high, low, close, volume, notional, tick_count)` tuples, or a path.

When every ticker is a `.bin` path, the files are memory-mapped, so data larger than RAM runs.

Keyword arguments:

| Argument | Meaning |
|---|---|
| `lookback` | Bars per window in `on_bar`. Defaults to the strategy's `lookback`, else 1. |
| Any key of `rr.DEFAULTS` | `initial_capital`, `leverage`, `commission_per_unit`, `commission_mode`, `slippage`, `price_volatility`, `spread`, `ohlcv_type`, `swap_per_unit`, `rng_seed`, `roll_cycle_months`, `roll_days_before_monthend`, `periods_per_year`. `EXECUTION_SPEC.md` §0a defines each. An unknown name is an error. |
| `ticker_config` | `{ticker: {setting: value}}` for one ticker's costs where they differ from the run's. |
| `exogenous` | `{ticker: sidecar}`: a `.exo.bin` path, or the records themselves (`{time: value}`). `on_bar` reads the latest value with `data.exogenous(ticker)`. See `EXOGENOUS_FORMAT.md`. |
| `start`, `end` | Epoch seconds UTC, inclusive. Only bars in that range run. |
| `result_json` | Write the full result document there (`RESULT_JSON_SCHEMA.md`). |
| `console_log` | Write everything the strategy prints there too, as JSON Lines stamped with each bar's time. It still reaches the terminal. |
| `open_viewer` | Open the run in the trade viewer when it finishes, with its result, bars and printed output loaded. On a machine with no desktop, the page's path is printed instead. |
| `viewer_html` | A path for that page. Given alone, it writes the page without opening a browser. |

Every cost defaults to zero. `templates/engine_config.json` holds illustrative non-zero costs, so a first run can be a costed fill rather than a frictionless one. Replace them with your own venue's figures before sizing anything off a result.

Runs are reproducible byte for byte at a fixed `rng_seed`.

## The `on_bar` callback

`on_bar(data)` runs once per timeline step, after the bar is sealed. An order it returns is eligible from the **next** bar onward (`EXECUTION_SPEC.md` §6).

In a one-ticker run, read the window straight off `data`. With several tickers, take a view first: `tv = data['AAPL']`. Views are zero-copy numpy arrays over the engine's memory.

| Attribute | Type | Description |
|---|---|---|
| `close`, `open`, `high`, `low` | float64 `(lookback,)` | Index `-1` is the current bar, `0` the oldest in the window. |
| `volume`, `notional` | float64 `(lookback,)` | Volume, and currency value (`0.0` if the data has none). |
| `ts` | int64 `(lookback,)` | Bar times, epoch seconds UTC. |
| `valid` | `bool` | `True` if this ticker's row closed at this step, `False` if it is carried forward. |

On `data` itself:

| Call | Returns |
|---|---|
| `data.position(ticker=None)` | Signed position after this bar's fills: positive long, negative short, `0.0` flat. |
| `data.position_state(ticker=None)` | A dict of `qty`, `avg_entry_price`, `take_profit`, `stop_loss`, `unrealized_pnl` and `entry_ts`. |
| `data.open_orders()` | Resting orders after this bar's fills, with their `order_id`s. |
| `data.action_results()` | How the actions returned on the previous bar went: `applied`, `reject_reason`. |
| `data.exogenous(ticker)` | The latest sidecar value at or before this bar, as bytes, or `None`. |
| `data.ts` | This step's time, epoch seconds UTC. |
| `data.tickers` | Ticker names, in ticker id order. |

The ticker argument may be left out in a one-ticker run.

### Several tickers

One wall clock runs the whole backtest and every row of every ticker is an event on it, so rows never need to line up. Gate on `tv.valid` so a carried-forward row is not mistaken for a fresh one.

```python
def on_bar(data):
    out = []
    for t in data.tickers:
        tv = data[t]
        if tv.valid and data.position(t) == 0 and tv.close[-1] > tv.close.mean():
            out.append(rr.buy_market(1, ticker=t))
    return out

result = rr.backtest(on_bar, ['AAPL.csv', 'MSFT.csv'], lookback=50)
```

`templates/multi_asset.py` is a fuller example.

### Return value

| Return | Meaning |
|---|---|
| `None` | No action this bar |
| An order or action | One order or action |
| A `list` or `tuple` of them | Several this bar |

## Orders and actions

```python
rr.buy_market(qty, *, ticker=None, tif='GTC', expiry_timestamp=None, take_profit=None, stop_loss=None)
rr.sell_market(qty, ...)                 # same keywords
rr.buy_limit(limit_price, qty, ...)      # fills at the limit or better
rr.sell_limit(limit_price, qty, ...)
rr.buy_stop(stop_price, qty, ...)        # triggers when price reaches the stop
rr.sell_stop(stop_price, qty, ...)
rr.close_position(qty=0, *, ticker=None) # 0 closes the whole position
rr.cancel_order(order_id, *, ticker=None)

rr.modify_order(order_id, *, price=None, take_profit=None, stop_loss=None)
rr.modify_position(*, ticker, take_profit=None, stop_loss=None)
rr.close_all()
rr.cancel_all()
```

- **Brackets**: `take_profit` and `stop_loss` attach to any entry (`EXECUTION_SPEC.md` §0b and §2). Bracket exits are folded into the closed trade's fees and slippage, not recorded as separate orders.
- **Closing**: `close_position()` closes the whole position, with the direction resolved by the engine. `close_position(qty=N)` closes `N` units and leaves the rest open (§11). It is rejected when flat.
- **Time in force**: `'GTC'` (the default), `'IOC'`, or `'GTD'` with `expiry_timestamp="YYYY-MM-DD HH:MM:SS"` (§6).
- **Order ids**: `data.open_orders()` lists every resting order with its `order_id`, for `cancel_order` and `modify_order`.

## Results

`backtest()` returns an `rr.Result`, and `print(result)` shows the headline table.

| Member | Description |
|---|---|
| `result.net_pnl`, `sharpe_ratio`, `win_rate`, ... | Every summary field is an attribute, and a key of `result.summary`. |
| `result.closed_trades` | Every round trip as a dict: `ticker`, `open_timestamp`, `close_timestamp`, `side`, `entry_price`, `exit_price`, `qty`, `gross_pnl`, `net_pnl`. Uncapped. |
| `result.trades_frame()` | The same as a pandas DataFrame. |
| `result.json()`, `result.write_json(path)` | The full result document: every fill, the order log with reject reasons, the equity curve. |
| `result.monte_carlo(simulations=10000, seed=0)` | Resamples the per-trade returns and returns the distribution's statistics. |
| `result.open_viewer(path=None)` | Writes the run into the trade viewer and opens it. |

**`total_slippage_cost` is not total transaction cost: it excludes spread.** Spread is in `entry_price` and `exit_price` but is not reported separately (§3). To measure a fill against the bar, diff `entry_price` against your own bars at `open_timestamp`; `examples/quickstart_run.py` shows how.

**Slippage is not floored at zero.** With `price_volatility > 0`, sampled slippage is uncapped and unclamped, so single fills and the total can be negative, which is genuine price improvement (`EXECUTION_SPEC.md` §1).

## Bars and files

- `rr.load_csv(path)` reads a CSV with a header row into a `BAR_DTYPE` array. Header names are matched loosely (`Date`, `<CLOSE>`, `Timestamp`), so MetaTrader exports work as saved. Separate date and time columns, date strings (read as UTC) and epoch seconds, milliseconds, microseconds or nanoseconds all work. Rows are sorted by time.
- `rr.to_bars(df)` turns a pandas DataFrame (bar columns, with a time column or a DatetimeIndex) into bars.
- `rr.write_bin(path, bars)` and `rr.read_bin(path)` write and read the `.bin` files `bin/reamer-csv-build` makes.
- `rr.write_exogenous(path, {time: value})` writes a sidecar for `exogenous=`.
- `fetchers/` downloads bars ready for `backtest()`: `alpaca_bars.py` (US stocks with a free paper-account key, crypto with no key) and `databento_bars.py` (needs `pip install databento`, and refuses a request Databento prices above `max_cost_usd`). `examples/fetch_and_run.py` runs a strategy on them. Vendor accounts, fees and terms are your own.

## The trade viewer

`open_viewer=True` writes a copy of `viewer/reamer-viewer.html` with the run embedded, next to `result_json` as `<name>.viewer.html` or else as `reamer-run.viewer.html` in the current directory. It charts each trade on its bars, with every value you printed (`rsi=61.2`) plotted beside it, the console lines for each bar, and a Monte Carlo simulation. To build the same page from files a C or C++ program wrote:

```bash
python3 -m reamer_research.viewer result.json --bars AAPL.bin --console console.jsonl
```

## Live orders

`rr.Relay` sends the same orders a backtest returns to Reamer Server, or to a receiver of your own over the published relay protocol (the relay section of `include/reamer_research_abi.h`). Linux and macOS.

```python
with rr.Relay('/run/reamer/strategy.sock', ['AAPL']) as relay:
    relay.send(rr.buy_market(10, stop_loss=180))
    relay.poll(500)
    print(relay.positions())
```

Moving a strategy from research to the live path is covered in `RESEARCH_TO_SERVER.md`.

## Contents

- **`reamer_research/`**: the package.
- **`examples/quickstart.py`**: the script at the top of this page.
- **`examples/quickstart_run.py`**: a longer end-to-end run on `sample_data/AAPL.csv` with the costs in `templates/engine_config.json`, which also measures entry cost against the bar.
- **`templates/`**: starting-point strategies (buy-and-hold, mean reversion, ATR bracket, multi-asset) to copy and change.
- **`examples/`**: longer strategies, each explained in its docstring.
- **`fetchers/`**: the market-data fetchers above.

## Code written for 4.8

It runs unchanged. `from reamer_research_binding import run_backtest` and `from engine.orders import buy_market` still import, and `run_backtest(StrategyClass, config, {"AAPL": bars})` still takes one config dict and returns the same result, now an `rr.Result`. 4.9.0 also raises a strategy's exceptions instead of ending the run quietly, reads `lookback` from an instance as well as the class, and no longer caps `closed_trades` at 4,096.

## License

Provided under the terms of the Reamer Research licence, LICENSE.txt in the kit.
