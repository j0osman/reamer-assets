---
title: Does my strategy code or data ever leave my machine?
description: No. Reamer Research and Reamer Server run entirely on your own machine, and neither sends strategy code, data, results or orders to Reamer Labs. Their only contact with Reamer Labs is activating or deactivating a licence, which sends the licence key and a one-way hash of the machine's identity. There is no telemetry, no usage analytics and no crash reporting.
stage: 5
order: 38
product: both
next: security-and-vendor-review, single-maintainer-risk, customer-reviews, local-python-backtesting-engine-no-upload
date: 2026-10-03
---

No. Reamer Research and Reamer Server run entirely on your own machine, and neither sends strategy code, data, results or orders to Reamer Labs. Their only contact with Reamer Labs is activating or deactivating a licence, which sends the licence key and a one-way hash of the machine's identity. There is no telemetry, no usage analytics and no crash reporting.

## What is sent, and when

- **On activation:** the licence key and a machine fingerprint, which is a SHA-256 hash of the machine's hostname and network hardware address. The licence server checks the key, binds it to that machine, and returns a signed licence.
- **On deactivation:** the same, to release the seat so you can move it to another machine.
- **At any other time:** nothing. Each later start checks the signed licence on your machine, with no network call, and a running process does not check in.

## What is never sent

- Strategy code, parameters or configuration
- Market data or any other input data
- Backtest results, orders, fills or positions
- Usage statistics, feature analytics or crash reports

Reamer Server's orders go to your broker through a connector you write, not through Reamer Labs.

## Checking it yourself

After activation, both products run with no internet access, so you can block outbound traffic and confirm they work. The Reamer Server deployment guide says that any traffic from a running process to the licence server is unexpected and should be reported. The kit also includes a vendor questionnaire pack with a data-handling statement for your firm's review.

## What Reamer Labs does hold

Your email address, the licensee's name, the licence key, the machine fingerprint and activation times. Payment details stay with Paddle, which handles checkout. See the [privacy policy](/privacy.html).

See also [Is there a backtesting engine I can call from Python that runs locally and never uploads my code?](/faq/local-python-backtesting-engine-no-upload.html)
