---
title: Deploying Reamer Server
description: Verify, license, configure and run Reamer Server with your own gate and broker connector, and the operational checks around a live process.
group: operations
order: 2
product: server
source: DEPLOYMENT.md
---

Sequences everything needed to go from "I have the ABI header and
library" to "my own binary, linking my own gate and broker connector, is
running and licensed." Each step links to the doc that actually owns that
surface rather than restating it.

**Most integrators reach a first simulated trade — a real `ExecutionReport`
from the local `fix-acceptor` in Step 4 — within a few hours of hands-on
work.** That is working time, and it does not start at Step 1.

**Have your license key in hand before you begin.** Step 2 activates a
license, and `reamer_server_run()` refuses to enter its loop without one.
No key ships in this archive — it arrives by email on payment (see
`README.md`). Until a key activates, Steps 1 and 3 are reachable and
**Step 4 is not**.

Going live against a real venue takes longer still, and depends on your own
gate/broker connector work.

## Step 1 — Get the header and library

Download the kit from a tagged release
(`server-kit-<version>-linux_x86_64.zip`).

**Verify the download before extracting it** — see "Verify your download"
below. Verifying first is both the correct order and the only order in
which the commands work as printed: the `.zip`, `.sha256`, `.sha256.sig`,
and `.zip.sig` are downloaded as siblings, and extraction creates the
`server-kit-4.5.0-linux_x86_64/` directory *next to* them, not around them.

Once verification passes, extract — see "Extract the kit" at the end of
this step for the commands, which must run after the verification above.

Contents: `include/reamer_server_abi.h`, `lib/libreamer_server_core.a`,
`lib/libreamer_license.a`, the standalone `bin/reamer-license`
activator, the standalone `bin/reamer-config-check` config validator,
this doc set (`README.md`, `DEPLOYMENT.md`, `CONFIG.md`, `BENCHMARK.md`,
`EXTENSION_PROTOCOL.md`, `LICENSING.md`, `RESEARCH_TO_SERVER.md`), a
`THIRD_PARTY_NOTICES` file, and a `sbom.cdx.json` CycloneDX SBOM.
There is no `reamer-server` binary to run directly — you link the
library into your own program, which implements a gate and a broker
connector as `ReamerGateVtable`/`ReamerBrokerConnectorVtable` and calls
`reamer_server_run()`. See "Step 3" below.

### Verify your download

Every release publishes a `.sha256` checksum, a detached Ed25519
signature over that checksum (`.sha256.sig`), and a detached Ed25519
signature over the zip's own bytes (`.zip.sig`), all alongside the eval
kit zip. Download all four files into the same directory.

**Run every command in this section from the directory holding the four
downloaded files** — the `.zip`, the `.sha256`, the `.sha256.sig`, and
the `.zip.sig`. Do not `cd` into the extracted kit first; the four files
are siblings of that directory, not members of it.

1. Check the file wasn't corrupted or tampered with in transit:

   ```
   sha256sum -c server-kit-4.5.0-linux_x86_64.zip.sha256
   ```

   Expected output: `server-kit-4.5.0-linux_x86_64.zip: OK`.

2. Check the checksum's detached signature, using the `release-sign`
   binary shipped in this kit's `bin/`.

   `release-sign` ships *inside* the archive, so before extracting there
   is no copy on disk to run. Extract just that one file next to the
   downloads:

   ```
   unzip -j server-kit-4.5.0-linux_x86_64.zip \
     'server-kit-4.5.0-linux_x86_64/bin/release-sign' -d .
   chmod +x release-sign
   ```

   Then verify, still from the directory holding the downloads:

   ```
   ./release-sign verify \
     server-kit-4.5.0-linux_x86_64.zip.sha256 \
     server-kit-4.5.0-linux_x86_64.zip.sha256.sig \
     b0ff9dcdcbd767ec78375629e10afec2baf18319df27e000ed6d8307cb93e68c
   ```

   Expected output: `signature verification OK`.

   Extracting the verifier from the archive it verifies is not a
   circularity this step can escape — see "What this step does and does
   not prove" below. Step 1's checksum is what binds the extracted
   `release-sign` to the archive you are about to trust.

   After extracting the full kit, the same binary is at
   `bin/release-sign` inside it; re-verifying from the kit root requires
   pointing at the parent directory:

   ```text
   bin/release-sign verify \
     ../server-kit-4.5.0-linux_x86_64.zip.sha256 \
     ../server-kit-4.5.0-linux_x86_64.zip.sha256.sig \
     b0ff9dcdcbd767ec78375629e10afec2baf18319df27e000ed6d8307cb93e68c
   ```

   The release-signing public key is:

   ```text
   b0ff9dcdcbd767ec78375629e10afec2baf18319df27e000ed6d8307cb93e68c
   ```

   This key is a separate trust domain from your license key: it attests
   that Reamer Labs built this archive, not that any particular license
   grant is valid.

   **What this step does and does not prove.** The verifying tool, the
   public key above, and these instructions all ship *inside the archive
   they authenticate*. Run as written, this proves the archive is
   internally consistent — it does not prove provenance, because anyone
   able to replace the archive could replace all three.

   **To get provenance, confirm the key out of band before trusting it.**
   The fingerprint above is the only value that matters; obtain it from a
   second, independent channel and compare:

   - Compare against the copy published on this page at reamerlabs.com,
     or mail **team@reamerlabs.com** and compare against the reply.
   - Compare against the same key published in the Reamer Research
     distribution's `DEPLOYMENT.md`, if you received that separately —
     one signing keypair covers both products.

   Once you have confirmed the fingerprint through a channel that did not
   arrive with the archive, pin it: record it in your own configuration
   management and verify future releases against your stored copy rather
   than against whatever the next archive contains. Verifying with a
   copied-and-pasted key from the same download is the step that gains
   you nothing.

