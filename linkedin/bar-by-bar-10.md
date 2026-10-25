# What is a quant research engine, and how is it different from a backtesting script? | Bar by Bar #10

A backtesting script does everything for one study: loads the data, loops over the bars, works out the fills, adds up the profit. A research engine takes everything in that script except the strategy and puts it in one place that every study calls.

The difference is less about features than about where the rules live. In a script, they live inside each study. In an engine, they live once, outside all of them.

That's what lets one engine support the whole cycle, not just one run:

1. **Test** over years of bars with realistic costs.
2. **Diagnose** a result by tracing it back to every order and fill.
3. **Sweep** parameters, knowing the runs differ only in what you changed.
4. **Report** in one fixed format, with metric formulas written down.
5. **Hand off to live** as a port of known size, not a second project.

A script is still the right tool when the idea is rough and may be dropped tomorrow, when the strategy needs something an engine doesn't model, or when you're learning how a backtester works.

Signs you've outgrown scripts:

- Two similar studies give results you can't compare.
- A bug fix in one script hasn't reached the others.
- An old result no longer reproduces.
- Sweeps are slow enough that you run fewer than you should.
- Going live means writing the strategy again.

The five steps, and what sits underneath them: https://reamerlabs.com/faq/research-engine-vs-backtest-script
