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
   - Claude Code installers install them into host repos at `.claude/skills/`
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

Applies to all workflow documents: files under `Guides/`, internal runbooks (`maintain-*.md`), and skill entry points (`SKILL.md` files under `Templates/skills/` and `.cursor/skills/`). Follow these rules whenever adding or substantially editing any of them.

### Required elements

- **Managed header**: file must include the managed header (`Managed-By`, `Managed-Id`, `Managed-Source`, `Managed-Adapter`, `Managed-Updated-At`).
- **Terminology**: use "advance intent" (never "Next Command").
- **Lean core, deep links**: keep core guides concise; if a tactic/procedure is detailed and rarely used, place it in a dedicated guide/appendix and link to it from the core guide.

### Standard structure for guides with gates

If the guide has any STOP points / gates, use this structure for the process-flow sections:

```
## Checkpoints
<one-line reference to process-flow for shared mechanics>
<enumerate this guide's workflow-specific gates -- one bullet per gate>

## Advance intent
<one-line reference to process-flow>
<workflow-specific advance behavior ONLY if it differs from the default>
```

**Checkpoints section rules:**
- Reference `Guides/Core/process-flow.md` for shared mechanics (do not restate what checkpoints are or how gate lines work).
- List only this guide's planned gates -- one bullet per gate describing what the agent presents and why it stops.

**Advance intent section rules:**
- Start with a one-line reference to process-flow (e.g., "Advance intent (and `auto`) semantics are defined in `Guides/Core/process-flow.md`.").
- Add workflow-specific behavior only when this guide's advance action differs from the default (e.g., "marks previous task complete and begins next task").
- Do not restate what advance intent means, how authorization works, or general `auto` semantics.
- Workflow-specific `auto` rules (skip conditions, auto-fix rules) belong here.

### Process-flow separation (do not restate general mechanics)

`Guides/Core/process-flow.md` is the single source of truth for:
- What a checkpoint/gate is and how it works
- The standard gate line variants (Continue, Blocked) and workflow completion line
- Advance intent recognition and authorization
- General `auto` semantics and universal STOP conditions
- 🟡 marker semantics (TODO, pending approval, must not clear without approval)
- Marker update protocol

**Do not restate these in individual guides.** Restating general mechanics creates contradictions when either source is updated independently. Proven failure mode: a guide's "Flow" section said markers are removed automatically when tests pass, contradicting process-flow's rule that markers require human approval.

Specifically, do not include:
- "Flow" or "Process Overview" subsections that embed checkpoint behavior or marker removal steps alongside phase sequencing. State only which phases exist and their order.
- Standalone "Progress Tracking" sections that restate 🟡 = TODO / no marker = complete.
- "Quick Reference" content that restates process-flow rules (checkpoint behavior, marker semantics, authorization). Quick references should contain only workflow-specific content (e.g., test commands, file paths, task structure).
- Inline restatements of marker removal timing (e.g., "remove 🟡 when complete") -- process-flow already defines when markers are cleared.

**What guides should contain:**
- Workflow-specific phase sequencing (which phases, their order, what each phase does)
- Workflow-specific gate descriptions (what the agent presents at each gate)
- Workflow-specific `auto` skip/fix rules
- Workflow-specific procedures, examples, and reference material

### Sanity scan (after every edit)

Run this scan after *each* guide edit, not just at the end of a batch. Deferring the scan to the end of a batch is the proven failure mode -- violations introduced early in the batch survive unchecked.

Checks:

**Structure:**
- Missing `## Checkpoints` section (if the guide has gates)
- Missing `## Advance intent` section (if the guide has gates)
- Inconsistent terms ("Next Command" instead of "advance intent")
- Stack-mismatched references (e.g., Xcode terms in Android-only sections)

**Process-flow separation:**
- Restated process-flow mechanics (marker semantics, checkpoint behavior, authorization rules)
- Standalone "Progress Tracking" or "Emoji System" sections that redefine 🟡

**Flow control integrity:**
- Any "remove 🟡" language that is not explicitly gated on advance intent or a checkpoint. Search for `remove 🟡` and `Remove 🟡` and verify each occurrence is tied to an advance intent trigger, not to an autonomous condition (e.g., "when complete", "as you complete them").
- Flow/sequence descriptions where marker removal appears before a STOP/checkpoint (wrong order -- the checkpoint must come first, marker removal happens on the subsequent advance intent).
- Examples or walkthroughs that show markers being removed without a preceding checkpoint + advance intent step.
- `auto` sections that list gates bypassed by `auto` without labeling which gates are hard (not bypassed). If a guide has both soft and hard gates, the Checkpoints section should label the hard ones.
- Inline "Checkpoint:" signposts that add mechanics not present in the `## Checkpoints` section (acceptable to say "Checkpoint: STOP"; not acceptable to add new rules about what happens at the checkpoint).

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

