# Changelog

## Unreleased

- 2026-02-13 Restructured repository toward IDE-neutral `.md` sources (`Guides/`, `Policies/`, `Templates/`, `Install/`).
- 2026-02-13 Removed legacy Cursor `.mdc` sources from this repo (Cursor `.mdc` is generated into host projects during install/update).
- 2026-02-13 Added migration-capable install/update discussion record (temporary; later removed).
- 2026-02-13 Added a Debugging Process rule to enforce evidence-first debugging and prevent “guessing fixes” loops.
- 2026-02-13 Expanded install/update support for multiple agent platforms coexisting in one repo:
  - Cursor installs to `.cursor/rules/ai-dev-process/` and `.cursor/skills/ai-dev-process-*/`
  - Claude Code installs to `.claude/agent/ai-dev-process/`
- 2026-02-13 Added/expanded Claude Code runbooks:
  - JetBrains (IntelliJ IDEA) + Claude Code
  - Android Studio + Claude Code (Android stack)
  - Xcode (MCP-hosted) + Claude Code (EXPERIMENTAL)
- 2026-02-13 Made installers update `.cursorignore` / `.claudeignore` safely using managed blocks (begin/end markers) to reduce clutter and prevent cross-agent ingestion.
- 2026-02-13 Added “update review” guidance to runbooks (record old/new submodule SHA and summarize diffs for approval).
- 2026-02-13 Updated unit testing guides:
  - use 🟡 only (no ✅ completion markers)
  - place test work docs and artifacts under `work/<branch>/testing/<suite>/...`
  - reference `docs/ai-dev-process/integration.md` instead of `xcode-commands.md` / `.cursor/*`
- 2026-02-13 Improved the Integration template for Xcode: clarified agent-filled command variables and multi-scheme/test-plan conventions; standardized `xcodebuild`/`.xcresult`/`xcresulttool` patterns.
- 2026-02-13 Refactored the Integration template to support LLM-managed blocks and human-owned overrides; stack-specific integration sections now live in `Templates/docs/ai-dev-process/integration-sections/` and are merged into the Integration doc by installers.
- 2026-02-13 Added `Install/integration-doc-install-update.md` as a shared installer guide for safely updating the Integration doc.
- 2026-02-13 Cursor installer now proposes permission-gated cleanup of the deprecated `.cursor/agent/ai-dev-process/` symlink install target when migrating to Cursor skills.
- 2026-02-13 Added `Guides/dev-retro.md` (dev-session retro) and a Cursor skill wrapper (`ai-dev-process-dev-retro`); deprecated `retro-prd`.
- 2026-02-13 Added `maintain-retro.md` as a post-change checklist for maintainers/LLMs.
- 2026-02-13 Added `assets.manifest.json` to enumerate assets and adapter applicability.

