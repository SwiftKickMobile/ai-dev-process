# Cursor adapter: install/update (LLM runbook)

This document is the canonical Cursor install/update runbook.

## Inputs (required reading)

- `assets.manifest.json`
- `Install/managed-header.md`
- `Install/conflict-precedence-policy.md`
- `Policies/safe-operations.md`
- `Templates/docs/ai-dev-process/integration.md`

Key responsibilities:
- Detect existing installs (including legacy copies of guides/rules) and plan a safe migration.
- Create/update `docs/ai-dev-process/integration.md` (and migrate any legacy integration command notes into it).
- Generate managed Cursor `.mdc` rule files into `.cursor/rules/ai-dev-process/`.
- Install `ai-dev-process` Cursor skills into `.cursor/skills/` (no symlinks; wrappers point at `Submodules/ai-dev-process/...` sources).
- Update `.cursorignore` (and, if present, `.claudeignore`) via managed blocks so Cursor and Claude installs can coexist without clutter.
- Never overwrite project-owned files; only overwrite managed files; treat lookalike files as legacy candidates.

## Migration-capable algorithm (required)

Follow the discover → classify → plan → confirm → execute workflow.

### 1) Discover (read-only)

- Identify whether `ai-dev-process` is already present as a submodule (and where).
- If this is an update, record the current submodule commit SHA (pre-update) and the intended new SHA (post-update).
- Inventory existing install artifacts:
  - `.cursor/rules/**`, `.cursor/**`
  - specifically scan for prior `ai-dev-process` install targets under `.cursor/**` (do not assume current targets only):
    - `.cursor/skills/**`
    - `.cursor/agent/**` (deprecated install target; see cleanup guidance below)
  - `docs/**`
  - any “integration glue” docs/notes (build/test commands, destinations, artifact paths), wherever they live (README, docs, CI scripts, etc.)
- Inventory any existing Integration doc candidates and existing rule/policy docs (do not assume specific filenames).

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
- Legacy candidates to supersede (create new managed files in canonical locations)
- Integration doc migration items (move legacy integration glue into `docs/ai-dev-process/integration.md`)
- Legacy cleanup proposals (explicitly permission-gated):
  - delete legacy candidates
  - or replace legacy candidates with symlinks to the new canonical locations
 - Deprecated install artifact cleanup proposals (explicitly permission-gated):
   - remove managed outputs that were installed by older versions of this runbook but are no longer part of the current install targets

Required gray-area checks (ask the human, then reflect the decision in the plan):
- Search for legacy debugging rule candidates (common examples: `.cursor/rules/debugging*.mdc`, `.cursor/rules/*debug*.mdc`, or other Cursor rules that encode project-specific logging/API conventions).
  - If found, propose:
    1) migrating any project-specific logging/API conventions they contain into `docs/ai-dev-process/integration.md`, then
    2) deleting those legacy debugging rule files (only with explicit approval).
  - When reporting discovery, list the candidates you found; do not emit “not found” lines for example filenames you didn’t find.

### 4) Confirm (human gate)

Present the plan and wait for human approval before writing.

If updating the submodule, include an “update review” section:
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
     - STOP and ask the human for the missing items before proceeding with the rest of the install.
   - When filling “Build / compile” and “Unit tests”, prefer non-interactive command-line commands (e.g., `xcodebuild ...`) over GUI instructions (“open Xcode…”). If you can’t produce command-line commands with high confidence, leave 🟡 placeholders and ask.
   - For Xcode projects: never invent a simulator/device model. If a canonical `xcodebuild -destination` string is not already established in-repo, propose one and ask the human to confirm before writing it.
   - Follow `Install/integration-doc-install-update.md` for how to update the Integration doc safely (managed blocks + human overrides).
2.5 Create/update ignore files (permission-gated if the files already exist and are project-owned):
   - Update `.cursorignore` by inserting/updating a managed block:
     - Exclude `.claude/**` so Cursor sessions don’t ingest Claude-specific assets by default.
     - Do NOT exclude `Submodules/ai-dev-process/**` here; use editor UI excludes for autocomplete/search clutter instead.
   - If `.claudeignore` exists, propose inserting/updating an equivalent managed block to exclude `.cursor/**` (ask approval before changing).
3. Generate managed Cursor `.mdc` rule files into `.cursor/rules/ai-dev-process/`.
4. Install `ai-dev-process` Cursor skills into `.cursor/skills/`.
5. If approved, perform legacy cleanup (delete or replace with symlinks).

Required Integration doc fields to request (minimum set):
- Build/compile command(s)
- Unit test command(s): run all + run a single test/subset
- How to capture full output (paths/artifacts the human should paste back)
- Simulator/device destination conventions (if applicable)
- Known evidence-capture limitations (if any)

