---
title: Reamer Server configuration
description: Every Reamer Server config field, its default and environment override, plus how the file is layered and validated.
group: integration
order: 7
product: server
source: CONFIG.md
---

Everything `reamer-server`'s own process needs to start (socket paths, the
three wire-protocol endpoints, and the dashboard/monitoring knobs) is
one `ServerConfig`, loaded through three layers:

```
struct defaults  →  optional JSON config file  →  env vars (env wins)  →  validate
```

- **Struct defaults** — hardcoded in `ServerConfig`, listed in the table
  below.
- **Config file** — optional. Path is `REAMER_SERVER_CONFIG_PATH` if set,
  else the per-machine default: `~/.config/reamer/reamer-server-config.json`
  on Linux/macOS, `%APPDATA%\Reamer\reamer-server-config.json` on Windows. A
  missing file is fine — defaults pass through unchanged. (Note the one
  deliberate asymmetry: `bin/reamer-config-check --config <path>` *rejects*
  a named path that does not exist, exit 1. A path you typed is a stated
  intent, so an absent one is a typo; only the unnamed default path above is
  allowed to be missing.) A present key
  fully replaces that field (not a deep merge); an absent key leaves
  whatever the defaults already had. Malformed JSON or a wrong-typed field
  throws at startup, naming the file path.
- **Env vars** — always take precedence over both the defaults and the
  file. An invalid value on a numeric env var is ignored with a stderr
  warning (keeps whatever the file/default already set) rather than
  crashing — the file layer is strict, the env layer stays lenient to match
  the pre-existing env-only deployment behavior.
- **Validation** — runs last, after all three layers are applied; throws
  `std::runtime_error` (caught by `reamer_server_run()`, printed to stderr
  via `logStderr`, `REAMER_SERVER_ERROR` returned) if a required field is
  still unset.

## No reload — restart is the only way to pick up a config change