3. Also check the detached signature over the zip's own bytes, not only
   its checksum — a party who could replace both the zip and its
   `.sha256` would produce a self-consistent pair that step 1-2 alone
   cannot catch:

   ```
   ./release-sign verify \
     server-kit-4.5.0-linux_x86_64.zip \
     server-kit-4.5.0-linux_x86_64.zip.sig \
     b0ff9dcdcbd767ec78375629e10afec2baf18319df27e000ed6d8307cb93e68c
   ```

   Expected output: `signature verification OK`. Same key, same
   circularity caveat, same out-of-band confirmation as step 2 — this is
   an additional check, not a replacement for it.

4. After extracting — see "Extract the kit" below, which must run
   first — verify every individual file in the kit against the
   `SHA256SUMS` manifest shipped inside it, from the kit root:

   ```text
   sha256sum -c SHA256SUMS
   ```

   Step 1's whole-archive checksum already covers these files, so this
   catches nothing new at download time. Its value is later: it lets you
   re-verify an individual file — after a partial copy between hosts, or
   to confirm nothing on a long-lived box has drifted —
   without re-downloading the zip.

5. `THIRD_PARTY_NOTICES` and `sbom.cdx.json` in the archive root name
   every open-source dependency statically linked into
   `libreamer_server_core.a`/`libreamer_license.a` (nlohmann/json,
   Asio, orlp/ed25519), with version and license.

### Extract the kit

Only after both verification commands above have passed, still from the
directory holding the downloads:

```
unzip -o server-kit-4.5.0-linux_x86_64.zip
cd server-kit-4.5.0-linux_x86_64
ls include/reamer_server_abi.h lib/libreamer_server_core.a
```

Every later step in this document runs from that
`server-kit-4.5.0-linux_x86_64/` directory unless it says otherwise.

Start at `README.md` in the archive root — what the product is, what
ships, and where the license key Step 2 needs comes from (it arrives by
email on payment and is not inside this archive).

### Supported build/runtime environment

`libreamer_server_core.a` in this release is built on Ubuntu 24.04 (glibc
2.39, the distro's default GCC) targeting the `x86-64-v2` CPU baseline
(SSE4.2/POPCNT-class instructions, ~2009+ x86-64 hardware). It is
deliberately not tuned to the build host's CPU, so it runs on any
x86-64-v2-capable machine. Link and run on a host with glibc >= 2.39 and
an x86-64-v2-capable CPU. Our release pipeline asserts the shipped
archive's actual glibc symbol floor on every release rather than trusting
the build host, so the figure above describes the artifact you received,
not the machine that produced it.

**Shared libraries the `bin/` tools need at runtime.** They are C++
programs and link the C++ runtime dynamically:

| Library | Why |
|---|---|
| `libc.so.6`, `libm.so.6` | glibc, per the floor above. |
| `libstdc++.so.6` | C++ standard library. |
| `libgcc_s.so.1` | GCC support runtime (unwinding). |

Nothing else. Confirm with `ldd bin/reamer-license` on your own host.
This matters on a minimal or distroless base image: those images
routinely omit `libstdc++`/`libgcc_s`, and the tools then fail to start
with a loader error naming the missing object rather than anything of
ours. Install your distribution's C++ runtime package (`libstdc++6` on
Debian/Ubuntu) or use a base image that includes it. Note that your own
server binary links `libreamer_server_core.a` **statically** and has
whatever runtime dependencies your linker gives it — this table covers
the shipped `bin/` tools only.

