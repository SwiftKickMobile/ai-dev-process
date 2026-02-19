# Changelog

## Unreleased

- 2026-02-19 **Working document conventions guide (`Guides/Core/working-doc-conventions.md`).** All working documents (planning docs, work specs, test work docs, test artifacts) now live under a single `working-docs/` top-level folder, organized by git branch path. The new guide centralizes the mechanics (branch detection, folder creation, path formula) that were previously repeated across workflow guides. Updated `work-spec-creation.md`, `unit-testing-guide.md`, `unit-test-writing-guide.md`, `unit-test-infrastructure-guide.md`, and the `ai-dev-process-unit-testing` skill template to reference the conventions guide. All four installer runbooks now add `working-docs/` to `.gitignore` via a managed block.
- 2026-02-18 **New skill: `ai-dev-process-update-installation`.** Lets the human ask the LLM to check for `ai-dev-process` submodule updates, pull the latest, show a changelog summary of what changed, and automatically re-run the appropriate adapter install/update runbooks. The skill reads from a new `docs/ai-dev-process/install-state.json` file (see below) to know which adapters are installed and what SHA was last applied.
- 2026-02-18 **Install state file (`docs/ai-dev-process/install-state.json`).** All four adapter runbooks (Cursor, JetBrains + Claude Code, Xcode + Claude Code, Android Studio + Claude Code) now write/update this JSON file on successful completion. It records the submodule SHA, last update date, and the list of installed adapters with their runbook paths -- enabling the `update-installation` skill to detect changes and re-run the right runbooks without manual configuration.
- 2026-02-18 **Process reflection step in retros.** Both `maintain-retro.md` (repo maintenance) and `Guides/dev-retro.md` (host-project dev sessions) now include a "Process reflection" step that asks the LLM to reflect on pattern violations, recurring friction, human corrections, and documentation gaps, then produce concrete suggestions for process/doc improvements.
- 2026-02-18 **Date determination rule.** `Install/managed-header.md` now requires LLMs to run `date +%Y-%m-%d` in the terminal for all date values rather than relying on system-prompt dates, which can be stale or timezone-mismatched.
- 2026-02-18 **ASCII-only content rule.** Normalized all smart/curly quotes and Unicode dashes to ASCII equivalents across the repo (28 files, 272 replacements). Added a content rule in `maintain-ai-dev-process.md` and a retro check in `maintain-retro.md` to prevent reintroduction. Smart quotes break `StrReplace` tooling.
- 2026-02-17 **Unified skill templates.** Cursor and Claude Code adapters now share a single set of skill templates at `Templates/skills/ai-dev-process-*/SKILL.md`. Each template uses YAML frontmatter with no baked-in managed marker; installers stamp the adapter-specific marker (`Managed-Adapter: cursor` or `claude-code`) at copy time. This eliminates the previous duplication where Cursor and Claude Code had separate template sets. Claude Code skills now install to `.claude/skills/ai-dev-process-*/SKILL.md` using Claude Code's native skills system, replacing the old `.claude/agent/` symlink approach. Manifest entries unified from `cursor-skill.*` / `claude-code-skill.*` to `skill.*`.
- 2026-02-17 **Fixed `.cursor/` over-analysis in Claude Code runbooks.** All Claude Code adapter runbooks (JetBrains, Xcode, Android Studio) updated so their Discover step notes `.cursor/` presence for `.claudeignore` setup but does not inventory or classify its contents. Previously, installers were wasting time analysing `.cursor/**` files and misclassifying Cursor-owned rules as "cursor rules without managed header."
- 2026-02-17 Added formal Planning Phase section to `work-spec-creation.md` with three stages (scope discussion, ideation/questions/discussion, API sketch), a 🟡 marker protocol for tracking open questions through to decisions in planning documents, and reordered the guide so Planning Phase precedes Requirements Normalization (requirements are an output of planning, not a precondition).
- 2026-02-14 Split `ai-dev-process-work-spec` Cursor skill into two separate skills: `ai-dev-process-work-spec-creation` (planning/PRD/spec authoring) and `ai-dev-process-work-spec-implementation` (begin/next/continue task execution). Each skill now loads only the guide relevant to its phase, improving focus and reducing context dilution. Renamed `Guides/Spec/work-spec.md` to `work-spec-creation.md` for consistency.
- 2026-02-13 Added Swift Package Manager integration section template (`swift-package.md`) for projects that use `Package.swift` alongside or instead of Xcode projects; updated installer guidance and manifest.
- 2026-02-13 Restructured repository toward IDE-neutral `.md` sources (`Guides/`, `Policies/`, `Templates/`, `Install/`).
- 2026-02-13 Removed legacy Cursor `.mdc` sources from this repo (Cursor `.mdc` is generated into host projects during install/update).
- 2026-02-13 Added migration-capable install/update discussion record (temporary; later removed).
- 2026-02-13 Added a Debugging Process rule to enforce evidence-first debugging and prevent "guessing fixes" loops.
- 2026-02-13 Expanded install/update support for multiple agent platforms coexisting in one repo:
  - Cursor installs to `.cursor/rules/ai-dev-process/` and `.cursor/skills/ai-dev-process-*/`
  - Claude Code installs to `.claude/agent/ai-dev-process/`
- 2026-02-13 Added/expanded Claude Code runbooks:
  - JetBrains (IntelliJ IDEA) + Claude Code
  - Android Studio + Claude Code (Android stack)
  - Xcode (MCP-hosted) + Claude Code (EXPERIMENTAL)
- 2026-02-13 Made installers update `.cursorignore` / `.claudeignore` safely using managed blocks (begin/end markers) to reduce clutter and prevent cross-agent ingestion.
- 2026-02-13 Added "update review" guidance to runbooks (record old/new submodule SHA and summarize diffs for approval).
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

