Managed-By: skai
Managed-Id: guide.update-installation
Managed-Source: Guides/Core/update-installation-guide.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-05-03

# Update Installation Guide

Updates the `skai` submodule and re-runs installed adapter runbooks.

## Gates

Core rule: every time the agent is waiting on the human, the message must end with a `⏳ GATE:` line. The only normal exception is full workflow completion, which uses `🏁 Complete. Let me know if anything needs adjustment.`

Use these standard gate lines:
- Planned gate: `⏳ GATE: Next: <what happens after your response>. Say "next" or what to change.`
- Blocked gate: `⏳ GATE: Blocked: <reason>. Resolve and say "next" to continue.`

The gate line is exclusively about advance intent. Place questions, proposals, options, drafts, and any other solicitation of human input *above* the gate line as discussion content. Never phrase a gate as "answer X, then I'll do Y" or "pick option A/B/C, then I'll proceed" -- that collapses discussion into advance intent and pre-commits the agent to execute on the next message regardless of how the human responds.

If the human's response contains discussion (answers, refinements, follow-up questions) but not explicit advance intent, treat the response as discussion: incorporate it (revise drafts, refine proposals, ask follow-ups) and re-gate. Do not auto-advance.

Planned gates are the expected review points of this workflow. At each planned gate:
1. Summarize what you did and what should happen next.
2. End with the planned gate line.
3. STOP and wait for the human.

In the planned gate line, `<what happens after your response>` should describe what the agent will do after the human gives advance intent. If the gate is non-standard, make it describe the exact human response or handoff needed to resume the workflow.

If an unexpected blocker prevents continued work, use the blocked gate line and STOP until the human resolves it.

Planned gates for this workflow:
- After reporting what changed (changelog summary or `git log` fallback), but before running the adapter install/update runbooks.

Workflow-specific blocked gates:
- `docs/skai/install-state.json` is missing, so the update workflow does not know what was previously installed.

## Advance intent

Advance intent moves past the current gate. Common signals: "next", "continue", "go ahead", "do it".

Rules:
- Recognized as approval to move past a gate only after you output a `⏳ GATE:` line.
- "we should...", "let's..." = discussion/context-setting, NOT authorization.
- Outside a gate, interpret "begin"/"next"/"continue" using the workflow's active-step rules below. Do not use them to skip required prerequisite checks.

`auto` = advance intent that bypasses planned gates only. Blocked gates always require explicit human resolution.
`auto to <milestone>` = auto-advance but STOP before the named planned gate. Use stable, workflow-specific milestone names.

Progress tracking:
- This workflow currently uses inline discussion rather than a required work document.
- The active state lives in the conversation plus `docs/skai/install-state.json`.
- At the changelog-review gate, STOP after summarizing what changed and wait for approval before re-running adapter runbooks.

Workflow-specific advance behavior:
- `auto` may bypass the changelog-review gate and continue directly into the adapter runbooks.
- `auto` does not bypass the blocked missing-install-state case.
- Use `runbooks` as the stable bounded-auto target for this guide.

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
- Gate: STOP after reporting what changed and end with `⏳ GATE: Next: Run the installed adapter update runbooks. Say "next" or what to change.`

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

Then complete the workflow.
