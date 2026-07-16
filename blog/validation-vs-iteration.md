---
title: Validation vs Iteration
description: Iteration refines an idea against its own results. Validation tests whether the refined result would survive data it hasn't seen. Whether it's safe to iterate against a validation result depends entirely on what that check is actually testing against.
date: 2026-07-16
---

Iteration and validation can look similar from the outside — both involve running a strategy again and looking at the result — but they're answering different questions. Whether it's safe to blur them together depends on something specific: what the validation step is actually checking against.

## Two different questions

Iteration refines an idea against its own results: something looks off, a filter gets added, a threshold gets adjusted, and the strategy runs again against the same data to see if the change helped. [Validation asks something else entirely](https://reamerlabs.com/blog/validation-is-not-backtesting): would this result hold up against data it hasn't seen, or a spread of plausible variations of what actually happened? One tests whether a change helped on the data already in front of you. The other tests whether any of it means something beyond that specific data.

## Why repeated iteration alone isn't evidence

After several rounds of tweaking, a strategy usually does look better than it did at the start — and it's tempting to read that improvement itself as a form of validation, as though careful, repeated refinement were evidence on its own. It isn't, on its own. Every one of those rounds tested the same dataset again, just in a slightly different shape. Fitting that one dataset more closely doesn't tell you the underlying pattern is real — the only way to find that out is to actually check against something the iteration process wasn't shaped around.

## Iterating against a validation check: it depends what the check is testing

Using a validation result to guide further iteration is a completely standard approach in plenty of research styles — walk-forward optimization and tuning against a held-out validation set both work exactly this way, deliberately. The part that matters is whether the check is actually being asked something new each time, or just being asked the same question again in a different shape.

A genuinely held-out check — a walk-forward window the strategy hasn't been tuned against, a true out-of-sample split — can be iterated against safely, because each round is a real test against data the refinement process hasn't already shaped itself around. [Reamer's Monte Carlo is a different kind of check](https://reamerlabs.com/blog/why-monte-carlo-matters): it bootstrap-resamples the trade sequence from the run that already happened, not new out-of-sample data. Tuning a strategy and re-running Monte Carlo on the same underlying trades, repeatedly, doesn't introduce anything the strategy hasn't already seen — it's a different resampling of the same information, not a look at something new. That doesn't make Monte Carlo iteration meaningless, but it does mean repeated tuning against it doesn't earn the same trust that iterating against genuinely new data would.

## The practical distinction

Before treating a validation result as safe to iterate against, it's worth being specific about what it actually checked: new data the process hasn't touched, or a different view of the same data it has. The first can reasonably guide another round. The second is closer to iteration wearing validation's name, and deserves the same skepticism as any other result measured on data the strategy has already been shaped around.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