**The shipped binaries retain their symbol tables — this is deliberate,
not an oversight.** `file bin/*` reports "not stripped" for all four.
Symbols are kept so that a crash in a support case yields a usable
backtrace and so you can verify for yourself what is in the binary you
are about to run. They cost disk, not runtime. Strip your own copies if
your deployment policy requires it; nothing in the product reads them.

There is no build-from-source path: `libreamer_server_core` and the
binaries under `bin/` are precompiled and shipped to you as released
artifacts. Only Linux is built and packaged today —
Reamer Server is server-side execution infra deployed on Linux hosts,
not a desktop tool.

## Step 2 — Activate the license

Run the standalone `reamer-license` binary (included in the release
kit alongside `reamer_server_abi.h`/`libreamer_server_core.a`):

```
bin/reamer-license activate <license-key>
bin/reamer-license status
```

**Running in a container?** Pin a stable hostname before you activate —
`LICENSING.md`'s "Self-Service Activation Failure Modes" explains why an
unpinned container looks like a new machine on every restart and gets
reactivation refused.

Activation is per-machine and per-product, not per-binary — it writes
this product's grant into `~/.config/reamer/activation.json`, keyed to
this machine's fingerprint. Run this kit's `reamer-license` once and
every Reamer Server binary on this machine passes its license check
afterward, including your own gate/broker wrapper from Step 3 below,
whatever language it's written in (a Go or C++ wrapper linking
`reamer_server_core` directly, a Python process calling into it via
ctypes — none of that matters, since none of them need to embed the
activation subcommands themselves).

Reamer Research is licensed separately and priced separately, so running
both products on one machine consumes one seat of each. A key belongs to
one product — fixed when it was issued — and `activate` reads the product
off the key, so either kit's `reamer-license` activates either key; there
is no wrong activator to use. The two grants coexist in the one
activation file; activating or deactivating one leaves the other intact.
`status` lists both. See `LICENSING.md`.
`reamer_server_run()` refuses to start the loop at all without an active
license, checked internally before anything else.

### Exit codes — `status` is safe to script

`status` lists every product's seat, since a machine may hold one or
both:

```text
research: not activated
server:   valid, expires in 42 days
```

The exit code refers to the Server seat — the one this kit's engine
requires — so a provisioning gate can be written directly:

```text
bin/reamer-license status && exec ./your-server-binary
```

| Code | Meaning |
| --- | --- |
| 0 | Valid |
| 2 | Not activated — no Server seat on this machine |
| 3 | Expired — term ended, re-issue required |
| 4 | Activation belongs to a different machine |
| 5 | Activation file failed signature verification |
| 1 | The command itself failed (bad arguments, network error) |

`1` never means "the license is in state X" — it means the command did
not complete. Branch on 2-5 to distinguish causes; test for 0 to gate.
The listing goes to stdout when the Server seat is valid and to stderr
otherwise.

`status --json` prints a single JSON object to the same stream instead
of the human-readable listing, for scripts parsing the result instead of
gating on exit code alone: `{"product":"server","status":"valid",
"exit_code":0,"this_machine":"...","products":[{"product":"research",...},
{"product":"server","status":"valid","is_this_build":true,"expires_at":...,
"expired_by_clock_rollback":false,"server_machine_id":"...","machine_id":"..."}]}`.
`products` carries one entry per product; the top-level fields describe
the Server seat, which is what `exit_code` refers to. `machine_id` is the fingerprint stored in
`activation.json`; `this_machine` is the one computed from the running
host — they must match, and a mismatch is what produces exit code 4.
`status --verbose` prints the same seat/machine/expiry detail as extra
human-readable lines instead. The two flags are mutually exclusive; any
other second argument to `status` exits 1 with a usage message.

**Exit 5 usually means stale, not malicious.** `activation.json` lives at
a fixed per-user path (`~/.config/reamer/activation.json` on Linux and
macOS, `%APPDATA%\Reamer\activation.json` on Windows) — not inside the
kit directory. A file left behind by an earlier or separate Reamer Labs
install is therefore still found by this one, and it is signature-checked
before the machine fingerprint is examined, so the first thing it can
fail is the signature. An activation signed by a different build's key
verifies as `Tampered` and exits 5.

If you get exit 5 on a machine you have not deliberately edited an
activation file on:

