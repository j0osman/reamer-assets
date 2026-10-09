---
title: What's a good commercial research engine for mid-frequency strategies on OHLCV bars?
description: A good one gives byte-identical results for the same inputs and seed, publishes the rules it fills orders by, models bid, ask, spread and slippage, runs fast enough to sweep parameters, keeps your strategy on your own machine and has a path to live trading. Reamer Research is built for this case. It costs $1,800 per seat per year, with a $225 30-day trial.
stage: 3
order: 14
product: research
next: what-is-reamer-research, who-its-for, deterministic-backtesting-with-slippage-and-spread, stop-and-target-in-same-bar, backtest-futures-roll-and-fx-swap
date: 2026-10-01
---

A good one gives byte-identical results for the same inputs and seed, publishes the rules it fills orders by, and models bid, ask, spread and slippage the way your broker charges them. It also runs fast enough to sweep parameters, keeps your strategy on your own machine and has a path to live trading. Reamer Research is a commercial engine built for exactly this case: systematic strategies on OHLCV bars, held from minutes to days.

The list matters more than any one name. Below is what to check in any engine, the questions that separate a serious one from a script with a price on it, and how Reamer Research answers each.

## What mid-frequency strategies on bars need

Mid-frequency means positions held from minutes to days, decided on bars rather than on every quote. That puts the weight on a few things:

- **Repeatable results.** The same data, settings and seed should give the same output every run. Otherwise a change in results could be the strategy or the tool, and you cannot tell which. See [What makes a backtest deterministic, and why does it matter?](/faq/what-makes-a-backtest-deterministic.html)
- **Fills you can read.** The rules for when an order fills and at what price should be written down: market, limit and stop orders, stops and targets on the same position, gaps, and order lifetimes. A result is only as trustworthy as those rules.
- **Costs that match the broker.** Bid and ask rather than one price, spread and slippage that can vary, commission, overnight financing, margin. At mid-frequency, costs decide whether a marginal edge survives. See [How realistic do slippage and spread need to be in a backtest?](/faq/how-realistic-slippage-and-spread.html)
- **What happens inside a bar.** Stops, targets and limits often trigger within a bar, so the engine needs a stated way to decide what was reached first.
- **Portfolios.** Several instruments in one run, with shared equity and margin, and costs set for each one.
- **Speed for sweeps.** One backtest is never the test. Checking that a result holds across nearby parameters means hundreds or thousands of runs. See [How do I know if my backtest is overfit?](/faq/is-my-backtest-overfit.html)
- **The full record.** Every order, fill and closed trade, not just a final number, so any result can be traced.
- **A route to live.** The research has to lead somewhere. See [What software do independent quants use to go from research to live trading?](/faq/independent-quant-research-to-live-stack.html)

## Questions to ask any commercial engine

1. **Can I read the fill rules before trusting a result?** If not, every number it produces is a claim you cannot check.
2. **If I run the same test twice, do I get the same bytes?** Run it and compare the output files.
3. **Can I reproduce the published speed on my own hardware?** A benchmark you cannot rerun is marketing.
4. **What leaves my machine?** Strategy code and data are your edge. Know exactly what the tool sends, and where.
5. **Which languages can drive it?** It should fit the code you already write, not ask you to rewrite it.
6. **What is not included?** Data, live trading, options, order-book data, Windows. A vendor that states its limits saves you weeks.
7. **What does it cost each year, per person, and can I try it on my own strategy first?**

## How Reamer Research answers them

[Reamer Research](/products/reamer-research.html) is a research engine delivered as a library with a stable C interface. Your program, in Python, C++ or any language that can call C, hands it bars, settings and a strategy, and gets back the full result. Everything below is documented in the kit:

- **Byte-identical runs.** A fixed `rng_seed` gives byte-identical output, including the randomised spread and slippage. In the published test, 20 of 20 runs matched by SHA-256.
- **A written execution specification.** Fill prices, bid and ask, spread and slippage, commission, margin, order lifetimes (GTC, IOC, GTD), gaps, stop-and-target collisions, overnight swap and futures rolls are each specified, and the engine is tested against that document. It ships with the kit.
- **Inside the bar.** A deterministic synthetic tick path runs through every bar. Orders fill at the first tick that meets their condition, and the earlier of a stop and a target wins.
- **Portfolios with per-instrument costs.** Many instruments in one run with shared equity and margin. Commission, spread, slippage, swap and bid or ask pricing can be set for each one. Non-price data, such as earnings dates, can be attached to an instrument and is only visible from its own timestamp onward.
- **Fast enough to sweep.** On an AMD EPYC 9575F, one backtest ran at about 1.72 million bars a second, and its speed varied by only 0.59% (coefficient of variation) across 2,000 runs. A 50-million-bar test, 10,000 tickers over 20 years with 990,000 stop-and-target orders, ran in 29.1 seconds. Each backtest is single-threaded, and separate backtests are safe to run at the same time from your own threads, so a sweep uses every core. A benchmark tool in the kit reruns the measurement on your own machine.
- **Python, with a cost.** The Python reference uses ctypes and numpy and ships with 15 example strategies and scripts. It is far slower per bar than C++: on a 2017 laptop, an 884,130-bar backtest took about 65 seconds from Python.
- **Local.** After a one-time activation, it runs offline. Only activation and deactivation contact the licence server; strategies, data and results stay on your machine.
- **The full record.** 31 summary metrics, including Sharpe, Sortino, Calmar and drawdown, plus every order, fill and closed trade, written as a versioned JSON report.
- **Full state on every bar.** The strategy sees its position in each instrument and every resting order, and can re-price an order, move a position's stop to breakeven or into profit, close everything or cancel everything. Each action reports whether it was applied and why not.
- **A route to live.** `reamer_relay_*` sends the orders your strategy returns to a running [Reamer Server](/products/reamer-server.html), a separate product, and reads the fills back as position state. The decision logic carries over; indicators move from a bar window to running state.

The method and raw data behind the speed and repeatability figures are public: see [Reamer Research findings on EPYC](/blog/reamer-research-findings-epyc.html).

## Scope

- **Bars, minutes to days.** Level 2 and level 3 order-book strategies and high-frequency trading call for a different kind of engine.
- **Equities, futures and FX on bars.** Options pricing and modelling sit outside it.
- **Code first.** You write the strategy and the program that runs it.
- **Linux x86-64 and macOS on Apple Silicon.**
- **Your own data,** with equity data already adjusted for splits and dividends.

## Price

$1,800 per seat per year, paid once, with no automatic renewal. A 30-day trial costs $225, and the licence key and kit arrive by email within minutes. The trial is the way to run your own strategy on your own data before deciding. See [pricing](/pricing.html).
