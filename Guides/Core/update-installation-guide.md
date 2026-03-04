Managed-By: skai
Managed-Id: guide.update-installation
Managed-Source: Guides/Core/update-installation-guide.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-04

# Update Installation Guide

Updates the `skai` submodule and re-runs installed adapter runbooks.

## Checkpoints

This guide follows the shared process-flow mechanics in `Guides/Core/process-flow.md` (checkpoints, advance intent, `auto`, and the standard gate line).

Workflow-specific gate points (this guide must STOP and wait at these checkpoints):
- If `docs/skai/install-state.json` is missing: STOP and ask the human to run initial install/update runbooks first.
- After reporting what changed (changelog summary or `git log` fallback): STOP and wait for the human to acknowledge before proceeding to runbooks.

At checkpoints, end checkpoint output with the standard gate line (see `Guides/Core/process-flow.md`).

## Advance intent

Advance intent (and `auto`) semantics are defined in `Guides/Core/process-flow.md`.

## Prerequisites

- `docs/skai/install-state.json` must exist (created by the initial install).
  - If missing, STOP and tell the human to run the initial install/update runbook for each adapter first.

## Procedure

### 1. Read the install state file

Read `docs/skai/install-state.json` and extract:
- `submodulePath` (default: `Submodules/skai`)
- `lastSHA`
- `lastUpdatedAt`
- `installedAdapters` (list of adapter entries with `runbook` paths)

### 2. Check for upstream updates

```
cd <submodulePath>
git fetch
```

Compare the current HEAD SHA against `origin/main` (or the configured tracking branch). Also compare `lastSHA` (from step 1) to the current submodule HEAD.

If both conditions hold -- HEAD matches `origin/main` AND `lastSHA` matches HEAD -- report "skai is up to date" and stop.

If either condition is false, continue with the remaining steps. Step 3 will either update the submodule or no-op if HEAD already matches the remote.

### 3. Update the submodule

Record the old SHA, then:

```
git submodule update --remote <submodulePath>
```

### 4. Report what changed

- Read `<submodulePath>/CHANGELOG.md`.
- Show the human all changelog entries dated **after** `lastUpdatedAt` from the state file.
- If no new entries exist (edge case: non-changelog-worthy internal changes), note that and show a brief `git log --oneline <oldSHA>..<newSHA>` instead.
- **Wait for the human to acknowledge** before proceeding.

### 5. Run adapter install/update runbooks

For each entry in `installedAdapters`:
- Read the runbook at `<submodulePath>/<runbook>` and execute it.
- Run them sequentially (one adapter at a time).
- Each runbook's migration-capable algorithm handles any necessary updates (new skills, changed policies, deprecated artifacts, etc.).

### 6. Update install state

Update `docs/skai/install-state.json` with:
- The new submodule HEAD SHA
- Today's date (run `date +%Y-%m-%d` -- see `Install/managed-header.md`)
- Per-adapter `lastRunAt` timestamps
