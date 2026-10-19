# What does professional-grade trading infrastructure actually include? | Bar by Bar #8

Less hardware than people expect and more discipline. For strategies that trade on bars, what separates professional infrastructure from a folder of scripts is a set of properties, and each one can be checked.

In research:

- **Same inputs, same result.** Otherwise no comparison between two runs can be trusted.
- **Fill and cost rules written down,** so results can be checked against them.
- **One engine for every study,** so every result is measured with the same ruler.

In live trading:

- **Risk checks outside strategy code,** so a bug in a strategy can't switch off the limits.
- **One ordered path for every order,** so two strategies can't each find room under a limit they break together.
- **A record of every decision:** what was asked for, whether it was accepted, why, and what the broker did.
- **Monitoring while it runs,** with alerts when something stops.
- **Safe restarts** that take positions from the broker, the one true record, not from a saved copy.

In the software:

- **Performance claims with their evidence:** the hardware, the method and the raw output.
- **Releases you can verify before running,** and interfaces that don't break on upgrade.

None of it needs a large team, a data centre or microsecond networking. These are design choices. Once built in, they don't need people to maintain them day to day.

The full list, with how to check each property: https://reamerlabs.com/faq/what-is-institutional-grade-trading-infrastructure
