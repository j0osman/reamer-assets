---
title: Running Reamer Server
description: What a Reamer Server restart keeps and loses, how event sequence numbers behave across one, how licence expiry affects startup, and the monitoring surface.
group: operations
order: 1
product: server
source: DEPLOYMENT.md, MONITORING.md, CONFIG.md
---

The core holds no durable state of its own. On start, order and position
state is re-seeded entirely from your broker connector's `pull_state()`,
so a process restart loses nothing the core was responsible for keeping.

## What a restart does

**Rebuilt from the venue:**

- **Order and position state.** Re-seeded through `pull_state()` on
  start.
- **Fills that landed during downtime.** They arrive in the next
  `pull_state()`, so report current state from it rather than a delta.
- **Open orders.** The core never cancels one during drain, and calls
  `pull_state()` once more before exit.

**Not carried across:**

- **In-flight intents at the moment of a kill.** Anything not yet at the
  venue is gone; anything already there is the venue's.
- **The event-bus sequence.** Numbering restarts at 0, even when the core
  reattaches a segment a previous run left behind.

The core keeps no second copy of position state to diverge from the
venue's, so drift in the usual sense cannot build up. `pull_state()` is
yours, and the core adopts whatever it reports as fact.

## Sequence numbers across a restart

A sequence that goes backwards means a restart. Treat a decrease as a
stale cursor and reset to the new value. The `gap` flag covers a
different case, events lost within one run, so for a definite restart
marker have your gate or connector publish a known event on startup and
key on that.

## Licence validity governs startup

- **A process already running** keeps trading past `expires_at`.
- **Any restart** requires a `Valid` licence, with no grace period. A
  drain, a redeploy, a reschedule and an evicted pod are all the same
  case.

Treat `license.expiring` and `license.expired` as a restart freeze and
renew ahead of one. [Licensing](/docs/licensing.html) carries the full
expiry position, and [pricing](/pricing.html) the renewal terms.

## Monitoring surface

| Surface | What it is |
| --- | --- |
| `/metrics` | 15 metrics in Prometheus text format, each with its own `HELP` and `TYPE` line. That is the whole set. |
| `/health` | Liveness probe endpoint. |
| ShmEventBus | Shared-memory event ring: every gate decision, order state change and fill. |

[Monitoring Reamer Server](/docs/server-monitoring.html) is the full
reference, with a starting alert table that gives a signal, threshold,
severity and action for each metric.

## Two rules that are not thresholds

- **Gate every automated restart on licence validity.** An expired seat
  turns a restart into an outage, so a supervisor restarts only when
  `bin/reamer-license status` exits 0.
- **Alert when `reamer_uptime_seconds` decreases.** That is a restart
  nothing in your deploy pipeline ordered.

## The live event stream

`ShmEventBus` is a shared-memory broadcast ring the core writes to, which
any number of readers in any number of processes attach to independently.
A reader's `since()` call returns every retained event newer than the
sequence it names. The ring holds a fixed event count set by
`event_buffer_capacity`, so a reader that falls further behind sees
`gap: true`.

Storage is your decision. Point whatever you already run at the stream;
the core's job stops at supplying it. `reference/cmd/event-tail` in the
kit is a working reader to start from.

## Two commands

| Command | What it does |
| --- | --- |
| `bin/reamer-diag-bundle` | Writes one JSON file carrying a `/health` and `/metrics` snapshot plus every retained event, with the monitoring auth token redacted. Attach it to a support email. |
| `bin/reamer-config-check` | Validates an edited config before you restart on it, because a path that does not resolve starts the core on defaults. Config changes take effect on restart, not on reload. |

[Deploying Reamer Server](/docs/server-deployment.html),
[configuration](/docs/server-config.html) and
[capacity and limits](/docs/capacity-and-limits.html) carry the rest of
the operational reference, including the envelope to check a proposed
deployment against.
