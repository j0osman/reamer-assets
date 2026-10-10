---
title: Reamer Server extension protocol
description: The Reamer Server integration contract: gate and broker connector vtables, order fields, both strategy wire protocols and the shared-memory event bus.
group: integration
order: 6
product: server
source: EXTENSION_PROTOCOL.md
---

Reamer Server ships as a closed-source compiled artifact — a header
(`reamer_server_abi.h`) and a static/shared library
(`libreamer_server_core.a`/`.so`). There is no source tree, but there is a
stable C ABI you link against: implement a gate and a broker connector as
plain C structs of function pointers, hand them to `reamer_server_run()`,
and that call *is* your program. No socket, no serialization, no second
process.

Two extension points work this way:

| Interface | What you provide | What it does |
|---|---|---|
| Gate | a `ReamerGateVtable` | one pre-trade decision per intent: pass or reject, given the intent and the venue's last known state |
| Broker connector | a `ReamerBrokerConnectorVtable` | talks to your venue, reports fills back, and is the sole source of venue state (positions, open-order count, connection status) |

Both vtables carry a `void* user_data` slot, so a C++ implementation closes
over its own object through a `static` trampoline (or a cgo callback, in
Go — see "Go, via cgo" below) exactly the way any C callback API works.
`reamer_server_run(gate_vtable, broker_vtable, config_path)` blocks running
the core loop until shutdown; everything before and after that call —
`main()`, CLI parsing, process supervision — is yours to write.

Strategies are unaffected by any of this — they still connect over the
unchanged Unix domain socket strategy-input protocol described in "Strategy
relay protocol" below. That's the one remaining wire-protocol extension
point; a strategy process, unlike a gate or connector, is never in the same
address space as core.

## Shutdown contract

On SIGTERM/SIGINT, `reamer_server_run()` does not stop immediately. It
stops accepting new `StrategyRequest`s, then keeps calling your broker
connector's `tick`/`poll_updates`/`poll_events`/`poll_published_events` for
up to `shutdown_drain_seconds` (config field / `REAMER_SHUTDOWN_DRAIN_SECONDS`
env var, default `5`; `0` disables draining) so whatever your connector
already has in flight — fills, acks, session events — is drained and
published exactly as during normal operation, rather than dropped. Draining
ends early once a pass finds nothing left to drain.

Core never calls `submit_cancel` on your behalf during this drain: every
open order is left exactly as it is at the venue. This follows directly
from the account-state design above — the venue is the sole source of
truth, core keeps no second copy, and a restart's first `pull_state` call
picks positions and open-order-count back up automatically. What draining
buys is operator visibility, not order safety: once it ends, core calls
`pull_state` one final time and publishes a reconciliation snapshot to the
monitoring endpoint before exiting, so the last dashboard view reflects the
venue's actual state rather than one that predates the shutdown signal.

If your connector needs to do its own work on shutdown (e.g. flush a local
log, close a session cleanly), do it inside `tick()`/`poll_events()` as
called during this drain window — there is no separate shutdown callback.

If your gate or broker connector implementation returns quickly and
correctly, core keeps running; there is no reconnect/backoff/timeout
machinery left for these two, because there is no connection to lose —
a function call either returns or it doesn't build. `is_available`/
`is_connected` are still the mechanism by which your own implementation
tells core it's degraded (a dropped venue session, a gate that can't
currently evaluate anything), and core's fallback behavior on `false` is
exactly what it was over the wire: a rejected `Ack` from the gate, a
disconnected `VenueState` from the connector.

There is deliberately no third callable extension point that derives or
holds its own copy of account state. Core keeps exactly one: whatever
your broker connector's `pull_state` callback last wrote into its output
struct, tagged `connected`. The gate receives that state on every check
and decides pass/reject from it — it never needs to maintain a position
ledger of its own, and there is nothing for core to reconcile, restore at
startup, or escalate an alarm about, because there is no second copy that
could ever drift from the venue's own truth.

**Before reading further:** `reference-skeleton/` (kit root) is a complete,
runnable `reamer_server_run()` wiring — accept-all gate, in-memory paper
broker — that builds and runs against nothing but this document and the
header. It is not a starting point for a real connector; it exists so you
can confirm your mental model of the wiring below matches reality before
you start on one.

## The header

`reamer_server_abi.h` is `extern "C"`, callable from C, C++, or any
language with a C FFI (Go via cgo, Rust via bindgen, etc.). Every struct
crossing the boundary is flat — no `std::string`, no STL containers.
Strings are fixed-capacity `char[REAMER_MAX_STRING_LEN]` (256) inline
fields, not pointer+length pairs — simplest possible ownership story (the
callee always owns its own buffer, nothing to free, nothing to outlive a
call) at the cost of a hard per-field length cap. `VenueState.positions` is
a fixed `ReamerPosition[REAMER_MAX_POSITIONS]` (256) array plus a
`position_count`, the same flat-array shape as every other bounded list on
this boundary (`poll_updates`, `poll_events`, `poll_published_events`).

Both limits are enforced, not advisory. A field longer than 255 bytes, or a
`VenueState` with more than 256 open positions, is rejected at the boundary
rather than silently truncated: the affected intent is rejected (`Ack{
accepted=false}`) or the affected broker submit is dropped, and the reason
is logged (`abi.field_overflow`). Core never hands your `Gate` or
`BrokerOutputConnector` implementation a truncated or partially-populated
struct.

`REAMER_ABI_VERSION` is a compile-time constant baked into the header, so a
version mismatch can't be caught at runtime — a customer building against a
mismatched header is comparing the constant to itself, which the compiler
folds away. Instead, the library pins `sizeof` of each vtable to its
expected size for the current `REAMER_ABI_VERSION` via `static_assert`,
so a header/library version mismatch (or an accidental
layout drift within the same version) fails to *compile* rather than
running against structs the library and your build disagree about the
layout of.

## Gate vtable

```c
typedef struct {
    void* user_data;
    int (*check)(void* user_data, const ReamerIntent* intent,
                 const ReamerVenueState* venue_state, ReamerAck* out_ack);
    bool (*is_available)(void* user_data);
} ReamerGateVtable;
```

Core calls `check()` once per intent — new order, cancel, and replace all
go through the same call, evaluated as a `ReamerIntent` — and always passes
the venue's current `ReamerVenueState` alongside it, pulled fresh from your
broker connector's `pull_state` immediately beforehand. If
`venue_state->connected` is false, core rejects the order itself
(`reason="venue disconnected"`) and never calls `check()` at all — this is
a fixed, non-configurable core rule, not a policy your gate can override.

Write your decision into `*out_ack` and return `0` for accepted, non-zero
for rejected — the return code and `out_ack->accepted` should agree;
core reads `out_ack` as the source of truth. `out_ack->request_id` is
overwritten by core with the intent's own id right after your call
returns, regardless of what you set it to, so there's no correlation
bookkeeping to get right on your side.

Cancels and replaces are also sent to your gate as a `ReamerIntent` — core
builds a synthetic one from the `ReamerCancelRequest`/`ReamerReplaceRequest`
fields before calling `check()`, so your gate only ever needs to implement
one decision shape.

**A synthetic intent populates only the fields its source request carries.
Every other field holds a default value, not data.** Validate accordingly —
a check written for new orders will misjudge a cancel if it reads a field
the cancel never set.

