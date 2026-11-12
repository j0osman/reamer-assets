---
title: Reamer Research for C++
description: The one-header C++20 wrapper for Reamer Research: building with CMake or pkg-config, strategies, fluent orders and brackets, options, results and errors.
group: start
order: 5
product: research
source: reference-cpp/README.md
---

One header, `include/reamer_research.hpp` (also in the kit's `include/`), over the Reamer Research C ABI. Include it, link `libreamer_research`, compile as C++20. There is nothing to build first.

```cpp
#include <reamer_research.hpp>
#include <algorithm>
#include <iostream>
namespace rr = reamer_research;

int main() {
  auto r = rr::backtest([](rr::Data& d) {
    auto c = d.close(), h = d.high(), l = d.low();
    if (c[0] == 0) return;                // warm-up: the window is still filling
    double atr = 0;
    for (size_t i = 1; i < c.size(); ++i)
      atr += std::max(h[i], c[i - 1]) - std::min(l[i], c[i - 1]);
    atr /= double(c.size() - 1);
    double mid = c.mean(), px = c.back();
    if (d.position() == 0 && px < mid - 2 * atr) d.buy_market(10).bracket(mid, px - 2 * atr);
    if (d.position() == 0 && px > mid + 2 * atr) d.sell_market(10).bracket(mid, px + 2 * atr);
  }, "AAPL.csv", {.lookback = 20, .open_viewer = true});
  std::cout << r << '\n';
}
```

That is a complete program, an ATR band reversion: it fades a close more than two ATRs from its 20-bar mean, with a bracket that targets the mean and stops two ATRs further out. It reads the CSV, runs the strategy, prints the summary and opens the run in the trade viewer (`viewer/README.md`). It is `examples/quickstart.cpp`.

**`EXECUTION_SPEC.md` (kit root) is authoritative** on fill, slippage, spread, commission and margin behaviour. `include/reamer_research_abi.h` is the ABI contract. This header ships in source (LICENSE.txt, Section 4): read it, change it, keep it in your own tree.

## Building

From the kit root, any of three ways.

**By hand**

```bash
c++ -std=c++20 -O2 run.cpp -Iinclude -Llib -lreamer_research -Wl,-rpath,"$PWD/lib" -o run
```

**CMake** (Linux and macOS)

```cmake
find_package(ReamerResearch 4 REQUIRED)
target_link_libraries(run PRIVATE ReamerResearch::reamer_research)
```

```bash
cmake -S . -B build -DCMAKE_PREFIX_PATH=/path/to/research-kit
```

The target carries the include path, C++20 and the library's location.

**pkg-config** (Linux and macOS)

```bash
export PKG_CONFIG_PATH=/path/to/research-kit/lib/pkgconfig
c++ run.cpp $(pkg-config --cflags --libs reamer-research) -o run
```

Each route sets an rpath, so the program finds the library at run time. The trade viewer page is found next to the library, at `<kit>/viewer/reamer-viewer.html`; set `REAMER_VIEWER_HTML` to use another copy. On a machine with no desktop (SSH, a server) the viewer path is printed and no browser is started.

The header needs a C++20 compiler and the standard library, nothing else.

## Strategies

A strategy is a lambda or function taking `rr::Data&`, or an object with `on_bar(rr::Data&)`. An object may also carry `lookback` (bars per window) and `on_progress(done, total)` (return `true` to stop the run). An exception thrown in either stops the run and is rethrown from `backtest()`.

```cpp
struct Breakout {
  size_t lookback = 20;
  void on_bar(rr::Data& d) {
    double close = d.close().back();
    if (d.position() == 0 && close >= d.high().max())
      d.buy_market(100).bracket(close * 1.04, d.low().min());
  }
};

auto r = rr::backtest(Breakout{}, "SPY.bin");
```

Orders are placed by calling methods on `Data`: `buy_market(qty)`, `sell_market(qty)`, `buy_limit(price, qty)`, `sell_limit`, `buy_stop(price, qty)`, `sell_stop`, `close_position(qty = 0)` (0 closes all of it) and `cancel(order_id)`. Each returns the order just placed, so brackets and time in force chain on: `.take_profit(p)`, `.stop_loss(p)`, `.bracket(tp, sl)`, `.gtc()`, `.ioc()`, `.gtd("2024-06-30 16:00:00")`. Actions on what is already open: `modify_order(id, price, tp, sl)`, `modify_position(tp, sl)`, `close_all()` and `cancel_all()`. An order placed on a bar is eligible from the next bar (`EXECUTION_SPEC.md` §6).

What `on_bar` reads:

| Call | Returns |
|---|---|
| `d.close()`, `open()`, `high()`, `low()`, `volume()` | A column over the window. `back()` or `ago(0)` is the current bar, `ago(1)` the one before, `[0]` the oldest. `sum(n)`, `mean(n)`, `max(n)` and `min(n)` cover the last `n` bars, or the whole window when `n` is 0. |
| `d.bar(ago)` | One whole bar: `ts`, `open`, `high`, `low`, `close`, `volume`, `notional`. |
| `d.ts()` | The current step's time, epoch seconds UTC. |
| `d.position()` | Signed position after this bar's fills: positive long, negative short, 0 flat. |
| `d.position_state()` | Size, average entry price, the attached brackets, unrealised P&L and entry time. |
| `d.open_orders()`, `d.action_results()` | Resting orders with their ids, and how the previous bar's actions went. |
| `d.exogenous()` | The latest value of the ticker's exogenous sidecar at or before this bar, if there is one. |

### Several tickers

Pass several files, or a `rr::Universe` of in-memory bars. Each ticker is named after its file (`AAPL.csv` is `"AAPL"`), and every call then names one: `d["AAPL"].close()`, `d.position("AAPL")`, `d.buy_market(10, "AAPL")`, `d.close_position("AAPL")`. One wall clock runs the backtest and every row of every ticker is an event on it, so rows never need to line up. `d["AAPL"].fresh()` says whether that ticker's row closed at this step or was carried forward.

```cpp
auto r = rr::backtest([](rr::Data& d) {
  for (const auto& name : d.names()) {
    auto t = d[name];
    if (t.fresh() && t.position() == 0 && t.close().back() > t.close().mean())
      d.buy_market(1, name);
  }
}, {"AAPL.csv", "MSFT.csv"}, {.lookback = 50});
```

## Data

`backtest()` takes a `.csv` or `.bin` path, a list of paths, or bars in memory (`rr::Bars`, a `std::vector<ReamerOhlcvBar>`). When every path is a `.bin` file, the files are memory-mapped, so data larger than RAM runs (`DEPLOYMENT.md`, "Run from `.bin` files").

- `rr::load_csv(path)` reads a CSV with a header row: a time or date column (plus an optional separate time column), open, high, low, close, and optionally volume. Header names are matched loosely (`Date`, `<CLOSE>`, `Timestamp`), so MetaTrader exports work as saved. Timestamps may be date strings, read as UTC, or epoch seconds, milliseconds, microseconds or nanoseconds. Rows are sorted by time.
- `rr::write_bin(path, bars)` and `rr::read_bin(path)` write and read `.bin` files, the same files `bin/reamer-csv-build` makes.
- `rr::write_exogenous(path, {{ts, "value"}, ...})` writes an exogenous sidecar (`EXOGENOUS_FORMAT.md`). Pass it to a run as `.exogenous = {{"AAPL", "aapl.exo.bin"}}`.

## Options

Every field has a default, so name only what differs:

```cpp
auto r = rr::backtest(strategy, "ES.bin", {
  .lookback = 30,
  .initial_capital = 100000,
  .leverage = 2,
  .costs = {.commission_per_unit = 2.5, .slippage = 0.25, .spread = 0.25},
  .rng_seed = 7,
  .start = 1704067200,          // epoch seconds UTC, inclusive
  .result_json = "es.json",
  .open_viewer = true,
});
```

`costs` is the execution config from `EXECUTION_SPEC.md` §0a: commission and its mode, slippage, `price_volatility`, spread, bid, ask or mid data, swap by weekday and the roll cycle. `ticker_costs` overrides it for named tickers. Designated initialisers must keep declaration order: `lookback`, `initial_capital`, `leverage`, `costs`, `rng_seed`, `ticker_costs`, `exogenous`, `start`, `end`, `periods_per_year`, `on_progress`, `result_json`, `viewer_html`, `open_viewer`.

## Results

`backtest()` returns an `rr::Result`, and `std::cout << r` prints its summary.

- `r.summary()` is the full `ReamerBacktestSummary`: P&L, fees, slippage, swap, win rate, profit factor, Sharpe, Sortino, drawdown and trade counts. `r.net_pnl()` is the headline.
- `r.closed_trades()` returns every round trip, however many there are.
- `r.json()` is the full result document (`RESULT_JSON_SCHEMA.md`: orders, fills, trades, equity curve), and `r.write_json(path)` saves it.
- `r.monte_carlo(simulations, seed)` resamples the closed trades and returns the distribution of outcomes.
- `r.write_viewer(path)` writes the trade viewer page with this run in it. `.open_viewer` and `.viewer_html` do the same as part of the run.
- `r.handle()` is the raw `ReamerBacktestHandle`, for any ABI call the wrapper does not cover.

Failures throw `rr::Error`: `code()` is the `ReamerResearchErrorCode` and `what()` the library's own message.

## Live orders

`rr::Relay` sends the same orders a backtest places to Reamer Server, or to a receiver of your own over the published relay protocol (the relay section of `include/reamer_research_abi.h`). Linux and macOS. Moving a strategy from research to the live path is covered in `RESEARCH_TO_SERVER.md`.

```cpp
rr::Relay relay("/run/reamer/strategy.sock", {"AAPL"});
relay.send(rr::buy_market(10).stop_loss(180));
```

## Examples

- **`quickstart.cpp`**: the program at the top of this page. `./quickstart your.csv` runs it on your own file.
- **`buy_and_hold.cpp`**, **`atr_stop_bracket.cpp`**, **`donchian_breakout.cpp`**: strategies on synthetic bars, written against the 4.x reference wrapper below.
- **`rejected_orders.cpp`**: submits malformed orders and reads the rejections back. A run whose orders were all rejected still succeeds; only the order log shows what the engine refused.

Build any of them as shown under Building.

## The 4.x reference wrapper

The same header keeps the earlier wrapper in namespace `reamer_research_ref`, so code written against it compiles unchanged: `run_backtest(bars_by_ticker, ticker_names, on_bar, config, capital, leverage)` with a raw `on_bar(window, ticker_count, positions, window_valid, out_orders)` callback, returning `std::optional<BacktestResult>`. Its `closed_trades()` is no longer capped at 4,096 trades.

## Overhead

The views `on_bar` reads are borrowed pointers into the engine's window, so no bar is copied; the wrapper adds a trampoline call per bar.

## License

Provided under the terms of the Reamer Research licence, LICENSE.txt in the kit.
