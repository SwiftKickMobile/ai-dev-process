# Install/Update Conflict & Precedence Policy

This policy defines how installers and update runbooks must behave in the presence of existing files.

## File classes

- **Managed file**: contains the managed header (`Managed-By: ai-dev-process`).
- **Managed symlink**: a symlink created by the installer that points at the expected repo-owned target path.
- **Managed block file**: a project-owned file that contains a delimited managed block (begin/end markers) that the installer may update in-place (e.g., ignore files).
- **Legacy candidate**: appears to be an older copy of a managed asset but lacks the managed header.
- **Project-owned file**: anything else (custom project content).

## Core rules

- Never overwrite project-owned files.
- Managed files may be overwritten deterministically.
- Managed symlinks may be overwritten deterministically if they already point to an `ai-dev-process` target.
- Managed block files may be updated deterministically, but only within the managed block.
- Legacy candidates must not be overwritten by default.
  - Migrate by generating new managed outputs in the current target locations.
  - Default migration plan should propose a cleanup action for any legacy candidates that are known install artifacts (delete or replace-with-symlink), but never execute without explicit human approval.

## When destination path already exists

- If destination does not exist: create it.
- If destination exists and is managed: overwrite/update it.
- If destination exists and is a symlink:
  - If it points to the expected `ai-dev-process` target: treat as managed symlink → update/replace as needed.
  - Otherwise: treat as project-owned → do not overwrite.
- If destination exists and contains a managed block: treat as managed block file → update only the block.
- If destination exists and is not managed:
  - Treat it as project-owned by default.
  - If it looks like a legacy candidate, classify it as legacy candidate and do not overwrite.
  - Create the new managed output in the current canonical location (e.g., `.cursor/rules/ai-dev-process/`) with a non-conflicting name only if needed.

## Cleanup step (always explicit)

Cleanup of legacy candidates is always a separate step:
- Present the list of legacy candidates.
- Ask whether to delete, keep, replace with a symlink, or strip overlapping content.
- Do not delete or strip without explicit approval.

