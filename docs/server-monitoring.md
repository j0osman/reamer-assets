---
title: Monitoring Reamer Server
description: The 15 Prometheus metrics, /health, the event stream, structured logs, licence alerts and a starting alert table for Reamer Server.
group: operations
order: 3
product: server
source: MONITORING.md
---

`reamer-server` publishes every order-flow and operational event it sees to
one shared pipeline event stream inside core (`EXTENSION_PROTOCOL.md`'s
"Shared pipeline event stream" section is the authoritative field-level
spec — this file only covers the one delivery path documented below). Under
Reamer Server's closed-source distribution model, a customer never gets
Reamer Server's source or a client SDK to build against, so storage is
entirely the customer's own decision: point whatever you already run — a
cloud database, Kafka, Postgres, DuckDB, flat files, or nothing durable at
all — at the stream below and archive it however you want. Reamer Server's
job stops at reliably supplying the stream, not picking where it ends up.

## The live event stream — `ShmEventBus`, not an HTTP route

An earlier version of `HttpMonitoringEndpoint` served a Server-Sent Events
stream at `/events`. That route is gone: it published the same
`PipelineEvent` records a second time over a slower transport, and forced
core to serialize an order/position dump on the hot path to feed it. The
one outlet for the live ops/intent-flow event feed is now `ShmEventBus` —
a shared-memory broadcast ring core writes to and any number of readers,
in any number of processes, attach to independently.
`EXTENSION_PROTOCOL.md`'s "Event bus segment layout" section is the
wire/layout-level spec a peer process implements to attach directly; the
same construct is what `bin/reamer-diag-bundle` below uses to pull recent
events into a bundle without a round trip through core.

A reader's `since(after_sequence)` call returns every retained
`PipelineEvent` newer than `after_sequence`, each one either an ops record
(`level`: `info` \| `warning` \| `error`, `code`, `message` — connection
state changes, gate/broker connectivity changes, fatal errors) or an
intent-flow record (`intent_id`, `strategy_id`, `instrument`, `stage`,
`reason`, `last_qty`, `last_price` — the same `stage` vocabulary
`accepted`/`rejected`/`filled`/`cancel_accepted`/`order_<status>`/etc. as
`PipelineEvent.stage` in `EXTENSION_PROTOCOL.md`). The ring is a fixed
event count (`event_buffer_capacity`, `CONFIG.md`), not a time window — a
reader that falls more than that many events behind sees `gap: true` on its
next `since()` call, meaning some events in between were already
overwritten and can't be recovered. There is no HTTP-facing replacement for
this: an external tool that isn't a linked-in peer process reads it via
`bin/reamer-diag-bundle`'s point-in-time snapshot below, not a live
subscription.

## `/metrics` — the complete exposed surface

`GET http://<monitor_host>:<monitor_port>/metrics` returns Prometheus text
format. Same auth rule as `/health` — if `monitor_auth_token` is set, the
request must present it or gets `401 Unauthorized`.

**Core exposes exactly these 15 metrics.** This is the whole set; there is
no other name to discover by scraping. Every one carries a `# HELP` and
`# TYPE` line in the response itself, and the types below are those lines
verbatim.

