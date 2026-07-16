---
title: Stop Orders, Gaps, and Reality
description: A limit order guarantees a price. A stop order only guarantees a trigger — once crossed, it fills at whatever the market gives, which can be meaningfully worse than the stop level itself when price gaps through it.
date: 2026-07-17
---

Limit orders and stop orders get lumped together as "orders with a price attached," but they promise fundamentally different things. A limit order guarantees the price, at the cost of maybe never filling. A stop order guarantees almost the opposite: once triggered, it will fill, but not necessarily anywhere close to the level that triggered it.

## What a stop order actually is

A stop order sits inactive until price crosses a specified level, and once triggered, it behaves like a market order from that point forward — filling at whatever price is actually available, not at the stop level itself. In a market moving in small, continuous steps, the difference between the stop level and the actual fill is usually minor. It's not always minor.

## Where gaps come from

Price doesn't always move continuously. Across a weekend, around a scheduled news release, at the open after low-volume hours — price can jump straight through a stop level without ever trading at it. A stop order in that situation still fills, but at the first price actually available on the other side of the gap, which can be meaningfully worse than the level that was supposed to protect the position. This isn't a modeling edge case to wave away; it's exactly the situation stop-losses exist for, and exactly the situation where the fill price and the stop price can diverge the most.

## Why conflating stops and limits changes a result

Modeling a stop-loss as though it fills precisely at the stop price — the way a limit order fills at its limit price — quietly removes the one risk stop orders are specifically exposed to. A strategy backtested this way looks like its downside is more contained than it actually is, in exactly the scenario (a fast, gapping move) where real risk tends to concentrate. The two order types need genuinely different fill logic: a limit fills at its price or not at all; a stop, once crossed, fills at whatever the market actually offers next.

## Reality, not pessimism

None of this means every stop fills badly — most of the time, in a liquid, continuously-traded market, a stop fills close to its level. The point is that "close, most of the time" is a different claim than "exactly, every time," and a backtest that assumes the second is quietly hiding the specific kind of loss stop orders are least protected against.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://github.com/reamerlabs/Reamer/blob/main/EXECUTION_SPEC.md) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
