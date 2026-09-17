# MV3 Exporter Runtime Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a small GPL-3.0 Manifest V3 runtime that reliably powers the existing ChatGPT export workflow on modern Chromium.

**Architecture:** Keep the legacy v2.9-derived source intact as provenance/reference and add focused modern modules for metadata, lifecycle-safe messaging, ChatGPT extraction, serialization, and packaging. The service worker owns extension coordination; page-facing code owns DOM extraction; pure serializers own output formats.

**Tech Stack:** Manifest V3, JavaScript, Chrome extension APIs, shell packaging, deterministic unit tests, and a disposable Chromium browser test harness.

**Spec:** `specs/001-mv3-exporter-runtime/spec.md`

## Global Constraints

- Use GPL-3.0 project code and preserve provenance notices.
- Do not copy proprietary Tampermonkey 5.x code, assets, branding, or implementation details.
- Use Manifest V3 and a service worker for new runtime code.
- Treat missing, closed, reloaded, or navigated tabs as normal lifecycle outcomes.
- Keep host permissions limited to supported ChatGPT origins.
- Keep conversation contents local; diagnostics must be redacted.
- Write a failing test before production code for each behavior change.
- Keep the current live Chrome/Tampermonkey profile outside the test workflow.

---

### Task 1: Establish modern project layout and build contract

**Files:**
- Create: `extension/manifest.json`
- Create: `extension/background/service-worker.js`
- Create: `extension/content/content-entry.js`
- Create: `extension/ui/README.md`
- Create: `scripts/build-extension.sh`
- Create: `tests/smoke/build-contract.test.js`
- Modify: `README.md`

**Interfaces:**
- Produces an unpacked MV3 directory at `dist/extension/`.
- The build script accepts no required arguments and exits nonzero on a missing input.

- [ ] **Step 1: Write the failing build-contract test**

Assert that the build output contains `manifest.json`, a service worker, and a
content entry file; assert that the manifest has `manifest_version: 3` and a
distinct extension name.

- [ ] **Step 2: Run the test and verify it fails**

Run: `node --test tests/smoke/build-contract.test.js`

Expected: FAIL because `extension/` and `dist/extension/` do not exist.

- [ ] **Step 3: Add the minimal MV3 shell and build script**

Copy only the new extension files into `dist/extension/`, emit a valid
Manifest V3 manifest, and do not package the legacy `src/` tree into the new
extension.

- [ ] **Step 4: Run the test and verify it passes**

Run: `node --test tests/smoke/build-contract.test.js`

Expected: PASS with the manifest and entry-point assertions satisfied.

- [ ] **Step 5: Commit**

Run: `git add extension scripts tests README.md && git commit -m "build: add modern mv3 extension shell"`

### Task 2: Add deterministic metadata parsing and URL matching

**Files:**
- Create: `extension/runtime/metadata.js`
- Create: `extension/runtime/matcher.js`
- Create: `tests/unit/metadata.test.js`
- Create: `tests/unit/matcher.test.js`

**Interfaces:**
- `parseMetadata(source) -> { name, namespace, version, matches, grants, runAt }`.
- `matchesUrl(metadata, url) -> boolean`.

- [ ] **Step 1: Write failing parser and matcher tests**

Cover a valid metadata block, a missing optional field, an invalid metadata
block, exact ChatGPT host matching, and rejection of an unrelated origin.

- [ ] **Step 2: Run the focused tests and verify they fail**

Run: `node --test tests/unit/metadata.test.js tests/unit/matcher.test.js`

Expected: FAIL because the modules and exported functions do not exist.

- [ ] **Step 3: Implement the smallest parser and matcher**

Support only the metadata fields required by the exporter and the URL patterns
needed by the first supported ChatGPT origins. Keep parsing independent of
Chrome APIs.

- [ ] **Step 4: Run the focused tests and verify they pass**

Run: `node --test tests/unit/metadata.test.js tests/unit/matcher.test.js`

Expected: PASS with no unhandled warnings.

- [ ] **Step 5: Commit**

Run: `git add extension/runtime tests/unit && git commit -m "feat: add userscript metadata matching"`

### Task 3: Implement lifecycle-safe tab messaging

**Files:**
- Create: `extension/runtime/tab-messenger.js`
- Create: `extension/runtime/tab-registry.js`
- Create: `tests/unit/tab-messenger.test.js`
- Create: `tests/unit/tab-registry.test.js`
- Modify: `extension/background/service-worker.js`

**Interfaces:**
- `sendToTab(tabsApi, tabId, message) -> Promise<{ status: "sent" | "stale" }>`.
- `TabRegistry.track(tabId)`, `.invalidate(tabId)`, `.has(tabId)`.

- [ ] **Step 1: Write failing stale-tab tests**

Test that a rejected `tabs.sendMessage` with `No tab with id` resolves to
`{status: "stale"}`, while an unrelated error remains rejected. Test that tab
removal invalidates registry state.

- [ ] **Step 2: Run the focused tests and verify they fail**

Run: `node --test tests/unit/tab-messenger.test.js tests/unit/tab-registry.test.js`

Expected: FAIL because the lifecycle modules do not exist.

- [ ] **Step 3: Implement the message boundary and registry**

Catch only known stale-tab failures, attach removal/navigation listeners once,
and ensure callers receive a bounded result instead of an unhandled promise.

