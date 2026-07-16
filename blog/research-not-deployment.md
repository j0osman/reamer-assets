---
title: Research, Not Deployment
description: Reamer produces validated strategy logic and stops there. It does not place trades, connect to a broker, or run anything live — not as a missing feature, but as a permanent, intentional edge of the product.
date: 2026-07-16
---

Reamer does not place trades. It does not connect to a broker. It has no live risk management, no order management system, no way to run a strategy against a real account. This isn't a gap on a roadmap somewhere — it's a permanent, deliberate edge of the product, stated up front so it never has to be discovered by surprise halfway through an evaluation.

Reamer owns experimentation. Other systems own execution.

## What "research, not deployment" means in practice

Reamer's job ends at a specific, well-defined point: [validated strategy logic](https://reamerlabs.com/blog/from-idea-to-evidence) — an idea that's been implemented, run against realistic execution costs, inspected tick by tick, and stress-tested against the possibility that its apparent edge is noise. That's the finished product. What happens to it next — wiring it into a live account, a broker connection, an order management system, or a hosting platform that keeps something running unattended — is a separate decision, made with separate tools, entirely outside what Reamer does.

This isn't "research first, deployment eventually." It's research *only*. Reamer is upstream of the deployment decision, not a participant in it.

## Why this is a design decision, not a missing feature

[Closing the trust gap](https://reamerlabs.com/blog/why-reamer-exists) between a trading idea and a decision to risk capital on it is one job. Running a live account — uptime, latency, order routing, operational risk — is a different one entirely. A product trying to be excellent at both tends to be mediocre at each. Reamer commits fully to the first and leaves the second, deliberately, to the tools already built for it.

Concretely, that means these stay out, permanently:

- **Live execution and broker integrations** — definitionally past the boundary; there's no version of "a little bit of live trading" that doesn't turn Reamer into a different product.
- **An order management system** — that's live-trading infrastructure, not research infrastructure.
- **Live risk management** — a live-account concern, distinct from the research-time risk metrics (drawdown, margin, robustness) that are already core to what Reamer does and stay in.
- **Cloud deployment of a running strategy, or strategy hosting** — running something live on someone's behalf is deployment, however it's hosted.

None of that is a statement that these things don't matter — only that they're a different job, already done well by tools built specifically for it.

## What's still in scope, despite the boundary

"Research only" doesn't mean "solo, offline, one script at a time." Research collaboration — a team having shared visibility into verified results — stays in scope, as does running distributed experiments across machines or accelerating research compute in the cloud, as long as every individual run stays exactly as deterministic as it would be locally. The boundary is about live accounts and running infrastructure, not about scale or working with other people.

## What happens after Reamer's job is done

Validated strategy logic isn't locked into Reamer. Once an idea has survived the loop, deploying it is a decision made entirely outside this product — through a broker directly, through a platform built for that stage of the problem like QuantConnect or NautilusTrader, or through infrastructure a team has already built and trusts. None of those are competitors to Reamer; they operate downstream, at a stage Reamer was never trying to occupy in the first place. Showing that handoff exists, plainly, is part of how you can tell this boundary is a real design decision and not an excuse for a feature that isn't finished.

## Why the boundary is a feature

A tool that stops at validated strategy logic can afford to take that stage seriously — going all the way through cost realism, portfolio interaction, and robustness testing instead of stopping at a first promising number. Ideas that survive [the full loop](https://reamerlabs.com/blog/the-research-loop) come out the other end ready to be trusted with a deployment decision, not just interesting enough to consider one. That depth is easier to justify when the product isn't also trying to be the infrastructure the strategy eventually runs on.

---

Full reference: [docs](https://reamerlabs.com/docs) · The workflow this maps onto: [reamerlabs.com](https://reamerlabs.com/#workflow)