## Cursor install targets

Create these directories in the host repo:
- `.cursor/rules/ai-dev-process/`
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
- `.cursor/rules/ai-dev-process/coding-patterns.mdc` ← `Policies/coding-patterns.md`
- `.cursor/rules/ai-dev-process/debugging-process.mdc` ← `Policies/debugging-process-rule.md`
- `.cursor/rules/ai-dev-process/error-handling.mdc` ← `Policies/error-handling.md`
- `.cursor/rules/ai-dev-process/unauthorized-changes.mdc` ← `Policies/unauthorized-changes.md`
- `.cursor/rules/ai-dev-process/safe-operations.mdc` ← `Policies/safe-operations.md`
- `.cursor/rules/ai-dev-process/swift-code-organization.mdc` ← `Policies/swift-code-organization.md` (Swift stack only)

## Installing Cursor skills (recommended; no symlinks)

Create these skills in the host repo under `.cursor/skills/` by copying the templates from the submodule.

Rules:
- Each destination `SKILL.md` is considered managed only if it contains the managed marker comment described in `Install/managed-header.md`.
- Overwrite only if destination is missing or already contains the managed marker.

Install these skill wrappers:
- `.cursor/skills/ai-dev-process-debugging/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/cursor/skills/ai-dev-process-debugging/SKILL.md`
- `.cursor/skills/ai-dev-process-work-spec/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/cursor/skills/ai-dev-process-work-spec/SKILL.md`
- `.cursor/skills/ai-dev-process-unit-testing/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/cursor/skills/ai-dev-process-unit-testing/SKILL.md`
- `.cursor/skills/ai-dev-process-unit-test-planning/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/cursor/skills/ai-dev-process-unit-test-planning/SKILL.md`
- `.cursor/skills/ai-dev-process-unit-test-infrastructure/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/cursor/skills/ai-dev-process-unit-test-infrastructure/SKILL.md`
- `.cursor/skills/ai-dev-process-unit-test-writing/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/cursor/skills/ai-dev-process-unit-test-writing/SKILL.md`
- `.cursor/skills/ai-dev-process-dev-retro/SKILL.md`
  - source: `Submodules/ai-dev-process/Templates/cursor/skills/ai-dev-process-dev-retro/SKILL.md`

## Legacy path adoption (required, permission-gated)

If the host repo already has legacy copies of these guides at older locations (common examples):
- `.cursor/work-spec.md`
- `.cursor/work-spec-implementation.md`

Then the installer MUST propose a cleanup plan:
- **Preferred**: delete them (only with explicit approval).
- **Alternative**: replace them with a short redirect note that points at:
  - the Cursor skills under `.cursor/skills/ai-dev-process-*/`
  - and/or the canonical submodule sources under `Submodules/ai-dev-process/Guides/Spec/...`

Rationale: leaving legacy copies in place increases the chance that humans/agents keep reading the wrong file out of habit.

## Deprecated Cursor agent-doc install target (required, permission-gated)

Older installs of `ai-dev-process` may have placed symlinked “agent docs” under:
- `.cursor/agent/ai-dev-process/` (deprecated)

Current installs use Cursor skills under `.cursor/skills/ai-dev-process-*/` instead (no symlinks).

Required behavior:
- If `.cursor/agent/ai-dev-process/` exists:
  - If it contains symlinks that point into `Submodules/ai-dev-process/...`, treat them as **deprecated managed symlinks** and propose deleting the whole directory (only with explicit approval).
  - If it contains non-symlink or project-authored content, treat it as project-owned and STOP to ask the human what to do.

## Deprecated Cursor skill: `ai-dev-process-retro-prd` (required, permission-gated)

Older installs may have installed:
- `.cursor/skills/ai-dev-process-retro-prd/` (deprecated)

It is superseded by:
- `.cursor/skills/ai-dev-process-dev-retro/`

Required behavior:
- If `.cursor/skills/ai-dev-process-retro-prd/` exists:
  - If it contains only managed skill files (managed marker comment present), propose deleting the deprecated skill directory (only with explicit approval).
  - Otherwise treat as project-owned and STOP to ask what to do.

## Legacy debugging rule (special case, permission-gated)

If the host repo contains `.cursor/rules/debugging.mdc` (legacy, no managed header), treat it as a gray area:
- It may contain project-specific logging conventions that should be migrated into `docs/ai-dev-process/integration.md`.
- The installer should propose deleting it (because debugging should be guided by `Guides/Core/debugging-guide.md` plus the Debugging Process rule), but must ask for explicit approval before deleting.