- [ ] **Step 4: Run the focused tests and verify they pass**

Run: `node --test tests/unit/tab-messenger.test.js tests/unit/tab-registry.test.js`

Expected: PASS, including the unexpected-error assertion.

- [ ] **Step 5: Commit**

Run: `git add extension/runtime extension/background tests/unit && git commit -m "fix: make tab messaging lifecycle safe"`

### Task 4: Add pure export serializers

**Files:**
- Create: `extension/export/normalize-conversation.js`
- Create: `extension/export/serialize-markdown.js`
- Create: `extension/export/serialize-json.js`
- Create: `extension/export/serialize-html.js`
- Create: `extension/export/serialize-text.js`
- Create: `tests/unit/export-serializers.test.js`
- Create: `tests/fixtures/conversation.json`

**Interfaces:**
- `normalizeConversation(input) -> Conversation`.
- `serializeMarkdown(conversation) -> string`.
- `serializeJson(conversation) -> string`.
- `serializeHtml(conversation) -> string`.
- `serializeText(conversation) -> string`.

- [ ] **Step 1: Write failing serializer tests**

Assert stable title/date handling, speaker ordering, escaped HTML, valid JSON,
and deterministic output for the fixture conversation.

- [ ] **Step 2: Run the focused tests and verify they fail**

Run: `node --test tests/unit/export-serializers.test.js`

Expected: FAIL because serializer modules do not exist.

- [ ] **Step 3: Implement pure serializers**

Use the fixture schema, avoid DOM and Chrome dependencies, and make filenames
derive from a sanitized title plus stable conversation identifier.

- [ ] **Step 4: Run the focused tests and verify they pass**

Run: `node --test tests/unit/export-serializers.test.js`

Expected: PASS with exact snapshot-like assertions.

- [ ] **Step 5: Commit**

Run: `git add extension/export tests/unit tests/fixtures && git commit -m "feat: add deterministic conversation serializers"`

### Task 5: Port the ChatGPT page adapter and export actions

**Files:**
- Create: `extension/content/chatgpt-adapter.js`
- Create: `extension/content/export-actions.js`
- Create: `extension/ui/export-panel.html`
- Create: `extension/ui/export-panel.js`
- Create: `tests/fixtures/chatgpt-conversation.html`
- Create: `tests/browser/export-actions.test.js`
- Modify: `extension/content/content-entry.js`
- Modify: `extension/background/service-worker.js`

**Interfaces:**
- `readConversation(document) -> ConversationInput`.
- `runExportAction(action, conversation, dependencies) -> Promise<ExportResult>`.
- Actions: `copy-text`, `screenshot`, `markdown`, `html`, `json`, `export-all`.

- [ ] **Step 1: Write failing browser-level action tests**

Load the synthetic conversation fixture, assert all six actions are visible,
and assert each action produces its expected output type.

- [ ] **Step 2: Run the browser tests and verify they fail**

Run: `node --test tests/browser/export-actions.test.js`

Expected: FAIL because the adapter, panel, and action handlers do not exist.

- [ ] **Step 3: Implement the adapter and actions**

Keep DOM selectors centralized in the adapter, route output through the pure
serializers, and make bulk export process one conversation at a time with
per-item results.

- [ ] **Step 4: Run the browser tests and verify they pass**

Run: `node --test tests/browser/export-actions.test.js`

Expected: PASS for all six actions and the fixture conversation.

- [ ] **Step 5: Commit**

Run: `git add extension/content extension/ui tests/browser tests/fixtures && git commit -m "feat: add ChatGPT export actions"`

### Task 6: Add lifecycle regression coverage and release documentation

**Files:**
- Create: `tests/browser/tab-lifecycle-regression.test.js`
- Create: `docs/TESTING.md`
- Modify: `README.md`
- Modify: `docs/ROADMAP.md`
- Modify: `specs/001-mv3-exporter-runtime/spec.md`

**Interfaces:**
- The browser regression test reports stale-tab counts and unhandled rejection
  counts for a repeated close/reload/navigation cycle.

- [ ] **Step 1: Write the failing lifecycle regression test**

Run an in-flight export while repeatedly closing, reloading, and navigating
fixture tabs. Assert zero unhandled rejections and that unrelated fixture tabs
continue responding.

- [ ] **Step 2: Run the regression test and verify it fails against the legacy path**

Run: `node --test tests/browser/tab-lifecycle-regression.test.js`

Expected: FAIL until the MV3 lifecycle-safe path is connected and the test
harness is configured.

- [ ] **Step 3: Connect the runtime and document the test workflow**

Document the disposable-profile command, expected outputs, and redaction rules.
Update roadmap/spec acceptance status only for checks that actually pass.

- [ ] **Step 4: Run the full verification set**

Run: `node --test tests/**/*.test.js && ./scripts/build-extension.sh`

Expected: PASS with a reproducible `dist/extension/` output.

- [ ] **Step 5: Commit**

Run: `git add tests docs README.md specs && git commit -m "test: verify mv3 lifecycle and exporter release path"`

## Verification checklist

- [ ] `git diff --check` passes.
- [ ] Unit tests pass.
- [ ] Browser tests pass in a disposable profile.
- [ ] Clean build produces `dist/extension/`.
- [ ] No proprietary Tampermonkey 5.x source or assets were added.
- [ ] No live Chrome profile was modified.
- [ ] Roadmap and active spec accurately report status.

