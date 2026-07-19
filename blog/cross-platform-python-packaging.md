---
title: "Cross-Platform Python Packaging"
description: pybind11 has no stable-ABI support, on any version. Shipping one compiled extension that works across four Python minor versions and three operating systems means building the same unmodified C++ many times over and bundling every result into a single wheel, not one clever binary.
date: 2026-07-19
tier: Engineering
---

A pure-Python package is one file tree that works on any interpreter. A compiled extension is not — a `.so` built against Python 3.11's C API is not generally loadable by Python 3.12, and pybind11 (the library Reamer's Python bindings are built on) has no stable-ABI, abi3-style escape hatch on any version. "Works on Python 3.10 through 3.13" cannot mean one binary here. It has to mean building the same unmodified source once per targeted interpreter and shipping every result together.

## What "one wheel" actually contains

A Reamer wheel bundles multiple compiled extension files side by side — one per targeted CPython minor version — plus the pure-Python package files the tests and docs actually import. Nothing about the install process picks between them manually: Python's own import machinery already knows to look for the extension suffix matching whichever interpreter is running (`cpython-312-x86_64-linux-gnu`, and so on), so dropping several correctly-named binaries into the same wheel and letting normal import resolution run is sufficient — no custom selection logic needed at install time, just filenames that follow the convention correctly.

## Where this gets genuinely hard: three operating systems, one build

Linux, macOS, and Windows each have their own rules for what a compiled extension needs to load correctly — different linker behavior, different bundled-library conventions, different minimum-OS-version constraints that have to be pinned consistently across every interpreter version built on that platform, not left to whatever each build happened to pick up. Getting this wrong doesn't fail loudly at build time; it fails at import time, on someone else's machine, in an environment that's inconvenient to reproduce — which is exactly why round-trip verification matters more here than in most packaging pipelines: installing the actual built wheel into a fresh virtual environment and running the real test suite against it, on every targeted platform and interpreter combination, rather than trusting that a successful compile implies a working install.

## The tag that makes one wheel installable everywhere

The wheel filename and its internal metadata use PEP 425's compressed tag format — a single filename encoding every targeted interpreter and platform combination, rather than one wheel per combination. That's what lets `pip install` resolve the correct one of several bundled binaries automatically, on whichever of the targeted Python versions happens to be running, without the user needing to know or care how many separate builds went into producing the file they just downloaded.

---

Full reference: [docs](https://reamerlabs.com/docs) · Installation: [reamerlabs.com/docs#install](https://reamerlabs.com/docs#install)
