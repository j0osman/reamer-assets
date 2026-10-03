---
title: What happens when my licence expires?
description: Anything already running keeps running. Expiry never stops a live process, including a Reamer Server instance that is trading. What it stops is starting a new process, so after expiry a restart will fail until you renew. Nothing renews or charges automatically; renewing is a new purchase.
stage: 6
order: 46
product: both
next: refunds, support, what-you-receive, seats, pricing
date: 2026-10-03
---

Anything already running keeps running. Expiry never stops a live process, including a Reamer Server instance that is trading. What it stops is starting a new process, so after expiry a restart will fail until you renew. Nothing renews or charges automatically; renewing is a new purchase.

## When the term ends

The term is 30 days or one year from the key's first activation, not from the date you bought it. Moving the seat to another machine does not restart it. `reamer-license status` shows how many days are left.

The end date is part of the signed licence and is checked on your machine, so it is enforced without a network connection. Changing the system clock or editing the activation file does not extend it.

## What happens at expiry

- **Running processes carry on.** A backtest finishes, and a Reamer Server instance keeps trading until it exits on its own.
- **New starts are refused.** A new backtest, a restarted server or a reboot needs a current licence, and there is no grace period.
- **Your files are untouched.** Results, configuration and code on your machine stay as they are.

## The warning

Reamer Server raises a `license.expiring` warning once at each of 30, 14, 7 and 1 days before the end date, which you can change. After expiry it raises a `license.expired` error every hour. That error means the process is still trading but will not come back if it stops. The Reamer Server monitoring guide suggests paging on it and pausing restarts and redeploys until the seat is renewed.

## Renewing

- **Nothing renews automatically.** Nothing is charged at the end of a term.
- **Renewal is a new purchase,** at the [pricing page](/pricing.html), by card or by invoice.
- **Buy it before the term ends** so a restart never finds the licence expired. The new key's term starts when you first activate it, so buying early does not cost you days.
- **A renewal ships the latest release.**

To switch a machine to the new key, deactivate the old key first, then activate the new one:

1. `reamer-license deactivate`, adding `--product research` or `--product server` if the machine holds both
2. `reamer-license activate` with the new key

The new key's term starts the moment you activate it, so switching early overlaps the two terms; make the switch close to the end date. A process that is already running is not stopped by the switch. A running Reamer Server may raise one `license.expired` error in the seconds between the two commands; that is expected, and it stops once the new key is active.

See [What counts as a seat?](/faq/seats.html) and [How much do Reamer Research and Reamer Server cost?](/faq/pricing.html)
