---
title: Trade viewer
description: The offline Reamer Research trade viewer: each closed trade charted on its bars beside the values your strategy printed, its console output, and a Monte Carlo simulation.
group: start
order: 6
product: research
source: viewer/README.md
---

`reamer-viewer.html` charts every closed trade of a backtest on its own bars,
next to whatever your strategy printed while it ran. It is one HTML file: open
it in any current browser. It needs no install, no server and no network, and
the files you add never leave the browser.

## Opening a run automatically

From Python or C++, ask for it in the run and the viewer opens on it when the
backtest finishes, with its result, bars and console output already loaded:

```python
result = rr.backtest(on_bar, "AAPL.csv", open_viewer=True)          # Python
```

```cpp
auto r = rr::backtest(on_bar, "AAPL.csv", {.open_viewer = true});   // C++
```

This writes a copy of the viewer with the run embedded, next to your
`result_json=` file as `<name>.viewer.html` or else as
`reamer-run.viewer.html` in the current directory, and opens it in your
default browser. Its path is also printed, and kept in `result.viewer_html`
in Python. On a machine with no desktop (SSH, a server) only the path is
printed; copy the page anywhere with a browser. Pass `viewer_html="path.html"`
(`.viewer_html` in C++) to choose the path; on its own, it writes the page
without opening a browser. In Python the console output is captured for the
page even if you don't pass `console_log=`.

From C or a shell, run the same step as a script once your program has
written its files:

```
python3 -m reamer_research.viewer result.json \
    --bars AAPL.bin --bars SPY=spy_m15.csv --console console.jsonl
```

(`python3 reference-python/reamer_viewer.py` does the same without the
package installed.)

Bar files are matched to tickers by name; `TICKER=path` names the ticker
yourself. `-o` sets the output path and `--no-open` skips the browser.

## What to add

To load files by hand, use the three cards on the start screen (**Result
JSON**, **Bar data** and **Console output**) or the matching buttons in the
header. You can also drop files on a card, or anywhere on the page, where each
file is recognised by its content. A green dot on a header button marks each
kind that is loaded, and the button shows the file's name.

| File | Where it comes from | Needed |
|---|---|---|
| Result JSON | `reamer_write_result_json()`, `result_json="result.json"` in Python, or `.result_json = "result.json"` in C++. Schema: `RESULT_JSON_SCHEMA.md`. | Yes |
| Bar data | The bars each ticker ran on: a `.bin` file from `bin/reamer-csv-build` or `reamer_write_bin()`, or a CSV. Name each file after its ticker (`AAPL.csv`, `AAPL.bin`, `AAPL_M15.csv`). If a name matches no ticker, the viewer asks which ticker it is. | Yes, to chart |
| Console log | `console_log="console.jsonl"` in Python, or any text log whose lines start with a timestamp (format below). | No |

CSV bar files need open, high, low and close columns, plus either a `Date` and
`Time` pair or one timestamp column (`timestamp`, `ts`, `datetime`). Comma,
semicolon and tab delimiters are all read, and MetaTrader-style `<DATE>`
headers work. Timestamps are read as UTC: epoch seconds or milliseconds, or
`YYYY-MM-DD HH:MM:SS` with `-`, `.` or `/` between the date parts.

## What it shows

- The run's summary metrics, and a sortable table of closed trades that you
  can filter by ticker and to winners or losers.
- The selected trade's details (entry, exit, take profit, stop loss,
  reward : risk, hold time, costs and P&L), and the trade on candles: entry
  and exit markers joined by a dotted line, take-profit and stop-loss levels
  tagged on the price axis, resting limit and stop orders from `order_log`,
  and the other trades in view drawn faint.
- Every number your strategy printed as `name=value` or `name: value` (an
  indicator, a signal, a position) as a series. A series whose values sit in
  the price range is drawn over the candles; any other series gets its own
  pane below. Click a series to hide or show it; right-click to move it
  between the price chart and its own pane.