| Field | New order | Cancel | Replace |
|---|---|---|---|
| `connection_id`, `strategy_id`, `intent_id` | set | set | set |
| `instrument` | set | set (from the target order) | set (from the target order) |
| `quantity` | set | **`0.0` — not set** | set (`new_quantity`) |
| `limit_price`, `stop_price` | set | **`0.0` — not set** | set (`new_limit_price`/`new_stop_price`) |
| `side`, `order_type`, `position_effect`, `time_in_force`, `expire_time` | set | **default — not set** | **default — not set** |
| ABI 4 order fields (`take_profit` … `payload`) | set | **default — not set** | `take_profit`, `stop_loss`, `trailing_offset` set (`new_take_profit`/`new_stop_loss`/`new_trailing_offset`); the rest default |

On a cancel, `intent_id` is the cancel's own `request_id`, and the order
being cancelled is identified by `target_order_id` on the originating
`ReamerCancelRequest` — a cancel has no size, so `quantity` is meaningless
there by design. A blanket `quantity <= 0` rejection in your gate will
therefore reject every cancel you receive. Gate a cancel on identity and
policy (instrument, strategy, venue state), not on order-shaped numeric
fields.

Rejecting a synthetic intent is a normal, valid outcome, so core does not
warn about it: the cancel or replace is simply not forwarded to your broker
connector, and the outcome is published as an `IntentFlowEvent` with stage
`cancel_rejected`/`replace_rejected` (see "Shared pipeline event stream"
below). If a
cancel appears to do nothing, read that event stream first — it carries
your gate's own `reason` string.

### Struct field layouts

`ReamerIntent`: `connection_id: uint64_t`, `strategy_id`, `intent_id`,
`instrument` (all `char[256]`), `side: ReamerSide` (0=Buy, 1=Sell,
2=SellShort), `order_type: ReamerOrderType` (0=Market, 1=Limit, 2=Stop,
3=StopLimit, 4=TrailingStop, 5=TrailingStopLimit, 6=MarketIfTouched,
7=LimitIfTouched, 8=MarketToLimit), `quantity: double`, `limit_price:
double`, `stop_price: double`, `position_effect: ReamerPositionEffect`
(0=Open, 1=Close), `time_in_force: ReamerTimeInForce` (0=Day, 1=Gtc,
2=Ioc, 3=Fok, 4=Gtd, 5=AtTheOpen, 6=AtTheClose), `expire_time: char[256]`.

Then the ABI 4 order fields, appended after `expire_time` so every ABI 3
field keeps its offset. Each is 0 or empty when not used, and core does
not interpret any of them: your gate decides what to allow and your
connector maps them to your venue.

| Field | Type | Meaning |
|---|---|---|
| `take_profit` | `double` | Attached take-profit exit, absolute price |
| `stop_loss` | `double` | Attached stop-loss exit, absolute trigger price |
| `stop_loss_limit` | `double` | Makes the stop-loss exit a stop-limit at this price |
| `stop_loss_trailing_offset` | `double` | Makes the stop-loss exit trail by this offset |
| `trailing_offset` | `double` | Trail amount for a trailing order type |
| `offset_type` | `ReamerOffsetType` | Unit of the offsets: 0=Price, 1=BasisPoints, 2=Ticks |
| `limit_offset` | `double` | Trailing-stop-limit: limit distance from the trailing stop |
| `trigger_type` | `ReamerTriggerType` | Price that triggers stops and touches: 0=Default (venue), 1=Last, 2=BidAsk, 3=Mid, 4=Mark, 5=Index |
| `display_quantity` | `double` | Iceberg: shown size |
| `min_quantity` | `double` | Minimum fill size |
| `exec_flags` | `uint32_t` | Bits: `REAMER_EXEC_POST_ONLY` 0x1, `REAMER_EXEC_REDUCE_ONLY` 0x2, `REAMER_EXEC_HIDDEN` 0x4, `REAMER_EXEC_ALL_OR_NONE` 0x8 |
| `contingency_type` | `ReamerContingencyType` | Link to another order: 0=None, 1=OCO, 2=OTO, 3=OUO |
| `linked_intent_id` | `char[256]` | The order `contingency_type` links to |
| `account` | `char[256]` | Venue account or sub-account |
| `destination` | `char[256]` | Venue, exchange or route |
| `payload_schema` | `uint32_t` | Your own tag for the payload format |
| `payload_len` | `uint32_t` | Payload bytes, at most `REAMER_MAX_PAYLOAD_LEN` (1024) |
| `payload` | `const uint8_t*` | Your own bytes, passed through untouched; `NULL` when `payload_len` is 0. Valid only during the call: copy it to keep it |

**Attached exits (brackets).** When `take_profit` or `stop_loss` is
nonzero, your connector receives the entry with them set and places the
exits at the venue: a native bracket or OCO order where the venue has
one, or its own emulation. Core tracks each exit as an order of its own,
`<intent_id>:tp` and `<intent_id>:sl`: opposite side, same quantity,
`position_effect` Close. With both set they are each other's OCO partner.
Report the exits' fills, cancels and rejects under those ids and they
reach the strategy like any other order's. The strategy can cancel or
replace an exit by its id. On a replace, `new_take_profit`,
`new_stop_loss` and `new_trailing_offset` move the entry's exits.

A connector built against ABI 3 keeps working. Core passes it a larger
struct whose ABI 3 fields keep their offsets, and the connector does not
see the new fields.

