# Feature Specification: Permissions and Security Hardening

**Feature ID:** 007
**Status:** proposed
**Target milestone:** M1–M4

## Goal

Make the modern extension safe for high-load agentic development and explicit
about what data and browser capabilities it can access.

## Requirements

1. Host permissions MUST be limited to supported ChatGPT/export origins.
2. The extension MUST not request `<all_urls>`, cookies, or webRequest access
   unless a separately specified feature proves the need.
3. Export content MUST remain local by default.
4. Logs and test artifacts MUST redact credentials, cookies, authorization
   headers, conversation bodies, and sensitive query values.
5. `innerHTML` and equivalent sinks MUST receive escaped or trusted generated
   content only.
6. Downloads and clipboard writes MUST be user-triggered or clearly attributable
   to an active export action.
7. Imported scripts MUST be treated as executable code and require explicit
   user confirmation before enablement.
8. CI MUST run dependency and static security checks available to the project.
9. Security-sensitive permission changes MUST update the relevant spec and
   constitution rationale.

## Acceptance criteria

- Manifest review shows only justified permissions.
- Redaction tests prove representative secrets and conversation text do not
  appear in logs or artifacts.
- Security checks run as part of `verify` and CI.
- A malicious fixture message cannot inject markup into HTML output or UI.

