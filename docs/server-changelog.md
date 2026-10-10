---
title: Reamer Server changelog
description: Every customer-visible change to Reamer Server by release, including ABI version bumps.
group: changelogs
order: 2
product: server
source: CHANGELOG.md
---

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Every
entry below describes a customer-visible change to the Reamer Server
product — the ABI, the shipped reference material, or the kit
contents — not internal refactors. Updated at release, alongside the tag.

## [4.5.0] - 2026-10-09

**ABI bump: `REAMER_ABI_VERSION` 3 → 4. Additive.** Every ABI 3 field keeps
its type, name and offset, the vtables are unchanged, and a strategy or
relay that sends ABI 3 frames needs no change. Rebuild your gate and
connector against the ABI 4 header to read the new fields.

### Added
- **Order fields on `ReamerIntent`**, appended after `expire_time`:
  attached exits (`take_profit`, `stop_loss`, `stop_loss_limit`,
  `stop_loss_trailing_offset`), `trailing_offset`, `offset_type` (price,
  basis points, ticks), `limit_offset`, `trigger_type` (last, bid/ask, mid,
  mark, index), `display_quantity` (iceberg), `min_quantity`, `exec_flags`
  (post-only, reduce-only, hidden, all-or-none), `contingency_type` (OCO,
  OTO, OUO) with `linked_intent_id`, `account` and `destination`.
- **User-defined payload** on `ReamerIntent`: `payload_schema`,
  `payload_len` and `payload`, up to `REAMER_MAX_PAYLOAD_LEN` (1024) bytes
  that core passes to your gate and connector untouched.
- `ReamerReplaceRequest` gains `new_take_profit`, `new_stop_loss` and
  `new_trailing_offset`.
- `REAMER_SIDE_SELL_SHORT`; order types `TRAILING_STOP`,
  `TRAILING_STOP_LIMIT`, `MARKET_IF_TOUCHED`, `LIMIT_IF_TOUCHED`,
  `MARKET_TO_LIMIT`; time in force `AT_THE_OPEN` and `AT_THE_CLOSE`.
- **Exit tracking.** An accepted order with `take_profit` or `stop_loss`
  set makes core track its exits as orders `<intent_id>:tp` and
  `<intent_id>:sl`, so the connector's reports for them reach the strategy
  and the strategy can cancel or replace them by id. The connector places
  the exits at the venue.
- Both strategy wire protocols carry the new fields in an optional
  extension block at the end of the NewOrder and Replace frames. A frame
  without it decodes as before. `EXTENSION_PROTOCOL.md` has the layout.
- Reference relay: JSON fields for every new field (`payload` as base64)
  and the new side, order type and time-in-force spellings. The reference
  gate rejects an order that uses a field the reference FIX broker does
  not implement.
- Rust skeleton: ABI 4 types.

This release also carries the Reamer Research ABI 7 changes (see the
Reamer Research changelog), including `reamer_relay_*`, a strategy relay
that sends research orders, brackets included, to this server's
local-mode strategy socket.

## [4.4.0] - 2026-10-09

No change to the Reamer Server binary or ABI. `reamer_server_abi.h` remains
at ABI 3. The version moves with the shared release number; this release
carries the Reamer Research ABI 6 changes (see the Reamer Research
changelog).

## [4.3.2] - 2026-10-03

### Fixed
- **`license.expiring` fired every 30 seconds in the last 14 days of a
  term.** The run loop remembered only the last threshold it emitted, so
  once two thresholds had been crossed it alternated between them on every
  recheck (30, 14, 30, 14, …). Each `license_warning_days` threshold now
  fires exactly once, a process started inside several thresholds fires
  only the smallest, and the thresholds may be listed in any order, as
  `CONFIG.md` and `MONITORING.md` already described.

### Added
- **`LICENSING.md` "Renew on the same machine"**: deactivate the old key,
  then activate the new one, and the one `license.expired` a running
  server may emit between the two.

## [4.3.1] - 2026-09-17

No change to the Reamer Server binary or ABI. `reamer_server_abi.h` remains at
ABI 3 and the core library is byte-identical to 4.3.0. This release corrects
documentation figures against the raw EPYC 9575F artifacts and adds a
versioning policy document.

### Fixed
- **`BENCHMARK.md` figures reconciled with the raw EPYC 9575F data.** The
  concurrent-strategy sweep's peak-throughput row was labeled N=32; the
  219,652 orders/sec peak is at N=34 (true N=32 is 218,116). The pinning
  section's run-to-run spread is corrected to 1.28x unpinned collapsing to
  1.02x pinned (was stated approximately as 1.30x to 1.01x), the raw N=62
  runs are the measured values, and the cpu-migration reduction is 118 to 63
  (47 percent, was stated as ~40 percent). The median pinning gain is
  reframed as load-dependent and not bankable, with the variance collapse as
  the reliable claim.

### Added
- **A written ABI versioning policy** for the server ABI. It states the
  contract: the surface only grows, and a header and library version mismatch
  is caught at compile time by the `abi_layout_guard` static_asserts rather
  than by a runtime fallback. It records the git-evidenced version history and
  sets support to the latest version only, communicated through this changelog
  at release.

## [4.3.0] - 2026-09-13

