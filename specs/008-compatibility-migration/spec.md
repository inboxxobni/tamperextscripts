# Feature Specification: Compatibility and Script Migration

**Feature ID:** 008
**Status:** proposed
**Target milestone:** M4

## Goal

Allow useful custom scripts to migrate from Tampermonkey without recreating the
entire proprietary product or hiding unsupported behavior.

## Requirements

1. Provide import and export for the supported userscript source format.
2. Provide a script list with explicit enable/disable state.
3. Support only GM APIs implemented behind tested interfaces.
4. Report unsupported `@grant`, `@require`, and metadata features before a
   script is enabled.
5. Preserve script source and metadata locally during migration.
6. Provide a migration guide using the user's ChatGPT exporter as the reference
   conversion.
7. Keep compatibility adapters separate from the core runtime.
8. Add one specification or test before adding each new compatibility API.

## Acceptance criteria

- The current exporter script can be imported into a disposable profile.
- Unsupported APIs produce an actionable report rather than partial silent
  execution.
- Enable/disable state survives worker restart.
- Exported scripts can be round-tripped without source loss.

