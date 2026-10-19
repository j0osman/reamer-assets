# Can one laptop run serious quant research and live trading? | Bar by Bar #7

For strategies that trade on bars, yes. Speed is rarely the limit. What decides it is how the machine's cores are shared, and whether it stays on and connected while the market is open.

The two jobs want different things:

- **Research wants throughput.** Every core it can get, and nobody is hurt if it slows down.
- **Live trading wants steady latency.** Few cores, and it must never wait for one.

The mistake is letting them compete. In one published benchmark on a 2017 four-core laptop, four live strategies, one per core, saw the middle order take 56 microseconds and the slowest 1% finish within 139. At eight strategies on the same four cores, the middle order barely moved, but the slowest 1% rose to 4.2 milliseconds.

Where a laptop stops being enough:

- **High-frequency trading,** which competes on network distance to the exchange.
- **More busy live strategies than physical cores.**
- **Trading that must not stop.** Laptops sleep, update themselves and change networks.

Sharing one machine:

1. Pin each live strategy to its own core and leave those cores alone.
2. Run research on the rest, or outside market hours.
3. Turn off sleep and automatic updates while trading.
4. Plan for a restart: rebuild positions and open orders from the broker's records, not the program's memory.

Where the limits sit, and how to measure your own machine: https://reamerlabs.com/faq/can-a-laptop-run-a-trading-desk
