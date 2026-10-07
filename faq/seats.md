---
title: What counts as a seat? Can I move it to another machine?
description: A seat is one product on one machine at a time, with unlimited runs and processes on that machine. To move it, deactivate on the old machine and activate on the new one. If the old machine is lost, support releases the seat by hand. A Reamer Research seat is also personal to one person.
stage: 6
order: 45
product: both
next: licence-expiry, pricing, what-you-receive, support, invoice-and-order-form
date: 2026-10-03
---

A seat is one product on one machine at a time, with unlimited runs and processes on that machine. To move it, deactivate on the old machine and activate on the new one. If the old machine is lost, support releases the seat by hand. A Reamer Research seat is also personal to one person.

## What one seat covers

- **One machine.** Running on two machines at once needs two seats. There is no floating or pooled licence.
- **One product.** Reamer Research and Reamer Server are licensed separately, and a key for one does not activate the other. A machine running both holds one seat of each.
- **Anything on that machine.** Any number of backtests, Reamer Server instances and strategy processes run under the one seat. Reamer Server is designed to run one server and a strategy process per core, all on one seat.

## Moving a seat

1. On the old machine, run `reamer-license deactivate`. This releases the seat.
2. On the new machine, run `reamer-license activate` with the same key.

That is the only way to move a seat, and you can do it yourself without contacting anyone. Moving does not restart the term; it still ends a fixed time after the key was first activated. If the old machine has died or been wiped and you cannot deactivate it, write to support@reamerlabs.com and the seat is released for you.

## What counts as "the machine"

The machine is identified by a hash of its hostname and network hardware address. Changing either, by renaming the host or reordering network interfaces, makes it look like a new machine. Deactivate before the change and activate after it. In containers, give the workload a stable hostname and keep the activation file on persistent storage, or every restart looks like a new machine. Each kit's `LICENSING.md` gives a working Kubernetes setup.

## Reamer Research: one person

A Reamer Research seat bought online belongs to the person whose email it was issued to at checkout. Only that person may use it, at home or at work. An employer can pay for it, but it stays personal to that employee. An employer can ask, once per term, for it to be reissued to a different employee. Each extra person needs their own seat. Reamer Server seats are not tied to a person.

See [pricing](/pricing.html) and [How much do Reamer Research and Reamer Server cost?](/faq/pricing.html)
