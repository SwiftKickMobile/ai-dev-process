# `ai-dev-process`

Reusable, mostly IDE-agnostic **agentic coding guides** and **policies** you can vendor into any repo.

This repo is designed to be installed as a **git submodule** and activated by an **LLM-runbook-driven installer** that:
- inspects the host repo (including legacy installs),
- proposes a migration plan,
- writes **managed files** only (safe updates),
- generates IDE-specific artifacts (e.g., Cursor `.mdc`) into the host repo.

## Quick start (recommended): ask your agent

- **Cursor**
  - “Install/update `ai-dev-process` using `Install/Cursor/install-update-cursor.md`.”
- **Android Studio + Claude Code**
  - “Install/update `ai-dev-process` using `Install/AndroidStudio-ClaudeCode/install-update-androidstudio-claudecode.md`.”

## How installs stay safe

- **Integration doc (project-owned)**: `docs/ai-dev-process/integration.md` is the single source of truth for project-specific commands/paths (build/test/lint/etc). Templates live in `Templates/`.
- **Managed files**: host-project files written by the installer have a required header (see `Install/managed-header.md`). The installer overwrites only files that already contain this header.
- **Legacy installs**: lookalike files without the header are treated as **legacy candidates** and are not overwritten by default (see `Install/conflict-precedence-policy.md`).

## Asset inventory

The authoritative list of assets and their intended usage is in `assets.manifest.json`.

High-level structure:
- `Core/`: IDE-neutral guides (e.g., debugging)
- `Policies/`: policies/rules (some are stack-specific)
- `Spec/`: work spec processes
- `Test/`: unit testing processes (currently Swift/Xcode-oriented; expanding over time)
- `Templates/`: templates copied into host repos (notably the Integration doc template)
- `Install/`: adapter runbooks + install/update policies

## Development guide (for contributors)

- Keep sources in this repo **IDE-neutral** (`.md`). IDE-specific outputs are generated into host repos by install/update runbooks.
- When you add/move/rename an asset, update `assets.manifest.json`.
- Update `CHANGELOG.md` for user-visible changes.
- Do not introduce scripts that mutate host repos; installers are LLM-runbook-driven and must follow the safety policies in `Policies/`.

