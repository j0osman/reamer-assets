---
title: How do I manage many research scripts without results drifting apart?
description: Separate what changes between studies from what must not. Each study should contain only its strategy logic, while fills, costs, data handling and metrics live in one shared, versioned engine that every study calls. Record the engine version, configuration, seed and data with every result, so any two results can be compared like for like.
stage: 1
order: 5
product: research
next: research-engine-vs-backtest-script, what-is-institutional-grade-trading-infrastructure, backtest-results-change-between-runs, what-makes-a-backtest-deterministic
date: 2026-09-29
---

Research usually starts with one script. It works, so the next idea starts as a copy of it, and the one after that as a copy of whichever version was closest. A year later there are dozens of scripts, each with its own small differences in how it fills orders, charges costs, cleans data and computes returns. Results from different studies can no longer be compared, because each one was measured with a slightly different ruler.

## How the drift happens

- **Fills defined differently.** One script fills at the signal bar's close, another at the next bar's open. One charges commission per side, another per round trip. Both look reasonable, and they give different answers for the same strategy.
- **A fix that reached only one copy.** A bug found and fixed in one script lives on in every script copied before the fix.
- **Data prepared differently.** Different adjustments, different handling of missing bars, different time zones, different date ranges.
- **Metrics computed differently.** Sharpe annualised by a different period count, returns measured on the position in one script and on the whole account in another. The same trades produce different headline numbers.
- **Environments that moved.** A library updated between two studies changes the older one's results the next time it runs, with no record of why.
- **No record of what produced a result.** A number in a notebook or spreadsheet, with no note of which script version, settings or data made it, cannot be reproduced or checked.

None of these is a big mistake. Each is a small, sensible local choice. Together they make a research history that cannot be trusted as a whole.

## The fix: one engine, many strategies

Split the work into two layers.

1. **What changes between studies:** the strategy logic. Signals, entries, exits, sizing. Each study contains only this.
2. **What must not change between studies:** how orders fill, how costs are charged, how data is loaded and aligned, how results are measured. This lives in one place, and every study calls it.

Then treat the shared layer with care:

- **Write its rules down.** A short document that says exactly how a stop fills through a gap, which price a buy pays, and how returns are computed turns "what the code happens to do" into something you can check and rely on. See [What Is an Execution Specification?](/notes/what-is-an-execution-specification.html)
- **Version it.** When the shared layer changes, the version changes, and every result records which version produced it.
- **Rerun what matters when it changes.** If a fix changes fills, rerun the studies you still rely on, so the whole history is measured with the same ruler.

## Record enough to reproduce every result

Store these with every result, alongside the numbers themselves:

1. The strategy's code version.
2. The engine or shared layer's version.
3. The full configuration, including costs and the random seed.
4. A checksum of the data used.

With those four, any result can be rerun and checked, and two results can be confirmed to be like for like before they are compared. Without them, a surprising result cannot be told apart from a changed setting.

## A test of whether it is working

Pick an old study and rerun it. If the numbers match what was recorded, the research history is sound. If they do not, and nobody can say why, the drift has already happened. Find the first difference before trusting any comparison across studies. See [Why do I get different backtest results each time I run it?](/faq/backtest-results-change-between-runs.html)

## Where Reamer Research came from

Reamer Labs began with exactly this problem. Its founder's research covered equity portfolios, sector ETF portfolios, currency pairs and crypto perpetuals, and every study was its own Python script in its own sandbox. Small differences between the scripts made results drift from one study to the next, and the research loops were too slow for strategies at any real scale. [Reamer Research](/products/reamer-research.html) is the result: one engine, built to one set of written execution rules, for every study. The story is told on the [About](/about.html) page.

## How Reamer Research keeps studies comparable

Every rule below is documented in the kit:

- **One engine, one written specification.** Every study runs through the same engine, and every fill, cost and accounting rule is written down in `EXECUTION_SPEC.md`. A study contains only its strategy, as a callback the engine calls on each bar, written in Python or in any language that can call a C interface. The kit includes worked Python examples, from a quickstart to multi-asset and risk-sized strategies.
- **One configuration format.** Costs, leverage, the type of price data and the seed are set in the same documented fields for every study, with per-instrument overrides.
- **One result format.** Each run can write a single JSON document with a version number, holding the summary metrics, every closed trade, fill and order, and the per-trade return series. `RESULT_JSON_SCHEMA.md` defines each field, including exactly how Sharpe, Sortino and Calmar are annualised, so a metric means the same thing in every study.
- **Identical output for identical inputs.** With a fixed seed, a run is byte-identical on every repeat, so rerunning an old study is a real check.
- **A stable interface.** From ABI 7 on, interface versions are additive: a program built against ABI 7 runs unmodified against a newer library, so upgrading does not mean rewriting every study. `ABI_VERSIONING_POLICY.md` in the kit states the rules.

Keeping your strategy code under version control, fixing a copy of your data and recording each run's configuration stay with you; the engine does not store your research history. What it removes is the largest source of drift: a different set of fill, cost and metric rules in every script.
