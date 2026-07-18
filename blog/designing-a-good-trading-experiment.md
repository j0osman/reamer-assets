---
title: Designing a Good Trading Experiment
description: A specific, falsifiable hypothesis can still be tested badly. Good experiment design means enough data to mean something, realistic costs from the start, no lookahead, a representative sample, and success criteria decided before the result exists.
date: 2026-07-16
---

A hypothesis can be specific, falsifiable, and precisely coded, and the experiment around it can still be designed badly enough to make the result meaningless. Specificity and good design are two different skills, and the second one is where a lot of otherwise-careful research quietly goes wrong.

## Enough data to mean something

A rule that only fires a handful of times over a short window hasn't been tested yet — it's been observed once or twice, with extra steps. Before trusting a result, it's worth asking plainly how many times the signal actually occurred and over what span. A pattern with five occurrences in three months and a pattern with five hundred occurrences over a decade are not carrying the same evidentiary weight, even if both produced an identical-looking headline number.

## [Realistic costs from the start, not bolted on afterward](https://reamerlabs.com/blog/why-execution-modeling-matters)

It's tempting to test an idea with everything set to zero — no commission, no slippage, no spread — "just to see if it works," and add realistic costs later as a check. This gets the order backwards. An idea that only looks good before costs isn't a slightly-worse version of the real idea; it's evidence against a different, easier hypothesis that was never actually the one worth testing. Setting commission, slippage, and spread deliberately before the first real run — not after a flattering zero-cost pass — is what makes the first result worth paying attention to at all.

## No lookahead

The signal at a given bar can only use information that would genuinely have been available at that bar — not the day's high before the day has finished, not a value that was only computable once later data arrived. This is one of the easiest ways an experiment lies to itself: the mistake is usually a single off-by-one in an indicator calculation, and the result is a strategy that looks unreasonably good because it's quietly allowed to see the future.

## A representative sample, not the flattering one

Choosing the exact market, instrument, or window that happens to show the pattern most clearly, after having looked at several, is re-fitting with extra steps — it produces [the same overconfidence as tuning parameters until the number improves](https://reamerlabs.com/blog/curve-fitting-vs-evidence), just applied to the choice of data instead of the choice of rule. A fair test uses a sample chosen for being representative, decided before looking at how the result would come out on each candidate.

## Decide what counts as success before you have the result

Once the experiment runs, the temptation to notice a plausible reason a middling result was still "good enough" is strong and very human. Deciding the acceptance bar in advance — [a Monte Carlo risk-of-ruin threshold](https://reamerlabs.com/blog/robustness-before-capital), a minimum number of trades, a drawdown limit — removes the option to move the goalposts after the fact, which is usually the actual moment an experiment stops being honest.

---

Full reference: [docs](https://reamerlabs.com/docs) · The execution model behind realistic costs: [execution specification](https://reamerlabs.com/execution-spec) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
