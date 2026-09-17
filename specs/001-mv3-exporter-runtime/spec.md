# Feature Specification: MV3 Exporter Runtime

**Feature ID:** 001
**Status:** proposed
**Target milestone:** M1–M3

## Problem

The working ChatGPT export workflow currently depends on Tampermonkey 5.5.0.
On Chromium 153, Tampermonkey can report repeated `No tab with id` errors and
can delay userscript injection or GM calls after browser startup. The public
Tampermonkey repository contains GPL source only through the 2.9-era code, so
the current implementation cannot be directly patched from this repository.

We need a small, independently maintainable GPL-3.0 extension that can run the
export workflow reliably on modern Chromium.

## User scenarios

### Scenario 1: Load the exporter

Given a disposable Chromium profile with the extension loaded, when the user
opens a supported ChatGPT conversation, then the exporter controls become
available without waiting for a background page to remain permanently alive.

### Scenario 2: Export one conversation

Given a loaded ChatGPT conversation, when the user selects Markdown, JSON, HTML,
plain text, or Screenshot, then the extension creates the requested output
without navigating away from the conversation.

### Scenario 3: Export all conversations

Given the user has access to the supported ChatGPT conversation list, when the
user selects Export All, then the extension processes conversations one at a
time, reports progress, and continues past a conversation that disappears or
changes during processing.

### Scenario 4: Survive tab lifecycle changes

Given an export or injection request is in flight, when its tab is closed,
reloaded, navigated, or moved into the back-forward cache, then the request is
cancelled or marked stale without an unhandled promise rejection and without
blocking other tabs.

### Scenario 5: Run with narrow scope

Given a page outside the supported ChatGPT origins, when the page loads, then
the extension does not inject exporter code or request page data.

## Functional requirements

1. The extension MUST use Manifest V3 for the modern Chromium build.
2. The extension MUST use a service worker rather than a persistent background
   page for new runtime code.
3. The extension MUST expose the six proven export actions: Copy Text,
   Screenshot, Markdown, HTML, JSON, and Export All. A settings action MAY be
   included when needed by the exporter.
4. The extension MUST keep export serialization separate from DOM discovery so
   formats can be tested without a browser.
5. The extension MUST treat a missing tab as a terminal stale-request result,
   not as an unhandled error.
6. The extension MUST remove or invalidate tab-scoped state when Chrome reports
   that the tab was removed or navigated.
7. The extension MUST bound retries and MUST NOT poll all browser tabs
   continuously.
8. The extension MUST restrict content-script matches and host permissions to
   the supported ChatGPT origins selected for the first release.
9. The extension MUST preserve user data locally and MUST NOT upload
   conversation contents.
10. The extension MUST provide a reproducible unpacked build for development.

## Non-functional requirements

- GPL-3.0 licensing and provenance notices remain visible in the project.
- The first supported browser is Chromium-based Chrome 153 or newer.
- A clean build must run without network access after dependencies are present.
- Unit tests must be deterministic and cover stale-tab behavior.
- Browser tests must use a disposable profile and synthetic fixture pages for
  sensitive conversation data.
- Unexpected failures must remain observable through structured, redacted
  diagnostics suitable for an upstream bug report.

## Proposed architecture

```text
ChatGPT page
    │ DOM extraction / UI actions
    ▼
content/exporter adapter ──> pure serializers ──> download or clipboard
    │
    └── lifecycle-safe message client
             │
             ▼
       MV3 service worker
             │
             └── tab registry with removal/navigation invalidation
```

The first implementation should live in new focused modules rather than
rewriting the legacy files in place. The legacy source remains available for
provenance and compatibility research. New code must have a distinct extension
ID, name, icons, and user-facing copy.

## Error handling

- `No tab with id`, `Receiving end does not exist`, and equivalent tab-lifecycle
  failures are recorded as stale-request outcomes at the message boundary.
- Permission, serialization, filesystem/download, and unexpected programming
  failures remain visible to the caller and test harness.
- Bulk export records per-conversation success/failure and continues only for
  failures classified as isolated stale or unavailable tabs.
- Diagnostics must exclude conversation text, cookies, tokens, and full URLs
  containing sensitive query data.

## Compatibility and migration

The runtime initially supports only the exporter capabilities needed by the
user's current scripts. Each additional GM API requires a separate spec or
explicit amendment to this one. The project does not promise compatibility
with proprietary Tampermonkey 5.x internals.

## Acceptance criteria

- A disposable Chrome profile loads the MV3 extension from a clean checkout.
- All six export actions are visible on a supported fixture conversation.
- Markdown, JSON, HTML, text, and screenshot outputs are generated with stable
  names and expected contents.
- A test closes/reloads/navigates the target tab during an in-flight request and
  observes no unhandled rejection.
- A non-ChatGPT fixture page receives no exporter injection.
- The test suite and clean build command pass from a fresh checkout.