1. Confirm the file is the one you expect: `ls -l
   ~/.config/reamer/activation.json` — check the modification date
   against when you activated *this* kit.
2. If it predates this activation, remove it and re-activate:
   ```text
   rm ~/.config/reamer/activation.json
   bin/reamer-license activate <license-key>
   ```

Removing the file is safe: it holds no state you cannot re-obtain by
activating again. Run `bin/reamer-license deactivate` first if the old
activation still holds a seat you need to release.

### Network and host requirements for activation

**`activate` and `deactivate` require network egress. Nothing else does.**
Once `activation.json` exists, every license check — including the one
`reamer_server_run()` performs on every start — is fully offline: it
verifies an Ed25519 signature over the stored activation against the
machine fingerprint and the local clock, and makes no network call. A
trading host that has been activated once needs no egress thereafter.

**`reamer-license` shells out to `curl`.** It does not link an HTTP or TLS
library; it forks `curl` (never through a shell) and reads its stdout.
Two consequences to plan for:

- **`curl` must be present on `PATH`.** It is absent from distroless and
  most hardened minimal base images. Add `curl` to the image that runs
  activation. Do not activate on a separate host and copy the resulting
  `activation.json` in: a grant is bound to the fingerprint of the
  machine that produced it, and copying it between machines is not
  supported (`LICENSING.md`). Without `curl`, `activate` fails with
  `failed to start curl subprocess (is curl installed and on PATH?)` or
  `network request failed (curl exit code 127)`.
- **`curl` is a runtime dependency, declared in `sbom.cdx.json` and
  `THIRD_PARTY_NOTICES` as an external runtime component.** It is not
  linked into `libreamer_server_core.a` or `libreamer_license.a`, and no
  Reamer Labs binary contains HTTP or TLS code.

Egress rule a network team can write directly:

