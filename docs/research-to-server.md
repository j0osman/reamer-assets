---
title: From Reamer Research to Reamer Server
description: What it takes to move a strategy from Reamer Research to live execution on Reamer Server, including the reamer_relay_* strategy relay and its limits.
group: integration
order: 5
product: both
source: RESEARCH_TO_SERVER.md
---

**This document ships in both kits and is written for either reader.**
Whichever archive you found it in, you are holding one half of the move —
the other half lives in the other product's kit. Where a file below is
only in one kit, that is stated on the line.

## The handoff

Reamer Research validates your strategy against historical OHLCV bars and
measures backtest performance. Reamer Server executes the same trading
logic live against a real venue.

**The server hosts no strategy, in any language.** It links a static C
archive into a `main()` that you write; strategies connect to it
out-of-process, over a wire protocol (the strategy relay — see
EXTENSION_PROTOCOL.md in the server kit). A pure-Python strategy can submit
a live order using nothing but the standard library `socket` and `json`
modules — the same relay a Go or C++ strategy uses. There is no
per-language cost difference in reaching the server; the work below is
identical regardless of what your research strategy is written in.

**The strategy relay ships in `libreamer_research`.** From ABI 7 (release
4.5.0), `reamer_relay_open()`, `reamer_relay_send()` and
`reamer_relay_poll()` send the same `ReamerOrderRequest` your `on_bar()`
returns to a running Reamer Server over its local-mode strategy socket, and
`reamer_relay_get_positions()` reads the fills back as
`ReamerPositionState`. See "Sending research orders live" below. The gate
and the broker connector are still yours to build.

**What the move actually costs**, in order of how much it touches your
strategy code:

**1. One-time deployment infra: gate and broker connector.**
Built once per deployment, not once per strategy. The strategy relay is
the exception: `libreamer_research` ships one for the local-mode socket,
and you write your own only for remote mode or a strategy outside the
research ABI. The gate and broker
connector link the server's C archive directly, so that layer is
language-locked to whatever links a C ABI. EXTENSION_PROTOCOL.md in the
server kit is the complete integration contract, and you write the
production linking code yourself — but you do not start from a blank file.
`reference/` in the server kit is a worked integration of this entire
layer in Go: a FIX 4.4 session, a broker connector, a gate, a strategy
relay, and an event-bus reader, with `demo-trade.sh` driving one order
from strategy to fill. Read it and adapt it. It is reference material
under the kit licence, not a supported surface and not an adapter for
your venue. The strategy relay, by contrast, is just a client of the wire
protocol and can be written in anything, including plain Python.

**2. Instrument-identity mapping.** Research `ReamerOrderRequest` keys on
`int ticker_id` — an index into the array you passed to
`reamer_run_backtest()`. Reamer Server identifies instruments by
**string name** (`intent.Instrument`, e.g. `"AAPL"`). `reamer_relay_open()`
takes a `ticker_names[]` array and sends `ticker_names[ticker_id]` as the
instrument, so pass it the same array you passed to `reamer_run_backtest()`.
Those names must also be the names your gate and broker connector know.
Keep the array in one place; an index that silently means a different
symbol than it did in the backtest is the worst class of bug this move can
produce.

That mapping is at least **checkable**. `reamer_run_backtest()` requires a
`ticker_names[]` array, and every instrument name in the result —
`ReamerClosedTrade.ticker`, and every `ticker` field in the result JSON — is
your own name from that array. So the backtest output carries the same identifiers your
Server-side `intent.Instrument` uses, and you can assert the two agree
instead of trusting an index. Diff the result's ticker set against your
instrument table as a build step; that turns the worst class of bug above
into a failed check.

**3. Running indicator state.** The one item that genuinely touches your
strategy's own code. In research, `on_bar` receives the full lookback
window on every call. Live, nothing hands you a window — a gate sees one
intent and the current venue state. Any indicator needing history (an ATR,
a moving average) must be restructured to be maintained as running state
by whichever of your processes computes it. This is a batch-to-event-driven
port, not a language rewrite, and it is the same shape of work whether your
research strategy was Python or C++.

**4. No configuration or data artifact carries across.** Not the config
file, not the data format, not the execution-model parameters. The
research execution model (fees, slippage, spread) is a *backtest*
simulation of costs; live, those costs are whatever your venue actually
charges. Do not port `engine_config.json` expecting it to mean anything to
the server.

None of the above is hard, and none of it is per-language — but all of it
is silent if you do not know about it in advance. Budget it as the
architecture work every backtest-to-live transition requires, not as a
tax on whichever language your research strategy happens to be written in.

## What changes: the callback model

Research and Server use different extension points, since research replays
bars and Server processes live intents:

