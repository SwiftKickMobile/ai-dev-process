# Maintain `ai-dev-process` (LLM maintainer runbook)

This file is instructions for an LLM making changes **to the `ai-dev-process` repo itself** (not installing into a host project).

## Safety defaults

- Do not commit unless explicitly asked.
- Do not delete files unless explicitly asked.
- Do not add dependencies unless explicitly asked.
- Keep edits minimal and reversible.

## Before you change anything

1. Read `README.md` to understand the intended system.
2. Identify what kind of change this is:
   - guide/policy content change
   - installer/runbook change
   - template change
   - repo metadata change (manifest/changelog/readme)
3. Propose a short plan and then proceed.

## Required bookkeeping (most common misses)

When you change assets, keep these in sync:
- `assets.manifest.json`
  - add/move/rename assets here
  - keep `id` stable when possible
- `README.md`
  - update the file-level Asset inventory for developer-facing guides
- `CHANGELOG.md`
  - add an entry under “Unreleased” for user-visible changes (prefix each line item with the date, `YYYY-MM-DD`)
 - Cursor skills (if you add/change them):
   - templates live at `Templates/cursor/skills/**/SKILL.md`
   - Cursor installer installs them into host repos at `.cursor/skills/`

## Content rules (project goals)

- Prefer IDE-neutral `.md` sources in this repo.
- IDE-specific outputs (e.g., Cursor `.mdc`) are generated into host repos by installer runbooks under `Install/`.
- Keep examples project-agnostic (use the shared fictional theme, currently “LumenNotes”).

## Installer invariants (do not regress)

The install/update runbooks must remain robust across:
- returning to a host repo later,
- updating the submodule,
- starting a fresh LLM session,
- running install/update again.

To preserve that:

- **Idempotency**: re-running install/update should converge to the same end state without manual cleanup.
- **Managed overwrites only**:
  - Generated host files are overwriteable only if they contain the managed header (`Install/managed-header.md`).
  - **Symlinks** are overwriteable only if they point at the expected `Submodules/ai-dev-process/...` targets (symlinks cannot contain headers).
  - **Managed blocks**: some project-owned files (e.g., ignore files, Integration doc) are updated only inside delimited managed blocks.
- **Legacy candidates are permission-gated**:
  - Identify legacy candidates.
  - Propose delete/replace (often replace-with-symlink), but do not execute without explicit approval.
- **Gray areas must be surfaced**:
  - Example: legacy `.cursor/rules/debugging.mdc` should be flagged and deletion should be proposed only after migrating any project-specific logging conventions into the Integration doc and getting approval.

## Integration doc architecture (do not regress)

The Integration doc is project-owned at `docs/ai-dev-process/integration.md`, but is structured to support safe automation.

Format rules:
- `Templates/docs/ai-dev-process/integration.md` must be **minimal** and must not contain “meta” guidance.
  - No instructions to the installer/LLM (those belong in `Install/integration-doc-install-update.md`).
  - No instructions to humans about how to clear 🟡 markers (those belong in `README.md`).

Ownership rules:
- Humans may edit only the **Special instructions / overrides** section (freeform).
- The installer owns:
  - the `required-values` “form” block (structure + restoration of missing fields)
  - stack-specific sections inside `BEGIN/END Managed-By: ai-dev-process` blocks.
- Stack-specific templates live under:
  - `Templates/docs/ai-dev-process/integration-sections/`

When changing Integration templates/sections:
- Keep managed block markers minimal and stable:
  - `<!-- BEGIN Managed-By: ai-dev-process | Section: <id> -->`
  - `<!-- END Managed-By: ai-dev-process | Section: <id> -->`
- Keep 🟡 markers only for true project-specific missing constants/mappings.
  - Do not mark variables (e.g., `<Scheme>`, `<TestPlan>`, `<TestTarget>`) with 🟡.
  - Do not mark standard procedures/patterns with 🟡.
- Ensure section templates treat placeholders like `<Scheme>`, `<TestPlan>`, `<TestTarget>` as **variables** (agent-filled per task context).
- Ensure section templates are **integration-doc-ready** content:
  - no 🟡 TODO lists
  - no “installer/human instructions” (those belong in `README.md` and `Install/integration-doc-install-update.md`)
  - include copy/pasteable CLI command templates with `<...>` variables
- Ensure `Install/integration-doc-install-update.md` remains the canonical installer guidance for:
  - managed block merge/update/remove behavior
  - `{human response}` placeholders + `INSTRUCTION:` formatting and restoration rules

## If you change install/runbooks

After editing anything under `Install/`:
- Ensure `Install/conflict-precedence-policy.md` still matches the runbooks (managed header vs managed symlink behavior).
- Ensure `assets.manifest.json` includes any new policy/guide/runbook IDs referenced by the installer.
- Ensure `README.md` Quick start prompts still point at the correct submodule path (`Submodules/ai-dev-process/...`).
- Ensure ignore-file behavior remains safe: update `.cursorignore` / `.claudeignore` via managed blocks and do not hide the submodule via agent ignore (use editor UI excludes instead).

## After changes (retro)

After making changes, run the post-change checklist in `maintain-retro.md`.

## How to guide an LLM (prompt template)

Use this as a starting prompt when maintaining this repo:

> You are updating the `ai-dev-process` repo.
> Follow `README.md` and `maintain-ai-dev-process.md`.
> Make minimal changes.
> Update `assets.manifest.json`, `README.md` (asset inventory), and `CHANGELOG.md` as needed.
> Do not commit or delete files unless I explicitly ask.
> After edits, summarize exactly what changed and list touched files.