`ReamerVenueState`: `positions: ReamerPosition[256]` (each
`{instrument: char[256], qty: int64_t}`, nanounits — see "Fixed-point OMS
fields" below), `position_count: size_t`, `open_order_count: size_t`,
`connected: bool`. This is your connector's last successful `pull_state`
write — the one and only account-state value core holds; there is no
richer or more current copy anywhere else in core to fall back to.

`ReamerAck`: `request_id: char[256]`, `accepted: bool`, `reason: char[256]`.

### Worked example: one `check()` call

1. A strategy submits a new order; core builds a `ReamerIntent`, calls
   your broker connector's `pull_state`, and (since `connected` is true)
   calls `gate_vtable->check(user_data, &intent, &venue_state, &ack)`.
2. Your implementation runs its own pre-trade logic against the
   `ReamerVenueState` it was handed and writes `{accepted: true, reason:
   ""}` into `*out_ack`, returning `0`.
3. Core reads `ack` back out of the struct you wrote and, since
   `accepted` is true, submits the resulting order to your broker
   connector's `submit_order` — in the same call stack, no round trip.

There is no timeout to configure and no degraded-answer fallback path
distinct from what your own `check()` returns — a call that never returns
is a hang in your code, in-process, same as any other function call would
be; keep `check()` fast and deterministic, since it runs on core's hot
path once per intent.

An exception thrown out of your `check()` or `is_available()`
implementation is caught at the boundary, not propagated: core logs it
(`gate.check.exception` / `gate.is_available.exception`) and degrades to
a safe default — a rejected `Ack` for `check()`, `false` for
`is_available()` — rather than crashing `reamer_server_run()`. A bug in
your implementation that throws costs you that one call's result, not
the process.

## Broker connector vtable

```c
typedef struct {
    void* user_data;
    void (*submit_order)(void* user_data, const ReamerIntent* intent);
    void (*submit_cancel)(void* user_data, const ReamerCancelRequest* request);
    void (*submit_replace)(void* user_data, const ReamerReplaceRequest* request);
    void (*pull_state)(void* user_data, ReamerVenueState* out_state);
    size_t (*poll_updates)(void* user_data, ReamerExecutionReport* out_updates, size_t max_reports);
    size_t (*poll_events)(void* user_data, ReamerOpsEvent* out_events, size_t max_events);
    size_t (*poll_published_events)(void* user_data, ReamerPipelineEvent* out_events, size_t max_events);
    void (*tick)(void* user_data);
    bool (*is_connected)(void* user_data);
} ReamerBrokerConnectorVtable;
```

Every one of these calls is synchronous — there is no fire-and-forget
mode left to design around, since a direct call either finishes or it
doesn't. `reamer_server_run()` spawns no threads of its own: every gate
and broker connector callback — this vtable and `ReamerGateVtable`'s
`check()` alike — runs sequentially, on the thread that called
`reamer_server_run()`. Your implementation does not need locks or
thread-safe state to be correct with respect to core calling it. Core
drives your connector every loop iteration:

- **`tick()`** — advance whatever session/reconnect state machine your
  implementation owns (a FIX heartbeat, a TCP reconnect timer). Called
  once per loop iteration and again after every submit, matching how
  often the previous wire-protocol connector's own internal `tick()`
  ran.
- **`submit_order`/`submit_cancel`/`submit_replace`** — mould the given
  struct into your venue's wire format and send it. Return promptly;
  there's no async ack path back to core from these three calls
  specifically — fills and rejects come back through `poll_updates`.
- **`pull_state`** — write your connector's current view of the venue
  (net positions per instrument, open-order count, `connected`) into
  `*out_state`. Core calls this once per intent, immediately before every
  gate check, so answer from whatever you already keep current
  internally (your last successful venue query, or a running tally
  updated as fills/acks arrive) — not with fresh venue I/O on this call's
  critical path if you can avoid it.
- **`poll_updates`/`poll_events`/`poll_published_events`** — fill the
  given array up to `max_*`, return how many you actually wrote. Core
  calls each once per loop iteration and drains whatever's pending;
  there's no minimum or maximum call rate to honor beyond "don't block."
- **`is_connected`** — true only once your venue session is actually
  usable for trading, not merely "the socket is open" — this gates
  whether core calls your gate at all (see "Gate vtable" above), matching
  the reference FIX connector example's own distinction between "socket
  open" and "trading session established."

### Struct field layouts

`ReamerOrderRecord`: `order_id`, (`char[256]`), `connection_id:
uint64_t`, `strategy_id`, `instrument` (`char[256]`), `side: ReamerSide`,
`order_type: ReamerOrderType`, `quantity: int64_t`, `filled_quantity:
int64_t`, `limit_price: int64_t`, `stop_price: int64_t`, `position_effect:
ReamerPositionEffect`, `status: ReamerOrderStatus`, `venue_order_id:
char[256]`, `total_commission: int64_t`, `time_in_force:
ReamerTimeInForce`, `expire_time: char[256]`. The five `int64_t` fields
are nanounits — see "Fixed-point OMS fields" below.

`ReamerExecutionReport`: `order_id`, `venue_order_id` (`char[256]`),
`status: ReamerOrderStatus`, `last_qty: int64_t`, `last_price: int64_t`,
`commission: int64_t` (nanounits — see "Fixed-point OMS fields" below),
`reason: char[256]`, `transact_time: char[256]`, `exec_id: char[256]`,
`is_cancel_reject: bool`, `last_rpt_requested: bool`.

### Fixed-point OMS fields

`quantity`, `filled_quantity`, `limit_price`, `stop_price`, and
`total_commission` on `ReamerOrderRecord`; `last_qty`, `last_price`, and
`commission` on `ReamerExecutionReport`; `qty` on `ReamerPosition` — these
are all `int64_t`, scaled by `REAMER_FIXED_SCALE` (`1e9`, nanounits):
the wire value for `1.5` is `1_500_000_000`. Core's OMS ledger accumulates
`filled_quantity` and `total_commission` across many fills, and binary
`double` drifts under repeated addition — a position that should net to
exactly zero can read as `-1e-13`. Scaled integers make "flat" and "fully
filled" exact integer equality, with no epsilon comparison anywhere in
core or in a correct connector implementation. Divide by
`REAMER_FIXED_SCALE` to recover a `double` for display or logging;
multiply and round/truncate to convert a `double` back to nanounits when
writing these fields — `(int64_t)(value * REAMER_FIXED_SCALE)` going in,
`(double)field / REAMER_FIXED_SCALE` coming out. Round rather than
truncate if your venue's own quantities are not exact in binary floating
point. `ReamerIntent`, `ReamerCancelRequest`, `ReamerReplaceRequest`
(strategy-submitted, see "Struct field layouts" above) and
`ReamerPipelineEvent` (monitoring) are unaffected by this and stay
`double` — the fixed-point scale applies only to the OMS-internal ledger
fields listed here.

`ReamerOrderStatus` (enum): 0=PendingNew, 1=New, 2=PartiallyFilled,
3=Filled, 4=Cancelled, 5=Rejected, 6=PendingCancel, 7=PendingReplace.

### Order-state transition legality

Core enforces which `ReamerExecutionReport`s may legally be applied to a
tracked order. Two rules, checked in this order:

1. **Terminal is terminal.** Once an order reaches `Filled`, `Cancelled`,
   or `Rejected`, no further report is applied to it — a fill, cancel, or
   any other report your connector emits after that point is rejected.
2. **No overfill.** A report is rejected if applying its `last_qty` would
   push `filled_quantity` above `quantity`, regardless of the order's
   current status.

Every other transition is legal, including a `PendingCancel`/
`PendingReplace` order resolving back to `New`/`PartiallyFilled` (the
venue rejected the cancel/replace action itself — the FIX
`OrderCancelReject`/`is_cancel_reject` path) and a report that reaches a
terminal state from any open status.

A rejected report is dropped: the tracked order's `status` and
`filled_quantity` are left unchanged, the strategy does not receive an
`OrderUpdate` for it, and core logs a warning
(`order_tracking.illegal_transition`, naming the order id and both the
current and reported status) plus publishes an `order_transition_rejected`
`ReamerPipelineEvent`. Your connector should not rely on core correcting
or reconciling a bad report on your behalf — get the report right at the
source.

### Duplicate, out-of-order, and unrecognized reports

Real venues resend reports under retransmission, and a `poll_updates` call
can surface a report for an order core no longer knows about. Core's
handling:

1. **Late report on a terminal order.** A fill, cancel, or any other
   report arriving after an order already reached `Filled`/`Cancelled`/
   `Rejected` is out of order by definition and is rejected by the
   terminal-is-terminal rule above (`order_tracking.illegal_transition`).
2. **Retransmitted fill, same `exec_id`.** Set `exec_id` (FIX tag 17,
   ExecID) on every `ReamerExecutionReport` your connector emits for a
   fill. Core tracks which `exec_id` values have already been applied to
   each order and rejects a repeat, via the same
   `order_tracking.illegal_transition` path, before it re-adds quantity or
   commission. **This is required, not optional**: FIX session-level
   sequence numbers (handled at your connector's session layer) dedupe
   transport-level resends only — a venue can legitimately resend the
   same execution
   under a fresh session after a reconnect, at a new sequence number, with
   the same ExecID. If your connector leaves `exec_id` empty, core falls
   back to the no-overfill check alone, and a resent fill that still fits
   under the ordered quantity will be double-counted into
   `filled_quantity` and `total_commission`.
3. **Report for an order core never tracked** — an unsolicited report, or
   one for an order this process didn't submit, or one already evicted
   under `max_tracked_orders` pressure — see the eviction note in
   `CONFIG.md`). Core looks the report's `order_id` up in
   its tracked-order table; on a miss, the report is dropped silently as
   far as the strategy and pipeline are concerned (no `OrderUpdate`, no
   `IntentFlowEvent`), but core still logs a warning
   (`order_tracking.unknown_order`, naming the order id and reported
   status) so it's visible in ops monitoring.

None of these three cases reach the strategy connection or mutate any
tracked order state.

`ReamerCancelRequest`: `connection_id: uint64_t`, `strategy_id`,
`request_id`, `target_order_id` (`char[256]`).

`ReamerReplaceRequest`: `connection_id: uint64_t`, `strategy_id`,
`request_id`, `target_order_id` (`char[256]`), `new_quantity: double`,
`new_limit_price: double`, `new_stop_price: double`, then ABI 4:
`new_take_profit: double`, `new_stop_loss: double`,
`new_trailing_offset: double` (0 = unchanged).

`ReamerOpsEvent`: `level: ReamerOpsEventLevel` (0=Info, 1=Warning,
2=Error), `message: char[256]`, `code: char[256]`.

`ReamerPipelineEvent`: `sequence: uint64_t`, `timestamp: char[256]`,
`category: ReamerPipelineEventCategory` (0=Ops, 1=IntentFlow), `level`,
`message`, `code`, `intent_id`, `strategy_id`, `instrument`, `stage`,
`reason` (all `char[256]`), `last_qty: double`, `last_price: double`,
`request_id: char[256]` (added in `REAMER_ABI_VERSION` 3; empty except on
`cancel_*`/`replace_*` stages, where it — not `intent_id`, which there
carries the *target order's* id — is the id to route the outcome back to
the requester by).
`poll_published_events` is **inbound**: core publishes whatever you return
from it onto the shared event bus, stamping each event with a fresh
sequence. It exists so your connector can surface pipeline events core
cannot otherwise see — transport diagnostics such as a relay disconnect or
a sequencer close.

It is **not** a way to read core's stream back. Returning events you read
off the bus makes core republish them under new sequences; if you re-read
the bus every iteration, one event becomes a self-sustaining loop running
at main-loop frequency, wrapping the ring and evicting real order events
before any reader can correlate them. Core defends against this: any event
you return that already carries a nonzero `sequence` — which only core
assigns — is dropped, and `connector.echoed_event` is logged once per
process. To consume core's stream, attach a reader to the shared-memory
event bus directly (see "Shared pipeline event stream" below).

The same guarantee applies to every method on this vtable: an exception
thrown from your `submit_order`/`submit_cancel`/`submit_replace`/
`pull_state`/`poll_updates`/`poll_events`/`poll_published_events`/`tick`/
`is_connected` implementation is caught, logged
(`broker.<method>.exception`, e.g. `broker.submit_order.exception`), and
degraded to a safe default — the submit/tick calls simply no-op for that
call, `pull_state` yields a default (disconnected) `VenueState`,
`is_connected` returns `false`, and the three poll methods return an
empty result — rather than crashing the process.

## Strategy socket protocol (`local` mode)

This is the protocol core speaks when `strategy_input_mode` is `local`
(the **default**). Core listens on a Unix domain socket at `socket_path`
(default `/tmp/reamer-server.sock`, mode `0600`), and strategies connect
to it directly — there is no relay process in this mode. Write a client
against this section and it talks to core with nothing in between.

Do not build a `local`-mode client from the "Strategy relay protocol"
section below. That protocol is a different shape: it carries an explicit
`connection_id` in each request because one relay multiplexes many
strategies behind a single connection to core. Here, one socket
connection *is* one strategy, so core assigns the `ConnectionId` from the
connection itself and there is no such field on the wire. A relay-shaped
`Intent` sent to this socket is rejected as `malformed request`.

### Framing

Identical to the relay's framing — the two protocols differ in payload
layout, not in how frames are delimited:

```
+----------------+----------------------+----------------------------+
| version (1B)   | payload length (4B)  | payload (length bytes)     |
+----------------+----------------------+----------------------------+
```

- `version` is `1`. Any other version byte is rejected and the connection
  closed — there is no negotiation.
- `payload length` is big-endian `uint32`, counting `payload` only (not
  the 5-byte header). Capped at 1 MiB; a longer length is rejected and
  the connection closed.
- `payload`'s first byte is a message-type discriminator; the rest is
  that message's fields in the order listed below.

Field encodings: integers are big-endian at the stated width; `double` is
IEEE-754 sent as its 8 raw bytes in the same big-endian order as a `u64`;
`string` is a `u16` big-endian byte length followed by that many raw
UTF-8 bytes, no null terminator; `bool` is one byte, `0` or `1`.

A payload that fails to decode — truncated, an unrecognized type byte, or
an out-of-range enum value — closes the connection and logs
`sequencer.connection_closed` with the offending type byte. Malformed
input is fatal to that connection, never guessed around. Frames may be
pipelined: core keeps reading, so a burst of frames written back-to-back
without waiting is valid.

### Strategy -> core

| Type byte | Name | Payload |
|---|---|---|
| 0 | NewOrder | `strategy_id: string`, `intent_id: string`, `instrument: string`, `side: u8`, `order_type: u8`, `quantity: double`, `limit_price: double`, `stop_price: double`, `position_effect: u8`, `time_in_force: u8`, `expire_time: string`, optional order extension block |
| 1 | Cancel | `strategy_id: string`, `request_id: string`, `target_order_id: string` |
| 2 | Replace | `strategy_id: string`, `request_id: string`, `target_order_id: string`, `new_quantity: double`, `new_limit_price: double`, `new_stop_price: double`, optional replace extension block |

**There is no `connection_id` field on any of these.** Core sets it from
the accepting connection. Sending one shifts every subsequent field by 8
bytes and the frame is rejected.

Enum values on the wire match the header's enums exactly: `side` is
`ReamerSide` (0 buy, 1 sell, 2 sell_short); `order_type` is
`ReamerOrderType` (0 market, 1 limit, 2 stop, 3 stop_limit, 4
trailing_stop, 5 trailing_stop_limit, 6 market_if_touched, 7
limit_if_touched, 8 market_to_limit); `position_effect` is
`ReamerPositionEffect` (0 open, 1 close); `time_in_force` is
`ReamerTimeInForce` (0 day, 1 gtc, 2 ioc, 3 fok, 4 gtd, 5 at_the_open, 6
at_the_close). A value outside these ranges is a decode failure, not a
silent default.

**Extension blocks (ABI 4).** A NewOrder or Replace frame may continue
past its last ABI 3 field with one block. A frame that stops where the
ABI 3 frame stopped decodes with every ABI 4 field at 0 or empty, so an
ABI 3 client needs no change, and its bytes are what they always were.

Order extension block, after `expire_time`: `ext_version: u8` (1),
`take_profit: double`, `stop_loss: double`, `stop_loss_limit: double`,
`stop_loss_trailing_offset: double`, `trailing_offset: double`,
`offset_type: u8`, `limit_offset: double`, `trigger_type: u8`,
`display_quantity: double`, `min_quantity: double`, `exec_flags: u32`,
`contingency_type: u8`, `linked_intent_id: string`, `account: string`,
`destination: string`, `payload_schema: u32`, `payload_len: u32`, then
`payload_len` raw bytes.

Replace extension block, after `new_stop_price`: `ext_version: u8` (1),
`new_take_profit: double`, `new_stop_loss: double`,
`new_trailing_offset: double`.

The block must end the frame. An unknown `ext_version`, an out-of-range
enum, a `payload_len` over 1024, or bytes left after the block is a
decode failure. Fields are as in "Struct field layouts" under "Gate
vtable" above.

### Core -> strategy

| Type byte | Name | Payload |
|---|---|---|
| 1 | OrderUpdate | `intent_id: string`, `instrument: string`, `side: u8`, `status: u8`, `filled_quantity: double`, `last_price: double`, `venue_order_id: string`, `commission: double`, `reason: string`, `timestamp: string` |
| 2 | ServerEvent | `code: string`, `message: string` |

`status` is `ReamerOrderStatus` (0 pending_new, 1 new, 2 partially_filled,
3 filled, 4 cancelled, 5 rejected, 6 pending_cancel, 7 pending_replace).

Type byte `0` (Ack) is **retired** and left unassigned, exactly as on the
relay protocol: per-request accept/reject is published as an
`IntentFlowEvent` on the shared pipeline event stream rather than sent
back over this socket. Read acceptance and rejection from the event bus
(see "Shared pipeline event stream"), not from this connection. The byte
is not reused, so a client still expecting type-0 framing fails closed
instead of misreading a later type byte.

These messages are addressed by connection: core writes an `OrderUpdate`
only to the connection that submitted the originating request, so unlike
the relay protocol there is no `remote_connection_id` to route on.

### Priority

`SetPriorityMsg` has no equivalent here — it is a relay-mode message.
Every `local`-mode connection runs at the default priority `0`, and
requests within a poll cycle are ordered by arrival. The strict-priority
semantics described under the relay protocol apply only when a relay is
multiplexing strategies.

## Strategy relay protocol

Use this protocol only when `strategy_input_mode` is `remote`. For the
default `local` mode, see "Strategy socket protocol" above — the payload
layouts differ.

Strategy input stays an out-of-process wire protocol, since a strategy process is never expected
to share an address space with core the way a gate or connector now can.
Your relay process owns whatever transport strategies actually connect
over (TCP, shared memory, anything) — that transport and its own wire
format are entirely up to you and outside this spec. What this protocol
covers is the single connection between your relay and the Reamer Server core,
over which your relay forwards decoded strategy requests and core sends
back acks/updates/events for your relay to route to the right strategy.

### Shared wire framing

```
+----------------+----------------------+----------------------------+
| version (1B)   | payload length (4B)  | payload (length bytes)     |
+----------------+----------------------+----------------------------+
```

- `version` is `1`. A frame with any other version byte is rejected and the
  connection is closed — there is no negotiation.
- `payload length` is big-endian `uint32`, the byte length of `payload`
  only (not including the 5-byte header). Capped at 1 MiB; a longer length
  is rejected and the connection closed.
- `payload`'s first byte is always a message-type discriminator, meaning
  defined per table below. Everything after it is that message's fields,
  in the order listed.

Field encodings: integers are big-endian, whatever fixed width the field
says (`u8`/`u16`/`u64`); `double` is IEEE-754, transmitted as its 8 raw
bytes reinterpreted as a `u64` and byte-swapped the same way any other
`u64` is; `string` is a `u16` big-endian length prefix followed by that
many raw bytes (UTF-8, no null terminator); `bool` is a single byte, `0`
or `1`.

A message that fails to parse (truncated, or an unrecognized type byte) is
dropped and the connection is closed — malformed input from a customer
process is treated as fatal to that connection, not something to guess
around.

Your relay may hold many strategy connections behind this one link to
core. Every message that refers to a specific strategy connection carries
a `remote_connection_id` — an opaque `u64` **you choose**, one per strategy
connection your relay is handling. Core adopts whatever value you send
directly as the `ConnectionId` it uses everywhere else (in `OpsEvent`
attribution, in the dashboard, in `setPriority`) — there's no separate
mapping table, so pick ids however is convenient for you (e.g. a
per-connection counter) and stay consistent for that connection's lifetime.

For `RelayedNewOrder`/`RelayedCancel`/`RelayedReplace`, there's no separate
`remote_connection_id` field — it's simply the `connection_id` field
already inside `Intent`/`CancelRequest`/`ReplaceRequest` (below). For
`OrderUpdateMsg`/`ServerEventMsg`/`SetPriorityMsg`, which have no field of
their own to reuse, it's an explicit leading field.

**Priority is strict, not weighted-fair, and applies within a batch.**
`SetPriorityMsg` sets an integer priority per connection (default `0`,
higher runs first). Within each internal poll cycle, core sorts every
request received in that cycle by priority alone — highest first,
unconditionally — and only falls back to arrival order to break a tie
between two requests at the *same* priority. There is no round-robin, no
weighting, and no fairness guarantee across priorities: a connection
sending a steady stream of requests at a higher priority than another
connection's will run ahead of it indefinitely, cycle after cycle. If
your relay multiplexes several strategies at different priorities behind
one connection to core, design for that: a lower-priority strategy can
be starved by a busier higher-priority one for as long as the busier one
keeps submitting, and the product does not detect or alert on this for
you.

`connectionIds()` on the core side is a cache, refreshed by a
`ConnectionIdsQuery`/`ConnectionIdsResponse` exchange core fires once per
`poll()` call — so it can be briefly stale by up to one poll interval,
never a live query.

### Relay -> core

| Type byte | Name | Payload |
|---|---|---|
| 0 | RelayedNewOrder | `Intent` |
| 1 | RelayedCancel | `CancelRequest` |
| 2 | RelayedReplace | `ReplaceRequest` |
| 3 | ConnectionIdsResponse | `count: u16`, then `count` × `u64` |
| 4 | Event | `PipelineEvent` — see "Shared pipeline event stream" below |

### Core -> relay

Type byte `0` (`AckMsg`) is retired: per-request accept/reject is no longer
sent back as a dedicated reply on this connection. Every `RelayedNewOrder`/
`RelayedCancel`/`RelayedReplace` already gets its outcome published as an
`IntentFlowEvent` (`stage` = `"accepted"`/`"rejected"`, or the `cancel_*`/
`replace_*` equivalents) on the shared pipeline event stream — see "Shared
pipeline event stream" below — which your relay attaches to directly, so
`AckMsg` was a second, redundant delivery of the same decision over this
slower connection. A relay built against this table before the change must
be rebuilt: it will stop receiving per-order acks on this connection and
should instead read acceptance/rejection from the event stream. The type
byte is left unused rather than reassigned, so an old relay build fails
closed on `decodeCoreMsg`'s default case instead of silently misreading a
later type byte as an ack.

The pattern a relay needs, since acks no longer arrive on this
connection: attach to the event bus directly, and key each
`IntentFlowEvent` back to the strategy connection that issued the
original `new_order`/`cancel`/`replace` through a `{strategy_id,
request_id}` lookup you populate when relaying the request to core, and
clear once the outcome is delivered or the strategy disconnects. Note
that for `cancel_*`/`replace_*` stages the id to match on is the event's
`request_id` field, not `intent_id` — `intent_id` there is the *target*
order's id, not the id the strategy submitted the request under.

| Type byte | Name | Payload |
|---|---|---|
| 1 | OrderUpdateMsg | `remote_connection_id: u64`, `intent_id: string`, `instrument: string`, `side: u8`, `status: u8`, `filled_quantity: double`, `last_price: double`, `venue_order_id: string`, `commission: double`, `reason: string`, `timestamp: string` |
| 2 | ServerEventMsg | `remote_connection_id: u64`, `code: string`, `message: string` |
| 3 | SetPriorityMsg | `remote_connection_id: u64`, `priority: i32` (sign-extended into a u64 on the wire) |
| 4 | ConnectionIdsQuery | (no fields) |

`Intent` on the wire mirrors `ReamerIntent`'s fields exactly (see "Gate
vtable" above), string-encoded per the framing rules above instead of
fixed-capacity `char[]`; its ABI 4 fields are the optional order
extension block described under "Strategy socket protocol", ending the
frame. `ReplaceRequest` likewise ends with the optional replace extension
block. `CancelRequest`:
`connection_id: u64`, `strategy_id: string`, `request_id: string`,
`target_order_id: string`. `ReplaceRequest`: `connection_id: u64`,
`strategy_id: string`, `request_id: string`, `target_order_id: string`,
`new_quantity: double`, `new_limit_price: double`, `new_stop_price:
double`. Note `OrderUpdateMsg`'s payload omits `connection_id` and
`found` — those are core-internal routing metadata, not meaningful to
send back over this connection; the target connection is instead the
message's own leading `remote_connection_id`.

### Strategy -> relay: the reference relay's own schema

Everything above defines relay <-> core. The other side of your relay --
what strategies speak to *it* -- is yours to choose, as stated at the top
of this section. Core neither sees nor constrains it.

The reference relay that ships in this kit had to pick something, and it
picked **newline-delimited JSON over TCP** (`--strategy-addr`, default
`:9100`). Its schema is documented here so you can drive the shipped
relay from any language without reading Go, and so you have a worked
example if you are writing your own. **This is the reference
implementation's choice, not a protocol Reamer Server mandates** -- replace it
wholesale in your own relay if a different transport suits you.

One JSON object per line. Inbound `type` is one of `new_order`,
`cancel`, `replace`; any other value is ignored. **A line that fails to
parse as JSON is skipped silently** -- there is no error reply, so a
malformed order looks exactly like an order that was never sent.

| Field | Types | Notes |
|---|---|---|
| `type` | all | `new_order`, `cancel`, or `replace` |
| `strategy_id` | all | Your identifier for the submitting strategy |
| `intent_id` | `new_order` | Your id for this order; echoed back on `order_update` |
| `instrument` | `new_order` | Passed through to your gate and connector |
| `side` | `new_order` | `buy`, `sell`, `sell_short` |
| `order_type` | `new_order` | `market`, `limit`, `stop`, `stop_limit`, `trailing_stop`, `trailing_stop_limit`, `market_if_touched`, `limit_if_touched`, `market_to_limit` |
| `quantity`, `limit_price`, `stop_price` | `new_order` | JSON numbers |
| `position_effect` | `new_order` | `open`, `close` |
| `time_in_force` | `new_order` | `day`, `gtc`, `ioc`, `fok`, `gtd`, `at_the_open`, `at_the_close` |
| `expire_time` | `new_order` | String, for `gtd` |
| `take_profit`, `stop_loss`, `stop_loss_limit`, `stop_loss_trailing_offset`, `trailing_offset`, `limit_offset`, `display_quantity`, `min_quantity` | `new_order` | JSON numbers, optional |
| `offset_type` | `new_order` | `price`, `basis_points`, `ticks`; optional |
| `trigger_type` | `new_order` | `default`, `last`, `bid_ask`, `mid`, `mark`, `index`; optional |
| `post_only`, `reduce_only`, `hidden`, `all_or_none` | `new_order` | JSON booleans, optional |
| `contingency_type` | `new_order` | `none`, `oco`, `oto`, `ouo`; optional |
| `linked_intent_id`, `account`, `destination` | `new_order` | Strings, optional |
| `payload_schema` | `new_order` | JSON number, optional |
| `payload` | `new_order` | Base64 string, at most 1024 bytes decoded; optional |
| `request_id` | `cancel`, `replace` | Your id for this request; echoed on `ack` |
| `target_order_id` | `cancel`, `replace` | The `intent_id` being acted on |
| `new_quantity`, `new_limit_price`, `new_stop_price` | `replace` | JSON numbers |
| `new_take_profit`, `new_stop_loss`, `new_trailing_offset` | `replace` | JSON numbers, optional |

**Every enum field above must be one of the exact spellings listed** --
`side`, `order_type`, `position_effect`, `time_in_force`, `offset_type`,
`trigger_type`, and `contingency_type` are rejected,
not defaulted, when the value doesn't match. A `new_order` with any
unrecognized enum value never reaches the core: the relay replies with
`{"type":"ack","request_id":"<intent_id>","accepted":false,"reason":"unrecognized <field>: <value>"}`
on the same connection instead, naming the bad field so a typo like
`"Limit"` (capitalized) is caught immediately instead of silently
executing as a market order.

Outbound, the relay writes the same one-object-per-line format back:

| `type` | Fields |
|---|---|
| `ack` | `request_id`, `accepted` (bool), `reason` |
| `order_update` | `intent_id`, `instrument`, `status` (int, table below), `filled_quantity`, `last_price`, `venue_order_id`, `commission`, `reason`, `timestamp` |
| `server_event` | `code`, `reason` (the event's message text) |

`status` is a **bare integer**, not a name:

| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|
| pending_new | new | partially_filled | filled | cancelled | rejected | pending_cancel | pending_replace |

Terminal states are `filled` (3), `cancelled` (4), and `rejected` (5);
the rest mean the order is still working.

**Most fields are emitted with `omitempty`, so a zero value is absent, not
`0`.** A partial fill of price 0.0 and a `status: 0` (pending_new) update
arrive with that key *missing*. Default every absent numeric to zero when
parsing -- do not treat a missing key as an error.

`ack`'s `accepted` field is the one exception: it is **always present**,
including `accepted: false` -- a rejection ack is not distinguishable from
an accepted one by key presence, only by the boolean value itself.

**No ordering is guaranteed between an `ack` and any `order_update` for
the same order.** The reference relay writes `ack` synchronously as soon
as it parses and validates the inbound line; `order_update` arrives
later and asynchronously, off the shared event bus. Depending on timing,
an `ack` can arrive before, interleaved with, or after the first
`order_update` for the same `intent_id`. A client must not assume `ack`
arrives first -- correlate messages by `intent_id` (`order_update`) or
`request_id` (`ack`), never by arrival order.

A single order, end to end:

```bash
printf '{"type":"new_order","strategy_id":"s1","intent_id":"o1","instrument":"AAPL","side":"buy","order_type":"limit","quantity":10,"limit_price":100.0,"time_in_force":"gtc"}\n' \
  | nc 127.0.0.1 9100
```

The reference relay's instrument allowlist applies to `instrument` --
see `DEPLOYMENT.md`. `TIMING_INSTRUMENTATION.md` uses this same schema to
drive order flow when measuring the kit.

## Shared pipeline event stream

The broker/venue connector and the gate now run in-process, so either can
read core's real-time `PipelineEvent` stream two ways: your broker
connector's `poll_published_events` vtable call (above), or attaching
directly to the shared-memory event bus core writes it into — the same
segment a strategy relay process (which is out-of-process by construction)
must use, since it has no vtable call to poll through. Core remains the
sole writer of the canonical sequenced stream and the sole point that
assigns `sequence` numbers.

**Which one to use:** prefer `poll_published_events` if you're a broker
connector or gate already holding the vtable — it's the same call you're
already making, with no second attach/cursor/gap-handling concern to
build. Prefer attaching `ShmEventBusReader` directly if you're a separate
process with no vtable call to poll through (an ops/monitoring tool, or
anything that isn't itself the gate/connector), or if you specifically
want the event stream independent of your connector's own poll cadence.

### Event bus segment layout

Core creates and owns a segment named by `REAMER_EVENT_BUS_SHM_NAME`
(default `/reamer_event_bus`) at startup. It has no request/response
pairing and no attach/liveness protocol: it's pure broadcast, one writer
(core) and any number of read-only attachers, and core never waits on a
reader.

Core's destructor `shm_unlink`s this segment on clean shutdown, and
segment creation best-effort unlinks any stale segment left by a prior
crashed run before creating its own. A peer attached across a core
restart will therefore see its segment disappear -- this is expected, not
a bug. `tryAttach()` is idempotent for exactly this reason: on losing the
segment, retry `tryAttach()` in your poll loop and it re-attaches cleanly
once core has recreated it.

If core's own segment creation fails (see `REAMER_EVENT_BUS_DEGRADED_IS_FATAL`
in CONFIG.md), core either refuses to start (default) or runs with a
permanently-empty bus -- `since()` always returns an empty batch. A peer
depending on this bus for its audit trail should treat a segment that
never appears as "core is running degraded," not "core hasn't started
yet," if it persists past core's normal startup time.

Offsets are given explicitly, in bytes from the start of the mapping.
**Every field is packed with no alignment padding**, so do not declare a
natural-alignment struct in C, C++, or Rust and map it over the segment —
`latest_sequence` sits at offset **16**, not 24, and a compiler that
inserts four bytes of padding after the four `u32`s will read garbage.
In C/C++ use an explicitly packed struct, or read each field at its
offset. In Rust use `#[repr(C, packed)]`, or read at offsets.

The two ring fields marked *atomic* are the concurrency contract, not a
decoration; the ordering rules are stated under "Memory ordering" below.

```
Header — 24 bytes total

  offset  size  field            notes
  ------  ----  ---------------  ------------------------------------------
       0     4  magic            u32, native byte order, 0x52454256 ("REBV")
       4     4  version          u32, native byte order, currently 1
       8     4  slot_bytes       u32, native byte order, fixed at creation
      12     4  ring_capacity    u32, native byte order, fixed at creation
      16     8  latest_sequence  u64, ATOMIC — sequence of the newest event
                                 written; 0 means nothing written yet

Ring — begins at offset 24; `ring_capacity` slots, stride
       (16 + slot_bytes) bytes each. Slot i begins at
       24 + i * (16 + slot_bytes).

  offset  size        field      notes
  ------  ----------  ---------  ------------------------------------------
      +0           8  sequence   u64, ATOMIC — 0 means "never written";
                                 otherwise the global sequence number of the
                                 event currently occupying this slot
      +8           4  length     u32, bytes of `payload` actually used;
                                 MUST be validated <= slot_bytes before use
     +12           4  reserved   u32, padding, always 0
     +16  slot_bytes  payload    one encoded PipelineEvent (see below);
                                 no frame header, no type discriminator —
                                 this ring only ever carries PipelineEvent
```

Total segment size is `24 + ring_capacity * (16 + slot_bytes)`.

**Byte order.** The header and slot-header integers above are **native**
byte order — they are plain memory in a segment shared between processes
on the same machine, never transmitted. The *payload* is different: it is
big-endian, as specified below. Do not apply one rule to both.

`slot_bytes` comes from `REAMER_EVENT_BUS_SHM_SLOT_BYTES` (default 4096);
`ring_capacity` from `REAMER_EVENT_BUFFER_CAPACITY` (default 4096, an event
count, not a time window). Both are fixed at creation and your process
must know them (matching config, or read them from the header) to compute
slot strides correctly — a mismatch fails attach closed.

### Slot payload encoding — PipelineEvent

Each slot's `payload` holds exactly one `PipelineEvent`, encoded as the
fields below **in this order, with no padding between them**. Read
`length` bytes; a decode that runs past `length` means a corrupt slot, and
you should abandon that slot rather than trust a partial event.

`request_id` (field 14) is empty for every stage except the `cancel_*`/
`replace_*` outcomes, where `intent_id` carries the *target order's* id
(not an id the requesting strategy can match its own request against) and
`request_id` carries the strategy's own request id instead — route
cancel/replace outcomes back to the requester by `request_id`, not
`intent_id`.

Primitive encodings, all **big-endian**:

- `u8` — one byte.
- `u16` — two bytes, big-endian.
- `u64` — eight bytes, big-endian.
- `double` — the IEEE-754 binary64 bit pattern as a big-endian `u64`.
- `string` — a `u16` big-endian byte length, followed by exactly that many
  bytes of UTF-8. No NUL terminator. A zero length is a valid empty string.

Field order:

| # | Field | Type |
|---|---|---|
| 1 | `sequence` | u64 |
| 2 | `timestamp` | string |
| 3 | `category` | u8 |
| 4 | `level` | u8 |
| 5 | `message` | string |
| 6 | `code` | string |
| 7 | `intent_id` | string |
| 8 | `strategy_id` | string |
| 9 | `instrument` | string |
| 10 | `stage` | string |
| 11 | `reason` | string |
| 12 | `last_qty` | double |
| 13 | `last_price` | double |
| 14 | `request_id` | string |

`category` and `level` are small enumerations; a reader **must** reject a
value outside the ranges below rather than casting it blindly, since the
segment is shared memory and a corrupt writer or a future version could
present anything:

- `category`: `0` = `Ops`, `1` = `IntentFlow`. Reject anything above `1`.
- `level`: `0` = `Info`, `1` = `Warning`, `2` = `Error`. Reject anything
  above `2`. `level` is meaningful only when `category` is `Ops`.

This table is the complete definition. Implementing the reader in C++,
Rust, or any other language requires nothing from the Go reference beyond
what is written here.

### Memory ordering

The torn-read protection is a seqlock, and it is only sound if the reader
uses the right ordering. Stated as a contract, so a port to a
weakly-ordered CPU (AArch64, POWER) is correct rather than
accidentally-correct:

- Load `latest_sequence` with **acquire** ordering.
- Load a slot's `sequence` with **acquire** ordering.
- Read a slot as: load `sequence` (acquire) into `s1`; if `s1` is not the
  sequence you expect, stop. Read `length`, validate it against
  `slot_bytes`, and **copy** the payload bytes out. Then load `sequence`
  (acquire) again into `s2`. **If `s2 != s1`, discard what you copied** —
  the writer overwrote the slot while you were reading it.
- Only decode the copy after that second load has confirmed it. Never
  decode in place out of the mapping.

Core writes with matching **release** ordering, so an acquire load that
observes a slot's `sequence` also observes the payload bytes written
before it.

The Go reference reader uses plain loads because Go's memory model and the
x86-64 target make them sufficient there; that is not a licence to use
relaxed loads in a C++ or Rust port. Use acquire.

**Attach**, from any peer process, any time: `shm_open(name, O_RDONLY)`
(retry until core has created it), `mmap` read-only, validate
`magic`/`version`/`slot_bytes`/`ring_capacity`. No PID registration, no
generation counter — core never tracks or waits on readers, so there's
nothing about your process worth registering.

**Reading:** keep your own cursor — the highest `sequence` you've
processed so far, starting at `0`. To read forward: compute
`oldest_retained = latest_sequence > ring_capacity ? latest_sequence - ring_capacity + 1 : 1`;
if `your_cursor + 1 < oldest_retained`, you've fallen behind far enough
that core already overwrote events you never read — a gap, described
below. Otherwise walk `sequence` from
`max(your_cursor + 1, oldest_retained)` up through `latest_sequence`,
reading each slot in turn and advancing your cursor as you go.

**Torn-read protection:** for each slot, read its `sequence` field before
copying the payload and again after; if the two don't match, the writer
raced ahead and overwrote this exact slot while you were mid-copy — discard
and stop (your next read attempt will pick up from wherever the writer has
settled). Core always invalidates a slot's `sequence` to `0` before
overwriting it, so a stale-but-matching read is never silently accepted as
current.

**Gap handling:** if your cursor was more than `ring_capacity` events
behind `latest_sequence`, some events you never read are already gone —
core's writer never blocks or waits on a slow reader, so on wraparound it
unconditionally overwrites the oldest slot. Detect this yourself (per the
`oldest_retained` check above) and resync your cursor to the oldest
sequence still retained rather than trying to recover events that are
already gone.

**Restart handling — the cursor-ahead case:** the sequence is not durable
across core restarts. Core reinitializes the segment header on start and
numbers from 1 again, even when it reattaches a segment left behind by a
previous run, so `latest_sequence` can be **lower** than the cursor you
are holding.

Check for this explicitly, because it is the opposite of the gap case and
the `oldest_retained` arithmetic above does not catch it: if
`latest_sequence < your_cursor`, core restarted. Reset your cursor to
`latest_sequence` and continue. A reader that instead waits for the
sequence to exceed its stale cursor stalls permanently — within the new
run the counter never reaches the old value, so every event is discarded.

Sequence numbers carry no meaning across runs; there is no ordering
relationship between an event numbered 40 before a restart and one
numbered 40 after. If you must distinguish restarts positively rather
than by inference, publish a known startup event from your own gate or
connector and key on that.

### Timestamp behavior under clock adjustment

`PipelineEvent.timestamp`, `ReamerOpsEvent`/`ReamerPipelineEvent`'s
`timestamp` field, and the stderr JSON log's `"timestamp"` field are all
wall-clock (`std::chrono::system_clock`) annotations for display and audit
purposes only — never an ordering key. `system_clock` can step backwards
under an NTP correction or a leap-second smear; core does not clamp
against this, so two timestamps recorded in true chronological order can
compare lexicographically out of order. This is expected, not a bug.

The actual ordering guarantee, per stream, is unaffected by clock
adjustment:

- **Pipeline event bus:** order and gaps are determined solely by the
  core-assigned `sequence` (see Reading/Gap handling above), which is a
  monotonic counter independent of wall-clock time.
- **stderr JSON log:** order is the line order in the stream itself —
  one `fprintf` per event, single writer, no re-sorting — which is
  already chronological regardless of the embedded `timestamp` value.

Never sort or diff audit records by comparing `timestamp` strings; use
`sequence` for the bus, or stream position for the stderr log.

If your relay process publishes events of its own (submitting on its own
existing command channel, same as before), it will see them echoed back on
the bus with a core-assigned `sequence`/`timestamp` — a well-behaved peer
tracks what it already knows it published rather than treating that echo
as new information.

### Strategy relay protocol addition

| Direction | Type byte | Name | Payload |
|---|---|---|---|
| Relay -> core | 4 | Event | `PipelineEvent` |

No `remote_connection_id` on submission — this is your relay process
publishing on its own behalf, not addressed to any one strategy
connection.

## Go, via cgo

Every example above is C-callable, which means a Go implementation is
possible without a wire protocol at all — link `libreamer_server_core`,
declare the two vtables' function pointers as `//export`ed cgo callbacks,
and call `reamer_server_run()` directly from a Go `main()`. This trades
process isolation (a panic in your Go code now takes core down with it,
same as any other in-process extension language) for eliminating the UDS
hop entirely, and costs a cgo boundary crossing on every vtable call.

Three cgo rules make this look harder than it is. None are specific to
this ABI — they apply to any C callback API — but each produces a
confusing error the first time:

1. **An `//export`ed Go function cannot be referenced as `C.name` in the
   file that defines it.** Assigning `gate.check = C.gateCheck` fails with
   `could not determine what C.gateCheck refers to`. You need a C
   trampoline that forwards to it.
2. **cgo drops `const` when it generates prototypes for exported Go
   functions.** Declaring your trampoline's target as
   `const ReamerIntent*` collides with cgo's own non-const declaration
   (`conflicting types for 'gateCheck'`). Declare the extern without
   `const` and cast inside the trampoline.
3. **cgo types C function-pointer struct fields as `*[0]byte`**, so Go
   cannot assign them directly (`cannot use ... as *[0]byte value`). Fill
   the vtable from C instead.

Putting those together, the working shape is a small C preamble:

```go
/*
#include "reamer_server_abi.h"

extern int gateCheck(void* ud, ReamerIntent* i, ReamerVenueState* vs, ReamerAck* a);
extern bool gateIsAvailable(void* ud);

static int check_tramp(void* ud, const ReamerIntent* i,
                       const ReamerVenueState* vs, ReamerAck* a) {
    return gateCheck(ud, (ReamerIntent*)i, (ReamerVenueState*)vs, a);
}
static bool avail_tramp(void* ud) { return gateIsAvailable(ud); }

// Fill the vtable from the C side — Go cannot assign these fields.
static void fill_gate(ReamerGateVtable* g, void* ud) {
    g->user_data = ud;
    g->check = check_tramp;
    g->is_available = avail_tramp;
}
*/
import "C"
```

with `//export gateCheck` / `//export gateIsAvailable` on the Go
functions, and `C.fill_gate(&gate, nil)` in place of field assignment.
The broker connector follows the same pattern across its nine methods.

A Go callback must not let a panic escape across the boundary — recover
inside each exported function. The same applies to any language whose
unwinding is undefined across an `extern "C"` frame (Rust callbacks
should be `extern "C"` and wrap their bodies in `catch_unwind`).

## Configuration

`strategy_input_mode` is the one runtime choice left in `CONFIG.md` —
Reamer Server always calls the gate and broker connector vtables you pass
into `reamer_server_run()` directly (there is no compiled-in reference
implementation of either, and no endpoint to configure for them), while
strategy input can still run the built-in local `Sequencer` instead of a
remote relay.

An unset `strategy_relay_endpoint` while `strategy_input_mode=remote` is a
startup error — Reamer Server refuses to start rather than guess. There is
nothing to configure for the gate or broker connector at all: passing
`NULL` for either vtable to `reamer_server_run()` is itself the error,
returned as `REAMER_SERVER_ERROR` before the loop starts.

No gate, connector, or strategy relay is compiled into core. Reference
source for all three ships in the kit's `reference/` tree as a working
starting point (see [Start from working code](/docs/connectors.html));
it is reference material, not a supported surface. This document and `reamer_server_abi.h` are the
complete contract: every vtable method, every struct field, both strategy
wire protocols, and the event bus layout are specified here byte for
byte. A gate and a broker connector that reach a first fill are on the
order of a hundred lines against that contract.

One design note carries over regardless of language: your connector and
relay hand-parse untrusted bytes off a socket — a venue feed, a
strategy's own connection — so treat that parsing as a trust boundary and
prefer a memory-safe language or a hardened parser for it, even though
the core-facing side is now an ordinary in-process function call.