| Field | Value |
| --- | --- |
| Direction | Outbound only (no inbound, no callback) |
| Host | `licenses.reamerlabs.com` |
| Port | TCP 443 |
| Protocol | HTTPS (TLS as provided by the host's `curl`) |
| Paths | `POST /activate`, `POST /deactivate` |
| When | Only during `reamer-license activate` / `deactivate` |
| Timeout | 30 s per request (`curl --max-time 30`) |
| Proxy | Honors `curl`'s standard `https_proxy` / `HTTPS_PROXY` and `NO_PROXY` environment variables |

The forked `curl` inherits the calling environment, so its own
configuration applies — proxy variables above, `CURL_CA_BUNDLE` and
`SSL_CERT_FILE` for a corporate TLS root, and `~/.curlrc`. Its stderr is
discarded, so on failure you get Reamer Server's exit-code message rather than
`curl`'s diagnostic; to see the underlying cause, reproduce the request
by hand:

```text
curl -v -X POST https://licenses.reamerlabs.com/activate
```

A `000` or connection error there is an egress or TLS problem, not a
licensing one.

There is no telemetry, heartbeat, or periodic re-validation channel. If
you observe traffic to `licenses.reamerlabs.com` from a running trading
process, that is not expected behavior — report it.

## Step 3 — Implement and link your gate and broker connector

Reamer Server has no compiled-in gate or broker connector — you implement
both as a `ReamerGateVtable` and a `ReamerBrokerConnectorVtable`
(`reamer_server_abi.h`), link `libreamer_server_core`, and call:

```c
ReamerServerStatus status = reamer_server_run(&gate_vtable, &broker_vtable, config_path);
```

from your own `main()`. This *is* your server binary — there is no
separate process to start or endpoint to point at, since both vtables are
called directly, in-process.

`EXTENSION_PROTOCOL.md` specifies both vtables method by method — every
parameter, every ownership and lifetime rule, the threading contract, and
the order-state transitions core enforces on what your connector reports.
Read it alongside the header; between them they are the complete
interface.

(If you're running strategies through a remote relay rather than the
built-in local Sequencer — `REAMER_STRATEGY_INPUT_MODE=remote` — that is
the one extension point which stays an out-of-process UDS protocol,
listening on `strategy_relay_endpoint`. Both its wire format and the
default `local` mode's are specified in `EXTENSION_PROTOCOL.md`.)

Adapt either reference to your own risk rules or venue — the ABI is the
entire interface; neither implementation ever needs Reamer Server's
source. There is no reconnect/backoff/timeout to configure for the gate
or broker connector: a vtable call either returns or your process is
blocked, exactly like any other function call in your own code.

## Step 4 — Test locally before touching a real venue

Before pointing your gate and broker connector at a real venue, run your
binary against stub vtables and drive one order through it. This needs
nothing beyond the kit: no venue account, no FIX counterparty, no extra
toolchain past the C/C++ compiler you already used to build.

**If you have a Go toolchain, see the finished version of this test
first.** `reference/demo-trade.sh` runs a fuller version of the same
shape — simulated FIX venue, relay, gate, and an order burst across
three instruments with one gate rejection — already built, in about
twenty seconds cold:

```bash
cd reference && ./demo-trade.sh
```

It is reference material, not a supported surface, and its timing
characteristics are its own rather than the core's (see `BENCHMARK.md`
for every number the product claims). But it is the whole path closing,
and reading a working integration is faster than imagining one from a
specification. The rest of this step is the same test built from
scratch against the header, which is what you will adapt for your own
gate and connector.

The shape of the test is three pieces you write once and keep:

1. **A stub broker connector.** `submit_order` records the intent and
   queues a `ReamerExecutionReport` with `status = FILLED`, `last_qty`
   and `last_price` in nanounits (`value * REAMER_FIXED_SCALE`), and a
   `venue_order_id` you make up. `poll_updates` drains that queue.
   `pull_state` reports `connected = true` plus whatever positions your
   fills implied. `tick`, `poll_events`, and `poll_published_events`
   can be no-ops returning 0.
2. **A stub gate.** `check` writes `accepted = true` into the `ReamerAck`
   and returns 0; `is_available` returns `true`. Add your real policy
   later — start by proving the plumbing.
3. **A strategy client.** One Unix-socket connection speaking the
   `local`-mode framing in `EXTENSION_PROTOCOL.md`'s "Strategy socket
   protocol": a 5-byte header, then type byte `0` and the `NewOrder`
   fields. Any language with sockets and byte packing will do.

Point `socket_path` at a scratch path, start your binary, connect the
client, and send one order. A correct integration prints your gate's
accept, your broker's fill, and returns an `OrderUpdate` (type byte `1`)
to the client with `status = 3` (filled). That exercises every part of
the loop a venue would: intent decode, gate decision, broker submit,
execution report, and the update path back to the strategy.

**Check the shutdown path in the same run.** Send `SIGTERM`. Core stops
accepting new requests, keeps calling your connector's `tick`/`poll_*`
for up to `shutdown_drain_seconds`, calls `pull_state` once more, and
publishes a final reconciliation snapshot before `reamer_server_run()`
returns `REAMER_SERVER_SUCCESS`. If your connector needs to flush a log
or close a session cleanly, that work belongs in `tick()`/`poll_events()`
during this window — there is no separate shutdown callback.

Once that passes, swap the stub broker for your venue connector and keep
the stub gate; then swap in your real gate policy. Changing one side at a
time keeps a failure attributable.

To size the core on this machine before you wire a real venue, run the
prebuilt `bin/server-bench` (activation required — Step 2). It drives a
concurrent-strategy sweep against `reamer_server_run()` and prints the
throughput/latency curve for your hardware; see "Measuring on your own
hardware" in `BENCHMARK.md`.



## Step 5 — Write a config

Full field reference and the 3-layer load order (defaults → JSON file →
env vars) live in `CONFIG.md`. There is no required field for the gate or
broker connector — both are passed as vtables directly, not addressed by
config:

```json
{
  "monitor_host": "127.0.0.1",
  "monitor_port": 8090,
  "monitor_auth_token": "replace-with-a-real-secret"
}
```

No config file named `reamer-server-config.json` ships in this kit — the
snippet above is a template to copy into a file yourself. Save it as
`~/.config/reamer/reamer-server-config.json` (or any path you choose), and
either point `REAMER_SERVER_CONFIG_PATH` at it or pass that path as
`reamer_server_run()`'s third argument.

Use this same file for the Step 4 local test, so the
`Authorization: Bearer replace-with-a-real-secret` header in Step 6 works
against the server you started there and Steps 4-6 run in sequence
exactly as printed.

**`replace-with-a-real-secret` is a placeholder.** It exists so the quickstart is runnable
end-to-end on a loopback interface. Replace it before any deployment that
is not a throwaway local test, and do not copy the reference config into
one.

**Unrecognized keys are rejected.** A misspelled field fails the load
rather than silently running at its default — run
`bin/reamer-config-check --config <path>` to confirm the file resolves the
way you intend, and read the field list it echoes. See `CONFIG.md`.

## Step 6 — Start your binary and confirm it's live

```
./your-server-binary
```

However you've wired your own CLI, `reamer_server_run()` blocks running
the core loop until `SIGTERM`/`SIGINT` (handled internally — your `main()`
does not need its own signal handlers for this), then returns
`REAMER_SERVER_SUCCESS` or `REAMER_SERVER_ERROR`. No binary in this kit
runs the server loop for you (see Step 1) — that's always your own binary,
linking your gate and broker connector.
Three small standalone binaries do ship, for the pieces of operator
tooling that don't need your vtables: `bin/reamer-license`
(`activate <key>` / `deactivate` / `status` — Step 2),
`bin/reamer-config-check --config <path>` (validates a config file offline
against the same precedence `reamer_server_run()` applies at startup,
without booting a server — see `CONFIG.md`'s "No reload" section; there's
no live-reload path, so this is how you catch a bad edit before restarting
rather than after), and `bin/reamer-diag-bundle --config <path>` (Step 7's
incident-report tool). Confirm your own binary is live via the
monitoring endpoint's HTTP routes (`monitor_host`/`monitor_port`, default
`127.0.0.1:8090`):

