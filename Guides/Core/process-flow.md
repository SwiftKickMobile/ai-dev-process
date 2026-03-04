Managed-By: skai
Managed-Id: guide.process-flow
Managed-Source: Guides/Core/process-flow.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-04

# Process flow (core)

Purpose: define shared workflow mechanics (checkpoints, advance intent, `auto`, and progress markers) so individual workflow guides can stay consistent without duplicating process rules.

## Checkpoints / gates

A checkpoint (gate) is a deliberate STOP point where the agent must wait for the human to advance the workflow.

At a checkpoint:
- The agent summarizes what it did and what it believes should happen next.
- The agent does not proceed until the human indicates advance intent.

## Standard gate line (required)

End every checkpoint output with exactly one of these lines:

- Continue:
  - `⏳ GATE: Next: <thing>. Say "next" or what to change.`
- Blocked:
  - `⏳ GATE: Blocked: <reason>. Resolve and say "next" to continue.`

Gate lines are strictly for defined checkpoints. Do not add a gate line at the end of any message that is not at a workflow-specific checkpoint listed in the guide's `## Checkpoints` section.

## Workflow completion

When a workflow finishes (all checkpoints exhausted, no more steps), do not use a gate line. Instead, state completion with:

`🏁 Complete. Let me know if anything needs adjustment.`

This is not a gate -- it does not require advance intent to proceed.

## Advance intent vs context-setting

Two separate concepts:

- Context-setting selects/activates a workflow:
  - Examples: "start implementation", "start debugging", "let's do a retro"
- Advance intent moves past the current checkpoint once a workflow is active.

Advance intent is not a strict command. Common examples:
- `"begin"`, `"next"`, `"continue"`
- "go for it"
- "proceed"
- "do it"

If it is unclear whether the human is setting context or advancing an existing workflow, STOP and ask which one they intend.

### Authorization recognition (gate-bound)

Approval to proceed is recognized only at a checkpoint.

Rules:
- Treat phrases like "we should...", "let's...", or "we'll..." as context-setting (discussion intent), not approval to execute changes.
- Treat "next", "do it", "go ahead", etc. as advance intent only when the agent is currently stopped at a checkpoint and the previous checkpoint output ended with the standard `⏳ GATE:` line.
- If the agent is not currently at a checkpoint, do not infer authorization to execute work. Propose the concrete plan and STOP at a checkpoint first.

## `auto`

`auto` means "advance intent, but bypass checkpoints".

### `auto` to a milestone (bounded auto)

Bounded auto is a universal capability: the human may specify an upper bound, e.g. "auto to <milestone>".

Meaning:
- Auto-advance as in `auto`, but STOP when you are about to begin the milestone.
- The milestone must refer to a stable identifier in the current workflow (e.g., a work spec task number, a test section name, etc.).
- If the milestone reference is unclear or there are multiple plausible interpretations, STOP and ask the human what milestone they mean.

Examples (infer from context):
- Work spec implementation: "next auto to task 7"
- Unit testing: "next auto to success tests"
- Any workflow with phases: "next auto to Phase 2"

`auto` does not bypass universal STOP conditions:
- missing/broken required tool or integration for the requested workflow
- ambiguity requiring product intent
- about to deviate from an explicit plan/spec/instruction (spec says X, about to do not-X)
- about to run a destructive/irreversible operation
- required evidence/artifacts are unavailable or new evidence contradicts the current hypothesis

## 🟡 markers (workflow-scoped)

This section applies only to skai workflow documents (work specs, planning docs, unit test plans, working docs), not to arbitrary developer notes.

- 🟡 means TODO.
- 🟡 may also mean "implemented but pending approval"; it must not be cleared until the human approves.

### Marker update protocol (individual updates)

Every 🟡 marker must be individually updatable via a stable identifier on the same line. Do not rely on line numbers.

Do not propagate markers up the hierarchy:
- Do NOT add 🟡 to a parent heading/ID line solely because it contains child bullets that have 🟡.
- Only put 🟡 on the specific unresolved item line(s) (questions/proposals/tradeoffs) so the number of 🟡 markers reflects the true number of open items.

Allowed stable identifiers (choose the best fit per artifact):
- Work specs: task number + title line (e.g., `3. **Do thing** 🟡`)
- Unit tests: function name (e.g., `@Test func testFoo() ... // 🟡`) or MARK section title
- Planning/working docs: explicit IDs (e.g., `🟡 [Q-03] ...`, `🟡 [DEC-02] ...`)

Tooling expectation:
- Individual updates: use editor search (scoped to the current working document), or CLI ripgrep (`rg`) to locate the identifier quickly before editing.
- Bulk updates (optional): constrain to a single working document or a clearly scoped section, never repo-wide.

