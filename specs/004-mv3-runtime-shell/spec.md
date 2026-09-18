# Feature Specification: Modern MV3 Runtime Shell

**Feature ID:** 004
**Status:** proposed
**Target milestone:** M1

## Goal

Provide a loadable, reproducible Manifest V3 extension shell that works on
current Chromium without depending on the legacy persistent background page.

## Requirements

1. The modern build MUST use Manifest V3.
2. New background coordination MUST run in a service worker.
3. The build MUST produce a clean unpacked extension in `dist/extension/`.
4. The manifest MUST declare only the capabilities used by implemented code.
5. Content scripts, service-worker modules, and UI assets MUST have explicit
   entry points.
6. The build MUST fail when an entry point or manifest field is missing.
7. Development and production builds MUST be distinguishable in metadata.
8. The extension MUST load in a disposable Chrome profile on Chrome 153+.
9. The legacy v2.9 source MUST remain available for provenance and research but
   MUST NOT be silently mixed into the MV3 output.

## Acceptance criteria

- `build` succeeds from a clean checkout after dependencies are installed.
- Chrome loads the unpacked output without a manifest error.
- The service worker starts, suspends, and restarts without losing durable
  configuration.
- The output contains no proprietary Tampermonkey 5.x code or assets.

