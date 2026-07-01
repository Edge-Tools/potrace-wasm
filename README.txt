potrace-wasm — Corresponding Source mirror
==================================================

This repository publishes the Corresponding Source for the WebAssembly
build of potrace (license: GPL-2.0-or-later) used in edgetools.io.

Contents
  build/      our build recipe: Dockerfile + helper scripts/config/patches.
              Rebuild with:  docker build build/
  upstream/   the exact upstream source archive(s) the build fetched,
              byte-identical and sha256-verified (see below).

Upstream sources:
  potrace.tar.gz
    https://potrace.sourceforge.net/download/1.16/potrace-1.16.tar.gz
    sha256 be8248a17dedd6ccbaab2fcc45835bb0502d062e40fbded3bc56028ce5eb7acc
