## Android Studio + Claude Code adapter: install/update (LLM runbook)

Purpose: install/update `skai` into an Android/Kotlin/Jetpack Compose project that uses Claude Code.

## Inputs (required reading)

- `assets.manifest.json`
- `Install/managed-header.md`
- `Install/conflict-precedence-policy.md`
- `Policies/safe-operations.md`
- `Policies/universal-stop-conditions.md`
- `Templates/docs/skai/integration.md`

### Key decisions (from discussion)

- This repo is the source of IDE-neutral `.md` guides.
- Adapter selection is stack-aware (infer from repo, then confirm with developer).
- Claude Code instructions file location:
  - Detect existing convention first.
  - Default if none exists: manage root `CLAUDE.md`.
- Managed-file safety:
  - Never overwrite project-owned content silently.
  - Only overwrite files that either do not exist, or contain the managed-by header.

---

## Install/update steps (host repo)

## Migration-capable algorithm (required)

Follow the discover → classify → plan → confirm → execute workflow.

### 1) Discover (read-only)

- Identify whether `skai` is already present as a submodule (and where).
- If this is an update, record the current submodule commit SHA (pre-update) and the intended new SHA (post-update).
- Inventory existing install artifacts:
  - Claude instruction files (`claude.md`, `CLAUDE.md`, `.claude/**`)
  - `docs/**`
- Note whether `.cursor/` exists (for `.claudeignore` setup). Do **not** inventory or classify its contents -- those belong to the Cursor adapter and are out of scope for this runbook.
- Inventory existing Integration doc candidates and any legacy command docs.

### 2) Classify

Classify each discovered artifact:
- **Managed**: has the managed header (`Managed-By: skai`) → safe to overwrite.
- **Managed symlink**: a symlink that points into `Submodules/skai/...` at the expected target path → safe to replace/update.
- **Legacy candidate**: looks like a managed asset but lacks the header → do not overwrite.
- **Project-owned**: custom → do not overwrite.

### 3) Plan (no changes yet)

Prepare a concrete plan:
- Files to create
- Files to update (managed only, including managed symlinks)
- Legacy candidates to supersede (create new managed files in canonical locations)
- Integration doc migration items (e.g., existing Gradle command notes into `docs/skai/integration.md`)

### 4) Confirm (human gate)

Present the plan and wait for human approval before writing.

If updating the submodule, include an "update review" section:
- Summarize changes between the old SHA and new SHA for relevant paths:
  - `Guides/`, `Policies/`, `Templates/`, `Install/`, `assets.manifest.json`, `README.md`
- If you cannot compute the diff yourself, STOP and ask the human to provide the diff output.

### 5) Execute (safe order)

1. Ensure submodule is present/updated.
2. Create/update `docs/skai/integration.md` (migrate legacy command docs into it; do not delete legacy docs by default).
3. Create/update the Claude instruction file (`claude.md` vs `CLAUDE.md`) using managed headers.
4. Install Claude Code skills into `.claude/skills/` (see "Installing Claude Code skills" below).
4.5 Create/update ignore files (permission-gated if the files already exist and are project-owned):
   - Update `.gitignore` by inserting/updating a managed block:
     - Add `working-docs/` so ephemeral working documents are not committed.
   - Update `.claudeignore` by inserting/updating a managed block:
     - Exclude `.cursor/**` so Claude sessions don't ingest Cursor-specific assets by default.
     - Do NOT exclude `Submodules/skai/**` here; use editor UI excludes for autocomplete/search clutter instead.
   - If `.cursorignore` exists, propose inserting/updating an equivalent managed block to exclude `.claude/**` (ask approval before changing).
5. Optionally propose cleanup of legacy candidates as a separate explicit step.
6. Write/update `docs/skai/install-state.json` (see "Install state file" below).

## Android adapter install targets

In the host repo, create:
- `docs/skai/` (Integration doc)
- `.claude/skills/skai-*/` (managed skills pointing to submodule sources)

## Testing stack defaults (initial)

Observed defaults from real Android projects:
- Unit tests commonly use JUnit4, plus MockK, plus kotlinx-coroutines-test when coroutines are involved.
- Instrumentation tests commonly use AndroidX JUnit + Espresso.

## Installing Claude Code skills (no symlinks)

Create these skills in the host repo under `.claude/skills/` by copying the shared templates from the submodule and inserting the managed marker comment immediately after the YAML frontmatter (see `Install/managed-header.md`).

Use `Managed-Adapter: claude-code` and `Managed-Id: skill.<skill-name>` (e.g., `skill.skai-debugging`).

