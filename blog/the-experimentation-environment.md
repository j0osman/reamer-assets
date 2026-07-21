---
title: Quantitative Research Infrastructure, Not a Platform
description: Why "quantitative research infrastructure" is the right category for Reamer, why "platform" specifically is the wrong word despite sounding close, and why the obvious alternatives — backtesting framework, research environment — each undersell what the product actually is.
date: 2026-07-16
---

Reamer is quantitative research infrastructure for systematic trading desks. That's a specific, deliberate choice of words, not a stylistic preference — the more obvious labels were each considered and rejected for concrete reasons, and "infrastructure" itself was chosen over a word it's easy to confuse it with.

## Not "backtesting framework"

Backtesting is real and central to what Reamer does — a strategy meets historical data, orders get filled, a result comes out. But that's one stage of a longer process, not the whole of it. Calling the whole product a "backtesting framework" names the middle step and leaves out everything on either side of it: forming the idea in the first place, inspecting exactly what happened after the run, deciding whether the result would survive data it hasn't seen, and carrying whatever was learned into the next idea.

A single backtest, in isolation, proves very little — a strategy can look excellent on one run and be indistinguishable from noise. Naming the category after that one stage makes it sound like the point is the run itself. It isn't. The point is what happens before and after it.

## Not "research environment," on its own

"Research environment" is closer, but it describes a place, not a discipline. A folder of notebooks, a spreadsheet, and a shared drive is also, technically, a research environment — it doesn't commit to anything about how rigorous the research inside it has to be. It doesn't distinguish evidence-based testing, where a result has to survive [realistic costs](https://reamerlabs.com/blog/why-execution-modeling-matters) and [a robustness check](https://reamerlabs.com/blog/why-monte-carlo-matters) before it's trusted, from open-ended tinkering, where a good-looking number is treated as the end of the story.

The word that was missing is the discipline itself, not just the setting it happens in.

## Not "platform" — even though "infrastructure" sounds close to it

This is the distinction most worth getting right, because the two words get used interchangeably and they shouldn't be. "Platform" — and "operating platform" specifically — implies hosting, orchestration, running things on someone's behalf, multi-tenant scale. Reamer runs locally, on one machine, with no server in the middle a customer depends on Reamer Labs to keep running. Calling it a platform would promise infrastructure ambitions it doesn't have and isn't building toward.

"Infrastructure" doesn't carry that implication on its own. A local database is infrastructure. An embedded toolchain is infrastructure. Neither is hosted, neither is multi-tenant, and calling either one "infrastructure" isn't a stretch — it's just accurate: something foundational and load-bearing that other work depends on. Reamer is infrastructure in exactly that sense — the thing a desk's research process runs on — without being a platform in the hosted, orchestrated sense at all. The two words sound like they're on the same spectrum. They aren't.

## What "quantitative research infrastructure" actually names

The category needed to name [the loop itself — idea, implementation, execution, inspection, validation, iteration](https://reamerlabs.com/blog/from-idea-to-evidence) — rather than any single stage of it, and it needed to name what that loop actually becomes once it's something a desk relies on, not just an individual researcher's side tool: something the research process is built on top of. "Quantitative research infrastructure" does both — it names the discipline (quantitative research) and the role the product plays in it (infrastructure, not a single pipeline stage wearing the whole product's name, and not a hosted platform it was never going to be).

"Experimentation environment" was the earlier answer to this same question, and its reasoning about the loop still holds — it's the "platform" adjacent confusion above that made the earlier phrasing worth revisiting, not a change in what the product actually does.

## Why this is worth being precise about

A category name is a promise about what to expect. Call something a backtesting framework and someone reasonably expects a tool for running backtests, not a way to reason about whether an idea deserves capital behind it. Call it a platform and someone reasonably expects hosting and multi-tenant infrastructure that isn't there. "Quantitative research infrastructure" is the name that's supposed to still describe Reamer accurately a decade from now, because it describes both the loop and the role, and neither is the part of this product likely to change.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model behind the "test" part of the loop: [execution specification](https://reamerlabs.com/spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
