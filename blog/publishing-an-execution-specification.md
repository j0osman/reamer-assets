---
title: "Publishing an Execution Specification"
description: "\"The code does whatever the code does\" is not a definition of behavior — it's an admission that behavior was never actually decided. Writing the execution model down as a testable specification, before writing the tests, is a different discipline than testing code after the fact."
date: 2026-07-19
tier: Engineering
---

Most software gets tested against its own implementation, implicitly: a test asserts the function returns what the function currently returns, and a change to the function quietly becomes the new correct answer unless someone happens to notice. That works for a lot of software. It doesn't work for an execution engine, where the entire product being sold is a claim about what a fill price *should* be — a claim that has to exist independently of whatever the code happens to compute, or it isn't a claim at all.

## Writing the rule down before writing the test

Reamer's execution behavior — fill price for every order type, how slippage and spread apply, which side of a bracket collision wins, how commission and swap accrue — is written down first, in prose and formulas, as [a standalone specification](https://reamerlabs.com/spec), independent of the C++ that implements it. That ordering matters: a specification written after the code tends to describe what the code does, bugs included. A specification written first, and tested against, describes what the code is supposed to do — and a mismatch between the two is a bug in the code, not a footnote in the docs.

## What a specification has to commit to, to be worth anything

A specification that hedges — "fills are generally realistic," "costs are modeled appropriately" — commits to nothing a test could ever fail. Reamer's spec commits to specific formulas and specific tie-breaking rules: exact fill-price expressions per order type, an explicit answer for which side of a same-bar TP/SL collision fires first, an explicit tolerance for cost-accounting identities. Every one of those commitments is written so that a test can either pass or fail against it — there's no room for "close enough" once a number is written down.

## The discipline this creates

Publishing the spec publicly changes the cost of getting it wrong later: a change to the execution model that isn't reflected in the published spec is now a visible inconsistency, not a quiet implementation detail. That's a real constraint on how the engine evolves — a fill-price formula can't change without the spec changing with it, and a spec change is a conscious decision, not a side effect of refactoring something else. The discipline isn't the document. It's what having a public, specific document forces every change after it to reckon with.

---

Full reference: [docs](https://reamerlabs.com/docs) · The document itself: [execution specification](https://reamerlabs.com/spec)
