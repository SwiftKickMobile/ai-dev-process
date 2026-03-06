# Cursor adapter: install/update (LLM runbook)

This document is the canonical Cursor install/update runbook.

## Inputs (required reading)

- `assets.manifest.json`
- `Install/managed-header.md`
- `Install/conflict-precedence-policy.md`
- `Policies/safe-operations.md`
- `Templates/docs/skai/integration.md`

Key responsibilities:
- Detect existing installs (including legacy copies of guides/rules) and plan a safe migration.
- Create/update `docs/skai/integration.md` (and migrate any legacy integration command notes into it).
- Generate managed Cursor `.mdc` rule files into `.cursor/rules/skai/`.
- Install `skai` Cursor skills into `.cursor/skills/` (no symlinks; wrappers point at `Submodules/skai/...` sources).
- Update `.cursorignore` (and, if present, `.claudeignore`) via managed blocks so Cursor and Claude installs can coexist without clutter.
- Never overwrite project-owned files; only overwrite managed files; treat lookalike files as legacy candidates.

## Migration-capable algorithm (required)

Follow the discover → classify → plan → confirm → execute workflow.

### 1) Discover (read-only)

- Identify whether `skai` is already present as a submodule (and where).
- If this is an update, record the current submodule commit SHA (pre-update) and the intended new SHA (post-update).
- Inventory existing install artifacts:
  - `.cursor/rules/**`, `.cursor/**`
  - specifically scan for prior `skai` install targets under `.cursor/**` (do not assume current targets only):
    - `.cursor/skills/**`
    - `.cursor/agent/**` (deprecated install target; see cleanup guidance below)
  - `docs/**`
  - any "integration glue" docs/notes (build/test commands, destinations, artifact paths), wherever they live (README, docs, CI scripts, etc.)
- Inventory any existing Integration doc candidates and existing rule/policy docs (do not assume specific filenames).

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
- Integration doc migration items (move legacy integration glue into `docs/skai/integration.md`)
- Legacy cleanup proposals (explicitly permission-gated):
  - delete legacy candidates
  - or replace legacy candidates with symlinks to the new canonical locations
 - Deprecated install artifact cleanup proposals (explicitly permission-gated):
   - remove managed outputs that were installed by older versions of this runbook but are no longer part of the current install targets

Required gray-area checks (ask the human, then reflect the decision in the plan):
- Search for legacy debugging rule candidates (common examples: `.cursor/rules/debugging*.mdc`, `.cursor/rules/*debug*.mdc`, or other Cursor rules that encode project-specific logging/API conventions).
  - If found, propose:
    1) migrating any project-specific logging/API conventions they contain into `docs/skai/integration.md`, then
    2) deleting those legacy debugging rule files (only with explicit approval).
  - When reporting discovery, list the candidates you found; do not emit "not found" lines for example filenames you didn't find.

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
     - STOP and ask the human for the missing items before proceeding with the rest of the install.
   - When filling "Build / compile" and "Unit tests", prefer non-interactive command-line commands (e.g., `xcodebuild ...`) over GUI instructions ("open Xcode…"). If you can't produce command-line commands with high confidence, leave 🟡 placeholders and ask.
   - For Xcode projects: never invent a simulator/device model. If a canonical `xcodebuild -destination` string is not already established in-repo, propose one and ask the human to confirm before writing it.
   - Follow `Install/integration-doc-install-update.md` for how to update the Integration doc safely (managed blocks + human overrides).
2.5 Create/update ignore files (permission-gated if the files already exist and are project-owned):
   - Update `.gitignore` by inserting/updating a managed block:
     - Add `working-docs/` so ephemeral working documents are not committed.
   - Update `.cursorignore` by inserting/updating a managed block:
     - Exclude `.claude/**` so Cursor sessions don't ingest Claude-specific assets by default.
     - Do NOT exclude `Submodules/skai/**` here; use editor UI excludes for autocomplete/search clutter instead.
   - If `.claudeignore` exists, propose inserting/updating an equivalent managed block to exclude `.cursor/**` (ask approval before changing).
3. Generate managed Cursor `.mdc` rule files into `.cursor/rules/skai/`.
4. Install `skai` Cursor skills into `.cursor/skills/`.
5. If approved, perform legacy cleanup (delete or replace with symlinks).
6. Write/update `docs/skai/install-state.json` (see "Install state file" below).

