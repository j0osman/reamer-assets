---
title: Licensing and activation
description: Seats, activation, exit codes, containers, machine migration, term licences, renewal, prices and the source-escrow policy for both products.
group: security
order: 3
product: both
source: LICENSING.md
---

## Seat Policy

One license key equals one seat. One seat is one machine, for one
product. One seat activates on exactly one machine at a time. Run more
than one machine concurrently: buy more seats.

There is no pooled or floating license. A key does not roam freely
between machines while still bound.

**Reamer Research and Reamer Server are licensed separately.** They are
priced separately, and a key issued for one product does not activate
the other — the product is fixed in each kit's binaries at build time
and signed into the activation grant, so presenting a Research key to a
Server binary is rejected at activation.

A machine running both products therefore holds two grants and consumes
one seat of each. Both grants live in the one
`~/.config/reamer/activation.json`, filed by product: activate each
product once, with that product's own kit activator. Activating the
second product does not disturb the first product's grant, and
deactivating one leaves the other intact.

Process count does not affect the seat. Any number of concurrent
backtests, server instances, strategy processes, or binaries you link
against the libraries run under one seat on one machine. This is the
intended shape of a Reamer Server deployment: one licensed machine, one
server process, and as many core-pinned strategy processes as the box
has cores. What consumes an additional seat is an additional machine.

Every licensed machine runs `activate` itself. Copying an activation
file between machines is not a supported deployment technique — see
"Containers" below.

## Where the activator lives

This document covers both products, Reamer Research and Reamer Server, which place the same
`reamer-license` binary differently:

| Kit | Path from the extracted root |
|---|---|
| Reamer Server (`server-kit-<version>-<platform>.zip`) | `bin/reamer-license` |
| Reamer Research (`research-kit-<version>-<platform>.tar.gz`) | `bin/reamer-license` |

Commands below are written as `reamer-license <subcommand>`. Prefix with
`./bin/reamer-license status` (same relative path in both kits), or put
the binary on `PATH`.

Both kits build this activator from the same sources
(`reamer-research/shared/`), and either copy operates on either product.
A license key belongs to one product, fixed when it was issued, so
`activate` takes the product from the key itself — there is no wrong kit
to run it from. `status` lists both products, and `deactivate --product
<name>` releases the seat you name. On a machine running both, one
activator handles both seats — see "Seat Policy" above.

The one thing that is per-kit is the `status` **exit code**, which
reports the seat that kit's own engine needs to start (see "Check
current status" below).

**The two copies are not byte-identical, and that is expected.** The
research and server release jobs build on different base images to meet
each product's own glibc floor (2.35 vs 2.39 — see
`SUPPORTED_PLATFORMS.md`), so compiling the same source on each produces
two different artifacts: different checksums, different sizes. This does
not affect behavior — both run the identical activation logic. Do not
expect the two kits' `SHA256SUMS` entries for `reamer-license` to match
each other; each manifest is only a checksum of the copy shipped in that
kit, and that is the one to verify against.

## Activate a Machine

Run the license CLI on the target machine:

```
reamer-license activate <license-key>
```

1. An unbound key activates unconditionally.
2. A key already bound to a DIFFERENT machine REFUSES activation. The
   server returns:
   ```
   License is active on another machine. Run `reamer-license deactivate`
   on that machine first, or contact support@reamerlabs.com to transfer
   your license.
   ```
3. Re-running `activate` with the SAME key on the machine it is already
   bound to succeeds — this is a no-op re-activation, not a new seat.

Check current status at any time:

```
reamer-license status
```

Reamer Research and Reamer Server are licensed separately, and one
machine may hold a seat for either or both. `status` reports every
product, so the output describes the machine rather than one product:

```
research: valid, expires in 42 days
server:   not activated
```

The exit code refers to the product **this kit's** engine requires — the
Research kit's `status` exits 0 when the Research seat is valid, the
Server kit's when the Server seat is. So it still gates a provisioning
script directly (`reamer-license status && start_server`). The exit code
names the reason:

