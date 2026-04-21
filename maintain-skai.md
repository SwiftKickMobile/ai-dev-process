# Maintain `skai` (LLM maintainer runbook)

This file is instructions for an LLM making changes **to the `skai` repo itself** (not installing into a host project).

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
4. After significant edits (multi-file changes, path moves, installer behavior changes), run `maintain-retro.md` and fix straightforward misses immediately.

## Required bookkeeping (most common misses)

When you change assets, keep these in sync:
- `assets.manifest.json`
  - add/move/rename assets here
  - keep `id` stable when possible
- `README.md`
  - update the Usage section for developer-facing skills (see "README Usage section conventions" below)
- `CHANGELOG.md`
  - add an entry under "Unreleased" for user-visible changes (prefix each line item with the date, `YYYY-MM-DD`)
  - always determine the date by running `date +%Y-%m-%d` in the terminal (see `Install/managed-header.md`, "Determining today's date")
  - treat changelog entries as release notes for humans:
    - prefer multiple short bullets over one giant "mega-entry"
    - each bullet should have a short bold headline and 1-2 sentences max
    - group by theme when many files change (core mechanics, policies, spec guides, test guides, templates, etc.)
 - Skills (if you add/change them):
   - shared templates live at `Templates/skills/skai-*/SKILL.md`
   - Cursor installer installs them into host repos at `.cursor/skills/`
   - Claude Code installer installs them into host repos at `.claude/skills/`
   - Codex installer installs them into host repos at `.agents/skills/`
   - **Wrapper pattern (invariant)**: skill templates must be thin wrappers -- a few lines that point the LLM at the corresponding Guide(s) in `Guides/`. All substantive logic lives in the Guide, not in the skill template. This keeps the files copied into host projects small and ensures the Guide is the single source of truth.
   - When adding a new skill: start by writing the Guide under `Guides/`, then create the skill wrapper that references it. This order prevents accidentally inlining logic into the skill template.
 - Install state file:
   - All adapter runbooks write `docs/skai/install-state.json` on successful completion.
   - The `update-installation` skill reads this file to determine which adapters to re-run and what SHA was last applied.
   - If you add a new adapter runbook, ensure it writes/merges its entry into this file.

## Guides/ subdirectory convention

All guides live under a topical subdirectory within `Guides/`. Do not place guides at the `Guides/` top level.

| Subdirectory | Purpose |
|---|---|
| `Core/` | Cross-cutting foundations: debugging, working-doc conventions, update installation |
| `Spec/` | Work spec creation and implementation |
| `Test/` | Unit testing (planning, infrastructure, writing) |
| `Process/` | Retro, process problem reporting, ticket creation |

When adding a new guide, place it in the most appropriate existing subdirectory. If none fits, propose a new subdirectory and document it here.

## Guide house style

Applies to all workflow documents: files under `Guides/`, internal runbooks (`maintain-*.md`), and skill entry points (`SKILL.md` files under `Templates/skills/`, `.cursor/skills/`, `.claude/skills/`, and `.agents/skills/`). Follow these rules whenever adding or substantially editing any of them.

### Required elements

- **Managed header**: file must include the managed header (`Managed-By`, `Managed-Id`, `Managed-Source`, `Managed-Adapter`, `Managed-Updated-At`).
- **Terminology**: use "advance intent" (never "Next Command").
- **Lean core, deep links**: keep core guides concise; if a tactic/procedure is detailed and rarely used, place it in a dedicated guide/appendix and link to it from the core guide.

### Standard structure for guides with gates

If the guide has any STOP points / gates, inline the process-flow operational template in both sections. Do not reference `Guides/Core/process-flow.md` as a runtime dependency -- agents skip indirect references.

