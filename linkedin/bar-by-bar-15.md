# How do I run several strategies through one broker account? | Bar by Bar #15

Put one process between the strategies and the broker. Strategies decide; only that process talks to the broker.

Why one connection, even where the broker allows several:

- **One place to check risk.** Account-wide limits can only be enforced where every strategy's orders meet.
- **One order of events.** When two strategies hit the same instrument at once, one sequence decides which went first, and the record shows it.
- **One session to keep healthy.** Logon, heartbeats and reconnects are handled once, not once per strategy.

Running each strategy as its own process also contains failures. One that crashes takes only itself down.

What the hub has to get right:

1. **Order IDs that can't collide.** Two strategies that both count from 1 will both send order "1".
2. **Routing back.** Every fill, rejection and cancel reaches the strategy that owns the order, and only that one.
3. **Who may cancel what.** Enforce it in the hub. Don't rely on strategies behaving.
4. **Fair order.** Arrival order or priority. Under strict priority, a busy strategy can starve a quiet one.
5. **Disconnects.** Leave a dropped strategy's orders working or cancel them. Make it a decision, not an accident.

The part most setups miss: the account is shared, but positions are not. If one strategy is long 100 and another is short 60, the broker shows long 40, and neither strategy's view matches it. Each strategy keeps its own position from its own fills. Account limits use the broker's net figure. Per-strategy limits need per-strategy positions, which the broker will never give you.

Reconcile regularly. If the strategies' positions don't add up to the broker's, a fill was missed or misrouted.

The full shape, and how many strategies one machine can take: https://reamerlabs.com/faq/multiple-strategies-one-broker-connection