| Surface | Reamer Research | Reamer Server |
|---|---|---|
| **Strategy callback** | `ReamerResearchStrategyVtable.on_bar()`, called once per bar with a borrowed window of OHLCV data | No single per-bar callback — strategies are out-of-process, connected over a wire protocol |
| **Gate decision** | Strategy writes orders directly into the caller-provided buffer; success is implicit | `ReamerGateVtable.check()` evaluates each intent before submission; accepts/rejects by writing an `Ack` |
| **Venue state** | Passed as a `ReamerOhlcvBar[]` window; position tracking is left to your strategy (or read from the `positions` array `on_bar` receives) | `ReamerVenueState` from your broker connector's `pull_state()`: net positions per instrument, open-order count, connected flag |
| **Order submission** | Orders are returned to the backtest engine; `reamer_relay_send()` sends the same orders to a running server | `ReamerBrokerConnectorVtable.submit_order()` sends your intent to the real venue; fills come back via `poll_updates()` |
| **Instrument identity** | `int ticker_id`, an index you supplied — but results carry your own string names when the run went through `reamer_run_backtest()` | `string` instrument name |

Your strategy's **decision logic** — entry signals, exit rules, position
sizing, stop/target levels, risk thresholds — does not change. What changes
is the plumbing: a research `on_bar` that evaluates "buy 100 shares if
ATR-bracket conditions are met" becomes a `reamer_relay_send()` of the same
order, plus a gate (your own risk
gate) that accepts or rejects it based on venue state. The *signal and
bracket arithmetic itself* stays the same — it now runs in your strategy
process, from running state instead of a bar window passed to `on_bar`.
The gate does not compute it; the gate only decides, from
`ReamerVenueState`, whether the resulting intent may reach the venue.

(See "running indicator state" above for what this means for anything
needing history, like an ATR or a moving average.)

## What stays the same

The core trading ideas are venue-independent and callback-independent.
Reamer Research ships an ATR breakout-bracket template — entry on ATR
breakout, tight stop if wrong, wide stop if right — under its
reference-python templates directory. (That file is in the research kit
only; if you are reading this from the server kit, you will not find it
here.) It expresses the same risk rules a Server gate expresses when it
checks position limits before accepting an order. Both constrain what orders
reach the venue; one runs per-bar in a backtest, one runs pre-trade live.
Neither changes when you move between products.

Port your trading logic directly: the ATR calculation is the same, the
entry/exit decision is the same, the position-sizing formula is the same.
Only the interface shifts — from returning orders in a per-bar callback to
sending them with `reamer_relay_send()` through a gate you implement.

## Sending research orders live

`reamer_relay_*` in `include/reamer_research_abi.h` (research kit) is the
contract. In brief:

- **Order IDs:** the n-th order sent through a handle (cancels excluded) is
  order ID n, as in a backtest, so `cancel_target_id` means the same thing.
- **Submission rules:** the same as a backtest. `is_close` takes the side
  opposite the position; `qty <= 0` on a reducing order closes the full
  position.
- **Brackets:** `take_profit` and `stop_loss` on an entry go to the server
  as Reamer Server ABI 4 order fields. Your broker connector places and
  links the exits at the venue; their fills arrive as `<intent_id>:tp` and
  `<intent_id>:sl` and move the relay position. Exits on an `is_close`
  order are rejected.
- **Gate decisions:** your gate's accept or reject goes to the server's
  ShmEventBus, not back to the relay. A rejected order never fills and
  never moves a relay position.
- **Positions:** `reamer_relay_get_positions()` fills `qty` and
  `avg_entry_price` from fills on this handle's connection.
  `take_profit`, `stop_loss`, `unrealized_pnl` and `entry_ts` are always 0.
- **What it sends:** new orders and single cancels, as
  `ReamerOrderRequest`. The relay has no replace, and the `on_bar` actions
  (`MODIFY_ORDER`, `MODIFY_POSITION`, `CLOSE_ALL`, `CANCEL_ALL`) have no
  relay equivalent; a live strategy that needs them sends cancels and new
  orders, or speaks the strategy socket protocol directly.
- **Reconnects:** after a reconnect, fills for orders sent on the earlier
  connection do not reach the handle, so its positions can lag. Reconcile
  against your broker connector's records.

**Round-trip one order (Linux, both kits, licences activated):**

1. Server kit root: build the reference gate, simulated acceptor and
   event-bus reader.
   ```bash
   cd reference
   go build -o /tmp/reamer-demo/fix-acceptor ./cmd/fix-acceptor
   go build -o /tmp/reamer-demo/cgo-gate ./cmd/cgo-gate
   go build -o /tmp/reamer-demo/event-tail ./cmd/event-tail
   ```
