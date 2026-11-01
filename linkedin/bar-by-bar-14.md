# How do I backtest with earnings dates or other non-price data without lookahead? | Bar by Bar #14

Stamp every record with the moment you could have had it, not the date it describes. Then join it to your bars "as of": each bar sees the latest record stamped at or before it, never a later one.

Most lookahead comes from the timestamps, not the join. Every record can carry three times:

1. **The period it describes.** "Q2 earnings."
2. **The event time.** When the company reported.
3. **When you could have had it.** When it reached you in a form your strategy could read.

Only the third is safe. Where the other two creep in:

- **Date-only stamps.** An after-close earnings release stamped with its date reads as midnight, hours before it existed.
- **Time zones.** 16:05 in New York is 20:05 or 21:05 UTC, depending on daylight saving.
- **Revisions.** A dataset that keeps only the latest value gives the backtest numbers nobody had at the time.
- **Reporting lag.** Fundamentals filed weeks later, stored against the quarter's end.
- **Changing coverage.** A vendor that only covers companies that still exist hides the failures.

Three rules for the join:

- Never join on "nearest". It will take a record from just after the bar.
- Carry values forward, never back. Before the first record, the value is missing.
- Know whether your bars carry their opening or closing time, because it decides which bar a record first appears on.

One test catches most of it: delay every record by a bar and rerun. Some loss is normal. If the whole edge disappears, check whether it depended on seeing data before you really could.

An engine can stop a record being read before its timestamp. Nothing can fix a wrong timestamp.

The full checklist, with the pandas join: https://reamerlabs.com/faq/backtest-with-exogenous-data
