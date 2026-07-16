---
title: Knowing When You're Done
description: Validation isn't supposed to run forever. Once the pre-decided checks have actually been cleared, more testing against the same result stops adding information and starts risking the same over-fitting problem validation exists to catch.
date: 2026-07-17
---

Testing can continue indefinitely if nothing says when to stop. More Monte Carlo runs, another stress scenario, one more look at the trade log for something that might still be off — each additional check feels like due diligence, and none of them individually feels like enough of a reason to call it finished. At some point, though, continuing to test the same result stops adding real information and starts introducing a version of the exact problem validation was built to catch.

## More testing isn't automatically more rigor

Running a validation check that's already been cleared, again, a slightly different way, in the hope of feeling more certain, isn't the same as learning something new about the strategy. Past a certain point, repeatedly probing the same result for reassurance functions less like validation and more like searching for the framing that finally feels comfortable — the same instinct behind tuning a strategy until it looks good, just aimed at the confidence level instead of the equity curve.

## The bar was already decided

This is exactly why the acceptance criteria from earlier in this loop — the risk-of-ruin threshold, the stress-test tolerance, the parameter-count sanity check — need to be decided before any of them are checked. Once they're checked and cleared, the process is done, by the same standard that was agreed to before the result existed. Continuing to test past that point isn't extra caution. It's relitigating a decision that was already made honestly, looking for a reason to feel even more sure than the evidence actually supports.

## Being done is a decision too

Knowing when to stop isn't a lesser skill than knowing how to test thoroughly — it's the same discipline applied at the other end of the process. An idea that clears its pre-decided bar is done, ready for a decision, not for another round of checking in search of more comfort than the process was ever designed to provide.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