| Metric | Type | Meaning |
|---|---|---|
| `reamer_uptime_seconds` | counter | Seconds since the process started. |
| `reamer_loop_iterations_total` | counter | Main-loop iterations since start. Advancing means the loop is alive — but see the stall section below: you cannot read this while it is stuck. |
| `reamer_poll_latency_ms` | gauge | Duration of the most recent strategy-input `poll()`, in milliseconds. Most recent sample only, not an average. |
| `reamer_orders_submitted_total` | counter | New-order requests submitted to the gate. |
| `reamer_orders_accepted_total` | counter | New orders accepted by the gate and sent to the broker. |
| `reamer_orders_rejected_total` | counter | New orders rejected, by the gate or by a disconnected venue. |
| `reamer_order_rejections_by_reason_total` | counter | The same rejections, broken out by a `reason` label. |
| `reamer_tracked_orders` | gauge | Orders currently held in core's in-memory order table, terminal ones included until evicted. Not the same as `/health`'s `open_order_count`. |
| `reamer_max_tracked_orders` | gauge | Configured capacity of that table (`max_tracked_orders`). The denominator for a saturation alert. |
| `reamer_broker_connected` | gauge | 1 if your broker connector reports connected, else 0. This is the signal that gates every gate check. |
| `reamer_relay_connected` | gauge | 1 if the remote strategy relay is connected, else 0. **Always 0 in local-socket mode** — do not alert on it there. |
| `reamer_relay_seconds_since_last_frame` | gauge | Seconds since the last successful frame from the relay. **0 in local-socket mode.** |
| `reamer_event_bus_live` | gauge | 1 if the shm event-bus segment is live, 0 if degraded. |
| `reamer_event_bus_latest_sequence` | counter | Latest sequence number published to the event bus. |
| `reamer_event_bus_ring_capacity` | gauge | Configured ring capacity (`event_buffer_capacity`). |

**Pre-attach strategy requests are silently dropped, not queued.** In
remote-relay mode, any strategy request submitted before
`reamer_relay_connected` reads `1` is discarded on the spot — the
underlying connection has no send buffer to hold it in, so there is no
retry, no queue to inspect, and no log line marking the drop.
`reamer_orders_submitted_total` simply stays at 0 until the relay
connects. This is expected behavior during relay startup or a
reconnect, not a fault: check `reamer_relay_connected` before concluding
"the relay never connected" from a flat `reamer_orders_submitted_total`.
Once the gauge reads `1`, submissions reach core normally.

Recommended thresholds for these are in "Alert table" below.

## `/health` — liveness/readiness probe

`GET http://<monitor_host>:<monitor_port>/health` reports broker
connectivity for orchestrator liveness/readiness checks (systemd,
Kubernetes, Docker). Same auth rule as `/metrics` — if `monitor_auth_token`
is set, the request must present it or gets `401 Unauthorized`. One-shot
JSON response, not a stream:

- Before the process has completed a main-loop iteration: `200 OK`,
  `{"status":"starting","broker_connected":false}`.
- Once looping and the broker connector reports connected: `200 OK`,
  `{"status":"ok","broker_connected":true}`.
- Once looping and the broker connector reports disconnected:
  `503 Service Unavailable`, `{"status":"degraded","broker_connected":false}`.

**`open_order_count` in the shutdown log line and `reamer_tracked_orders`
on `/metrics` count different things, and they routinely disagree.**
`open_order_count` is orders still working *at the venue* — not yet
filled, cancelled or rejected. `reamer_tracked_orders` is entries in
core's in-memory order table, which retains orders that have reached a
terminal state until they are evicted. A healthy process with
`open_order_count: 0` and `reamer_tracked_orders` in the hundreds is
normal: everything filled, nothing still working. Alert on the two
separately — saturation is a `reamer_tracked_orders` question, working
exposure is an `open_order_count` question.

