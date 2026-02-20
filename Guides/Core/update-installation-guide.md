# Update Installation Guide

Updates the `ai-dev-process` submodule and re-runs installed adapter runbooks.

## Prerequisites

- `docs/ai-dev-process/install-state.json` must exist (created by the initial install).
  - If missing, STOP and tell the human to run the initial install/update runbook for each adapter first.

## Procedure

### 1. Read the install state file

Read `docs/ai-dev-process/install-state.json` and extract:
- `submodulePath` (default: `Submodules/ai-dev-process`)
- `lastSHA`
- `lastUpdatedAt`
- `installedAdapters` (list of adapter entries with `runbook` paths)

### 2. Check for upstream updates

```
cd <submodulePath>
git fetch
```

Compare the current HEAD SHA against `origin/main` (or the configured tracking branch).

If the submodule HEAD already matches the remote, report "ai-dev-process is up to date" and stop.

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

Update `docs/ai-dev-process/install-state.json` with:
- The new submodule HEAD SHA
- Today's date (run `date +%Y-%m-%d` -- see `Install/managed-header.md`)
- Per-adapter `lastRunAt` timestamps