Rules:
- Each destination `SKILL.md` is considered managed only if it contains the managed marker comment described in `Install/managed-header.md`.
- Overwrite only if destination is missing or already contains the managed marker.

Install these skills:
- `.claude/skills/skai-debugging/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-debugging/SKILL.md`
- `.claude/skills/skai-work-spec-creation/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-work-spec-creation/SKILL.md`
- `.claude/skills/skai-work-spec-implementation/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-work-spec-implementation/SKILL.md`
- `.claude/skills/skai-suggestion/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-suggestion/SKILL.md`
- `.claude/skills/skai-dev-retro/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-dev-retro/SKILL.md`
- `.claude/skills/skai-unit-testing/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-unit-testing/SKILL.md`
- `.claude/skills/skai-unit-test-planning/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-unit-test-planning/SKILL.md`
- `.claude/skills/skai-unit-test-infrastructure/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-unit-test-infrastructure/SKILL.md`
- `.claude/skills/skai-unit-test-writing/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-unit-test-writing/SKILL.md`
- `.claude/skills/skai-update-installation/SKILL.md`
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
- The adapter ID for this runbook is `androidstudio-claudecode`; the runbook path is `Install/AndroidStudio-ClaudeCode/install-update-androidstudio-claudecode.md`.

## Legacy artifact cleanup (permission-gated)

During discovery, if you find artifacts that appear to be from older installations (symlinks pointing into the submodule, guide copies without managed headers, skills that have been renamed or replaced), propose a cleanup plan:

- **Managed symlinks** (symlinks into `Submodules/skai/...`): safe to delete (installer-created artifacts, no approval needed). Remove the containing directory if empty after cleanup.
- **Non-symlink or project-authored content**: treat as project-owned and STOP to ask the human what to do.

### Step 0 -- Inspect current state

- Identify whether `skai` is already present as a submodule (check `.gitmodules`).
- Identify any existing Claude Code instruction files:
  - `CLAUDE.md` at repo root
  - `claude.md` at repo root
  - `.claude/` folder (if present)
  - Any org-specific equivalents (search for "Claude" or "AI instructions" docs)
- Identify any existing project docs containing integration details (to migrate into Integration doc):
  - `README.md`
  - docs folder
  - build/test command notes (Gradle commands, CI notes)

### Step 1 -- Ensure submodule exists and is updated

- If missing: add submodule at project-chosen location (recommended: `Submodules/skai`).
- Init/update submodule recursively.

### Step 2 -- Ensure Integration doc exists (project-owned source of truth)

Create (if missing) a single Integration doc in the host project, path chosen by the project.

Default:
- `docs/skai/integration.md`

Populate (or migrate) key integration details:
- How to build, lint, and run unit tests (`./gradlew test`, module-specific variants, CI parity)
- How to run instrumentation / UI tests if applicable
- Where logs/output should be captured from and pasted back to the agent
- Any project-specific constraints (minSdk, build variants, flavors)

Do not delete legacy docs; add pointers if you migrate content.

If you cannot find the required integration information in-repo:
- Create/seed the Integration doc from `Templates/docs/skai/integration.md`.
- Fill only what you can source with high confidence.
- Add explicit 🟡 placeholders for missing items.
- STOP and ask the human for the missing items before proceeding with Step 3.
 - Prefer non-interactive command-line commands (e.g., `./gradlew ...`) over GUI instructions. If you can't produce command-line commands with high confidence, leave 🟡 placeholders and ask.

Follow `Install/integration-doc-install-update.md` for how to update the Integration doc safely (managed blocks + human overrides).

Required Integration doc fields to request (minimum set):
- Build/compile command(s)
- Unit test command(s): run all + run a single test/subset
- How to capture full output (paths/artifacts the human should paste back)
- Emulator/device conventions (if applicable)
- Known evidence-capture limitations (if any)

### Step 3 -- Create/update Claude Code instructions

Decide instruction file location:
- If `claude.md` exists, use it.
- Else if `CLAUDE.md` exists, use it.
- Else if `.claude/` exists, follow the project's existing pattern.
- Else create/manage root `CLAUDE.md`.

The instruction file should:
- Include a managed-by header.
- Point to the Integration doc path.
- Point to the `skai` submodule guides that are relevant for Android work.
- Include the core safety/process rules (no unauthorized changes; remove 🟡 markers; stop conditions).

### Step 4 -- Update flow

On update:
- Update the submodule reference.
- Re-run migration step (Step 2) to keep the Integration doc complete.
- Refresh the instruction file content if it is managed (header present).

