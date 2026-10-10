---
title: Vendor questionnaire pack
description: Standing answers for a security questionnaire: architecture, licence checks, encryption, support model, disclosure policy, data handling and escrow.
group: security
order: 2
product: both
source: VENDOR_QUESTIONNAIRE_PACK.md
---

Hand this document as-is to your own infra, risk or procurement team. It
covers both closed-source products — Reamer Research and Reamer Server —
and is the standing answer to the security questionnaire, support model,
vulnerability disclosure policy, data-handling statement, and
business-continuity note a firm's onboarding process asks for.
For the security-hardening record — the internal adversarial pass already
performed and the defects it closed — see `SECURITY_HARDENING.md`.

---

## 1. Security questionnaire

**Architecture and source access.**
Both products are strictly closed-source. Customers never receive the
engine's source, internal headers, or repository access. Each product ships
as a versioned, platform-tagged archive containing a stable public C ABI
header, a precompiled binary library, a license-activation CLI, and
buildable reference integration code — for Reamer Research, `reference-cpp`
and `reference-python`; for Reamer Server, `reference-skeleton` (C++),
`reference-skeleton-rust`, and `reference` (a Go FIX 4.4 integration), plus
the extension-protocol specification — never the private implementation. The archive is self-describing: its
README lists the exact contents a reviewing party receives.

**Authentication and access control.**
Access to the software is gated by an offline, term-limited,
machine-locked license (Ed25519-signed activation grant). License checks
run locally after activation — no network call is made during normal
operation; the license server is contacted only on explicit activation or
deactivation. See `LICENSING.md`.

**Encryption.**
License validity is verified with an embedded public key against an
Ed25519 signature, entirely offline. Activation/deactivation calls to the
license server use HTTPS. No trading data or strategy logic is ever
transmitted to Reamer Labs — see Section 4.

**Security review posture.**
Both products are internal-pipeline software. An internal adversarial
pass has been performed against the ABI/IPC and license boundaries of both
products: memory-safety and license-path defects were found and fixed, each
with regression tests and AddressSanitizer + UndefinedBehaviorSanitizer
verification. The findings and their fixes are recorded in
`SECURITY_HARDENING.md`, against a documented ABI/IPC and license boundary
model. No independent third-party attestation has been performed, and none is
represented as existing. Concerned about security? Test it yourself against
the compiled artifacts you receive — any issue found is fixed immediately.

**Incident history.**
No security incidents to date (position as of 2026-08-23, the date of
this pack).

---

## 2. Support model

**Channel.** Licensing and technical support requests go to
**support@reamerlabs.com** (see Section 6's contacts table).

**Response time.** Best-effort acknowledgment targets, by severity, measured
in business hours (see **Supported hours** below):

| Tier | Definition | Target acknowledgment |
|---|---|---|
| Sev 1 — Production-down | Licensed live instance down or rejecting all orders | 1 business day |
| Sev 2 — Degraded | Partial functionality lost; a workaround exists | 2 business days |
| Sev 3 — Question / non-urgent | No operational impact (integration, docs, feature ask) | 5 business days |

"Acknowledgment" means triage begins, not resolution. As with the patch
timeline below, this is a single-maintainer vendor, so each figure is a
good-faith, best-effort target, not a contractual SLA in this document — a
signed order form (`legal/`) is where these same figures are committed
contractually. An issue affecting a customer's ability to trade (order flow
blocked, gate misbehaving, broker connector down) is a Sev 1 and is
prioritized ahead of a documentation or feature request.

**Escalation.** The severity tiers above set response targets; they are not
a tiered support organization. There is one support contact, and Reamer Labs
does not claim a support desk it does not have. To escalate an open request,
reply to the same thread marked urgent and state the trading impact; that
reaches the same person faster than opening a second channel.

