# Feature Specification: Userscript Engine and Lifecycle Safety

**Feature ID:** 005
**Status:** proposed
**Target milestone:** M2

## Goal

Implement the smallest reliable userscript runtime needed by the exporter and
future custom scripts, with explicit handling for modern Chromium tab and
service-worker lifecycle events.

## Requirements

1. Parse `@name`, `@namespace`, `@version`, `@match`, `@include`, `@exclude`,
   `@run-at`, `@grant`, and `@require` only when each is implemented and tested.
2. Match URLs deterministically and reject unsupported origins before injection.
3. Support `document-start` injection for approved scripts.
4. Keep registration and execution state separate from tab-scoped state.
5. Invalidate tab state on tab removal, navigation, and extension context
   invalidation.
6. Treat `No tab with id`, `Receiving end does not exist`, and equivalent
   lifecycle errors as bounded stale-request outcomes.
7. Unexpected errors MUST remain visible through the diagnostic logger and test
   result.
8. Every message request MUST have a correlation ID and bounded completion path.
9. The engine MUST not poll all tabs continuously.
10. The engine MUST survive service-worker suspension and restart.

## Acceptance criteria

- Metadata and matching tests pass for supported fields and reject unsupported
  behavior clearly.
- A browser regression test closes, reloads, and navigates tabs during in-flight
  requests with zero unhandled rejections.
- A worker restart does not create duplicate registrations.
- An unrelated page receives no userscript injection.