| Code | Meaning |
| --- | --- |
| 0 | Valid |
| 2 | Not activated — this product has no seat on this machine |
| 3 | Expired — term ended, re-issue required |
| 4 | Activation belongs to a different machine |
| 5 | Activation file failed signature verification |
| 1 | The command itself failed (bad arguments, network error) |

A `1` always means the command did not complete, never that the license
is in a particular state. The listing is printed to stdout when this
kit's product is valid and to stderr otherwise.

`status --verbose` adds the seat id, expiry, and both machine
fingerprints — `bound machine` (the one recorded in `activation.json`)
and `this machine` (the one computed from this host right now) — to the
same stdout/stderr stream as plain `status`, as human-readable extra
lines. The two must be identical; a mismatch is exactly what produces
exit code 4, `InvalidMachine`.

`status --json` prints one JSON object instead, to the same stream
(stdout on exit 0, stderr otherwise), for scripts that want to parse the
result rather than match on the message text:

```
{"product":"research","status":"valid","exit_code":0,
 "this_machine":"...",
 "products":[
   {"product":"research","status":"valid","is_this_build":true,
    "expires_at":1790254030,"expired_by_clock_rollback":false,
    "server_machine_id":"...","machine_id":"..."},
   {"product":"server","status":"missing","is_this_build":false,
    "expires_at":0,"expired_by_clock_rollback":false,
    "server_machine_id":"","machine_id":""}]}
```

`products` carries one entry per product. The top-level
`product`/`status`/`exit_code` describe the product this kit's engine
requires, which is the one `exit_code` refers to; `is_this_build` marks
that same entry inside the array.

`machine_id` is the fingerprint stored in `activation.json`;
`this_machine` is the one computed from the running host. Comparing the
two is the whole of the machine check, so a script can report a
`4`/`InvalidMachine` precisely rather than guessing at the cause.

`status` (and `status --verbose`) is unchanged if you're not passing
`--json`. `--verbose` and `--json` are mutually exclusive — pass one or
the other, not both. Any other second argument to `status` is rejected
with exit code 1 and a usage message.

A `5` most often means the activation file is **stale**, not that anyone
tampered with it. `activation.json` lives at a fixed per-user path
(`~/.config/reamer/activation.json`; `%APPDATA%\Reamer\activation.json`
on Windows), outside any install directory, so a file left by an earlier
or separate Reamer Labs install is still found — and the signature is checked
before the machine fingerprint, so a mismatched signing key surfaces as
`5` rather than `4`. Delete the file and re-run `activate`. Run
`deactivate` first if that old activation still holds a seat.

## Migrate to a New Machine

Move a seat from an old machine to a new one:

1. On the OLD machine, run:
   ```
   reamer-license deactivate
   ```
   This clears the machine binding. Only the machine currently holding
   the binding can deactivate it.

   If the machine holds seats for **both** products, a bare `deactivate`
   will not guess which one to release — it stops and lists the choices.
   Name the seat explicitly:
   ```
   reamer-license deactivate --product server
   ```
   Any `reamer-license` releases any product's seat: which kit the binary
   came from, and which directory you run it in, do not affect it.
2. On the NEW machine, run:
   ```
   reamer-license activate <license-key>
   ```

This deactivate-then-activate sequence is the only supported migration
path. There is no direct machine-to-machine transfer.

**Hazard: lost or unavailable old machine.** If the old machine is gone,
wiped, or otherwise unreachable, you cannot self-service the deactivate
step. Contact support@reamerlabs.com to release the seat manually.

## Self-Service Activation Failure Modes

A machine's identity is `machineFingerprint()`:
`SHA-256(hostname:first_mac)`. The four failure modes below all trace back
to that fingerprint, and every one of them looks like a self-service error
to a first-time user. Each is expected behavior, not a bug.

