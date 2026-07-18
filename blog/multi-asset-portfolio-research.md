---
title: "Multi-Asset Portfolio Research"
description: Running the same independent logic across many tickers isn't the same as running many separate single-asset backtests — they share one equity curve and one leverage pool, and that constraint only shows up once they're actually run together.
date: 2026-07-19
tier: Practical Research Workflows
---

The simplest way to test a strategy across many instruments is to run it once per ticker and add up the results afterward. It's also a quietly wrong model of what a portfolio actually is. A real account has one pool of capital and one leverage limit shared across every position it holds — three instruments that each individually look fine can collectively demand more margin than the account has, and a strategy that never sees them compete for the same capital never gets tested against that.

## One equity curve, many tickers, still one constraint

A multi-asset run in Reamer keeps every ticker's decisions independent — a strategy declares `tickers` and loops over them, each one's entry and exit logic depending only on its own data — while every position still draws against the same shared equity and the same leverage ceiling:

```python
class MyStrategy:
    tickers = ["AAPL", "SPY", "QQQ"]

    def on_bar(self, data):
        out = []
        for t in self.tickers:
            tv = data[t]
            if not tv.valid:
                continue
            if tv.close[-1] > tv.open[-1] and tv.position.qty == 0:
                out.append(buy_market(1.0, ticker=t))
        return out if out else None
```

Nothing about that loop looks different from three separate single-asset strategies. What's different is invisible in the code: an entry attempt on any ticker is checked against the total notional across every other open position, so a portfolio that happens to want to enter several correlated instruments at once will start seeing rejections a single-asset backtest of any one of them would never produce.

Independent decisions are the common case, not the only one — [a strategy that ranks or weights tickers against each other needs a genuinely different code shape](https://reamerlabs.com/blog/researching-cross-asset-relationships), not a loop like this one.

## Mixed schedules are the other thing a per-ticker view hides

Equities and forex don't trade on the same calendar, and a portfolio mixing them needs `alignment_mode="union"` rather than the default intersection — every timestamp from any ticker gets a step, and `tv.valid` turns `False` for whichever ticker has no bar at that particular moment. Skipping that check in union mode isn't a cosmetic omission; it's the difference between a strategy that correctly sits out when its data isn't there yet and one that silently acts on stale or default values it was never supposed to see.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
