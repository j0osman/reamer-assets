---
title: Which operating systems are supported? Is there a Windows version?
description: Reamer Research runs on Linux x86-64 with glibc 2.35 or later, and on macOS 11 or later on Apple Silicon. Reamer Server runs on Linux x86-64 with glibc 2.39 or later. There is no Windows version of either.
stage: 4
order: 28
product: both
next: integration-time, performance, market-data, research-to-server-move
date: 2026-10-02
---

Reamer Research runs on Linux x86-64 with glibc 2.35 or later, and on macOS 11 or later on Apple Silicon. Reamer Server runs on Linux x86-64 with glibc 2.39 or later. There is no Windows version of either.

## What that means in practice

- **Reamer Research on Linux:** Ubuntu 22.04 or later, Debian 12 or later, or RHEL 9 or later.
- **Reamer Research on a Mac:** any Apple Silicon Mac on macOS 11 (Big Sur) or later. Intel Macs are not supported. The Mac has its own kit.
- **Reamer Server:** Ubuntu 24.04 or later, or Debian 13 or later. RHEL 9 does not meet the floor, because the server's library needs newer C library functions.
- **Both on one machine:** the higher floor applies, so glibc 2.39 or later.

To check a Linux machine, run `ldd --version`; the first line shows its glibc version.

## Not supported

- **Windows.** Neither product is built for it.
- **Linux on ARM,** such as AWS Graviton, and any other processor or C library combination.

Running outside this list may work but is untested and unsupported. If your setup is outside it, such as a Linux virtual machine on Windows, ask support@reamerlabs.com before buying.

## Containers and virtual machines

Both run in containers and virtual machines on a supported Linux. The licence key is tied to a machine's identity, which includes its hostname, so give the container or machine a fixed hostname. Otherwise each restart looks like a new machine and reactivation is refused. The kits include a working container setup.

See [What is Reamer Research?](/faq/what-is-reamer-research.html), [What is Reamer Server?](/faq/what-is-reamer-server.html) and [pricing](/pricing.html).
