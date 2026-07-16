---
title: Where Trading Ideas Come From
description: Trading ideas rarely start as hypotheses. They start as noticed regularities from a chart, a paper, someone else's writeup, or a side observation made while testing something else entirely — and none of that origin determines whether they're real.
date: 2026-07-16
---

Before a trading idea is a hypothesis, it's usually just a noticed regularity — something that seems to happen more often than chance would suggest, spotted somewhere ordinary. Where it came from has surprisingly little to do with whether it's worth testing, and even less to do with whether it's real.

## The usual sources

**A chart, looked at long enough.** A pattern that seems to repeat — a certain setup before a certain move — noticed just from spending time looking at price action, with no formal theory behind it yet.

**Published research or someone else's writeup.** A factor, an anomaly, a strategy shape described in a paper or a blog post, tested on someone else's data, in someone else's market, under someone else's cost assumptions.

**A side effect of testing something else.** Sometimes the most interesting idea shows up while you're implementing a completely different one — a filter that seems to matter more than expected, a subset of the data where everything behaves differently, a pattern in the trades that got rejected rather than filled.

**A cross-asset observation.** Something noticed in one instrument while actually researching a different one — two markets that seem to move together, or one that seems to lead the other by a bar or two.

**A discarded idea's near-miss.** An idea that failed validation but failed in an interesting, specific way — not randomly, but in a way that suggests a variation might behave differently. The original didn't survive, but the reason it didn't points somewhere.

## What all of these have in common

None of them are validated facts yet, regardless of how they arrived. A pattern from your own chart-watching and a factor from a peer-reviewed paper are in exactly the same epistemic position the moment you're considering testing them: both are claims that haven't been checked against realistic costs, your specific data, or the possibility that they're noise. A published result proves the idea worked somewhere, under some set of assumptions — not that it will work under yours.

This matters because "it's published" or "someone credible said so" quietly get treated as partial validation, when they aren't. They're a reason to consider testing an idea sooner rather than later. They're not a reason to skip the testing.

## Why the source doesn't change what happens next

Once an idea is going to be tested, Reamer doesn't distinguish where it came from. A hunch from a chart and a replication of a well-known published factor get implemented the same way — as ordinary Python in `on_bar` — and run against [the same execution realism](https://reamerlabs.com/blog/why-execution-modeling-matters), [the same robustness check](https://reamerlabs.com/blog/why-monte-carlo-matters). The trigger for reaching for that process is always earlier than "I've decided to formally test this": it's the moment a noticed regularity, from wherever it came from, turns into "wait, is that actually real?"

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
