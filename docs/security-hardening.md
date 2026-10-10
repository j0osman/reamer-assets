---
title: Security hardening record
description: The internal adversarial security pass on the ABI, IPC and licence boundaries of both products: the defects found and how each fix was verified.
group: security
order: 1
product: both
source: SECURITY_HARDENING.md
---

## Scope of this document

This log records the **internal adversarial security pass** already performed
against the ABI and license/IPC boundaries of both products
(Reamer Research, Reamer Server), and the fixes it produced. It is written
for a security review: it names the classes of defect found and
closed, and the verification each fix carries, without exposing source.

This is **internal review**, performed by the party that built the product.
It is not a substitute for an independent third-party audit; no external
attestation has occurred. The findings below sit against a documented
ABI/IPC and license boundary model, summarized in this log.

**Positioning, stated plainly:** the memory-safety and license-path defects
below were found and fixed under an internal pass with regression tests and
sanitizer verification. Concerned about security? Test it yourself against the
kit's binaries and the boundary model summarized below. Any issue found is
fixed immediately. Nothing here claims that external third-party attestation
has occurred.

## Verification standard

Every fix in the memory-safety and cryptographic-path findings below was
verified clean under a fresh **AddressSanitizer + UndefinedBehaviorSanitizer**
build, and each shipped with regression tests exercising the original defect.
Where a finding was characterized rather than code-changed, that is stated.

## Findings found and closed

### S1 — Integer-overflow / out-of-bounds in exogenous-data offsets

The `.exo.bin` sidecar reader stored per-entry pointers computed from
attacker-controllable `offset`/`length` fields without validating them
against the blob. A crafted sidecar could point entries past the blob or
wrap the offset arithmetic, yielding out-of-bounds reads.

**Fixed.** The provider constructor now validates every entry's
`[offset, offset+length)` against the blob size, written as
`offset <= blob_sz && length <= blob_sz - offset` specifically to avoid the
integer-wrap the naive `offset + length <= blob_sz` form inherits.
Regression tests cover all five original cases: offset past blob, oversized
length, offset wraparound, length wraparound, and a valid slice, plus the
entry-count mismatch.

### S2 — Unbounded string read at the inbound ABI boundary

`read_cstr` — the sole inbound conversion for every fixed `char[]` field a
customer's vtable fills — constructed a `std::string` from the raw pointer
with no length bound. A fully-populated field with no NUL terminator read
past the field into adjacent struct memory.

**Fixed.** The scan is now bounded to `REAMER_MAX_STRING_LEN` via `strnlen`;
an unterminated field truncates rather than over-reads. The ABI header now
states the NUL-termination / truncation contract explicitly. A regression
test fills every string field of an inbound event with non-NUL bytes and
verifies truncation with no OOB read.

### S3 — Command argument injection in the license activation HTTP call

The license-activation curl invocation appended a URL after its flags with
no `--` separator; a URL beginning with `-` could be parsed by curl as an
option rather than a positional argument.

**Fixed.** A `--` separator is inserted before the URL, after all
`-H` / `--data-binary` / `Authorization` flags. Verified against a stub curl
capturing the real argv from `reamer-license activate`. The remaining
unpinned-`PATH` curl-resolution risk is recorded as an accepted risk rather
than bundling an HTTP client to remove it.

### S4 — License-server override was reachable on its own

The activation endpoint could be redirected through an environment
variable on its own — a second, undocumented control path.

**Fixed.** The override is now honored only inside our internal test
harness and has no effect in a customer deployment, so activation always
reaches the Reamer Labs license server.

### S5 — Undefined behavior in the vendored Ed25519 signing path

`-fwrapv` was masking signed-overflow / left-shift UB across three files of
the vendored `orlp/ed25519` used for grant-signature verification — carry
propagation in `sc.c` and `fe.c`, an `int32` overflow in `fe_tobytes`'
packing phase, and `select()` in `ge.c`. None had been caught because no
existing test exercised `ed25519_sign` / `ed25519_create_keypair`.

**Fixed.** Each site was corrected with a well-defined unsigned-shift helper
and `-fwrapv` dropped from the target. Added a sign/verify round-trip test
across 200 seeds plus tamper and empty-message cases. The vendored tree is
now **pinned to a known upstream commit** (`b1f19fab`), recorded in the SBOM
generator and in the header, with the source-modification note corrected.

### S7 — Release-archive authenticity

Release verification signed only the `.sha256` checksum sidecar, not the
archive bytes; a replaced archive+checksum pair could pass.

**Fixed.** The archive/zip bytes are now signed directly, in addition to the
checksum signature, so a substituted archive is detected. The release public
key is published out-of-band from the download (documented human/email
channel independent of the archive), closing the key-provenance circularity.
Both are documented in `DEPLOYMENT.md` as additive checks alongside the
existing `.sha256` / `.sha256.sig` steps.

### S8, S9 — Characterized, no code change required

- **S8** (license `last_seen.json` remedy): the remedy was already documented
  in `LICENSING.md` and matches the binary's own error string. Closed with no
  code change.
- **S9** (TPM attestation decision): recorded as a made decision, not a
  pending placeholder.

### Concurrency

Three real data races on boundary-adjacent state were found and fixed by
**ThreadSanitizer**-driven work.

## What this pass does and does not cover

- **Covers:** the inbound ABI conversion boundary (bar and event structs,
  string fields, exogenous sidecars), the license activation / grant-
  verification path and its cryptographic dependency, release-artifact
  authenticity, and boundary-adjacent concurrency.
- **Does not cover:** an independent external firm's adversarial review under
  the customer-artifact-only constraint. No such external attestation has been
  performed. Both products are internal-pipeline software: assess them
  against your own threat model, and report anything you find.

## Pointers

- `VENDOR_QUESTIONNAIRE_PACK.md` — security questionnaire, vulnerability
  disclosure and patch policy, data-handling and business-continuity notes.
- `sbom.cdx.json`, `THIRD_PARTY_NOTICES` — component inventory, including the
  pinned Ed25519 provenance from S5.