```
curl -H "Authorization: Bearer replace-with-a-real-secret" http://127.0.0.1:8090/health
```

The token in that header is not a default and has no meaning to the
server: it works only because it is the literal placeholder string in the
reference config from Step 5. Substitute whatever you set in
`monitor_auth_token`.

**`monitor_auth_token` defaults to empty, and an empty token means the
monitoring endpoint is unauthenticated.** `/health` and `/metrics` expose
order submit/accept/reject counts, gate-rejection reasons, and connection
state — operational detail about live trading that you should treat as
sensitive. The only thing standing between that and any local process is
the default loopback bind, which is a real control but a single one:
anything on the host, including another container sharing the network
namespace, can read it.

Set `monitor_auth_token` on every deployment, not only the ones bound
off-host. Startup validation enforces the narrower rule — it refuses to
run if `monitor_host` is non-loopback while the token is empty — but that
is a backstop against the worst case, not the recommended posture. There is no event-stream
route here — `ShmEventBus` (`MONITORING.md`) is the outlet for the live
ops/intent-flow event feed; `/health` and `/metrics` below answer from a
handful of stored scalars instead.

Point an existing Prometheus (or any scrape-compatible) deployment at
`/metrics` on the same host/port for order submit/accept/reject counts,
gate-rejection reasons, event bus and tracked-order occupancy, broker/relay
connection state, and uptime/loop/poll-latency gauges (see `CONFIG.md`):

```
curl -H "Authorization: Bearer replace-with-a-real-secret" http://127.0.0.1:8090/metrics
```

Wire the same host/port's `/health` route into your process supervisor's
liveness/readiness check (see `MONITORING.md` for the response shape):

```
# systemd unit
ExecStartPre=/usr/bin/curl -fs -H "Authorization: Bearer replace-with-a-real-secret" http://127.0.0.1:8090/health

# Docker / Kubernetes probe
curl -fs -H "Authorization: Bearer replace-with-a-real-secret" http://127.0.0.1:8090/health || exit 1
```

The header is required whenever `monitor_auth_token` is set — which
Step 5 recommends for every deployment — so a probe written without it
gets `401` and your supervisor reports a healthy process as down. Supply
the token from your secret store rather than inlining it as shown here.

## Step 7 — Operational notes

If you are porting a strategy from Reamer Research, start by reading
`RESEARCH_TO_SERVER.md` for a guide to the callback-model translation
before implementing your gate and broker connector.

- **Alerting.** `MONITORING.md`'s "Alert table" is the recommended
  starting set — signal, threshold, severity and action for every
  operational metric, plus the full 15-metric `/metrics` reference above
  it. Wire that before go-live; it is what stands between "the process
  runs" and "we can operate it."
- **Capacity envelope.** See `CAPACITY_AND_LIMITS.md` for max sustained
  order rate, max event throughput, and the `max_tracked_orders`/
  `event_buffer_capacity` overflow behavior — the single reference to
  check a proposed deployment's expected load against.
