# How do I manage many research scripts without results drifting apart? | Bar by Bar #6

Separate what changes between studies from what must not. Each study holds only its strategy logic. Fills, costs, data handling and metrics live in one shared, versioned layer that every study calls.

Drift starts innocently. The second idea begins as a copy of the first script, the third as a copy of whichever was closest. A year later, each one fills orders, charges costs and computes returns slightly differently:

- One fills at the signal bar's close, another at the next open.
- A bug fixed in one copy lives on in every copy made before the fix.
- Sharpe is annualised by a different period count.
- A library update quietly changes an old study's numbers.

None of these is a big mistake. Together they mean every study was measured with a different ruler.

The fix:

1. **Write the shared rules down:** how a stop fills through a gap, which price a buy pays, how returns are computed.
2. **Version the shared layer,** and rerun the studies you rely on when it changes.
3. **Record four things with every result:** strategy code version, engine version, full configuration including seed, and a checksum of the data.

Then test it: rerun an old study. If the numbers match, your research history is sound. If they don't and nobody knows why, the drift has already happened.

The full approach: https://reamerlabs.com/faq/research-scripts-drifting-apart