```
## Gates

Core rule: every time the agent is waiting on the human, the message must end with a `⏳ GATE:` line. The only normal exception is full workflow completion, which uses `🏁 Complete. Let me know if anything needs adjustment.`

Use these standard gate lines:
- Planned gate: `⏳ GATE: Next: <what happens after your response>. Say "next" or what to change.`
- Blocked gate: `⏳ GATE: Blocked: <reason>. Resolve and say "next" to continue.`

Planned gates are the expected review points of this workflow. At each planned gate:
1. Summarize what you did and what should happen next.
2. End with the planned gate line.
3. STOP and wait for the human.

In the planned gate line, `<what happens after your response>` should describe what the agent will do after the human gives advance intent. If the gate is non-standard, make it describe the exact human response or handoff needed to resume the workflow.

If an unexpected blocker prevents continued work, use the blocked gate line and STOP until the human resolves it.

Workflow-specific gate notes (optional):
- If a planned gate has workflow-specific semantics rather than the default "review complete, say `next` to advance" behavior, state the exact gate response and what `Next` means there.
- Review/handoff gates are one common example, but not the only allowed variation.
- Recognition test for a non-standard planned gate:
  - If the human response needed there is not simply "review complete; `next` means approve-and-advance", treat it as non-standard.
  - If leaving the gate standard would misdescribe what the human should do next, clear 🟡 markers too early, or advance to the wrong phase, treat it as non-standard.
  - For any non-standard planned gate, document the exact `⏳ GATE: Next: ...` response, what kind of human response resumes the workflow, what `Next` means there, and how progress markers track that gate.
- For any non-standard planned gate, use 🟡 markers in the most natural workflow artifact to show what remains open for that gate.
- Make the marker lifecycle explicit:
  - which 🟡 markers track gate/phase state vs local working state
  - what specific human intent approves clearing them
  - that the agent removes them only after that human intent is received
- If the workflow has guide-specific blocked cases discovered through retros, list them here. Otherwise keep blocked behavior generic.

Planned gates for this workflow:
- <gate 1>
- <gate 2>

## Advance intent

Advance intent moves past the current gate. Common signals: "next", "continue", "go ahead", "do it".

Rules:
- Recognized as approval to move past a gate only after you output a `⏳ GATE:` line.
- "we should...", "let's..." = discussion/context-setting, NOT authorization.
- Outside a gate, interpret "begin"/"next"/"continue" using the workflow's active-phase rules below. Do not use them to skip phases or clear unrelated 🟡 markers.

`auto` = advance intent that bypasses planned gates only. Blocked gates always require explicit human resolution.
`auto to <milestone>` = auto-advance but STOP before the named planned gate. Use stable, workflow-specific milestone names.

Progress tracking:
- Default rule: 🟡 = TODO or pending approval. Do not clear 🟡 without human approval.
- At a planned gate, advance intent is the approval signal for clearing the guide-owned 🟡 markers completed by the phase that just finished.
- Ordering rule: the agent first stops and waits at the gate, then removes the approved 🟡 markers only after the human gives advance intent.
- If a workflow needs a custom marker lifecycle (for example, inline discussion markers resolved during a human-led discussion loop), define that exception explicitly and narrowly below.
- If a workflow uses both gate/phase markers and local working markers, keep them distinct: local approvals clear local working markers; advance intent clears the gate/phase marker.
- For any custom marker lifecycle, specify both:
  - the artifact where the 🟡 markers live
  - the exact human intent that authorizes their removal

<workflow-specific advance behavior>
```

**Gates section rules:**
- Copy the standardized gates template verbatim, then add workflow-specific planned gates.
- If the guide has a custom review/handoff gate, explicitly state the exact `⏳ GATE: Next: ...` response and what human action resumes the workflow.
- Keep blocked-gate behavior generic unless the workflow has concrete blocker cases discovered through retros.
- Every place the guide says the agent must STOP and wait on the human must correspond to a `⏳ GATE:` line in the runtime behavior.
- After the template block, list this guide's workflow-specific gates.

**Advance intent section rules:**
- Copy the standardized advance intent template verbatim (the block above starting "Advance intent moves past").
- After the template block, add workflow-specific behavior (if any): custom advance actions, `auto` milestone names, custom marker-lifecycle exceptions, hard gates that `auto` does not bypass.

### Process-flow house style

`Guides/Core/process-flow.md` is the **house style reference** for process-flow mechanics. It is the canonical definition used by maintainers to ensure consistency across guides. It is NOT a runtime reference that agents read during workflow execution.

The operational core is inlined in each guide using the standard template above. When process-flow house style changes:
1. Update `Guides/Core/process-flow.md` (the canonical definition).
2. Update the standard template in this section.
3. Propagate the template changes to all workflow guides.

**What guides should contain (inline):**
- The standardized gates and advance intent template (copied verbatim)
- Workflow-specific gate descriptions, phase sequencing, `auto` skip/fix rules
- Workflow-specific procedures, examples, and reference material

**What guides should NOT contain:**
- References to `Guides/Core/process-flow.md` as a required runtime read
- "Flow" or "Process Overview" subsections that embed checkpoint behavior or marker removal steps alongside phase sequencing. State only which phases exist and their order.
- Standalone "Progress Tracking" sections that restate 🟡 = TODO / no marker = complete.
- Inline restatements of marker removal timing (e.g., "remove 🟡 when complete") unless gated on advance intent.

### Flow reflection (required for guides with gates)

After editing a guide with gates, imagine a fresh agent using it from the start of the workflow and mentally step through the first few human/agent exchanges plus each gate transition.

