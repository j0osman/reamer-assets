---
title: "Scaling In and Reversing a Position Correctly"
description: Real position management isn't strictly flat, full-open, or full-close. A framework that only knows those three states forces a strategy's logic to work around the tool instead of expressing the actual idea.
date: 2026-07-19
tier: Practical Research Workflows
---

A lot of backtesting frameworks quietly assume a position is either flat, fully open, or fully closed — three states, clean transitions between them. Real trading has more shapes than that: adding to a position that's working, closing half of one to lock in part of a gain while leaving the rest open, flipping from long to short on a signal strong enough to override the existing position rather than just closing it out first. A framework that can't represent those states doesn't stop a strategy from having that logic — it just forces the strategy to work around the tool's limits instead of expressing the idea it was actually built to test.

## Adding to a position changes its cost basis, correctly

A same-side order submitted while already in a position adds to it rather than being rejected or silently opened as a second, separate position. The quantity increases, and the entry price updates to the weighted average across all lots — not held at the original fill, and not overwritten by the new one. That distinction isn't cosmetic: unrealized P&L, a trailing stop measured from entry, any risk check reading `entry_price` all need the position's real weighted cost basis, not a stale number that no longer reflects what's actually at risk.

## Closing part of a position, or flipping it entirely

A close-side order can take exactly part of a position off — closing five units of a ten-unit long leaves the other five open at its existing entry price, unaffected. And a close-side order sized *larger* than the current position doesn't just close it and stop: it closes the existing position and opens the opposite side for the excess, in one order. Selling fifteen units against a ten-unit long closes the long and opens a five-unit short — a full reversal, netted into a single transition, instead of requiring a strategy to submit a close order and a separate opposite-side entry and reason about the gap between the two.

```python
return sell_market(15.0, ticker=self.ticker)   # closes a 10-unit long, opens a 5-unit short
```

## None of this bypasses the margin check

A scale-in fill is still a new fill, checked against available margin the same as any entry — the resulting total notional across every open position still can't exceed equity times leverage. A strategy that scales in aggressively without a real ceiling in its own logic starts seeing rejections rather than silently over-leveraging, which is the correct failure mode: [a rejected order visible in the log](https://reamerlabs.com/blog/why-replay-matters), not a fill that should never have happened in the first place.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://reamerlabs.com/spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
