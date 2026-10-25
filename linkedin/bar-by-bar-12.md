# What is an order management engine, and does a small trading operation need one? | Bar by Bar #12

It sits between your strategies and your broker. Every trading system already does its jobs somewhere, often scattered through strategy code. The question isn't whether to have order management, but whether to keep it scattered or put it in one place.

The five jobs:

1. **Track every order's state:** working, partly filled, pending cancel, filled. Once finished, nothing more applies, and nothing fills beyond its size.
2. **Put every order into one sequence,** so two strategies can't each find room under a limit they break together.
3. **Check each order before it leaves,** including cancels and changes.
4. **Handle what comes back.** Brokers resend fills after a reconnect; without a check on each fill's own ID, the position doubles on paper. While disconnected, reject new orders rather than queue them.
5. **Record every decision,** so a bad day has an answer.

You can manage without one when one strategy trades one account, the size is small enough that a mistake is a lesson, someone watches whenever it trades, and orders are simple.

You need one once any of these is true:

- More than one strategy trades the same account.
- Real money is at stake.
- The system runs unattended.
- You use limits, stops, cancels or changes to working orders.
- A partner, allocator or auditor needs to see what happened.

It isn't your strategy or your risk model. It's the place they meet the broker.

The full breakdown, including what an engine is not: https://reamerlabs.com/faq/what-is-an-order-management-engine
