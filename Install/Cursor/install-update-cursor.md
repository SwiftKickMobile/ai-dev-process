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
- Create/update `docs/ai-dev-process/integration.md` (and migrate legacy `xcode-commands.md` content into it).
- Generate managed Cursor `.mdc` rule files into `.cursor/rules/ai-dev-process/`.
- Place convenient guide links/copies into `.cursor/ai-dev-process/`.
- Never overwrite project-owned files; only overwrite managed files; treat lookalike files as legacy candidates.

## Migration-capable algorithm (required)

Follow the discover → classify → plan → confirm → execute workflow.

### 1) Discover (read-only)

- Identify whether `ai-dev-process` is already present as a submodule (and where).
- Inventory existing install artifacts:
  - `.cursor/rules/**`, `.cursor/**`
  - `docs/**`
  - legacy docs like `xcode-commands.md`
- Inventory any existing Integration doc candidates and existing rule/policy docs.

### 2) Classify

Classify each discovered artifact:
- **Managed**: has the managed header (`Managed-By: ai-dev-process`) → safe to overwrite.
- **Legacy candidate**: looks like a managed asset but lacks the header → do not overwrite.
- **Project-owned**: custom → do not overwrite.

### 3) Plan (no changes yet)

Prepare a concrete plan:
- Files to create
- Files to update (managed only)
- Legacy candidates to supersede (create new managed files in canonical locations)
- Integration doc migration items (move legacy `xcode-commands.md` content into `docs/ai-dev-process/integration.md`)

### 4) Confirm (human gate)

Present the plan and wait for human approval before writing.

### 5) Execute (safe order)

1. Ensure submodule is present/updated.
2. Create/update `docs/ai-dev-process/integration.md` (migrate legacy command docs into it; do not delete legacy docs by default).
3. Generate managed Cursor `.mdc` rule files into `.cursor/rules/ai-dev-process/`.
4. Link/copy key guides into `.cursor/ai-dev-process/` for convenient prompting.
5. Optionally propose cleanup of legacy candidates as a separate explicit step.

## Cursor install targets

Create these directories in the host repo:
- `.cursor/rules/ai-dev-process/`
- `.cursor/ai-dev-process/`

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
- `.cursor/rules/ai-dev-process/error-handling.mdc` ← `Policies/error-handling.md`
- `.cursor/rules/ai-dev-process/unauthorized-changes.mdc` ← `Policies/unauthorized-changes.md`
- `.cursor/rules/ai-dev-process/safe-operations.mdc` ← `Policies/safe-operations.md`
- `.cursor/rules/ai-dev-process/swift-code-organization.mdc` ← `Policies/swift-code-organization.md` (Swift stack only)

## Installing guide copies for prompting

Copy (not symlink) these repo files into `.cursor/ai-dev-process/` as managed files:
- `Core/debugging-guide.md`
- `Spec/work-spec.md`
- `Spec/work-spec-implementation.md`
- `Spec/retro-prd.md`
- `Test/unit-testing-guide.md`
- `Test/unit-test-planning-guide.md`
- `Test/unit-test-infrastructure-guide.md`
- `Test/unit-test-writing-guide.md`