**Containers and VMs: reactivation refused after every restart.** A
container's hostname typically changes on every restart, so
`machineFingerprint()` produces a new value each time and the license
server sees what looks like a brand-new machine. Reactivation is refused
with the same "already active on another machine" error described above,
because as far as the server can tell, it is.

*Remedy:* pin a stable hostname across restarts (most container runtimes
support this — set it explicitly rather than letting the platform assign
one), or run `reamer-license deactivate` before each restart and
`activate` again after, or contact support@reamerlabs.com for
containerized-deployment guidance.

**All-virtual-interface machines: fingerprint collision.** `first_mac`
skips known virtual interfaces (`docker`, `veth`, `br-`, `virbr`, `tun`,
`wg`) when picking a MAC address. A machine with no other interface to fall
back to — some CI runners, heavily virtualized containers — reports
`00:00:00:00:00:00`. Two such machines produce the identical fingerprint
and can collide on activation, one blocking the other.

*Remedy:* same as above — a stable, unique hostname is the workaround,
since it is half of the fingerprint input even when the MAC half is
degenerate. On a heavily virtualized fleet this is the operative
requirement, not a suggestion: assign each host a unique, persistent
hostname before activating, and treat hostname assignment as part of
provisioning rather than something the platform picks. Two hosts that
share a hostname *and* degrade to the all-zeros MAC share a fingerprint,
and the second to activate is refused.

The fingerprint is `SHA-256(hostname:first_mac)`, and `reamer-license`
has no subcommand that prints it, so check the two inputs directly on
each host before activating — `hostname`, and the MAC of the first
non-virtual interface (`ip link`). If two hosts agree on both, they will
collide.

**Treat the all-zeros-MAC case as unspecified.** A future release may
refuse activation outright on a machine with no non-virtual interface,
rather than issuing a fingerprint that can collide. Assign unique,
persistent hostnames as above and your fleet is correct under either
behavior. If your environment cannot avoid all-virtual interfaces, raise
it with support@reamerlabs.com before you build around the current
behavior.

**Renamed or reordered NICs: fingerprint shifts.** `first_mac` is picked
from whichever qualifying interface sorts first. Renaming, reordering, or
rebonding interfaces on an activated machine can change which MAC that is,
which changes the fingerprint, which makes a previously-activated machine
look new on the next license check.

*Remedy:* avoid renaming or reordering network interfaces on a licensed
machine. If unavoidable, `deactivate` and `activate` again afterward.

**`Tampered` status.** Every activation record is signed, and
`reamer-license` verifies that signature against a public key baked
into the release binary. `Tampered` means the signature did not verify:
the `activation.json` on this machine does not match what the license
server issued.

*Remedy:* do not hand-edit `activation.json` — in particular, editing the
stored dates to extend a term is exactly what this check exists to catch,
and it will report `Tampered` rather than a longer term. Run
`reamer-license activate <license-key>` again to obtain a fresh,
correctly signed record. If a clean re-activation still reports
`Tampered`, the binary or the activation file is corrupt; verify the kit's
signature (`DEPLOYMENT.md` Step 1) and contact support@reamerlabs.com.