Config is loaded once, at `reamer_server_run()` startup, and never re-read.
There is no reload path and no `SIGHUP` handler, deliberately. This is the
same restart-safety story as everywhere else in this product — a process
restart is always safe and never loses state (state is re-seeded from the
broker connector's `pull_state()`, not held durably in core; see
`DEPLOYMENT.md`'s "Restart safety" note) — so a config change is one more
reason to restart, not a special case needing a live-reload path. Adding
`SIGHUP` handling would mean partially re-initializing a running process
(new socket bindings, a new monitor listener, a new shm segment name) while
it may be holding open tracked orders, which is a materially riskier
operation than the clean stop/start `reamer_server_run()` already supports
via `shutdown_drain_seconds`. That extra risk was judged not worth it for a
config surface this small.

**Validate before you restart.** Because a bad edit is otherwise only
discovered when the new process fails to start, run the standalone
`reamer-config-check` binary (shipped precompiled in the kit's
`bin/`) against the edited file first:

```
bin/reamer-config-check --config /path/to/reamer-server-config.json
```

It runs the exact same defaults → file → env → validate precedence
described above — without starting the server, opening the socket, or
touching the event bus — and exits 0 after echoing **every** resolved
field, or exits 1 with the same file-path-qualified error
`reamer_server_run()` itself would produce. This is especially useful
mid-incident, when a config is being hand-edited under time pressure and a
syntax typo turning into a failed restart is the last thing you want to
discover live.

**Unrecognized keys are rejected, not ignored.** Any JSON key not in the
table below fails the load — in `reamer-config-check` and in
`reamer_server_run()` alike, so the two can never disagree about what a
valid config is. The error names each offending key and suggests the
nearest known one:

```text
unrecognized key in config file at /etc/reamer/server.json:
  "max_tracked_ordres" -- did you mean "max_tracked_orders"?
```

This is deliberate. A misspelled key that loads clean runs the real field
at its default: `max_tracked_ordres: 50000` silently caps order tracking
at 5,000, and nothing in the output says so. Rejecting the file is the
only behavior that makes the typo visible before the trading session.

The echoed field list is the second half of that guarantee — read it to
confirm the value you edited actually took effect, rather than trusting
that a clean exit means what you intended. `monitor_auth_token` is
reported as `(set)` / `(not set)` rather than echoed, since this output
is routinely pasted into support tickets.

## Fields

| JSON key | Env var | Type | Default | Required |
|---|---|---|---|---|
| `socket_path` | `REAMER_SOCKET_PATH` | string | `/tmp/reamer-server.sock` | no |
| `monitor_host` | `REAMER_MONITOR_HOST` | string | `127.0.0.1` | no |
| `monitor_port` | `REAMER_MONITOR_PORT` | uint16 | `8090` | no |
| `monitor_auth_token` | `REAMER_MONITOR_AUTH_TOKEN` | string | `""` (disabled) | no |
| `monitor_allow_query_token` | `REAMER_MONITOR_ALLOW_QUERY_TOKEN` | bool | `false` | no |
| `max_tracked_orders` | `REAMER_MAX_TRACKED_ORDERS` | size_t | `5000` | no |
| `strategy_input_mode` | `REAMER_STRATEGY_INPUT_MODE` | `"local"` \| `"remote"` | `local` | no |
| `strategy_relay_endpoint` | `REAMER_STRATEGY_RELAY_ENDPOINT` | string | `""` | **yes, iff `strategy_input_mode == "remote"`** |
| `uds_initial_backoff_seconds` | `REAMER_UDS_INITIAL_BACKOFF_SECONDS` | int | `1` | no |
| `event_bus_shm_name` | `REAMER_EVENT_BUS_SHM_NAME` | string (POSIX shm segment name) | `/reamer_event_bus` | no |
| `event_buffer_capacity` | `REAMER_EVENT_BUFFER_CAPACITY` | size_t | `4096` | no |
| `event_bus_shm_slot_bytes` | `REAMER_EVENT_BUS_SHM_SLOT_BYTES` | uint32 | `4096` | no |
| `event_bus_degraded_is_fatal` | `REAMER_EVENT_BUS_DEGRADED_IS_FATAL` | bool | `true` | no |
| `license_warning_days` | `REAMER_LICENSE_WARNING_DAYS` | comma-separated ints | `30,14,7,1` | no |
| `license_expired_warning_interval_seconds` | `REAMER_LICENSE_EXPIRED_WARNING_INTERVAL_SECONDS` | int | `3600` | no |
| `shutdown_drain_seconds` | `REAMER_SHUTDOWN_DRAIN_SECONDS` | int | `5` | no |

`socket_path` is only used when `strategy_input_mode == "local"` (it's the
`Sequencer`'s own listen socket for direct strategy connections); it's
ignored in `remote` mode, where `strategy_relay_endpoint` is used instead.
Startup validation rejects any `strategy_input_mode` other than `local` or
`remote` outright — a misspelled or out-of-enum value never falls back to
`local` silently.

`monitor_auth_token`, once set, gates every HTTP monitoring endpoint route
(`/health` and `/metrics`) behind a shared secret: a request must
present it as an `Authorization: Bearer <token>` header, or it gets
`401 Unauthorized`. Left empty (the default), the endpoint is
unauthenticated, matching its original behavior. Startup validation refuses
to run at all if `monitor_host` is set to anything other than a loopback
address (`127.0.0.1`, `localhost`, `::1`) while `monitor_auth_token` is
still empty — an unauthenticated monitoring endpoint may never be exposed
off-host.

`monitor_allow_query_token`, when set `true`, additionally accepts the token
as a `?token=<token>` query parameter. This is **off by default and unsafe
to enable**: query strings leak into proxy access logs, browser history, and
`Referer` headers, so an accepted query token is effectively a credential
that ends up outside your control. Enable it only for a client that cannot
set request headers (e.g. a browser `EventSource`) — `curl`, `requests`, and
every other documented client (`MONITORING.md`) can send the header instead
and should.

`/metrics`, on the same `monitor_host`/`monitor_port`, is a scrape-friendly
Prometheus text-format endpoint alongside `/health`. Both are pull-only,
point-in-time routes — there is no push-based
event stream over HTTP; `ShmEventBus` is the outlet for the live ops/
intent-flow event feed (`MONITORING.md`). `/metrics` returns counters/
gauges: order submitted/accepted/rejected counts (and rejections broken out
by gate/venue-disconnect reason), tracked-order occupancy against
`max_tracked_orders`, shm event bus state (`latest_sequence`, ring
capacity), broker/relay connection state and seconds since the relay's
last successful frame, and process uptime/loop-iteration count/poll
latency. Point an existing Prometheus (or any scrape-compatible) deployment
at it directly.

**Secrets:** prefer `REAMER_MONITOR_AUTH_TOKEN` over writing
`monitor_auth_token` into the JSON config file when a secrets manager
(Vault, AWS Secrets Manager, k8s Secrets, etc.) can inject it as an env
var at process start — env vars already win over the file per this
file's layering above, so this is a deployment-pattern preference, not a
new mechanism. If a plaintext file is used to store the token anywhere
(the config file itself, or an env-file sourced into the process), set
`chmod 600` on it so only the process owner can read it.

There is no config surface for the gate or broker connector at all — both
are linked directly into the process as a `ReamerGateVtable`/
`ReamerBrokerConnectorVtable` passed straight into `reamer_server_run()`,
not addressed by endpoint or bounded by a timeout. A `NULL` vtable is
itself the startup error (`REAMER_SERVER_ERROR`), checked before this
`ServerConfig` load even runs. See `EXTENSION_PROTOCOL.md`'s "Gate
vtable" and "Broker connector vtable" sections for the full contract.

`uds_initial_backoff_seconds` still applies to `strategy_relay_endpoint`
— the one remaining out-of-process, UDS-based extension point (see
"Strategy relay protocol" in `EXTENSION_PROTOCOL.md`).

`event_bus_shm_name`/`event_buffer_capacity`/`event_bus_shm_slot_bytes`
configure the real-time `PipelineEvent` broadcast bus (`ShmEventBus`, see
`EXTENSION_PROTOCOL.md`'s "Shared pipeline event stream" section) — a
separate shared-memory segment Reamer Server creates and owns.
`event_buffer_capacity` is a fixed event count, not a time window: it caps
how far back a reader (gate process, broker/venue connector, strategy relay
— each attaching read-only, directly, in its own process) can catch up via
`since()` before hitting a gap; older events are simply gone, overwritten
oldest-first once the ring fills. `event_bus_shm_slot_bytes` bounds one
encoded event's size.

`event_bus_degraded_is_fatal` controls what happens if the event bus
segment itself fails to create (`/dev/shm` unavailable, a restrictive
container, a tightened `RLIMIT`). Every failure
is logged with `errno`, and a startup `event_bus.live`/`event_bus.degraded`
ops event is always emitted either way. With the default `true`, a
degraded bus refuses to start the server (`REAMER_SERVER_ERROR`) rather
than silently running with no audit trail — set `false` only for a
deployment that deliberately doesn't need this bus and should keep running
without it.

`max_tracked_orders` bounds the in-memory map Reamer Server uses to look up
an order by its id (for cancel/replace operations) and attach broker updates
to the correct order. Exceeding this cap evicts the oldest tracked order
(insertion-order FIFO, the same pattern as the event buffer above) and emits
an `order_tracking.evicted` warning via `logStderr`, naming the evicted
order id and the active cap. Once evicted, that order's future updates
(fills, cancels, replaces, broker state changes) are no longer tracked or
reported, and a cancel/replace request against its id falls into the
"unknown order" rejection path. A deployment running more than
`max_tracked_orders` concurrently open orders should raise the value via the
`max_tracked_orders` field (JSON) or `REAMER_MAX_TRACKED_ORDERS` env var.

`license_warning_days` sets the day-thresholds before `expires_at` at which
the run loop escalates a `license.expiring` warning (through the same
`logStderr`/`monitor->publishEvent()`/pipeline-event sinks used elsewhere).
Each threshold fires once, the first recheck tick that crosses it, not on
every recheck after. A process started inside several thresholds fires only
the smallest — one started 10 days from expiry with the default schedule
fires once at the 14-day threshold, not again until 7. The list may be in
any order. Once the
license has actually expired, this schedule stops applying and
`license_expired_warning_interval_seconds` takes over instead: it paces how
often the run loop re-emits a `license.expired` error event for an
already-expired (or tampered/invalid-machine/missing) activation. Expiry
never halts a running process — only a *new* `reamer_server_run()` refuses
to start on a non-`Valid` license; a process already running keeps running
indefinitely, just with these recurring warnings.

`shutdown_drain_seconds` bounds the graceful-shutdown drain phase:
on SIGTERM/SIGINT, the run loop stops accepting new `StrategyRequest`s and
keeps ticking the broker connector and monitoring endpoint to drain
whatever the connector already has in flight (fills, acks, events) for up
to this many seconds, or until a pass finds nothing left to drain,
whichever comes first — then publishes one final reconciliation snapshot
before exiting. `0` disables draining entirely (an immediate exit on
signal). No orders are ever cancelled during this drain; see
EXTENSION_PROTOCOL.md's "Shutdown contract" section for the full contract.

## Diagnostic environment variables

These are **not** config-file fields and have no JSON key — setting them in
the config file is an unrecognized-key error. They are environment-only
diagnostics, listed here because this document is the authoritative config
surface and an operator should not have to find them in a benchmark
appendix.

| Env var | Effect |
|---|---|
| `REAMER_TIMING_DEBUG` | Set to any non-empty value to emit latency traces to **stderr**. Three line kinds, and these are all of them: `[TIMING] input->poll(1): <ms> batch_size=<n>` once per non-empty poll; `[TIMING] dispatch: total=<ms> [OUTLIER_DETECTED]` for any single dispatch exceeding 200 ms; and `[TIMING_DETAIL] intent=<id> frame_decode=<ms>` per decoded intent, on both the `local` and `remote` strategy-input paths. There is no per-order `pull_state`/`check`/ack breakdown — see `TIMING_INSTRUMENTATION.md`. An idle server emits nothing at all. |

**Do not leave `REAMER_TIMING_DEBUG` set in production.** It writes an
unbuffered line to stderr per polled batch; under load that is a
synchronous write on the hot path and will itself distort the latencies it
reports. Enable it to diagnose a specific outlier, then unset it and
restart. It changes no trading behavior — it only adds output.

There is no config-file equivalent, deliberately: this is a per-incident
switch flipped on a single process, not deployment state.

## Example config file

```json
{
  "strategy_input_mode": "remote",
  "strategy_relay_endpoint": "/var/run/reamer/strategy-relay.sock",
  "event_bus_shm_name": "/reamer_event_bus",
  "monitor_host": "0.0.0.0",
  "monitor_port": 8090,
  "monitor_auth_token": "replace-with-a-real-secret",
  "uds_initial_backoff_seconds": 2
}
```

Any field left out of the file keeps its struct default (or whatever an env
var sets on top of it).