Required Integration doc fields to request (minimum set):
- Build/compile command(s)
- Unit test command(s): run all + run a single test/subset
- How to capture full output (paths/artifacts the human should paste back)
- Simulator/device destination conventions (if applicable)
- Known evidence-capture limitations (if any)

## Cursor install targets

Create these directories in the host repo:
- `.cursor/rules/skai/`
- `.cursor/skills/`

## Generating Cursor rule files (`.mdc`)

The Cursor adapter writes `.mdc` files directly into the host repo.

Rules:
- Each generated `.mdc` begins with the managed header (see `Install/managed-header.md`).
- For updates, overwrite only when the destination is missing or already has the managed header.
- Determine stack (Swift/Xcode vs Android/Kotlin, etc.) by inspecting the host repo, then ask the human to confirm before writing.

Initial supported stacks:
- Swift/Xcode: generate `.mdc` with `globs: ["**/*.swift"]`.
- Android/Kotlin: generate `.mdc` with `globs: ["**/*.kt", "**/*.kts"]`.

Recommended generated rules (filenames are stable):
- `.cursor/rules/skai/coding-patterns.mdc` ← `Policies/coding-patterns.md`
- `.cursor/rules/skai/debugging-process.mdc` ← `Policies/debugging-process-rule.md`
- `.cursor/rules/skai/error-handling.mdc` ← `Policies/error-handling.md`
- `.cursor/rules/skai/unauthorized-changes.mdc` ← `Policies/unauthorized-changes.md`
- `.cursor/rules/skai/safe-operations.mdc` ← `Policies/safe-operations.md`
- `.cursor/rules/skai/universal-stop-conditions.mdc` ← `Policies/universal-stop-conditions.md`
- `.cursor/rules/skai/swift-code-organization.mdc` ← `Policies/swift-code-organization.md` (Swift stack only)

## Installing Cursor skills (recommended; no symlinks)

Create these skills in the host repo under `.cursor/skills/` by copying the shared templates from the submodule and inserting the managed marker comment immediately after the YAML frontmatter (see `Install/managed-header.md`).

Use `Managed-Adapter: cursor` and `Managed-Id: skill.<skill-name>` (e.g., `skill.skai-debugging`).

Rules:
- Each destination `SKILL.md` is considered managed only if it contains the managed marker comment described in `Install/managed-header.md`.
- Overwrite only if destination is missing or already contains the managed marker.

Install these skills:
- `.cursor/skills/skai-debugging/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-debugging/SKILL.md`
- `.cursor/skills/skai-work-spec-creation/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-work-spec-creation/SKILL.md`
- `.cursor/skills/skai-work-spec-implementation/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-work-spec-implementation/SKILL.md`
- `.cursor/skills/skai-unit-testing/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-unit-testing/SKILL.md`
- `.cursor/skills/skai-unit-test-planning/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-unit-test-planning/SKILL.md`
- `.cursor/skills/skai-unit-test-infrastructure/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-unit-test-infrastructure/SKILL.md`
- `.cursor/skills/skai-unit-test-writing/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-unit-test-writing/SKILL.md`
- `.cursor/skills/skai-suggestion/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-suggestion/SKILL.md`
- `.cursor/skills/skai-dev-retro/SKILL.md`
  - source: `Submodules/skai/Templates/skills/skai-dev-retro/SKILL.md`
- `.cursor/skills/skai-update-installation/SKILL.md`
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
- The adapter ID for this runbook is `cursor`; the runbook path is `Install/Cursor/install-update-cursor.md`.

## Legacy artifact cleanup (permission-gated)

During discovery, if you find artifacts that appear to be from older installations (symlinks pointing into the submodule, guide/rule copies without managed headers, skills that have been renamed or replaced), propose a cleanup plan:

- **Managed symlinks** (symlinks into `Submodules/skai/...`): propose deleting them (only with explicit approval).
- **Legacy candidates** (look like managed assets but lack the managed header): do not overwrite. Propose deletion or replacement, but only with explicit approval. If they contain project-specific content (e.g., logging conventions, custom rules), propose migrating that content into `docs/skai/integration.md` first.
- **Non-symlink or project-authored content**: treat as project-owned and STOP to ask the human what to do.

Rationale: leaving legacy copies in place increases the chance that humans/agents keep reading the wrong file out of habit.
