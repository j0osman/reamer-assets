---
title: "The .reamer File: Making a Backtest Result Portable"
description: A result someone else can't independently reproduce isn't evidence — it's a claim asking to be trusted. A self-contained result file, carrying its own config, seed, and fills, is what turns "trust me" into "check it yourself."
date: 2026-07-19
tier: Engineering
---

A backtest result that exists only as numbers pasted into a message, or a plot with no underlying data, asks the person looking at it to simply believe it. That's true even if the result is completely honest — the format itself doesn't carry anything that would let someone else independently confirm it. A `.reamer` file exists specifically to close that gap: it's a single, self-contained artifact that carries everything needed to verify a result, not just the result itself.

## What "self-contained" actually has to include

Reproducing a claim means being able to run the exact same thing again and get the exact same answer — which means the file has to carry the execution config that produced it, the random seed, the data paths the run used, and the full record of what happened: every closed trade, every order, every cost applied. Leave any one of those out and the file stops being self-contained — it becomes a result plus an implicit assumption that whoever opens it already has the missing piece, which defeats the actual point of writing the file down in the first place.

## Why this matters more than it looks like it should

The value of a `.reamer` file isn't that it's convenient to open later — plenty of formats are convenient. It's that a colleague, a reviewer, or your own future self months later can regenerate the exact result from the file alone, without having to trust that the original run was described accurately. That property is what separates a reproducible claim from a well-described one. A well-described result can still be wrong in a way nobody catches, because there's nothing to actually check it against. A reproducible one either holds up when regenerated, or it doesn't — and that's a real answer, not a matter of how convincing the description sounds.

## Portable, not just personal

Because the file is self-contained, it's also portable in the ordinary sense — it can move to a different machine, sit unopened for a year, or be handed to someone who had no part in producing it, and still mean exactly what it meant the day it was written. That's a direct consequence of not depending on anything outside itself: no external state to have drifted, no separate config file that might not be the one that was actually used, no reliance on remembering which run this even was.

---

Full reference: [docs](https://reamerlabs.com/docs) · Reading results: [reamerlabs.com/docs#results](https://reamerlabs.com/docs#results)
