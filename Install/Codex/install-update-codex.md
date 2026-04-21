# Codex adapter: install/update (LLM runbook)

Purpose: install/update `skai` into any repo using Codex.

This adapter is **stack-aware** (Xcode/Swift, Android/Kotlin, etc.). It detects the project stack from repo signals, then applies stack-specific guidance from addenda files.

## Assumptions (stop if false)

Assume the following, unless the host repo already establishes a different working convention:

- Codex instructions live at `.agents/AGENTS.md`.
- Agent-facing skills live at `.agents/skills/skai-*/` (managed skill wrappers pointing to submodule sources).
- Mixed-agent repos may also contain `.cursor/**` or `.claude/**`; those belong to their own adapters and are out of scope for Codex install/update.

If any assumption is false in the host repo's setup, STOP and ask the human what file/path/convention to use.

## Inputs (required reading)

- `assets.manifest.json`
- `Install/managed-header.md`
- `Install/conflict-precedence-policy.md`
- `Policies/safe-operations.md`
- `Policies/universal-stop-conditions.md`
- `Templates/docs/skai/integration.md`
- `Templates/agents/codex/AGENTS.md`

## Migration-capable algorithm (required)

Follow the discover -> classify -> plan -> confirm -> execute workflow.

### 1) Discover (read-only)

- Identify whether `skai` is already present as a submodule (and where).
- If this is an update, record the current submodule commit SHA (pre-update) and the intended new SHA (post-update).
- **Detect stack** from project signals:
  - Xcode/Swift: `.xcodeproj`, `.xcworkspace`, `Package.swift`, `*.swift`
  - Android/Kotlin: `build.gradle`, `settings.gradle`, `*.kt`, `*.kts`
  - Otherwise: generic (no stack addendum needed)
  - Confirm the detected stack with the developer.
- **Detect other agent context** (for coexistence rules):
  - Note whether `.cursor/` exists (for optional `.cursorignore` setup).
  - Note whether `.claude/` exists (for optional `.claudeignore` setup).
  - Do **not** inventory or classify other agents' contents -- those belong to other adapters and are out of scope.
- Inventory existing install artifacts:
  - Codex instruction files (`.agents/AGENTS.md`, `AGENTS.md`)
  - Codex skill files (`.agents/skills/**`)
  - `docs/**`
- Identify any existing docs containing integration details (to migrate into Integration doc).

### 2) Classify

Classify each discovered artifact:
- **Managed**: has the managed header (`Managed-By: skai`) -> safe to overwrite.
- **Managed skill file**: has the managed marker comment described in `Install/managed-header.md` -> safe to overwrite.
- **Managed symlink**: a symlink that points into `Submodules/skai/...` at the expected target path -> safe to replace/update.
- **Legacy candidate**: looks like a managed asset but lacks the header -> do not overwrite.
- **Project-owned**: custom -> do not overwrite.

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
2. Create/update `docs/skai/integration.md` (migrate legacy command docs into it; do not delete legacy docs by default).
   - If you cannot find the required integration information in-repo:
     - Create/seed the Integration doc from `Templates/docs/skai/integration.md`.
     - Fill only what you can source with high confidence.
     - Add explicit 🟡 placeholders for missing items.
     - STOP and ask the human for the missing items before proceeding.
   - Prefer non-interactive command-line commands over GUI instructions. If you can't produce command-line commands with high confidence, leave 🟡 placeholders and ask.
   - **Stack-specific integration doc guidance**: if a stack was detected, read the corresponding addendum for additional rules:
     - Xcode/Swift -> `Install/Codex/stack-xcode.md`
     - Android/Kotlin -> `Install/Codex/stack-android.md`
   - Follow `Install/integration-doc-install-update.md` for how to update the Integration doc safely (managed blocks + human overrides).
3. Create/update `.agents/AGENTS.md` by copying `Templates/agents/codex/AGENTS.md` and stamping the managed header with `Managed-Id: template.codex-agents`, `Managed-Source: Submodules/skai/Templates/agents/codex/AGENTS.md`, and `Managed-Adapter: codex`.
4. Install Codex skills into `.agents/skills/` (see "Installing Codex skills" below).
5. Create/update ignore files (permission-gated if they already exist and are project-owned):
   - Update `.gitignore` by inserting/updating a managed block:
     - Add `working-docs/` so ephemeral working documents are not committed.
   - If `.cursorignore` exists, propose inserting/updating a managed block to exclude `.agents/**` so Cursor sessions don't ingest Codex-specific assets by default (ask approval before changing).
   - If `.claudeignore` exists, propose inserting/updating a managed block to exclude `.agents/**` so Claude sessions don't ingest Codex-specific assets by default (ask approval before changing).
   - Do NOT exclude `Submodules/skai/**` here; use editor UI excludes for autocomplete/search clutter instead.
6. Optionally propose cleanup of legacy candidates as a separate explicit step.
7. Write/update `docs/skai/install-state.json` (see "Install state file" below).

Required Integration doc fields to request (minimum set):
- Build/compile command(s)
- Unit test command(s): run all + run a single test/subset
- How to capture full output (paths/artifacts the human should paste back)
- Device/simulator/emulator conventions (if applicable)
- Known evidence-capture limitations (if any)

## Installing Codex skills

Create these skills in the host repo under `.agents/skills/` by copying the shared templates from the submodule and inserting the managed marker comment immediately after the YAML frontmatter (see `Install/managed-header.md`).

Use `Managed-Adapter: codex` and `Managed-Id: skill.<skill-name>` (e.g., `skill.skai-debugging`).

Rules:
- Each destination `SKILL.md` is considered managed only if it contains the managed marker comment described in `Install/managed-header.md`.
- Overwrite only if destination is missing or already contains the managed marker.

Install these skills:
- `.agents/skills/skai-debugging/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-debugging/SKILL.md`
- `.agents/skills/skai-work-spec-creation/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-work-spec-creation/SKILL.md`
- `.agents/skills/skai-work-spec-implementation/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-work-spec-implementation/SKILL.md`
- `.agents/skills/skai-suggestion/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-suggestion/SKILL.md`
- `.agents/skills/skai-dev-retro/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-dev-retro/SKILL.md`
- `.agents/skills/skai-unit-testing/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-unit-testing/SKILL.md`
- `.agents/skills/skai-unit-test-planning/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-unit-test-planning/SKILL.md`
- `.agents/skills/skai-unit-test-infrastructure/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-unit-test-infrastructure/SKILL.md`
- `.agents/skills/skai-unit-test-writing/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-unit-test-writing/SKILL.md`
- `.agents/skills/skai-update-installation/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-update-installation/SKILL.md`

## Install state file

After a successful install or update, write/update `docs/skai/install-state.json` so the `update-installation` skill can detect changes and re-run the appropriate adapters.

Format:

```json
{
  "managedBy": "skai",
  "submodulePath": "Submodules/skai",
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
- The adapter ID for this runbook is `codex`; the runbook path is `Install/Codex/install-update-codex.md`.

## Legacy artifact cleanup (permission-gated)

During discovery, if you find artifacts that appear to be from older installations (symlinks pointing into the submodule, guide copies without managed headers, skills that have been renamed or replaced), propose a cleanup plan:

- **Managed symlinks** (symlinks into `Submodules/skai/...`): safe to delete (installer-created artifacts, no approval needed). Remove the containing directory if empty after cleanup.
- **Non-symlink or project-authored content**: treat as project-owned and STOP to ask the human what to do.
