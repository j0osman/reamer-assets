---
title: Supported platforms
description: Operating systems, architectures and glibc floors for Reamer Research and Reamer Server, and how to check a binary against your host.
group: operations
order: 9
product: both
source: SUPPORTED_PLATFORMS.md
---

The build and toolchain matrix Reamer Research and Reamer Server release
binaries actually run on. Every number below is asserted against the
shipped binary on every release — the floors are read back out of the
built artifact's dynamic symbols, not copied from the build host.

This file ships in both product kits and covers both products, because
the two floors differ and the difference matters when you run them on one
image. See "Running both products on one image" below.

## Reamer Research

| | |
| --- | --- |
| **OS** | Linux (x86-64), macOS (arm64), Windows through WSL2 (x86-64) |
| **Linux glibc floor** | **2.35** — the shipped `libreamer_research.so` requires no `GLIBC_` symbol newer than 2.35 |
| **macOS deployment target** | 11.0 (Big Sur), Apple Silicon |
| **C++ standard** | C++20 |
| **Windows** | Through WSL2 only, using the Linux x86-64 kit. No native Windows build |

Distributions meeting the 2.35 floor include Ubuntu 22.04 and later,
Debian 12 and later, and RHEL 9 and later.

### Windows through WSL2

On Windows, Reamer Research runs inside WSL2 from the same Linux x86-64
kit. Nothing is installed on the Windows side. The distribution inside
WSL2 must meet the same 2.35 glibc floor, so Ubuntu 22.04 or later works.

- **WSL2, not WSL1.** WSL1 has no Linux kernel and is not supported.
  `uname -r` inside the distribution ends in `-WSL2` on WSL2.
- **The licence binds to the WSL machine ID.** WSL2 gives its VM a new MAC
  address on every restart, so under a WSL kernel the machine fingerprint
  uses the distribution's `/etc/machine-id` in place of the MAC. An
  activation survives `wsl --shutdown` and a Windows reboot. Unregistering
  or reinstalling the distribution creates a new machine ID, so deactivate
  first, as you would before moving to a new machine. See
  [Licensing](/docs/licensing.html).
- **Keep data inside the WSL filesystem** (`~/...`), not under `/mnt/c`.
  Files under `/mnt/c` cross the Windows filesystem bridge and are much
  slower to read.

A Windows runner with Ubuntu 24.04 under WSL2 checks this: it activates a
key, runs the quickstart, restarts the WSL VM, and requires the same
fingerprint, a valid licence and byte-identical output after the restart.

## Reamer Server

| | |
| --- | --- |
| **OS** | Linux (x86-64) |
| **Linux glibc floor** | **2.39** — higher than Reamer Research; see below |
| **C++ standard** | C++17 |
| **macOS / Windows** | Not supported targets |

Distributions meeting the 2.39 floor include Ubuntu 24.04 and later and
Debian 13 and later. RHEL 9 (glibc 2.34) does **not** meet it.

The server's floor is higher because `libreamer_server_core.a` is built
against glibc 2.38 headers and carries undefined references to the C23
`strtol` family introduced there (`__isoc23_strtol`, `__isoc23_strtoll`,
`__isoc23_strtoul`, `__isoc23_strtoull`). Any binary linking the archive
inherits those references, so a host below 2.38 fails at link or load.
2.39 is the supported floor because that is the glibc the shipped
archives are built and tested against; 2.38 is the hard technical
minimum. Confirm for yourself against the kit you received:

```bash
nm -u lib/libreamer_server_core.a | grep -o '__isoc23_[a-z]*' | sort -u
```

## Running both products on one image

**The higher floor governs.** An image hosting both Reamer Server and
Reamer Research needs **glibc 2.39 or newer**. Reamer Research alone runs
on 2.35; adding Reamer Server to that image does not lower the server's
requirement.

Check a candidate image before you build on it:

```bash
ldd --version | head -1
```

And confirm what a shipped artifact actually demands:

```bash
objdump -T lib/libreamer_research.so \
  | grep -oP 'GLIBC_\K[0-9]+\.[0-9]+' | sort -V | tail -1
```

The value printed is the highest glibc symbol version that artifact
requires. If it exceeds your host's glibc, the binary will fail to load.

## Out of scope

No other OS, architecture, or libc combination is built, tested, or
supported. This is a deliberate set — Linux
x86-64 for both products, plus macOS arm64 for Reamer Research's local
research workflow — not an exhaustive one. Running outside this matrix is
possible but unverified and unsupported.