- The console lines for the bars in view, with the selected trade's lines
  marked and its signal, entry and exit bars tagged. The signal bar is the bar
  the entry order was submitted on, taken from `order_log`. Hovering a bar
  highlights what it printed, and hovering a line puts the crosshair on its
  bar. **Whole run** lists everything, including output printed outside
  `on_bar`, and the search box searches the whole log. Clicking a line from
  outside the chart's window opens the trade that was open on its bar.
- Above the chart, a readout of one bar: its prices, every value your
  strategy printed for it, and its messages. It shows the trade's signal bar
  (or its entry bar) until you hover another bar.
- The equity curve. Clicking a point opens the trade that closed there.
- A Monte Carlo simulation of the run, using the same method as
  `reamer_run_monte_carlo()`: each simulation draws the run's per-trade
  returns (`returns[]`) with replacement, one draw per closed trade,
  compounding from the initial capital, with equity floored at 0. It shows
  the median and 5th percentile final equity, the probability of loss and of
  ending below half the capital, max drawdown percentiles, and where this run
  ranks. There is a fan chart of equity per trade (5–95% and 25–75% bands,
  the median, and this run) and a histogram of final equity. You can set the
  number of simulations and the seed. The viewer's random draws are its own,
  so its figures match the engine's statistically but not digit for digit.
  The simulation needs at least 2 closed trades.

The chart does not pan or zoom: it shows a fixed window of the selected trade,
from **Context** bars (40 by default) before its entry to the same number
after its exit. Change Context to widen or narrow it. `j` and `k` (or the
arrow keys) step through trades. Times are UTC.

The exit reason is inferred from prices (an exit at the stop or target level
is labelled as such) because the result document does not record what
triggered an exit.

## Console log format

The Python package writes JSON Lines when you pass `console_log=`: one record
per line printed, from `stdout` and `stderr` alike. Your output still reaches
the terminal as before.

```json
{"ts": 1704275100, "stream": "stdout", "text": "sma_fast=0.67626 sma_slow=0.67607 rsi=36.0"}
{"ts": null, "stream": "stdout", "text": "[init] strategy loaded"}
```

`ts` is the epoch second of the bar being processed when the line was printed:
the latest bar timestamp among the tickers with a fresh bar, which is the
engine's wall clock (`EXECUTION_SPEC.md`). It is `null` for lines printed
outside `on_bar`, such as in the strategy's constructor.

From C, C++ or anything else, either write the same JSON Lines, or print each
line with the bar's timestamp in front and save the output:

```
2024-01-03 09:45:00 sma_fast=0.67626 sma_slow=0.67607 rsi=36.0
2024-01-03 09:45:00 ENTRY long: fast crossed above slow
```

A line with no leading timestamp belongs to the stamped line above it. An
epoch second (`1704275100 ...`) works in place of the date. In a text log,
lines starting with `error`, `warn` or `traceback` are styled as errors.

For a number to become a series, print it as `name=value` or `name: value`,
with the same name on every bar. Names that appear on fewer than 2% of
printing bars are treated as one-off messages and not plotted. If a bar prints
the same name more than once, its last value is used.

In a run of several tickers, name the ticker on each line, as
`AUDUSD_M15 mid=0.67609 z=-1.25`, or in the key, as `AUDUSD_M15.mid=0.67609`.
The ticker's name up to its first `_` or `.` (`AUDUSD`) works too when no other
ticker shares it. Each ticker then gets its own series, and the chart, the
readout and the console list for the bars in view show only the charted
ticker's lines and lines that name no ticker. **Whole run** and the search box
still cover every line. Showing, hiding or moving a series does the same for
the series of that name on every ticker. A line that names no ticker belongs
to all of them, so in a run of several tickers a bare `mid=` printed for each
ticker on the same bar ends up as one series holding the last ticker's value.

```python
def on_bar(d):
    for t in d.tickers:
        tv = d[t]
        if tv.valid:
            print(f"{t} mid={tv.close.mean():.5f}")
```
