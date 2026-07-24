---
title: "One Compiled Module, Every Asset Class"
description: Reamer briefly shipped a separate compiled Python module per asset class, each hiding the fields the others didn't need. It was removed the same cycle it was built, once splitting modules turned out to cost more than it protected.
date: 2026-07-19
tier: Engineering
---

For a short period during futures support development, Reamer built `_reamer_py` as three separate compiled modules — one each for equities/FX, futures, and crypto — gated by a build flag, each exposing only the execution-model fields relevant to its own asset class. Futures got `roll_cycle_months`; the others didn't see it at all. The reasoning at the time seemed sound: why expose a futures-specific field to someone backtesting equities?

## Why it got removed in the same cycle it shipped

Three concrete costs showed up almost immediately, not eventually. First, the split had zero actual entitlement benefit — Reamer's license is per-machine and all-inclusive, so hiding fields behind a build flag protected nothing a paying user was supposed to be kept out of. Second, it broke something real: pybind11's type registry is process-wide, so two separately-compiled modules exposing overlapping type names collide the moment both get imported in the same Python process — which is exactly what mixing asset classes in one script requires, and exactly what a single all-inclusive license should have made possible in the first place. Third, a wheel-specific compile define meant a feature built for one wheel variant silently never reached the GUI or the test suite, because those were never told which variant they were supposed to be testing against.

## What replaced it

One compiled module, `_reamer_py`, with every asset-class field — futures' `roll_cycle_months`, and any future asset-class-specific field — always present and always Python-settable, inert unless a strategy actually sets it. The same pattern the FX-specific `swap_per_unit` field already used before any of this happened: presence isn't gated by which build is installed, behavior is gated by whether a strategy actually uses it. Per-ticker behavior differences come from `ticker_overrides`, a separate mechanism, entirely independent of which module got imported.

## The general lesson, not just this specific one

Hiding unused surface area behind a compile-time split looks like good API hygiene until the split itself becomes the thing that breaks — and the failure mode here wasn't subtle: a script that worked fine backtesting one asset class in isolation stopped working the moment it tried to backtest two at once, for a reason that had nothing to do with the strategy logic. One module, with fields that are simply unused when irrelevant, turned out to be both simpler to maintain and strictly more capable than the split it replaced.

---

Full reference: [docs](https://reamerlabs.com/docs)
