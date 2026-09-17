# tamperextscripts Constitution

## Purpose

Build a focused, GPL-3.0, modern Chromium userscript and export tool for
agentic development workflows, beginning with reliable ChatGPT Markdown, JSON,
HTML, screenshot, and bulk export.

## Principles

### 1. GPL and provenance clarity

The project may reuse code covered by the repository's GPL-3.0 license and
must retain notices. Code, assets, names, and implementation details from
proprietary Tampermonkey releases are outside the project source boundary.

### 2. Reliability over feature breadth

A small workflow that survives browser startup, reloads, tab closure, and
service-worker suspension is more valuable than broad compatibility with every
legacy GM API. New APIs require a concrete user workflow and a test.

### 3. Least privilege

Permissions and host matches must be limited to the supported sites and
features. Any broader permission requires a written security rationale in the
feature specification.

### 4. Lifecycle-safe messaging

Tab disappearance, navigation, back-forward-cache transitions, and extension
context invalidation are expected browser events. Message paths must make these
events bounded, observable, and non-fatal to unrelated tabs.

### 5. Testable boundaries

Metadata parsing, matching, export serialization, message routing, and stale
tab handling must have deterministic tests. Browser tests cover only behavior
that cannot be proven at a lower seam.

### 6. Agent-readable change history

Every substantial capability is specified under `specs/`, planned in a checked
task list, and linked from `docs/ROADMAP.md`. Agents must be able to resume
work by reading those files without relying on chat history.

## Change control

Any change that expands permissions, changes the license/provenance boundary,
or changes the primary export format requires an update to the constitution
and the affected specification before implementation.

