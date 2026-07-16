---
title: Modeling Bid/Ask Correctly
description: A buy pays the ask and a sell receives the bid — spread is a directional cost paid on every round trip, not symmetric noise. Getting the data's own bid/ask/midpoint convention wrong silently biases every fill the same way.
date: 2026-07-17
---

Spread gets treated, sometimes, as a small symmetric wobble around the "real" price — as likely to help a fill as hurt it. It isn't symmetric. A buy pays the ask. A sell receives the bid. Spread is a cost paid in the same direction every single time, on every round trip, and modeling it as anything else understates a real, structural cost.

## Why direction is the whole point

The ask is always at or above the bid — that gap is the spread, and it exists specifically so that whoever is providing liquidity gets compensated for the risk of doing so. A trader crossing that spread to enter or exit a position pays it every time, regardless of which direction the trade goes: buying costs the ask, selling receives the bid, and the difference between the two is gone the moment the trade executes, win or lose. A fill model that doesn't reflect this — that just fills at a single mid-price — is quietly assuming a cost away rather than measuring it.

## Why the data's own convention matters

[Historical OHLCV data itself](https://reamerlabs.com/blog/why-ohlcv-execution-is-harder-than-it-looks) represents one of three things: bid prices, ask prices, or a midpoint between them. Which one it is changes which direction the spread needs to be applied — treating ask-quoted data as if it were midpoint data, for instance, doesn't just introduce a small error, it applies the spread adjustment in a direction that's already partially baked into the raw numbers, compounding rather than correcting the bias. Knowing what the underlying data actually represents, and configuring `ohlcv_type` to match it, is what keeps the spread adjustment pointed the right way instead of stacking on top of a data convention that already accounts for part of it.

## A cost, not a rounding error

None of this is a fine adjustment for extra precision. On a strategy that trades often, or trades instruments with a wide spread relative to the size of a typical move, the spread paid on entry and exit can be a meaningful fraction of the entire result — sometimes the difference between an apparent edge and no edge at all. Modeling it correctly, in the right direction, using the right assumption about what the raw data represents, is not a detail bolted onto the real measurement. It's part of what makes the measurement real.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model this rests on: [execution specification](https://github.com/reamerlabs/Reamer/blob/main/EXECUTION_SPEC.md) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
