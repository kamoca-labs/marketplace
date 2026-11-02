# Changelog

This repository is an unversioned catalog. Entries use dates and commit evidence rather than invented plugin version numbers. Individual plugin releases belong to their own repositories.

## 2026-11-02 — Design catalog entry

### New features

- Register the standalone design plugin and document setup, approved responsive mockups, alternate states, system consolidation and optional Figma/FigCraft integration.

### Migration

- Point visual-only work to `/design:style`, replacing `/craft:style` in craft 0.16.0.

Evidence: [catalog registration](https://github.com/kamoca-labs/marketplace/commit/a4f48f5). This entry is delivered with the registration branch when it reaches the default branch; the catalog itself has no semantic version tag.

## 2026-10-27 — Standalone catalog

### Improvements

- Point each plugin entry to its standalone kamoca-labs GitHub repository rather than a folder in the former monorepo.
- Translate the catalog and maintenance documentation to English.

### Fixes

- Correct the Git usage link to point at the renamed troubleshooting section.

Evidence: [standalone sources](https://github.com/kamoca-labs/marketplace/commit/81d88a9), [translation](https://github.com/kamoca-labs/marketplace/commit/d82a9cc), [link fix](https://github.com/kamoca-labs/marketplace/commit/0f23824).

## 2026-10-11 — Docker catalog entry

### New features

- Add the Docker plugin for consistent create/tune workflows and environment conventions.

### Improvements

- Standardize English command names, keep project decisions in the repository and identify the marketplace owner separately from the kamoca brand.

Evidence: [Docker entry](https://github.com/kamoca-labs/marketplace/commit/a755fb8), [owner metadata](https://github.com/kamoca-labs/marketplace/commit/65f2e2a).

## 2026-10-06 — Initial catalog

### New features

- Introduce the personal plugin marketplace and add the Git and Craft domains.
- Document installation, command composition, hook enforcement and maintenance workflows.

### Breaking changes

- Rename the marketplace from kauan to kamoca and remove the work-specific GitLab plugin from the general catalog. Existing installations need the corresponding marketplace migration.

Evidence: [initial catalog](https://github.com/kamoca-labs/marketplace/commit/e10b05f), [Craft entry](https://github.com/kamoca-labs/marketplace/commit/63cb6c9), [GitLab removal](https://github.com/kamoca-labs/marketplace/commit/8ef039e), [rename](https://github.com/kamoca-labs/marketplace/commit/d468e0e).
