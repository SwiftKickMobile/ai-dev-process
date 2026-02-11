# Changelog

## Unreleased

- Restructured repository toward IDE-neutral `.md` sources (`Core/`, `Policies/`, `Templates/`, `Install/`).
- Removed legacy Cursor `.mdc` sources from this repo (Cursor `.mdc` is generated into host projects during install/update).
- Added migration-capable install/update discussion record in `DISCUSSION.md`.
- Added a Debugging Process rule to enforce evidence-first debugging and prevent “guessing fixes” loops.
- Expanded install/update support for multiple agent platforms coexisting in one repo:
  - Cursor installs to `.cursor/rules/ai-dev-process/` and `.cursor/agent/ai-dev-process/`
  - Claude Code installs to `.claude/agent/ai-dev-process/`
- Added/expanded Claude Code runbooks:
  - JetBrains (IntelliJ IDEA) + Claude Code
  - Android Studio + Claude Code (Android stack)
  - Xcode (MCP-hosted) + Claude Code (EXPERIMENTAL)
- Made installers update `.cursorignore` / `.claudeignore` safely using managed blocks (begin/end markers) to reduce clutter and prevent cross-agent ingestion.
- Added “update review” guidance to runbooks (record old/new submodule SHA and summarize diffs for approval).
- Updated unit testing guides:
  - use 🟡 only (no ✅ completion markers)
  - place test work docs and artifacts under `work/<branch>/testing/<suite>/...`
  - reference `docs/ai-dev-process/integration.md` instead of `xcode-commands.md` / `.cursor/*`
- Improved the Integration template for Xcode: clarified agent-filled command variables and multi-scheme/test-plan conventions; standardized `xcodebuild`/`.xcresult`/`xcresulttool` patterns.
- Added `assets.manifest.json` to enumerate assets and adapter applicability.

