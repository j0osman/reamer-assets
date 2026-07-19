---
title: "Testing a Trading Engine"
description: Most software testing asks whether code runs without crashing. Testing an execution engine against a published specification asks something stricter — does this exact scenario produce the exact fill the spec says it should, not just a plausible-looking one.
date: 2026-07-19
tier: Engineering
---

A test suite that checks "the backtest ran and produced a result" catches crashes. It does not catch a fill price that's wrong by a consistent, plausible-looking amount — the kind of bug that's far more dangerous than a crash, because nothing about the output looks broken. Testing a trading engine seriously means testing against a specific, written answer for each scenario, not against "did something reasonable happen."

## What the 282 checks are actually organized around

Reamer's conformance suite isn't 282 variations on the same assertion. It's organized around the categories of behavior the [execution specification](https://reamerlabs.com/spec) actually commits to: fill-price correctness per order type (market, limit, stop, and their interaction with slippage and spread), time-in-force and expiry behavior (GTC, IOC, GTD, and what happens at each boundary), bracket collision resolution (which side fires first when both are in range on the same bar), position-state transitions (scale-in, partial close, netted reversal), margin and cost-accounting identities, and multi-asset alignment behavior. Each category is tested with the specific edge cases that category tends to get wrong — a stop order exactly at a gap boundary, a bracket where both levels sit inside the same bar, a reversal sized larger than the existing position.

## Why edge cases, not the common case

The common case in each of those categories is usually easy to get right and easy to eyeball. The edge cases are where an execution engine's actual correctness lives — what happens when a limit price sits exactly on the bar's high, when TIF expiry and a fill attempt land on the same tick, when a scale-in changes a weighted-average entry price mid-run. A suite that only exercises the common case would pass while missing almost everything that actually matters about whether the engine can be trusted.

## What running the suite on every change actually buys

The conformance suite runs against every change to the execution core, not just before a release. That turns "did this change break correctness" from a question someone has to remember to ask into one the change either passes or fails automatically, before it ships. A specification that isn't continuously tested against tends to drift from the code that's supposed to implement it — the suite is what keeps that gap from opening quietly.

---

Full reference: [docs](https://reamerlabs.com/docs) · The specification the suite verifies: [execution specification](https://reamerlabs.com/spec)
