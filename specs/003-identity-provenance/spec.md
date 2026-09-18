# Feature Specification: Identity, Branding, and Provenance

**Feature ID:** 003
**Status:** proposed
**Target milestone:** M0–M1

## Goal

Establish a clearly distinct GPL-3.0 project identity while preserving accurate
credit and provenance for the public Tampermonkey 2.9-era source used as a
reference or starting point.

## Requirements

1. The extension MUST use a distinct name, extension ID, icons, descriptions,
   UI copy, and website links.
2. The project MUST retain the GPL-3.0 license and existing copyright notices
   for code that remains covered by them.
3. The project MUST document that the public source baseline is Tampermonkey
   2.9-era code and that proprietary Tampermonkey 5.x implementation details
   are excluded.
4. New modules MUST identify their project authorship and license consistently.
5. Build metadata MUST never claim to be an official Tampermonkey release.
6. The repository MUST include a NOTICE/provenance document mapping retained
   legacy directories to their license and origin.
7. User-facing migration documentation MUST explain that existing userscripts
   and the new extension are separate from the proprietary Tampermonkey product.

## Acceptance criteria

- A clean build shows only the new project identity.
- No new icon, title, or description uses Tampermonkey branding.
- License and provenance documents are present and linked from the README.
- A reviewer can identify legacy-derived and newly written code boundaries.