Required post-edit simulation:
- Explicitly simulate at least:
  - one normal transition
  - one blocked case
  - one `auto` case
- If the simulation exposes a gap, patch the guide immediately and re-run the affected simulation.
- If the simulation exposes no issues, record `No issues found in simulation` in the active working notes before moving to the next guide.

For each stop, verify:
- why the agent is stopping
- the exact `⏳ GATE:` line the agent should emit
- what human response resumes the workflow
- whether that response is advance intent, local approval, or some other workflow-specific action
- which workflow-owned artifact is tracking that gate or phase
- which specific 🟡 marker is still present while waiting
- what 🟡 marker changes, and only after what human intent
- what exact artifact change happens after that human intent

Look specifically for:
- waits on the human that do not end with a `⏳ GATE:` line
- gates where `Next` is unclear or misleading
- planned gates that have no workflow-owned artifact or no gate/phase marker to clear
- marker removals that happen before approval
- non-standard gates whose custom semantics are implied but not stated
- top-level summaries that do not match the actual gated sequence

### Pre-edit workflow model (required for guides with gates)

Before rewriting a guide with gates:
- Extract the workflow model first:
  - happy-path progression
  - workflow-owned artifact(s)
  - marker lifecycle
  - blocker categories
- Simulate at least:
  - one normal transition
  - one blocked/tooling failure
  - one ambiguity/spec flaw
  - one `auto` case
- Only then apply the standardized gate/advance-intent template.

### Sanity scan (after every edit)

Run this scan after *each* guide edit, not just at the end of a batch. Deferring the scan to the end of a batch is the proven failure mode -- violations introduced early in the batch survive unchecked.

Checks:

**Structure:**
- Missing `## Gates` section (if the guide has gates)
- Missing `## Advance intent` section (if the guide has gates)
- Inconsistent terms ("Next Command" instead of "advance intent")
- Mixed gate terminology ("checkpoint" and "gate" used interchangeably without a workflow-specific reason)
- Stack-mismatched references (e.g., Xcode terms in Android-only sections)
- If the guide creates working docs or artifacts, stale path layouts that skip the required `session-name` folder

**Process-flow template compliance:**
- Gates section missing the standardized gates template (the block starting "Core rule: every time the agent is waiting on the human...")
- Advance intent section missing the standardized advance intent template (the block starting "Advance intent moves past the current gate.")
- References to `Guides/Core/process-flow.md` that imply it is a required runtime read (e.g., "This guide follows the shared process-flow mechanics in...")
- Standalone "Progress Tracking" or "Emoji System" sections that redefine 🟡

**Flow control integrity:**
- Any place where the guide says the agent waits for the human but does not end with a `⏳ GATE:` line in the described runtime behavior.
- Any planned gate that does not have a workflow-owned artifact and a gate/phase marker that remains while waiting for advance intent.
- Any "remove 🟡" language that is not explicitly gated on advance intent or a documented workflow-specific exception. Search for `remove 🟡` and `Remove 🟡` and verify each occurrence is tied either to a gate/advance-intent step or to an explicitly documented custom lifecycle.
- Any guide where the planned gate exists but the post-approval artifact change is not explicit (for example, no statement of which marker is removed after advance intent).
- Flow/sequence descriptions where marker removal appears before a STOP/gate (wrong order -- the STOP comes first, marker removal happens on the subsequent advance intent unless a documented local exception applies).
- Examples or walkthroughs that show markers being removed without a preceding gate + advance intent step, unless the guide explicitly documents a narrower exception (for example, inline discussion markers resolved during human-approved discussion).
- `auto` sections that list gates bypassed by `auto` without labeling which gates are hard (not bypassed). If a guide has both soft and hard gates, the `## Gates` section should label the hard ones.
- Inline "Gate:" signposts that add mechanics not present in the `## Gates` section (acceptable to say "Gate: STOP"; not acceptable to add new rules about what happens at the gate unless they are already defined in the gate model).
- If the guide defines working-doc paths, any examples or artifact paths that place files directly under `working-docs/<branch-path>/...` instead of `working-docs/<branch-path>/<session-name>/...`.

## README Usage section conventions

The Usage section documents each **skill** (not sub-processes or internal maintenance workflows). Each entry follows this format:

1. Heading: `### <Human-readable name> (skill <skill-name>)` (no colon)
2. One-paragraph description of what the workflow does.
3. **Prerequisites:** what's needed before starting.
4. **Phases:** numbered list of the workflow's phases. Each phase is one line: `<number>. **<Phase name>.** <What happens>.`
   - If a phase has sub-steps, use a numbered sub-list under the phase.
   - If an entire phase is optional, mark it `(optional)` in the phase name.