- **Restart safety.** Reamer Server holds no durable state of its own, so
  a process restart loses nothing core was responsible for keeping. On
  start, order and position state is re-seeded entirely from the broker
  connector's `pull_state()`.

  **Why this holds, so you can check the reasoning rather than take it on
  trust:** core keeps no second copy of position state to diverge from the
  venue's. It does not persist positions across runs, does not carry a
  running tally it reconciles later, and does not treat its in-memory view
  as authoritative — the venue is the sole source of truth for order and
  position state, and `reamer_server_abi.h` states that as a contract
  (see `pull_state`, and the shutdown contract: core never cancels an open
  order during drain, and calls `pull_state` once more before exit). Drift
  in the usual sense — two copies of the truth that disagree after a
  restart — is structurally absent because there is only ever one copy.

  **What that does not promise.** This is a statement about core, not about
  your integration end to end. Correctness still depends on things you
  build and operate:
  - `pull_state()` is **yours**. If your connector reports the venue's
    state inaccurately or incompletely, core adopts that inaccuracy as
    fact. Core cannot detect it.
  - Fills that occur while the process is down are not lost — but core
    only learns about them through the next `pull_state()`, so your
    connector must report the venue's *current* state, not a delta since
    it last spoke to core.
  - In-flight intents at the moment of a kill are not resumed. Core is not
    holding them for you; anything not yet at the venue is gone, and
    anything already at the venue is the venue's.
  - **The event-bus sequence restarts at 0.** "No durable state" covers
    the sequence counter too: on start, core reinitializes the shared
    segment's header and begins numbering from 1 again, even when it
    reattaches a segment a previous run left behind. A consumer that read
    up to sequence 8831 will, after a restart, see sequences in the
    hundreds.

    **A sequence that goes backwards means a restart, not a gap.** Treat a
    decrease as "core restarted; my cursor is stale" and reset your cursor
    to the new value — do not wait for the sequence to climb back past
    your last-seen number, because it never will within that run, and you
    would discard every event until it did. The `gap` flag reports lost
    events *within* one run and does not signal this case: across a
    restart there is no ordering relationship between old and new sequence
    numbers at all. If you need to tell restarts apart with certainty,
    have your gate or connector publish a known event on startup and key
    on that rather than on the counter.

  Killing and restarting the process is safe in the sense that core will
  rebuild its view from the venue, not in the sense that an arbitrary
  mid-flight moment is replayed.

  One precondition: **a restart requires a `Valid` license.** Expiry never
  halts a running process, but it does refuse every new
  `reamer_server_run()`, with no grace period. A process running past its
  `expires_at` is still trading and still safe to leave alone — but it will
  not come back if you stop it, and neither will a pod the scheduler
  evicts. Treat a `license.expiring` or `license.expired` event as a
  restart freeze until the license is renewed: alert on both (see
  `MONITORING.md`), and renew before you drain, redeploy, or reschedule.
- **Config changes require a restart, not a reload.** There is no
  `SIGHUP`/live-reload path — a deliberate choice, not a gap; see
  `CONFIG.md`'s "No reload" section for why. Validate an edited config
  with `bin/reamer-config-check --config <path>` before restarting, so a
  typo is caught before it turns into a failed start.
- **No startup recovery wait.** There is no second, core-derived copy of
  account state to seed at startup — the first gate check simply uses
  whatever your broker connector's first `pull_state` call writes. If
  the venue session isn't up yet, that call reports
  `connected: false` and every intent is rejected (`reason="venue
  disconnected"`) until it is — a fixed, non-configurable core rule, not
  a startup phase to wait through.
- **No outage escalation subsystem.** A disconnected broker connector
  fails every gate check immediately and per-intent (see above) rather
  than accumulating toward a timed escalation threshold; there is nothing
  further to configure here.
- **License expiry.** Every license is a term license (`expires_at`,
  anchored to first activation — see `LICENSING.md`).
  Reamer Server re-checks validity periodically while running, in addition
  to the check at startup. Expiry never halts a running process: a
  currently-running `reamer_server_run()` keeps trading past `expires_at`,
  logging escalating `license.expiring`/`license.expired` ops events
  (`CONFIG.md`'s `license_warning_days`/
  `license_expired_warning_interval_seconds`). Only a **new** process start
  is blocked by an expired, tampered, or otherwise invalid license — it
  fails cleanly at startup with a status-specific message instead of
  entering the run loop at all.
- **stderr logging** — everything Reamer Server writes to stderr is one
  JSON object per line; the process does not rotate
  it, so wire it into `journald` or your container runtime's log driver
  (see `MONITORING.md`'s "stderr — process-level structured log lines"
  section).
- **Incident reports.** Run `bin/reamer-diag-bundle --config <path>`
  against the affected instance and attach the output file to your support
  request — see the vendor questionnaire pack's support-model section for
  response-time expectations and the contact address. The pack ships in
  the kit and is published as the [vendor questionnaire
  pack](/docs/vendor-questionnaire.html). It is the standing
  answer to security-questionnaire, support-model, vulnerability-disclosure,
  data-handling, and business-continuity questions, and is written to be
  handed straight to an infra/risk/procurement team. The bundle reads
  the same config file, scrapes `/health`/`/metrics` from the running
  process, and pulls every ops/intent-flow event still retained on the
  event bus, with `monitor_auth_token` redacted from the output — see
  `MONITORING.md`'s "Diagnostic bundle" section for the exact contents.

