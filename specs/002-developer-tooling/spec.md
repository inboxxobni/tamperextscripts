# Feature Specification: Developer Tooling and Verification Infrastructure

**Feature ID:** 002
**Status:** proposed
**Target milestone:** M1 foundation, then continuous maintenance

## Problem

The repository is a legacy shell-built extension with no modern dependency
manifest, test runner, local development server, structured diagnostics, or
continuous verification contract. That makes browser regressions difficult to
reproduce and makes parallel agent work risky: an agent cannot quickly know how
to build, test, inspect logs, or package evidence for another agent.

## Goal

Provide a small, reproducible developer loop for the modern extension:

```text
edit → lint/type-check → unit tests → build/watch → browser tests
      → redacted diagnostics → CI artifacts
```

The tooling must support both human contributors and unattended agents without
requiring access to the user's live Chrome profile.

## User scenarios

### Scenario 1: Start development

Given a clean checkout with dependencies installed, when a contributor runs the
documented development command, then the extension is built into `dist/` and a
local development server or file-watch loop reports rebuild status clearly.

### Scenario 2: Diagnose a browser failure

Given a failing extension test or disposable browser run, when the contributor
requests diagnostics, then the tooling emits timestamped, structured events
with category, severity, component, and correlation ID, while redacting
conversation contents, cookies, tokens, and sensitive query parameters.

### Scenario 3: Verify a change

Given a code change, when the contributor runs the verification command, then
format/lint checks, unit tests, build checks, and browser tests run in a
deterministic order and return a nonzero exit code for any required failure.

### Scenario 4: Hand off to another agent

Given a failed or successful verification run, when an agent packages the
result, then the output identifies the exact command, browser/build version,
test counts, failure summaries, and artifact paths without embedding secrets
or private conversation data.

## Functional requirements

1. The project MUST declare modern development dependencies and scripts in one
   standard project manifest.
2. The project MUST provide these commands or equivalent documented commands:
   `dev`, `build`, `test`, `test:unit`, `test:browser`, `lint`, and `verify`.
3. `build` MUST produce a clean, loadable unpacked extension under `dist/`.
4. `dev` MUST watch source changes and report rebuild failures without leaving a
   stale successful build indistinguishable from the current build.
5. Browser tests MUST launch against a disposable profile and synthetic fixture
   pages by default.
6. Browser tests MUST support a headed mode for human inspection and a headless
   mode for CI.
7. The logger MUST support levels `debug`, `info`, `warn`, and `error`.
8. Every diagnostic event MUST include an ISO timestamp, component, event name,
   severity, and correlation ID.
9. Diagnostic output MUST redact cookies, authorization headers, API keys,
   conversation bodies, and sensitive URL query values.
10. Expected stale-tab and closed-context events MUST be distinguishable from
    unexpected extension failures.
11. `verify` MUST write a machine-readable summary and a human-readable report
    to a predictable artifacts directory.
12. CI MUST run the same required verification command used locally, subject to
    the browser availability of the runner.
13. CI artifacts MUST include test reports, build output metadata, and redacted
    diagnostics on failure.
14. The tooling MUST document how an agent selects one focused test and how it
    runs the complete verification loop.

## Non-functional requirements

- A clean verification run must be repeatable without the user's Chrome
  profile, account, or private data.
- Unit tests must finish quickly enough for every implementation checkpoint.
- Log volume must be bounded by default; verbose tracing must be opt-in.
- A failing browser test must preserve a screenshot, console log, and test
  report when the runner supports those artifacts.
- The tooling must work on macOS and in a Linux-based CI environment.
- Build output must be reproducible from a clean checkout after dependencies
  are installed.

## Proposed components

### Project manifest and scripts

Use one standard JavaScript project manifest as the command authority. The
scripts call small shell or JavaScript helpers rather than duplicating command
logic across agent instructions.

### Logger

Create a dependency-light logger with a stable event shape:

```json
{
  "timestamp": "2026-09-18T12:00:00.000Z",
  "level": "info",
  "component": "tab-messenger",
  "event": "tab-stale",
  "correlationId": "export-123",
  "data": { "tabId": 42, "reason": "removed" }
}
```

Tab IDs may be retained for diagnosis; page content, cookies, full URLs, and
credentials may not.

### Build/watch server

The development command watches focused modern source directories, rebuilds
`dist/`, and serves fixture/test assets locally when browser tests need a URL.
The server must report its bound address and stop cleanly on interrupt.

### Test harness

The harness separates unit tests from browser tests. Browser tests use a
disposable profile, fixture HTML, deterministic downloads, console collection,
and failure artifacts. Tests for the live ChatGPT site are optional diagnostics,
never required CI gates.

### Verification reporter

The reporter records commands, durations, pass/fail status, versions, and
artifact paths in JSON plus a concise Markdown summary. It must preserve the
first useful failure and avoid dumping entire logs into terminal output.

## Error handling

- Tool commands fail fast on missing dependencies or invalid build output.
- Browser cleanup runs after both success and failure.
- Logger failures never hide the original extension failure.
- Redaction runs before terminal output and artifact persistence.
- CI reports unavailable browser capabilities separately from product test
  failures.

## Acceptance criteria

- A new contributor can discover all required commands from `README.md` and
  `AGENTS.md`.
- `npm run build` or its documented equivalent produces `dist/` from a clean
  checkout.
- `npm run test:unit` runs deterministic unit tests.
- `npm run test:browser` runs fixture-based browser tests in a disposable
  profile and preserves failure artifacts.
- `npm run verify` runs the complete local gate and writes JSON/Markdown
  summaries.
- A test proves that sensitive fields are redacted from diagnostics.
- A test proves that stale-tab events are classified as expected lifecycle
  outcomes rather than uncaught errors.
- CI configuration invokes the same verification gate and uploads redacted
  artifacts.

