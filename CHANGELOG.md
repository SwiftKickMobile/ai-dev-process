# Changelog

## Unreleased

- Restructured repository toward IDE-neutral `.md` sources (`Core/`, `Policies/`, `Templates/`, `Install/`).
- Removed legacy Cursor `.mdc` sources from this repo (Cursor `.mdc` is generated into host projects during install/update).
- Added migration-capable install/update discussion record in `DISCUSSION.md`.
- Updated unit testing guides:
  - use 🟡 only (no ✅ completion markers)
  - place test work docs and artifacts under `work/<branch>/testing/<suite>/...`
  - reference `docs/ai-dev-process/integration.md` instead of `xcode-commands.md` / `.cursor/*`
- Added `assets.manifest.json` to enumerate assets and adapter applicability.