No change to the Reamer Server binary or ABI. `reamer_server_abi.h` remains at
ABI 3 and the core library is byte-identical to 4.2.0. This release carries
new benchmark evidence and the documentation built on it; the version moves
with the shared release number (`kReamerVersion`, one number across both
products).

### Changed
- **`BENCHMARK.md` now includes a many-core verification section** measured on
  a 64-core AMD EPYC 9575F. It confirms the concurrent-strategy latency elbow
  tracks physical core count — previously stated as an expectation to verify,
  now backed by data (clean-latency region and throughput plateau extend to
  roughly one strategy per physical core; peak plateau ~200–220k orders/s).
  The ABI-only end-to-end P99 on this hardware is ~12µs, so the earlier
  laptop-only "P99 occasionally exceeds 100µs under burst" caveat does not
  reproduce there.
- **New tuning guidance from the same run:** the event ring buffer is a
  cache-fit knob (the shipped 16 MB default sits at the measured throughput
  peak; sizing it up costs throughput, it does not add it), and pinning
  strategy threads to cores is a run-to-run *repeatability* lever (collapses
  unpinned throughput spread from ~1.3× to ~1.0×), not a throughput lever.

## [4.2.1] - 2026-09-12

No change to the Reamer Server binary. The version moves with the shared
release number (`kReamerVersion`, one number across both products) for a fix
that is entirely inside Reamer Research's Python reference binding — see that
product's changelog. `reamer_server_abi.h` remains at ABI 3, the core
library is byte-identical to 4.2.0, and only the kit's version stamp and the
`server-kit-<version>` filename change.

### Changed
- **The kit now ships `SECURITY_HARDENING.md` in place of the
  security-review statement of work.** The statement of work was a
  forward-looking scope for an independent external firm review; the
  hardening log is the record of the internal adversarial pass already
  performed and the memory-safety and license-path defects it found and
  fixed, each with sanitizer and regression-test verification.

## [4.2.0] - 2026-09-09

The kit now ships `reference/`, a working reference spine, alongside
the existing shape-only skeletons. It is a FIX 4.4 session, a broker
connector, a strategy relay and an event-bus reader, exercised by its own
suite (96 tests, 0 failures) against ABI 3. `demo-trade.sh` takes the
extracted kit to a filled order over real FIX in about 13 seconds,
including the Go build.

The two tiers answer different questions: the skeletons confirm the shape
of `reamer_server_run()` wiring; the spine shows the whole path once it
closes. Neither is required to reach a running server, and neither is a
venue adapter for your venue — the header plus `EXTENSION_PROTOCOL.md`
remain the complete integration contract.

Fixed in the reference connector: `poll_published_events` returned events
read off the shared event bus, which core correctly rejected as echoes
(`connector.echoed_event`). That callback is inbound only. It now drains a
bounded queue of the connector's own transport diagnostics, and a FIX
session drop — invisible to core, since it happens on a socket below the
ABI — is surfaced as `SESSION_DISCONNECTED` on the bus. To consume core's
stream instead, attach a reader directly, as `cmd/event-tail` shows.

`reamer_server_abi.h` remains at ABI 3, and the core library is unchanged.

## [4.1.14] - 2026-09-06

Reamer Research and Reamer Server are licensed separately. A licence key
now activates only its own product's binaries: the product is compiled
into each kit, carried inside the signed grant, and checked against the
minted record at activation, so a Research key cannot activate a Server
build.

Activation is per machine *and* per product. `~/.config/reamer/activation.json`
holds one grant per product, so activating Server on a machine that
already runs Research leaves the Research grant intact. A machine
running both consumes one seat of each.

**Kits built before 4.1.14 cannot activate against the current licence
server, and 4.1.14 kits cannot use a pre-4.1.14 activation file.** This
is a coordinated cutover: re-cut both kits from this tag and re-activate
each machine. Any existing `activation.json` must be discarded — the old
flat format is treated as absent, not migrated.

`DEPLOYMENT.md` no longer documents copying an `activation.json` between
hosts to license container replicas. Every licensed machine runs
`activate` itself; a copied grant fails on any host that does not
reproduce the original's fingerprint. Containerised deployments are
documented as a single licensed machine.

`reamer_server_abi.h` remains at ABI 3, and the core library is
unchanged.

## [4.1.13] - 2026-09-05

`reamer_server_abi.h` remains at its own, independently versioned ABI 3,
and the core library and its reference material are unchanged.

### Added
- **`LICENSE.txt` ships in the kit.** The archive previously carried no
  licence terms at all. The file sits in the kit root and is covered by
  `SHA256SUMS`.
- **Activation requires accepting the terms.** `reamer-license activate`
  now displays `LICENSE.txt` and requires an explicit `accept` before it
  contacts the licence server. Declining consumes no seat, and the
  acceptance record is written only after activation succeeds. Because
  activation was then machine-global, accepting once covered every
  Reamer Labs product on that machine (activation became per product in
  4.1.14).

  For unattended provisioning set **`REAMER_ACCEPT_LICENSE=accept`**.
  Activation with no terminal and no such variable **fails** rather than
  defaulting to acceptance, so existing non-interactive provisioning that
  activates a licence must set it.

  `reamer-license` reads `LICENSE.txt` from the kit root, one level above
  `bin/`. Moving `bin/` out of the extracted tree makes activation fail
  with a message naming the cause.

Earlier releases, 3.0.0 through 4.1.12, are listed in `CHANGELOG.md` in the kit.
