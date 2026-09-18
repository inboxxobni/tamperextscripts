# Feature Specification: Release and Upstream Collaboration

**Feature ID:** 009
**Status:** proposed
**Target milestone:** M5

## Goal

Make the project reproducible for contributors and useful to upstream
maintainers investigating Chromium 153/Tampermonkey lifecycle failures.

## Requirements

1. Every important checkpoint MUST produce a commit with a focused message.
2. Every important checkpoint MUST be pushed to the configured GitHub remote.
3. Releases MUST include source provenance, license information, supported
   Chromium versions, known limitations, and reproducible build instructions.
4. CI MUST run the documented verification gate on every pull request.
5. Failed CI runs MUST preserve redacted logs, screenshots, console output, and
   test summaries when available.
6. The project MUST maintain a minimized stale-tab/startup reproduction based
   on the observed Chromium 153 failure.
7. Upstream reports MUST include exact versions, reproduction steps, timings,
   expected/actual behavior, and redacted diagnostics.
8. The project MUST distinguish a clean-room successor fix from a patch to
   proprietary Tampermonkey 5.x code.
9. Public release artifacts MUST use the new project's identity and extension
   ID.

## Acceptance criteria

- A new contributor can build and verify from the documented instructions.
- CI runs the same required checks as local verification.
- A maintainer can reproduce the stale-tab failure from a committed fixture or
  script without access to private conversation data.
- A release candidate passes the full verification checklist and has a complete
  provenance note.

