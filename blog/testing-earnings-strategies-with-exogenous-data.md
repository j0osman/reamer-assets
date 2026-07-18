---
title: "Testing Earnings Strategies with Exogenous Data"
description: An earnings backtest is only honest if the data becomes visible on the date it was actually announced, not the date it happened to land in a spreadsheet. Exogenous data's point-in-time resolution is what makes that distinction real instead of assumed.
date: 2026-07-19
tier: Practical Research Workflows
---

An earnings-surprise strategy lives or dies on one detail that's easy to get wrong without ever noticing: the exact bar at which the surprise becomes knowable. A dataset keyed by fiscal quarter, joined against price data by date, will happily let an EPS number sit next to bars that came before the company actually announced it — the join succeeds, the backtest runs, and the strategy quietly gets to react to information a day or two before the market could have. That's not a small bias. It's the single easiest way to manufacture an edge that was never real — exactly the [no-lookahead discipline good experiment design requires](https://reamerlabs.com/blog/designing-a-good-trading-experiment), applied to the specific case of event data instead of price data.

## Reading a value as of a specific bar, not a specific date

Reamer attaches an arbitrary, freeform value to a ticker at a specific timestamp — an earnings surprise, a guidance note, anything JSON-serializable — and `on_bar` reads back whatever entry is current as of that exact bar, never a later one. The engine doesn't parse or validate the value; it only tracks which entry is the latest-known one at each step, which means the actual moment a strategy can react to an earnings number is the timestamp it was attached at, not the fiscal period it describes.

```python
tv = data["AAPL"]
if not tv.exogenous_valid:
    return None
info = json.loads(tv.exogenous)
if info.get("eps_surprise", 0) > 0.02:
    return buy_market(10, ticker="AAPL")
```

## The discipline is in the sidecar file, not the engine

The engine enforces point-in-time correctness once data is attached — it won't leak an entry before its own timestamp. It has no way to know if the timestamp itself is wrong, and that's exactly where a naive earnings dataset goes bad: an announcement made after market close on Tuesday and reflected in Wednesday's price action needs a Tuesday-evening or Wednesday-morning timestamp, not a Tuesday date that predates when the number actually existed publicly. Getting that timestamp right, sourced from the real announcement time rather than a report's nominal date, is the actual work behind a trustworthy earnings backtest — the engine's job is to hold that line once it's set correctly, not to catch a wrong one.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