**`expired (system clock appears to have moved backward...)`.** Alongside
`activation.json`, the activation directory
(`~/.config/reamer/`; `%APPDATA%\Reamer\` on Windows) holds a second,
unsigned file, `last_seen.json` — a local high-water mark of the latest
time the license check has ever observed, used to catch a system clock
rolled backward to keep an expired key looking unexpired. A `last_seen`
value that is itself corrupt (e.g. a future timestamp written by some
earlier fault) trips this guard even on a machine whose clock never moved,
and prints `expired` for a key that is not.

*Remedy:* delete `last_seen.json` (not `activation.json`) and re-run
`reamer-license status`. This does not touch your activation or seat —
it only resets the local rollback watermark, which is rebuilt from the
current clock reading on the next check.

## Containers and Kubernetes: the working setup

The failure modes above describe what goes wrong. This section is the
configuration that avoids all of them, for the deployment shape most
users actually use.

**Two host requirements, both easy to miss in a minimal image.**

1. **`curl` must be on `PATH`.** `reamer-license` performs activation
   by shelling out to `curl` as an external process — no HTTP or TLS code
   is linked into any Reamer Labs binary, which is why `curl` is documented as
   a runtime dependency in the kit's SBOM rather than as a library.
   A distroless or `scratch` image has no `curl`, and `activate` fails
   with `failed to start curl subprocess (is curl installed and on
   PATH?)`. Add it to whatever image runs activation.

2. **`$HOME` must be set and writable.** Activation state is written to
   `$HOME/.config/reamer/activation.json` (Linux/macOS;
   `%APPDATA%\Reamer\activation.json` on Windows). A container running as
   a user with no home directory writes to `./.config/reamer/` instead,
   which usually means it is lost on restart.

**Activate in the container, on a pinned hostname.** This is the
supported configuration. Every licensed machine runs `activate` itself;
there is no path that activates elsewhere and distributes the result.

### Pin a stable hostname, activate once per pod identity

Give the workload a hostname that survives restarts, so
`machineFingerprint()` stays constant and the activation remains valid.

In Kubernetes, a `StatefulSet` does this by construction: pod names are
stable and ordinal (`app-0`, `app-1`), and the pod hostname follows the
pod name. A `Deployment` does not — it assigns a fresh random pod name on
every restart, so every restart looks like a new machine. **Use a
`StatefulSet` for any licensed workload**, or set the hostname explicitly.

Pair it with a persistent `$HOME/.config/reamer` so the activation written
on first start is still there on the next one:

```yaml
apiVersion: apps/v1
kind: StatefulSet
spec:
  serviceName: reamer
  replicas: 1
  template:
    spec:
      containers:
        - name: reamer
          env:
            - name: HOME
              value: /var/lib/reamer
          volumeMounts:
            - name: reamer-state
              mountPath: /var/lib/reamer/.config/reamer
  volumeClaimTemplates:
    - metadata:
        name: reamer-state
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Mi
```

Each replica consumes one seat, and the seat stays attached to that
ordinal across restarts and rescheduling.

### Do not copy `activation.json` between machines

Activating on one host and distributing the resulting `activation.json`
to others is not supported. A grant is bound to the fingerprint of the
machine that produced it, so a copied file fails on any host that does
not reproduce that fingerprint, reporting `activation is bound to a
different machine` (exit code 4). Each licensed machine activates its
own key, and that activation is what records the machine against your
key server-side.

Treat `activation.json` as a credential regardless: it is a signed
grant. Keep it out of image layers, secrets stores and version control.

### Kubernetes

Reamer Server is a single-machine, core-pinned deployment. The server
process and its strategy processes are pinned to distinct cores on one
box, communicating over local Unix domain sockets. That already provides
the fault isolation a pod-per-process layout would buy, and it is the
only shape in which the UDS path exists at all — containerising the
server and its strategies as separate pods replaces that local socket
with a network hop, which is the one cost this architecture cannot
absorb.

So use the manifest above for a containerised **single** licensed
machine, where it solves hostname stability and state persistence. Do
not read it as a clustering story: scaling Reamer Server means a larger
machine, or another licensed machine, not more replicas of one seat.

### If both halves of the fingerprint are degenerate

A container with no physical NIC reports `first_mac` as
`00:00:00:00:00:00` (see "All-virtual-interface machines" above). The
hostname is then the *only* distinguishing input, which is a further
reason to set it explicitly and uniquely rather than accept a generated
one. Two containers that share a degenerate MAC *and* a hostname produce
the same fingerprint and will collide on activation.

### Verifying before you rely on it

`status --verbose` prints the fingerprint this host computes, and needs
no license to do it:

```bash
reamer-license status --verbose
#   bound machine: <fingerprint recorded in activation.json, or (none)>
#   this machine:  <fingerprint computed from this host right now>
```

Run it inside the container, restart the container, and run it again.
**`this machine` must be identical across the restart.** If it changes,
the hostname is not yet pinned correctly and the pod will fail its
*next* activation, not its current one — which is why this is worth
checking before you build a deployment around it.

Once activated, `bound machine` and `this machine` must also match each
other. They are the two sides of the machine check: `bound machine` is
read from `activation.json`, `this machine` is recomputed from the live
host on every check, and a mismatch is exactly what reports
`activation is bound to a different machine` (exit code 4).

Then confirm the license itself is healthy:

```bash
reamer-license status   # expect: valid  (exit 0)
```

`status` exits non-zero on any non-valid state, so it scripts cleanly as a
container readiness check.

## Term Licenses

Every license is a term license: it carries a term length (`term_days`),
set once at issuance. There is no perpetual, non-expiring license.

Expiration is computed as `first_activated_at + term_days`, not from
issuance time. `first_activated_at` is set once, on the very first
successful activation of a key, and never reset afterward — deactivating
and reactivating (even on a new machine) does not grant a fresh term.

A license past its expiration is refused outright at `activate` time
(new activations only), and the client enforces the same expiration
offline afterward — a locally edited clock or activation file cannot
extend it. `expires_at` is part of the server-signed grant, so tampering
with it locally breaks signature verification.

An already-running process is never killed by license expiration. Reamer Labs
checks the license periodically while running and emits escalating
warnings as expiration approaches and after it passes, but a live process
keeps running until it exits on its own. Expiration only blocks the
**next** process start.

To renew, buy a new key at reamerlabs.com/pricing.html before the term
expires. Nothing renews automatically. A renewal ships the latest release;
an update released mid-term is sent on email request.

### Renew on the same machine

A machine holds one key per product, so the old key must be released
before the new one can be activated:

1. ```
   reamer-license deactivate
   ```
   Add `--product research` or `--product server` if the machine holds
   seats for both.
2. ```
   reamer-license activate <new-license-key>
   ```

The new key's term starts the moment it is activated, so switching early
overlaps the two terms; make the switch close to the old key's end date.
A process already running is not stopped by the switch.

Between the two commands the machine holds no license for that product.
A running Reamer Server rechecks every 30 seconds, so a recheck that lands
in that window emits a `license.expired` error (at most one, since the
error is paced hourly). That alert is expected
during a renewal; the process keeps trading, and the next recheck after
`activate` finds the new key valid and the errors stop. There is no
separate "restored" event.

Term length is fixed to one of two tiers: a 30-day trial or 1 year.

## Price

One published price per product, per seat, bought online as a one-time
payment:

| Product | 30-day trial | Per seat, per year |
|---|---|---|
| Reamer Research | $225 | $1,800 |
| Reamer Server | $900 | $7,200 |

Research licenses the machine where you develop a strategy. Server
licenses the machine that trades it — the server process and every
core-pinned strategy process beside it on the same box, under one seat.

A Reamer Research key bought online is an individual license: personal to
the one person named at checkout, with no restriction on live or paper
trading. LICENSE.txt, Section 2A, states the terms.

Prices are per seat and additive across machines: three Server machines
are three seats. A machine running both products holds one seat of each
and is billed for both.

Card payment runs through Paddle. An annual licence can also be paid by
bank transfer against a Paddle invoice; see [pricing](/pricing.html). All sales are final.

Seat policy is one machine at a time, same as every license — see
above.

## Business Continuity: Source Escrow

Reamer Labs is a single-maintainer vendor. There is no standing
source-escrow arrangement run or funded by Reamer Labs, and no
third-party escrow agent is retained today.

A customer that wants a continuity safeguard beyond the standard license
terms may set up a **release-triggered source escrow arrangement
independently, at their own cost**, with an escrow agent of their
choosing, naming Reamer Labs as depositor. Reamer Labs will cooperate with
the deposit, verification, and update obligations such an agreement
reasonably requires, for example periodic source deposits and
build-verification sessions.

**Escrow is not a free add-on.** Two separate costs fall on the customer.
First, the escrow agent's own fees, paid by the customer directly to the
agent. Second, an annual escrow-participation fee paid to Reamer Labs, which
covers Reamer Labs' periodic source deposits and build-verification sessions
for the arrangement. This participation fee is in addition to the license
fees, is set per arrangement in the separate escrow agreement, and recurs for
as long as the arrangement is in effect.

This section states the policy Reamer Labs will agree to. It is not a
signed template. The contract terms, the choice of escrow agent, and the
deposit cadence are a per-customer commercial negotiation. The points below
are the positions Reamer Labs holds in that negotiation.

**What the escrow protects.** The arrangement protects the customer's own
running deployment against the loss of the vendor. It is a continuity
safeguard, not a source license. While Reamer Labs supports the product,
the customer has no source rights at all. The deposit stays with the escrow
agent. The customer cannot read it and cannot compel its release.

**What triggers a release.** Reamer Labs will agree only to a defined,
closed list of release conditions, each one an externally verifiable fact.
The list is the following and nothing wider:

1. Reamer Labs enters bankruptcy, liquidation, or an equivalent insolvency
   process, or is dissolved.
2. Reamer Labs ends the product. This is shown by either a formal written
   end-of-product notice from Reamer Labs, or Reamer Labs stopping shipping
   updates and stopping all response to support requests for a continuous
   period the agreement names, where Reamer Labs does not resume within a
   cure period after written notice.

A change of control is not a release condition. If Reamer Labs is acquired
and the acquirer keeps supporting the product on the existing terms, no
release occurs. A release condition can arise only if the acquirer then
meets one of the two points above. Reamer Labs will not sign an arrangement
whose trigger fires on the sale of the business itself, and will not agree
to a trigger worded as a judgment of vendor health, such as "unable to
support" or "material decline," because such a trigger is not verifiable.

The quality, speed, or completeness of support while Reamer Labs continues
to operate the product is not a release condition. A missed response
target, degraded or partial support, or a dispute over support quality does
not release the source. Those matters are handled by the remedies in the
support terms, not by the escrow.

**How a release is controlled.** The arrangement must hold the following
process, so that no party can force a release by asserting a trigger that
has not occurred:

1. The customer cannot self-certify a trigger. A release request goes to
   the escrow agent, not straight to release.
2. On a request, the escrow agent notifies Reamer Labs and opens an
   objection period the agreement names. If Reamer Labs objects within that
   period, the release halts.
3. The escrow agent releases only on documentary proof of a listed
   condition, such as a filed insolvency petition, a company-registry
   record, or Reamer Labs' own written end-of-product notice.
4. If the customer and Reamer Labs disagree on whether a condition
   occurred, the matter goes to arbitration or expert determination named
   in the agreement. The deposit stays locked until that decision. The
   default outcome of a dispute is no release.

**What a release grants.** On a verified release, the source passes to the
customer under a limited license for that customer's internal use only, to
maintain and operate their own deployment. There is **no redistribution**
of the original source or any derivative work, to any party, under any
circumstance. There is no right to resell, sublicense, or build a competing
or redistributable product. The license does not transfer ownership. Reamer
Labs' closed-source posture is a hard condition of the release, and it
survives the release.

An escrow arrangement is set up on request to hello@reamerlabs.com.

## Out of Scope

Reamer Labs does not offer pooled or floating multi-seat licensing (one key,
N concurrent machines, checked out on demand). Each concurrent machine
requires its own key. This is a deliberate simplification: one-key-per-seat
matches per-seat pricing and needs no license-server-side seat-count
bookkeeping beyond the single machine binding described above.
