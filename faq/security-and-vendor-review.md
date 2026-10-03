---
title: Is it secure? Can it pass my firm's vendor review?
description: Each kit ships the documents a vendor review usually asks for, including a filled-in security questionnaire, a software bill of materials and signed release files. The security testing behind them was done by Reamer Labs itself; no outside firm has audited either product. Whether that is enough depends on your firm's policy, and you can test the binaries yourself.
stage: 5
order: 40
product: both
next: customer-reviews, single-maintainer-risk, data-privacy, are-benchmarks-verified
date: 2026-10-03
---

Each kit ships the documents a vendor review usually asks for, including a filled-in security questionnaire, a software bill of materials and signed release files. The security testing behind them was done by Reamer Labs itself; no outside firm has audited either product. Whether that is enough depends on your firm's policy, and you can test the binaries yourself.

## What ships for the review

- **A vendor questionnaire pack,** written to send as-is to an infrastructure, risk or procurement team. It covers architecture, access control, encryption, the support model, vulnerability disclosure and patch policy, data handling and business continuity.
- **A software bill of materials** in CycloneDX format, listing every third-party component and its version. There are only a few: a JSON library, a networking library, a pinned Ed25519 signing library, and the system's own curl for activation.
- **Signed releases.** Every download comes with a checksum and two Ed25519 signatures, one over the checksum and one over the archive itself. The deployment guide shows how to verify them and how to confirm the signing key through a channel separate from the download.
- **A security hardening log** recording the defects found and fixed at the C interface and in the licence code, such as out-of-bounds reads and an argument-injection risk in activation. Each fix has a regression test and was checked with memory and undefined-behaviour sanitisers.

## What it does not have

- **No outside audit or certification.** The hardening was an internal review by the people who built the product, and the log says so.
- **No source code.** Both products are closed source. Reviewers get the public header, the compiled library and the documents, not the engine's implementation.
- **No support desk.** One person handles support and security reports, on best-effort targets rather than a contractual SLA unless one is agreed in an order form.

## What helps the review

Both products run inside your network and send nothing to Reamer Labs beyond licence activation, which limits what a reviewer has to assess. See [Does my strategy code or data ever leave my machine?](/faq/data-privacy.html) For a firm buying Reamer Server, the [For firms](/firms.html) page covers order forms and procurement.
