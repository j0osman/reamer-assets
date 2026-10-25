# What software do independent quants use to go from research to live trading? | Bar by Bar #11

The tools differ from person to person. The layers don't. Almost every systematic setup ends up with the same five, because each does a job the others can't:

1. **Market data.** History for research and a live feed, with live bars built exactly like historical ones.
2. **A research engine.** One set of fill, cost and metric rules for every study.
3. **An execution pipeline.** Order state, one sequence for every strategy's orders, and a pre-trade risk check outside strategy code.
4. **A broker connection.** Sending orders, handling fills, rejections and reconnects. The broker is the one true record of your positions.
5. **Monitoring and records.** Alerts while it runs, and a durable log of every order, decision and fill.

Most unplanned work sits between layers 2 and 3. Four things rarely carry over as they are: indicators that read a lookback window become running values, list positions become broker symbols, simulated costs become real ones, and the data format changes. The decision logic itself should carry over unchanged. If it has to be rewritten, research and live are testing different things.

Whatever you adopt for the engines, some layers are always yours: the strategies, the risk rules, the choice of data and broker, and the glue between them. Those are your edge and your risk tolerance.

For bar strategies holding minutes to days, all five usually fit on one well-configured machine.

Each layer, and three common ways to assemble them: https://reamerlabs.com/faq/independent-quant-research-to-live-stack