## Deployment Topology

**Single process is the supported unit.** One `reamer-server` process
owns one `socket_path`, one `monitor_host:monitor_port`, one
`event_bus_shm_name`. Everything below builds on that.

**Multiple cores per host.** Supported, if each process gets a distinct
`socket_path`, `monitor_port`, and `event_bus_shm_name` (`CONFIG.md`).
There is no product-level allocator for these — assign distinct values
yourself, or through your orchestrator. Same mechanism "Version
coexistence" below uses for canaries.

**Warm standby / failover.** Not a product feature. Reamer Server has no
built-in leader election, health-based failover, or standby-promotion.
Given the restart-safety guarantee above (no durable state;
`pull_state()` fully re-seeds on start), a cold-standby pattern is
achievable at the orchestration layer — an external supervisor starts a
second instance, pointed at the same venue/broker connector config,
once it detects the primary's `/health` failing — but the product does
not coordinate this for you.

**Licensing.** Every concurrent process, including a warm-standby
instance, needs its own activated license seat — see
`LICENSING.md`'s one-machine-per-seat policy. An idle
standby box is not exempt.

**Orchestrator fit.** `/health` (`MONITORING.md`) is a conventional
liveness/readiness probe, so k8s/systemd-style supervision fits at the
single-process level described above. It implies nothing beyond that
about coordinated multi-instance behavior.

## Upgrade, rollback, and coexistence

Under a term license this is the recurring event you'll experience most
often — updates arrive continuously, not once at purchase.

### Two version numbers, two different questions

A release carries two independent version numbers, and they answer
different questions:

- **Product version** (`4.0.0`-style, semver) — what changed. Read the
  release notes for the tag you're moving to.
- **`REAMER_ABI_VERSION`** (`include/reamer_server_abi.h`, currently `4`) —
  whether the C struct/vtable layout changed. It moves when a struct or
  vtable grows or changes; most product releases ship with it unchanged.

### Validate before you cut over

1. **Relink and rebuild.** Point your build at the new release's
   `include/`/`lib/` and rebuild your binary. This *is* the ABI
   compatibility check, not a separate step: `static_assert`s compiled
   into the library pin `sizeof(ReamerGateVtable)`/
   `sizeof(ReamerBrokerConnectorVtable)` to the current
   `REAMER_ABI_VERSION`, so a header/library mismatch fails your build
   with a named error instead of linking something with a silently
   different layout. A clean rebuild is the confirmation signal — there is
   no separate runtime ABI-version check to run.
2. **Validate your config against the new binary.**
   `bin/reamer-config-check --config <path>` (see `CONFIG.md`) — catches a
   schema or field change independent of the ABI question above.
3. **Smoke-test before touching production.** Run the new build against a
   non-production venue/environment first and confirm Step 6's `/health`
   and `/metrics` both report as expected.

### Cutover and rollback

Given the restart-safety guarantee above (no durable state, positions
always re-seeded from `pull_state()`), upgrading and rolling back are the
same procedure run in opposite directions — there is no data migration
step, because there is nothing to migrate:

- **Upgrade:** stop the running process (`SIGTERM`; let
  `shutdown_drain_seconds` run its drain window), swap in the new binary,
  start it.
- **Rollback:** keep the previous version's binary/build artifacts on hand
  until the new one is confirmed healthy. If it isn't, stop it and start
  the old binary — same procedure, reversed.

Confirm the license is `Valid` (`bin/reamer-license status`, exit code 0)
before starting either direction. Both stop a running process and start a
new one, and a non-`Valid` license refuses the start — so an expired seat
turns a routine upgrade into an outage, and blocks the rollback that would
have undone it.

### Version coexistence

There is no built-in canary mechanism, and two processes cannot share
`socket_path`, `event_bus_shm_name`, or the monitor port — the same
collision class that rules out running two same-version instances on one
host without distinct resource names (see `CAPACITY_AND_LIMITS.md`).
A side-by-side canary is achievable, not automatic: give the canary
instance its own socket path, event bus shm name, and monitor port, and
point it at either a paper/staging venue or a scoped instrument subset via
your own gate logic. There is no product feature that does this
allocation for you.
