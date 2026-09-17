# Agent Working Agreement

## Project intent

`tamperextscripts` is a GPL-3.0 successor project derived from the public
Tampermonkey 2.9-era source. It targets modern Chromium and prioritizes a
small, reliable ChatGPT export workflow before broad userscript-manager
compatibility.

## Required reading

Before implementation work, read:

1. `docs/ROADMAP.md` for sequencing and scope.
2. The active feature specification under `specs/`.
3. The active implementation plan under `docs/superpowers/plans/` when one exists.
4. `.specify/memory/constitution.md` for project invariants.

## Working rules

- Preserve GPL-3.0 notices and identify the v2.9-derived portions.
- Keep the new extension's name, icons, and UI distinct from Tampermonkey.
- Treat the existing working Tampermonkey installation as a reference only;
  never modify the user's Chrome profile from this repository.
- Prefer narrow permissions and explicit host matches over `<all_urls>`.
- Write a failing test before production code for every behavior change.
- Keep browser-facing code behind small modules that can be tested without a
  live browser where practical.
- Treat closed, reloaded, or navigated tabs as normal lifecycle events.
- Do not swallow unexpected errors; only suppress known stale-tab conditions.
- Update the relevant spec and roadmap status when behavior or scope changes.

## Completion evidence

An implementation is complete only when the relevant automated checks pass, the
extension can be built from a clean checkout, and the user-facing workflow has
been verified in a disposable Chromium profile.