Formatting rule:
- When referencing a file or directory path within this repo in `README.md`, use a markdown link so humans can click it in the browser (do not leave bare backticked paths).

Do not add entries for:
- Sub-processes that are invoked by other skills (e.g., the filing phase of the process-improvement guide is invoked by the retro -- it appears as a phase within that entry, not as its own heading).
- Internal maintenance workflows (e.g., `maintain-tickets.md` -- these belong in the Development guide section).

## Content rules (project goals)

- Prefer IDE-neutral `.md` sources in this repo.
- IDE-specific outputs (e.g., Cursor `.mdc`) are generated into host repos by installer runbooks under `Install/`.
- **ASCII only for quotes and dashes.** Use straight quotes (`"`, `'`), hyphens (`-`), and double-hyphens (`--`) for em-dashes. Never use smart/curly quotes (`"` `"` `'` `'`) or Unicode dashes (`-` `--`). These break tooling (e.g., `StrReplace`) that matches on ASCII equivalents.
- Keep examples project-agnostic (use the shared fictional theme, currently "LumenNotes").

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
  - **Symlinks** are overwriteable only if they point at the expected `Submodules/skai/...` targets (symlinks cannot contain headers).
  - **Managed blocks**: some project-owned files (e.g., ignore files, Integration doc) are updated only inside delimited managed blocks.
- **Legacy candidates are permission-gated**:
  - Identify legacy candidates.
  - Propose delete/replace (often replace-with-symlink), but do not execute without explicit approval.
- **Gray areas must be surfaced**:
  - Example: legacy `.cursor/rules/debugging.mdc` should be flagged and deletion should be proposed only after migrating any project-specific logging conventions into the Integration doc and getting approval.

## Integration doc architecture (do not regress)

The Integration doc is project-owned at `docs/skai/integration.md`, but is structured to support safe automation.

Format rules:
- `Templates/docs/skai/integration.md` must be **minimal** and must not contain "meta" guidance.
  - No instructions to the installer/LLM (those belong in `Install/integration-doc-install-update.md`).
  - No instructions to humans about how to clear 🟡 markers (those belong in `README.md`).

Ownership rules:
- Humans may edit only the **Special instructions / overrides** section (freeform).
- The installer owns:
  - the `required-values` "form" block (structure + restoration of missing fields)
  - stack-specific sections inside `BEGIN/END Managed-By: skai` blocks.
- Stack-specific templates live under:
  - `Templates/docs/skai/integration-sections/`

When changing Integration templates/sections:
- Keep managed block markers minimal and stable:
  - `<!-- BEGIN Managed-By: skai | Section: <id> -->`
  - `<!-- END Managed-By: skai | Section: <id> -->`
- Keep 🟡 markers only for true project-specific missing constants/mappings.
  - Do not mark variables (e.g., `<Scheme>`, `<TestPlan>`, `<TestTarget>`) with 🟡.
  - Do not mark standard procedures/patterns with 🟡.
- Ensure section templates treat placeholders like `<Scheme>`, `<TestPlan>`, `<TestTarget>` as **variables** (agent-filled per task context).
- Ensure section templates are **integration-doc-ready** content:
  - no 🟡 TODO lists
  - no "installer/human instructions" (those belong in `README.md` and `Install/integration-doc-install-update.md`)
  - include copy/pasteable CLI command templates with `<...>` variables
- Ensure `Install/integration-doc-install-update.md` remains the canonical installer guidance for:
  - managed block merge/update/remove behavior
  - `{human response}` placeholders + `INSTRUCTION:` formatting and restoration rules

## If you change install/runbooks

After editing anything under `Install/`:
- Ensure `Install/conflict-precedence-policy.md` still matches the runbooks (managed header vs managed symlink behavior).
- Ensure `assets.manifest.json` includes any new policy/guide/runbook IDs referenced by the installer.
- Ensure `README.md` Quick start prompts still point at the correct submodule path (`Submodules/skai/...`).
- Ensure ignore-file behavior remains safe: update `.cursorignore` / `.claudeignore` via managed blocks and do not hide the submodule via agent ignore (use editor UI excludes instead).

## After changes (retro)

After making changes, run the post-change checklist in `maintain-retro.md`.

## How to guide an LLM (prompt template)

Use this as a starting prompt when maintaining this repo:

> You are updating the `skai` repo.
> Follow `README.md` and `maintain-skai.md`.
> Make minimal changes.
> Update `assets.manifest.json`, `README.md` (asset inventory), and `CHANGELOG.md` as needed.
> Do not commit or delete files unless I explicitly ask.
> After edits, summarize exactly what changed and list touched files.
