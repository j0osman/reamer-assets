---
title: "Order Lifetime: IOC, GTD, and Why It Matters"
description: An order doesn't just either fill or not — it stops being eligible to fill at some point too. Treating every order as though it waits indefinitely quietly tests a more patient strategy than the one that was actually written.
date: 2026-07-19
tier: Practical Research Workflows
---

Most backtesting examples implicitly treat every order as good indefinitely — submit it, and it sits there until it fills, however long that takes. Real orders don't work that way, and a strategy whose actual logic is "get in now or don't get in at all" behaves differently depending on whether that's genuinely modeled or just quietly assumed away by a framework that only knows one kind of order lifetime.

## Two different ways an order can stop waiting

An Immediate-Or-Cancel order attempts to fill within the same bar it's submitted on and is cancelled at that bar's close if it doesn't — it never persists to the next one. A market IOC order fills essentially by construction, since market orders don't wait for a price level; a limit IOC order only fills if [that specific bar's price action actually reaches the limit](https://reamerlabs.com/blog/how-limit-orders-really-behave), and if it doesn't, the order is gone rather than carried forward hoping the next bar does better.

```python
return buy_limit(128.0, 10.0, "IOC", ticker=self.ticker)
```

A Good-Till-Date order is the opposite shape — it stays open across as many bars as it takes, but only up to an explicit expiry timestamp, after which its status becomes distinct from both a fill and a manual cancel, so which orders ran out the clock is always visible in the order log rather than indistinguishable from the ones that were actively cancelled.

## What changes if this isn't modeled

A strategy backtested with no time-in-force specified is implicitly a strategy willing to wait arbitrarily long for its entry — every unfilled limit order just sits there until it eventually fills or the run ends. That's a legitimate thing to test. It's a different thing from a strategy whose actual edge specifically depends on speed — a breakout entry, a reaction to a news release, anything where the setup stops being valid a few bars after it appears. Backtest that kind of strategy without the right time-in-force and the extra patience isn't a rounding error: the fills that show up because the order was allowed to linger aren't fills the real strategy would ever have gotten, and the backtest ends up validating a strategy nobody actually intended to write.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
