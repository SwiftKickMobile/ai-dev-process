# Xcode (MCP-hosted) + Claude Code adapter: install/update (EXPERIMENTAL)

Purpose: install/update `ai-dev-process` into a repo where developers use **Xcode (26.3+) MCP-hosted Claude Code**.

This runbook is **experimental** because the exact Xcode+MCP conventions may differ by environment.

## Assumptions (stop if false)

Assume the following, unless the host repo already establishes a different working convention:

- Claude instructions file is rooted at either `CLAUDE.md` or `claude.md` (prefer existing; default to `CLAUDE.md`).
- Claude ignore file is `.claudeignore`.
- Agent-facing skills live at `.claude/skills/ai-dev-process-*/` (managed skill wrappers pointing to submodule sources).
- Mixed-agent repos may also contain `.cursor/**`; Claude sessions should ignore Cursor-specific assets by default.

If any assumption is false in the host repo's setup, STOP and ask the human what file/path/convention to use.

## Inputs (required reading)

- `assets.manifest.json`
- `Install/managed-header.md`
- `Install/conflict-precedence-policy.md`
- `Policies/safe-operations.md`
- `Templates/docs/ai-dev-process/integration.md`

## Migration-capable algorithm (required)

Follow the discover → classify → plan → confirm → execute workflow.

### 1) Discover (read-only)

- Identify whether `ai-dev-process` is already present as a submodule (and where).
- Infer stack (signals: `.xcodeproj`, `.xcworkspace`, `Package.swift`, `*.swift`).
- If this is an update, record the current submodule commit SHA (pre-update) and the intended new SHA (post-update).
- Inventory existing install artifacts:
  - Claude instruction files (`claude.md`, `CLAUDE.md`, `.claude/**`)
  - `docs/**`
- Note whether `.cursor/` exists (for `.claudeignore` setup). Do **not** inventory or classify its contents -- those belong to the Cursor adapter and are out of scope for this runbook.
- Identify any existing docs containing integration details (to migrate into Integration doc).

### 2) Classify

Classify each discovered artifact:
- **Managed**: has the managed header (`Managed-By: ai-dev-process`) → safe to overwrite.
- **Managed symlink**: a symlink that points into `Submodules/ai-dev-process/...` at the expected target path → safe to replace/update.
- **Legacy candidate**: looks like a managed asset but lacks the header → do not overwrite.
- **Project-owned**: custom → do not overwrite.

### 3) Plan (no changes yet)

Prepare a concrete plan:
- Files to create
- Files to update (managed only, including managed symlinks)
- Legacy candidates to supersede (create new canonical outputs)
- Integration doc migration items
- Legacy cleanup proposals (permission-gated)

### 4) Confirm (human gate)

Present the plan and wait for human approval before writing.

If updating the submodule, include an "update review" section:
- Summarize changes between the old SHA and new SHA for relevant paths:
  - `Guides/`, `Policies/`, `Templates/`, `Install/`, `assets.manifest.json`, `README.md`
- If you cannot compute the diff yourself, STOP and ask the human to provide the diff output.

### 5) Execute (safe order)

1. Ensure submodule is present/updated.
2. Create/update `docs/ai-dev-process/integration.md` (migrate legacy command docs into it; do not delete legacy docs by default).
   - If you cannot find the required integration information in-repo:
     - Create/seed the Integration doc from `Templates/docs/ai-dev-process/integration.md`.
     - Fill only what you can source with high confidence.
     - Add explicit 🟡 placeholders for missing items.
     - STOP and ask the human for the missing items before proceeding.
   - Prefer non-interactive command-line commands (e.g., `xcodebuild ...`) over GUI instructions ("open Xcode…"). If you can't produce command-line commands with high confidence, leave 🟡 placeholders and ask.
   - Never invent a simulator/device model. If a canonical `xcodebuild -destination` string is not already established in-repo, propose one and ask the human to confirm before writing it.
   - Follow `Install/integration-doc-install-update.md` for how to update the Integration doc safely (managed blocks + human overrides).
