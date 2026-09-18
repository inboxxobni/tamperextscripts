# Feature Specification Index

These specifications are the durable work queue for `tamperextscripts`. Each
feature is independently reviewable and should be implemented in dependency
order. Agents should read the relevant spec and its linked implementation plan
before changing code.

| ID | Specification | Depends on | Milestone |
|---|---|---|---|
| 001 | [MV3 exporter runtime](001-mv3-exporter-runtime/spec.md) | 003, 004, 005, 006 | M1–M3 |
| 002 | [Developer tooling and verification](002-developer-tooling/spec.md) | 003 | M1, continuous |
| 003 | [Identity, branding, and provenance](003-identity-provenance/spec.md) | — | M0–M1 |
| 004 | [Modern MV3 runtime shell](004-mv3-runtime-shell/spec.md) | 003, 002 | M1 |
| 005 | [Userscript engine and lifecycle safety](005-userscript-engine/spec.md) | 004 | M2 |
| 006 | [ChatGPT exporter](006-chatgpt-exporter/spec.md) | 004, 005 | M3 |
| 007 | [Permissions and security hardening](007-security-hardening/spec.md) | 004, 005, 006 | M1–M4 |
| 008 | [Compatibility and script migration](008-compatibility-migration/spec.md) | 005, 006, 007 | M4 |
| 009 | [Release and upstream collaboration](009-release-upstream/spec.md) | 002, 005, 006, 007 | M5 |

## Implementation order

```text
003 → 002 → 004 → 005 → 007 → 006 → 008 → 009
                         ↘ 001 integration gate ↗
```

Spec 001 is the end-to-end integration target. Specs 002–009 describe the
bounded capabilities needed to reach it and to maintain the project afterward.