**Supported hours.** Reamer Labs operates from **09:00–18:00 India Standard
Time (IST, UTC+5:30) on weekdays**, excluding public holidays in Pune,
Maharashtra, and does not offer 24/7 or follow-the-sun desk coverage.
A message sent outside these hours, including one sent immediately after a
market-hours incident, is picked up at the next covered hour rather than
in real time — plan any escalation path that needs faster coverage for a
live incident around that fact rather than assuming it.

**What to send during an incident.** Run
`bin/reamer-diag-bundle --config <path-to-your-config>` (ships alongside
`bin/reamer-license` and `bin/reamer-config-check` — see `DEPLOYMENT.md`)
against the affected instance and attach the resulting JSON file to the
support email. It captures a `/health` and `/metrics` snapshot plus every
ops and intent-flow event still retained on the instance's event bus at
the time it was run, with the monitoring auth token redacted — this
replaces sending whatever log lines you think to paste, and is the single
artifact most likely to shorten the first response.

---

## 3. Vulnerability disclosure and patch policy

Report suspected vulnerabilities to **team@reamerlabs.com**. Do not open a
public issue or disclose a suspected vulnerability publicly before a fix
ships.

- **Acknowledgment:** best-effort, within 2 business days of report.
- **Patch timeline:** best-effort, triaged by severity — this is a
  single-maintainer vendor, so timelines are a good-faith target, not a
  contractual SLA. Critical issues (remote exploit, license-bypass,
  data-integrity) are prioritized first.
- **Coordinated disclosure:** report privately; Reamer Labs will confirm
  receipt, work the fix, and agree a disclosure date with the reporter
  before any public discussion of the issue.

---

## 4. Data-handling statement

Reamer Labs collects the minimum necessary to issue and validate a
license. See the privacy policy at https://reamerlabs.com/privacy for the
full policy; the relevant points for procurement are:

**Collected:** email address (license delivery, renewal notices); the
licensee's name (the one person an individual license is issued to);
the record that the terms were accepted, with version and time; the
license key; and, at activation, a machine fingerprint (a one-way SHA-256
hash of hostname and network hardware address), a server-generated
machine ID and the activation timestamp (license validation).

**Explicitly not collected:** strategy code, backtest configurations, or
trading data; usage telemetry, feature analytics, or crash reports;
payment card or billing details (handled entirely by Paddle, PCI DSS
compliant); physical addresses or phone numbers.

**Where it runs:** Reamer Research and Reamer Server run
entirely on the customer's own infrastructure. All trading data, strategy
logic, and order flow processed by a running instance stays on that
infrastructure — Reamer Labs has no network access to a running instance
and never receives or sees this data. The only network contact between the
software and Reamer Labs is the license activation/deactivation call
described above.

As a standing product position: Reamer Labs is never the broker, never
custodies funds, and never gives investment advice — Reamer Server manages
orders against a broker connection the customer owns, not one Reamer Labs
operates.

---

## 5. Business-continuity note

Reamer Labs is a single-maintainer vendor. There is no standing
source-escrow arrangement run or funded by Reamer Labs, and no third-party
escrow agent is retained today.

A customer that wants a continuity safeguard beyond the standard license
terms may set up a **release-triggered source escrow arrangement
independently, at their own cost**, naming Reamer Labs as depositor. This
is a continuity safeguard for the customer's own operations, not a source
license. It grants no redistribution or derivative-work rights.

The release is gated on a defined, closed list of triggers, each an
externally verifiable fact: Reamer Labs entering insolvency or dissolution,
or Reamer Labs ending the product. An acquisition where the buyer keeps
supporting the product is not a trigger. The quality or speed of support
while Reamer Labs operates the product is not a trigger. A release passes
the source under an internal, maintenance-only license and never transfers
ownership. Full policy, including the release-control process and how any
such arrangement is negotiated: `LICENSING.md`, "Business Continuity:
Source Escrow."

---

## 6. Contacts

| Purpose | Contact |
|---|---|
| Purchases / commercial | hello@reamerlabs.com |
| Licensing / technical support | support@reamerlabs.com |
| Vulnerability reports / security correspondence | team@reamerlabs.com |
| Privacy / data requests | privacy@reamerlabs.com |
