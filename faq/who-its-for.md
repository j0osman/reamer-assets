---
title: Who are Reamer Research and Reamer Server for, and who are they not for?
description: They are for systematic quants who write their own code and trade mid-frequency strategies on bars, held from minutes to days, whether independent or employed at a firm. They are not for high-frequency or order-book (level 2 or level 3) strategies, options modelling, discretionary or point-and-click trading, or anyone who wants market data, a hosted platform or a ready-made broker connection included.
stage: 4
order: 26
product: both
next: supported-languages, supported-platforms, market-data, broker-connections, pricing
date: 2026-10-02
topics: systematic-trading, quantitative-analysis
---

They are for systematic quants who write their own code and trade mid-frequency strategies on bars, held from minutes to days, whether independent or employed at a firm. They are not for high-frequency or order-book strategies, options modelling, or traders who want to work without code.

## A good fit

- **You trade rules, not judgement calls,** and you write them in code: Python, C++ or another language that can call C.
- **Your strategies run on bars of any type,** intraday to multi-day: time, tick, volume, dollar, imbalance or your own custom rule. You build the bars yourself.
- **You want to check results, not trust them:** the same seed gives the same bytes, and the fill rules are written down.
- **You want everything on your own machine,** with nothing uploaded.
- **For [Reamer Server](/faq/what-is-reamer-server.html) in particular:** you are taking strategies live at your own broker, and you or your team can write the broker connection and the pre-trade check in C, C++, Rust or Go.

Independent quants and firms buy the same products at the same prices. A Reamer Research key bought online is personal to the one person whose email it was issued to at checkout, so each researcher on a team needs their own.

## Not a fit

- **High-frequency or order-book strategies.** Both products work on bars and order intents, not on level 2 or level 3 data, and are not built for the latency race.
- **Options.** There is no options pricing, greeks, exercise or expiry modelling.
- **Discretionary or point-and-click trading.** There is no charting interface or strategy builder; you write code.
- **You want data included.** No market data ships. You bring your own bars or pull them from your own Alpaca or Databento account with the bundled fetchers, and equities must already be adjusted for splits and dividends.
- **You want it hosted.** Both run on your own machines; there is no cloud version.
- **You want a broker connection out of the box.** Reamer Server connects to any broker you write a connector for. The kit has a worked FIX 4.4 example, not a certified connection.
- **Windows.** [Reamer Research](/faq/what-is-reamer-research.html) has no native Windows build but runs through WSL2; Reamer Server runs on Linux x86-64 only.

If you are unsure, a $20 month of Reamer Research, or the $2,000 30-day Reamer Server trial, is the way to check fit on your own strategy. See [pricing](/pricing.html).
