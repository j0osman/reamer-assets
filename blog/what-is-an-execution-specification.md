---
title: What Is an Execution Specification?
description: "The code does whatever the code does" isn't a definition of behavior — it's an admission that behavior hasn't actually been defined. A written execution specification is the difference between a claim you can check and a black box you have to trust.
date: 2026-07-17
---

Ask how a fill price gets calculated, and "however the code calculates it" is not an answer — it's a description of the absence of one. An execution specification exists specifically to close that gap: a written, authoritative definition of what the engine is supposed to do, checked against what it actually does, rather than the two being treated as automatically the same thing.

## Why "the code is the spec" isn't good enough

Code embodies behavior; it doesn't explain or justify it. A bug produces behavior too — indistinguishable, from the code's own perspective, from intended behavior, since the code is doing exactly what it was written to do either way. Without a separate, written definition of what's supposed to happen, there's no way to tell a bug from a feature except by convention or assumption. A specification exists to remove that ambiguity: it states, independently of the implementation, what a fill price should be, what a bracket collision should resolve to, what a rejected order should look like — and the implementation is then checked against that statement, not the other way around.

## The relationship between spec, tests, and code

The specification is the authoritative source. [Tests exist to verify that the implementation conforms to it](https://reamerlabs.com/blog/behavioral-testing-for-trading-engines) — not to define behavior themselves, and not to be redefined whenever a test happens to fail. When behavior needs to change, the specification changes first, deliberately, and the tests and any historical baselines get updated to match it in the same change — not quietly patched around a failing test without updating the document the test was supposed to be checking against. This ordering is what keeps the specification meaningful: if the document could be silently reinterpreted to match whatever the code currently does, it would stop being a specification and become a description after the fact.

## Why this matters beyond any one engine

A specification is what makes a claim about execution checkable by someone who didn't write the code. Anyone can read the document, understand what a fill is supposed to look like in a given scenario, and verify — independently, without needing to read or trust the implementation — whether a specific result conforms to what was promised. That's a fundamentally different kind of trust than "the developers say it works," and it's only available to a system that bothered to write down what "works" actually means before being asked.

---

Full reference: [docs](https://reamerlabs.com/docs) · Read the specification itself: [execution specification](https://github.com/reamerlabs/Reamer/blob/main/EXECUTION_SPEC.md) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