2. Start the acceptor, then the server. The reference gate accepts AAPL,
   SPY and QQQ. The strategy socket defaults to `/tmp/reamer-server.sock`.
   ```bash
   /tmp/reamer-demo/fix-acceptor > /tmp/reamer-demo/acceptor.log 2>&1 &
   /tmp/reamer-demo/cgo-gate --config config/server.json > /tmp/reamer-demo/gate.log 2>&1 &
   ```
3. Research kit root: save the program below as `relay_one_order.c`.
   Build it and run it.
   ```bash
   cc -std=c11 -Iinclude relay_one_order.c -Llib -lreamer_research -Wl,-rpath,"$PWD/lib" -o relay_one_order
   ./relay_one_order /tmp/reamer-server.sock
   ```
   Expect `AAPL qty=10 avg_entry_price=0` and exit status 0. The simulated
   acceptor fills market orders at 0.00.
4. Read the order on the ShmEventBus. Expect `stage=accepted`,
   `stage=order_new` and `stage=filled` for `instrument=AAPL`.
   ```bash
   timeout 2 /tmp/reamer-demo/event-tail
   ```
5. Stop the server and the acceptor: `kill %1 %2`.

```c
/* relay_one_order.c: one market order through reamer_relay_* */
#include <stdio.h>
#include <string.h>
#include "reamer_research_abi.h"

static int fail(const char* what) {
    ReamerResearchErrorCode code; char msg[512];
    reamer_get_last_error(&code, msg, sizeof msg);
    fprintf(stderr, "%s failed: code %d: %s\n", what, (int)code, msg);
    return 1;
}

int main(int argc, char** argv) {
    const char* names[] = {"AAPL"};
    ReamerRelayHandle h;
    if (reamer_relay_open(argc > 1 ? argv[1] : "/tmp/reamer-server.sock", "relay-e2e", names, 1, &h))
        return fail("reamer_relay_open");

    ReamerOrderRequest buy;
    memset(&buy, 0, sizeof buy);
    buy.valid = true;
    buy.ticker_id = 0;
    buy.order_type = REAMER_RESEARCH_ORDER_TYPE_BUY;
    buy.kind = REAMER_RESEARCH_ORDER_KIND_MARKET;
    buy.side = REAMER_RESEARCH_SIDE_BUY;
    buy.tif = REAMER_RESEARCH_TIF_GTC;
    buy.qty = 10;
    if (reamer_relay_send(h, &buy, 1)) return fail("reamer_relay_send");

    ReamerPositionState pos;
    for (int i = 0; i < 20; ++i) {
        if (reamer_relay_poll(h, 100)) return fail("reamer_relay_poll");
        reamer_relay_get_positions(h, &pos, 1);
        if (pos.qty != 0) break;
    }
    printf("AAPL qty=%g avg_entry_price=%g\n", pos.qty, pos.avg_entry_price);
    reamer_relay_close(h);
    return pos.qty == 10 ? 0 : 1;
}
```

## Where to go next

**Every document named below ships in the Reamer Server kit** and is
published in these docs.

**For the full server-side ABI contract:** see EXTENSION_PROTOCOL.md. It
covers the gate and broker connector vtables in full, the strategy relay
wire protocol, the default `local`-mode strategy socket protocol, and the
shared event stream — each specified byte for byte.

**On the venue connector, scope it before you start.** Two reference
points ship, at opposite ends. `reference-skeleton/` (server kit, C++ and
Rust) shows the minimum shape — an accept-all gate and a paper broker
that fills in-memory, about a hundred lines against the header — and
exists only to confirm the wiring matches what you expect. `reference/`
(server kit, Go) is the other end: a FIX 4.4 session that logs on,
sequences, heartbeats and reconnects, wired to a broker connector that
turns execution reports into ABI structs, exercised end to end against a
simulated acceptor by `demo-trade.sh`. Neither is an adapter for **your**
venue. What is *not* small is the distance between a session that works
against a simulated acceptor and a connector fit for your venue in
production. Budget for these explicitly, because none of them are
optional and all of them are yours:

- `ResendRequest` (35=2) handling. A connector that ignores it loses
  fills after any sequence gap.
- Sequence-number persistence across restarts.
- Gap-fill and session recovery that does not tear the session down on
  the first gap.
- Credential handling, venue certification, and whatever conformance
  suite your venue requires.

That division is the design, not a gap in it: Reamer Server supplies the
order core, which has to be right and is the same for everyone, and you
build the edge that touches your venue, your credentials, and your risk
policy, your own way.

**For the step-by-step integration path:** start with DEPLOYMENT.md. It
walks from downloading the ABI to testing your gate and connector against
a local FIX acceptor (no real venue needed) to writing a config and
booting your server.
