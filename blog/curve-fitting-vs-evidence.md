---
title: Curve Fitting vs Evidence
description: A strategy with many adjustable parameters, tuned against the same data it's later evaluated on, will eventually look good almost regardless of whether it captures anything real. Recognizing curve fitting is about noticing the shape of the process, not just the result.
date: 2026-07-17
---

Given enough adjustable parameters and enough attempts, almost any strategy can be tuned to look good on a specific, fixed set of historical data — not because it captures something real, but because enough knobs turned enough times will eventually align with whatever noise happens to be sitting in that data. Curve fitting isn't a rare mistake. It's the default outcome of tuning freely against the same data a result is later judged on, and recognizing it is more about noticing the shape of the process than about anything visible in the final number.

## The warning signs are in the process, not the output

A curve-fit result and a genuinely good one can look identical at the end — that's exactly what makes curve fitting dangerous. The signs worth watching for show up earlier: a large number of free parameters relative to how much data is available to constrain them, parameters that were adjusted specifically in response to how the equity curve looked after a previous run, or a strategy that went through many rounds of tuning against the exact same historical window it's ultimately being evaluated on. None of these guarantee the result is fake. All of them raise the odds enough to be worth taking seriously.

## Why more parameters need more suspicion, not more confidence

Each additional free parameter gives a strategy one more way to bend toward whatever pattern happens to exist in a fixed dataset, real or not. A strategy with two or three parameters, chosen for a specific, statable reason, is making a much narrower bet than one with a dozen thresholds and filters, each nudged slightly during tuning until the result improved. More flexibility isn't inherently a red flag — a genuinely complex market phenomenon might need it — but it does raise the bar for how skeptical the eventual result deserves to be treated.

## Evidence is what a curve fit doesn't have

A result built on a real, persistent pattern doesn't depend on having been tuned against the exact data it's judged on — it should hold up reasonably well against [a fresh, independent check the tuning process never touched](https://reamerlabs.com/blog/validation-is-not-backtesting). That's the practical difference: evidence survives being tested somewhere the fitting process wasn't looking. A curve fit, by construction, only really works where it was fitted, and its usual first sign of trouble is exactly that — a result that looks strong on the tuned data and considerably weaker the moment it's asked to do the same thing anywhere else.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