3. Create/update the Claude instruction file (`claude.md` vs `CLAUDE.md`) using managed headers.
4. Install Claude Code skills into `.claude/skills/` (see "Installing Claude Code skills" below).
5. Create/update ignore files (permission-gated if they already exist and are project-owned):
   - Update `.gitignore` by inserting/updating a managed block:
     - Add `working-docs/` so ephemeral working documents are not committed.
   - Update `.claudeignore` by inserting/updating a managed block:
     - Exclude `.cursor/**` so Claude sessions don't ingest Cursor-specific assets by default.
     - Do NOT exclude `Submodules/ai-dev-process/**` here; use editor UI excludes for autocomplete/search clutter instead.
6. Optionally propose cleanup of legacy candidates as a separate explicit step.
7. Write/update `docs/ai-dev-process/install-state.json` (see "Install state file" below).

Required Integration doc fields to request (minimum set):
- Build/compile command(s)
- Unit test command(s): run all + run a single test/subset
- How to capture full output (paths/artifacts the human should paste back)
- Device/simulator destination conventions (if applicable)
- Known evidence-capture limitations (if any)

## Installing Claude Code skills (no symlinks)

Create these skills in the host repo under `.claude/skills/` by copying the shared templates from the submodule and inserting the managed marker comment immediately after the YAML frontmatter (see `Install/managed-header.md`).

Use `Managed-Adapter: claude-code` and `Managed-Id: skill.<skill-name>` (e.g., `skill.ai-dev-process-debugging`).

Rules:
- Each destination `SKILL.md` is considered managed only if it contains the managed marker comment described in `Install/managed-header.md`.
- Overwrite only if destination is missing or already contains the managed marker.

Install these skills:
- `.claude/skills/ai-dev-process-debugging/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/skills/ai-dev-process-debugging/SKILL.md`
- `.claude/skills/ai-dev-process-work-spec-creation/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/skills/ai-dev-process-work-spec-creation/SKILL.md`
- `.claude/skills/ai-dev-process-work-spec-implementation/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/skills/ai-dev-process-work-spec-implementation/SKILL.md`
- `.claude/skills/ai-dev-process-dev-retro/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/skills/ai-dev-process-dev-retro/SKILL.md`
- `.claude/skills/ai-dev-process-unit-testing/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/skills/ai-dev-process-unit-testing/SKILL.md`
- `.claude/skills/ai-dev-process-unit-test-planning/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/skills/ai-dev-process-unit-test-planning/SKILL.md`
- `.claude/skills/ai-dev-process-unit-test-infrastructure/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/skills/ai-dev-process-unit-test-infrastructure/SKILL.md`
- `.claude/skills/ai-dev-process-unit-test-writing/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/skills/ai-dev-process-unit-test-writing/SKILL.md`
- `.claude/skills/ai-dev-process-update-installation/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/skills/ai-dev-process-update-installation/SKILL.md`

## Install state file

After a successful install or update, write/update `docs/ai-dev-process/install-state.json` so the `update-installation` skill can detect changes and re-run the appropriate adapters.

Format:

```json
{
  "managedBy": "ai-dev-process",
  "submodulePath": "Submodules/ai-dev-process",
  "lastSHA": "<current submodule HEAD SHA>",
  "lastUpdatedAt": "<yyyy-mm-dd>",
  "installedAdapters": [
    {
      "adapter": "<adapter-id>",
      "runbook": "<submodule-relative runbook path>",
      "lastRunAt": "<yyyy-mm-dd>"
    }
  ]
}
```

Rules:
- If the file does not exist, create it with this adapter's entry.
- If the file already exists, **merge**: update `lastSHA`, `lastUpdatedAt`, and upsert this adapter's entry in `installedAdapters` (preserve entries from other adapters).
- The adapter ID for this runbook is `xcode-claudecode`; the runbook path is `Install/Xcode-ClaudeCode/install-update-xcode-claudecode.md`.

## Deprecated symlink install target (required)

Older installs may have placed symlinked guides under:
- `.claude/agent/ai-dev-process/` (symlinks pointing into `Submodules/ai-dev-process/...`)

Current installs use managed skills under `.claude/skills/ai-dev-process-*/` instead (no symlinks).

Required behavior:
- If `.claude/agent/ai-dev-process/` exists:
  - If it contains symlinks that point into `Submodules/ai-dev-process/...`, delete them (these are installer-created artifacts, no approval needed).
  - If it contains non-symlink or project-authored content, treat as project-owned and STOP to ask the human what to do.
  - Remove the directory if empty after cleanup.
