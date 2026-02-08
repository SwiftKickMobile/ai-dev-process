# Install/Update Conflict & Precedence Policy

This policy defines how installers and update runbooks must behave in the presence of existing files.

## File classes

- **Managed file**: contains the managed header (`Managed-By: ai-dev-process`).
- **Legacy candidate**: appears to be an older copy of a managed asset but lacks the managed header.
- **Project-owned file**: anything else (custom project content).

## Core rules

- Never overwrite project-owned files.
- Managed files may be overwritten deterministically.
- Legacy candidates must not be overwritten by default.
  - Migrate by generating new managed outputs in the current target locations.
  - Leave legacy candidates in place unless the human approves cleanup.

## When destination path already exists

- If destination does not exist: create it.
- If destination exists and is managed: overwrite/update it.
- If destination exists and is not managed:
  - Treat it as project-owned by default.
  - If it looks like a legacy candidate, classify it as legacy candidate and do not overwrite.
  - Create the new managed output in the current canonical location (e.g., `.cursor/rules/ai-dev-process/`) with a non-conflicting name only if needed.

## Cleanup step (always explicit)

Cleanup of legacy candidates is always a separate step:
- Present the list of legacy candidates.
- Ask whether to delete, keep, or strip overlapping content.
- Do not delete or strip without explicit approval.