Broker connectivity is the one liveness signal here — it's also the
signal that gates every gate check (`EXTENSION_PROTOCOL.md`'s "Gate
protocol" section), so `/health` reports exactly what determines whether
core is currently accepting order flow at all.

### Both HTTP routes are served by the main loop — know what that means

`/health` and `/metrics` are serviced from inside the main loop itself,
on the same thread, as a non-blocking poll once per iteration. There is no
separate HTTP thread. Answering a request **is** a loop iteration.

The consequence matters when you design alerts: **if the main loop stops
iterating, both routes stop answering.** They do not return an error
saying the loop is stuck — the connection hangs until your client's
timeout. `reamer_loop_iterations_total` on `/metrics` is the counter that
would tell you the loop is advancing, and it is precisely the value you
cannot read while it is stuck.

**Do not** write a stall alert as "scrape `/metrics` and alert when
`reamer_loop_iterations_total` stops advancing." That rule can never fire:
a stalled loop serves no scrape, so the counter never arrives to be seen
as flat, and the rule reports stale-target rather than the stall.

**Do this instead.** Treat an unreachable or timing-out `/health` as
loop-stalled-or-worse, and let ordinary process supervision handle it:

1. Probe `/health` on a fixed interval with an **explicit client
   timeout** — a hang must fail the probe, not block the prober. In
   Kubernetes, `livenessProbe` with `timeoutSeconds` set does this.
2. Alert and restart on **2 consecutive** failures, per the alert table
   below.
3. **Restart only if the license is `Valid`.** A stalled process that is
   restarted on an expired license does not come back — see `LICENSING.md`
   and the restart-freeze rule.

**A stalled process does not exit on its own.** Core runs no watchdog
timer, applies no deadline to a loop iteration, and never self-terminates
on a stall. The process stays alive, keeps its monitor port bound, and
holds every open order exactly as it stands at the venue. It will not free
the port or release the seat for you — recovery is your supervisor's job,
which is why the probe above must have a client-side timeout and an
escalation that actually restarts the process rather than waiting for it
to die. A stalled instance also still counts against its license seat
until it is stopped.

What this costs you is resolution, not detection: a hung probe does not
distinguish "loop stalled" from "process killed" or "host unreachable."
The operator response is the same for all three, which is why core does
not ship a separate out-of-band liveness channel to tell them apart. The
kit as shipped does not provide that distinction.

For a signal that does not depend on core's HTTP path at all, watch the
venue side: fills and acknowledgements stop arriving while the loop is
stalled, and your broker connector sees that directly.

## Reading the event stream live

A peer process (your gate, your broker connector, your OMS, or any other
process running as the same OS user) attaches a reader to the
shared-memory event bus and polls it on whatever cadence it wants — no
HTTP round trip, no client library.

**No reader ships in this kit.** The segment layout is fully specified in
`EXTENSION_PROTOCOL.md`'s "Event bus segment layout" — magic, version,
`slot_bytes`, `ring_capacity`, `latest_sequence`, then the ring slots —
and a reader is a short piece of work against that spec in any language
with shared-memory access. The attach signature a reader needs:

```go
// NewReader(segmentName, ringCapacity, slotBytes) — the last two must
// match core's config, or attach fails closed.
// segmentName is a POSIX shm name (core creates it with shm_open), not a
// filesystem path. The reader resolves it the way Linux backs it, so
// "/reamer_event_bus" and "/dev/shm/reamer_event_bus" both attach; a
// reimplementation in another language must call shm_open, or open
// /dev/shm/<name> — opening the POSIX name as a path yields ENOENT.
reader := shmbus.NewReader("/reamer_event_bus", 4096, 4096)
var cursor uint64
for running {
    if !reader.IsAttached() {
        // Idempotent: also the correct response to core restarting.
        if attached, err := reader.TryAttach(); !attached || err != nil {
            continue
        }
    }
    events, gap, err := reader.Since(cursor)
    if err != nil {
        continue
    }
    if gap {
        // You fell behind and the ring wrapped past your cursor —
        // events were missed. Record it; do not treat it as normal.
    }
    for _, evt := range events {
        cursor = evt.Sequence
        // evt.Category is Ops or IntentFlow — see the section above.
    }
}
```

**If you are integrating in C++ or Rust, you implement the reader
yourself.** No C++ reader header ships — the core is delivered as a
precompiled archive with only `reamer_server_abi.h` exposed, and the
event-bus reader is not part of the ABI. This is a real piece of work to
scope, but it is bounded and fully specified: `EXTENSION_PROTOCOL.md`'s
"Event bus segment layout" gives the complete binary layout (header
magic/version/`slot_bytes`/`ring_capacity`/`latest_sequence`, then the
ring slots), and the attach, gap-detection, and core-restart semantics
are documented alongside it. Note the one easily-missed detail that
section states explicitly: the segment header and slot headers are
native byte order, while the `PipelineEvent` payload inside each slot is
big-endian.

Two alternatives if you would rather not write one:

- **If your process is the gate or the broker connector**, use the
  `poll_published_events` vtable call instead — you already hold the
  vtable, and it carries the same `PipelineEvent` stream with no second
  attach or cursor to manage. `EXTENSION_PROTOCOL.md` recommends this
  path for in-process peers regardless of language.
- **For point-in-time capture**, use `bin/reamer-diag-bundle` (below),
  which requires no integration at all.

Point your own durable sink at the loop above — write straight to your
database, push onto a queue, append to a file — however your own tooling
already handles the rest of your infrastructure. For a one-off,
point-in-time capture instead of a running subscriber, use
`bin/reamer-diag-bundle` below.

## Diagnostic bundle — `bin/reamer-diag-bundle`

`reamer-diag-bundle --config <path> [--out <path>]` is the single command
that answers "what do I send Reamer Labs support" (see the vendor
questionnaire pack's support-model section for response times and the
contact address: the [vendor questionnaire
pack](/docs/vendor-questionnaire.html), which also ships in the kit). It ships as a
standalone binary next to `bin/reamer-license` and
`bin/reamer-config-check` — see `DEPLOYMENT.md`.

Run against the affected instance's config file:

```bash
bin/reamer-diag-bundle --config /etc/reamer-server/config.json --out incident.json
```

It reads that config to find the same `monitor_host`/`monitor_port` and
`event_bus_shm_name` the running process uses, then in one pass:

- scrapes `/health` and `/metrics` from the monitoring endpoint (using
  `monitor_auth_token` from the config if set);
- attaches a `ShmEventBusReader` to the event bus and pulls every ops and
  intent-flow record still retained (`since(0)`), noting `gap: true` if the
  reader attached late enough that some history was already overwritten;
- writes one JSON file combining both, stamped with the Reamer Server
  version and a generation timestamp.

A scrape or shm-attach failure is recorded in the output (`"ok": false`,
`"attached": false`) rather than aborting the tool — "the process is
unreachable" is itself useful information during an incident, so the tool
still produces a bundle in that case.

**Safe to send:** `monitor_auth_token` is never written to the output file
— the config section only reports whether a token is set
(`monitor_auth_token_set: true/false`), never the token itself. The
resulting file is intended to be attached directly to a support email.

## stderr — process-level structured log lines

Everything Reamer Server itself writes to stderr (startup failures,
connector diagnostics also published to `ShmEventBus`, the fatal top-level
error if the process dies) is one JSON object per line, not free text:

```
{"timestamp":"2026-08-10T16:23:26.016096Z","level":"error","code":"license.missing","message":"..."}
```

Same field vocabulary as an `OpsEvent`/`PipelineEvent` (`level`: `info` \|
`warning` \| `error`, `code`, `message`), plus a `timestamp` — no separate
log schema to learn. `reamer-server` does not rotate or manage this
output as a file — that's your process supervisor's job, consistent with
core staying minimal and the operator owning surrounding infrastructure:

- **systemd**: `StandardError=journal` (the default) sends it straight to
  `journald`; read it with `journalctl -u reamer-server -o cat`, which
  prints each JSON line as-is for downstream parsing.
- **Docker/Kubernetes**: the container runtime's own log driver
  (`json-file`, `journald`, `fluentd`, etc.) captures stderr already —
  point it at your log-aggregation pipeline (ELK, Splunk, Datadog) the
  same way you would for any other JSON-logging process.

## Alert on the license events

Two event codes reach both sinks — stderr and the event stream — and both
warrant an alert:

| Code | Level | Meaning | Action |
|------|-------|---------|--------|
| `license.expiring` | `warning` | `expires_at` is within a `license_warning_days` threshold. Fires once per threshold crossing (default `30,14,7,1`). | Renew now. |
| `license.expired` | `error` | The license is expired, tampered, invalid-machine, or missing. Re-emitted every `license_expired_warning_interval_seconds` (default 3600). | Renew before any restart. |

`license.expired` does **not** mean the process stopped — expiry never
halts a running process. It means the process has become unrestartable: a
new `reamer_server_run()` is refused on a non-`Valid` license, with no
grace period. Page on it and freeze restarts, drains, redeploys, and
reschedules until the seat is renewed. See `DEPLOYMENT.md`'s "Restart
safety" for the operational rule and `CONFIG.md` for the two knobs.

**During a renewal,** a single `license.expired` is expected. Renewing on the
same machine means `reamer-license deactivate` then `activate` with the new
key, and a recheck (every 30 s) that lands between the two sees no license.
The next recheck after `activate` finds the new key valid and the errors
stop; there is no "restored" event, so resolve that page by hand once
`bin/reamer-license status` exits 0. See `LICENSING.md`'s "Renew on the
same machine".

## Alert table

Starting thresholds, not tuned values. They are chosen to fire on a real
fault and stay quiet through ordinary operation; tune the durations
against your own venue's reconnect behaviour and your strategy's cadence
once you have a week of baseline. Probe interval is **5 s** throughout
unless stated.

| Signal | Threshold | Severity | Action |
|---|---|---|---|
| `/health` unreachable or timing out | 2 consecutive failures (client timeout set, ≤3 s) | page | Restart — **only if `bin/reamer-license status` exits 0.** A stalled process never self-exits; see the stall section above. |
| `/health` returns `503` / `"degraded"` | > 30 s continuous | page | Venue session is down. Every intent is being rejected `reason="venue disconnected"`. Escalate to whoever owns the connector. |
| `reamer_broker_connected == 0` | > 30 s | page | Same condition from the metrics side. Use one or the other, not both, or you page twice. |
| `reamer_relay_seconds_since_last_frame > 60` | 1 scrape | page | Strategy relay is stale — orders have stopped arriving. **Only meaningful when `strategy_input_mode: remote`;** this reads 0 in local mode, so scope the rule to remote deployments. |
| `reamer_relay_connected == 0` | > 30 s, remote mode only | page | Relay link dropped. Same scoping caveat. |
| `reamer_tracked_orders / reamer_max_tracked_orders > 0.8` | 1 scrape | warn | Eviction is approaching. Raise `max_tracked_orders` (`CONFIG.md`) or shorten strategy order lifetime. See `CAPACITY_AND_LIMITS.md`. |
| `reamer_tracked_orders / reamer_max_tracked_orders > 0.95` | 1 scrape | page | Eviction imminent; tracked state for live orders is about to be dropped. |
| `rate(reamer_orders_rejected_total) / rate(reamer_orders_submitted_total) > 0.1` | over 5 min, ≥20 submissions | warn | Break out by `reamer_order_rejections_by_reason_total`. A shift in the dominant `reason` label is the signal, not the absolute rate — a gate correctly refusing a misbehaving strategy looks identical to a fault at this level. |
| `reamer_poll_latency_ms` | > 5× its own 1 h median, sustained 5 min | warn | Poll is slowing. Check CPU contention first, then `TIMING_INSTRUMENTATION.md`. There is no absolute threshold here that is meaningful across deployments. |
| `reamer_event_bus_live == 0` | 1 scrape | warn | Event bus degraded; core keeps trading, but every reader has gone blind. Page instead if the event stream feeds an audit or compliance sink. |
| `increase(reamer_loop_iterations_total)` flat | — | **do not use** | Cannot fire. A stalled loop serves no scrape; use the `/health` rule above. See the stall section. |
| `license.expiring` | on emission | warn | Renew now. |
| `license.expired` | on emission | page | **Restart freeze.** The process is still trading but will not come back if stopped. |

**Two rules that are not thresholds.** First, gate every automated restart
on license validity — an expired seat turns a restart into an outage.
Second, `reamer_uptime_seconds` decreasing is a restart you did not order;
alert on it if nothing in your deploy pipeline should have caused one.
