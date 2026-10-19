# Where should pre-trade risk checks sit in a trading system? | Bar by Bar #9

Between the strategies and the broker, as one separate step that every order passes through. Orders from all strategies are put into one sequence first. Then the check makes one decision per order, pass or reject, before anything is sent.

The position matters as much as the rules:

- **After sequencing,** so the check sees the account as it is, including the order just before this one. Check two strategies at the same moment against the same limit, and each can find room while together they go over it.
- **Before the broker,** because a sent order can't be unsent. A check after the fact is monitoring, not prevention.

Why not inside the strategy? The code that makes the mistake is the code meant to catch it. A strategy sees its own position, not the account's. And tightening a limit shouldn't mean redeploying trading logic.

Why not leave it to the broker? Broker checks protect the broker, can't know your strategy's limits, and act after the order has left your system.

How a good check behaves:

- **Every order, including cancels and changes.** A rule that rejects any order with a quantity of zero may reject every cancel.
- **Fails closed.** If it can't decide, the order is rejected.
- **No orders while disconnected.** Reject at once rather than queue and send later.
- **Every rejection recorded with its reason,** so it can be counted and explained.

What the check needs to see, and the limits most setups start with: https://reamerlabs.com/faq/where-pre-trade-risk-checks-belong
