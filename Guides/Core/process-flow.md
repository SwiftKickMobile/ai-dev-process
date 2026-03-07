Managed-By: skai
Managed-Id: guide.process-flow
Managed-Source: Guides/Core/process-flow.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-07

# Process flow (house style reference)

Purpose: canonical definitions for process-flow mechanics (gates, advance intent, `auto`, and progress markers). This file is a **maintainer reference** used to ensure consistency across workflow guides. It is NOT a runtime dependency -- agents should not need to read this file during workflow execution. The operational core is inlined in each guide using the standard template defined in `maintain-skai.md`.

## Gates

Core rule: every time the agent is waiting on the human, the message must end with a `⏳ GATE:` line.

The only normal exception is full workflow completion, which uses:

`🏁 Complete. Let me know if anything needs adjustment.`

This is not a gate. It does not require advance intent.

### Planned gates

Planned gates are the expected review points of a workflow.

At a planned gate:
- The agent summarizes what it did and what should happen next.
- The message ends with:
  - `⏳ GATE: Next: <what happens after your response>. Say "next" or what to change.`
- The agent STOPs and waits for the human.

In the planned gate line, `<what happens after your response>` should describe what the agent will do after the human gives advance intent. If the gate is non-standard, it should instead describe the exact human response or handoff needed to resume the workflow.

### Blocked gates

Blocked gates are unexpected stops that require human input before the workflow can continue.

Use:

`⏳ GATE: Blocked: <reason>. Resolve and say "next" to continue.`

Blocked gates may occur at any point in a workflow. They are not limited to the planned gates listed in a guide.

Keep blocked-gate behavior generic unless a workflow has concrete blocker cases discovered through retros and documented in that guide.

## Advance intent

Advance intent moves past the current gate. Common signals include:
- `"begin"`
- `"next"`
- `"continue"`
- `"go ahead"`
- `"proceed"`
- `"do it"`

### Gate-bound by default

Rules:
- Recognize advance intent as approval to move past a gate only after the agent has output a `⏳ GATE:` line.
- Treat phrases like "we should...", "let's...", or "we'll..." as context-setting or discussion intent, not execution approval.
- If the agent is not currently at a gate, do not infer workflow advancement from collaborative phrasing.

### Local approval vs advance intent

Some workflows use local working markers inside an active phase or discussion loop.

In those workflows:
- Human approval of a specific local item may authorize clearing that item's local 🟡 marker.
- That local approval does **not** automatically count as advance intent for the whole workflow.
- The workflow still advances phases only when it reaches a gate and the human then gives advance intent there.

Use a local approval model only when the workflow explicitly documents that narrower exception.

## `auto`

`auto` means: apply advance intent repeatedly, bypassing planned gates until the workflow reaches completion or a required stop.

Blocked gates always require explicit human resolution. `auto` does not bypass them.

### `auto to <target>`

Bounded auto is a universal capability: the human may specify an upper bound, e.g. `auto to <target>`.

Meaning:
- Auto-advance as in `auto`, but STOP before the named workflow target.
- The target must refer to a stable identifier in the current workflow.
- The guide should define workflow-specific target names when helpful.
- If the target reference is unclear or there are multiple plausible interpretations, STOP and ask the human what target they mean.

Examples (infer from context):
- Work spec implementation: `"next auto to task 7"`
- Unit testing: `"next auto to success tests"`
- Any workflow with phases: `"next auto to Phase 2"`

### Universal required stops

`auto` does not bypass these required stops:
- missing or broken required tool/integration for the requested workflow
- ambiguity requiring product intent
- about to deviate from an explicit plan/spec/instruction
- about to run a destructive or irreversible operation
- required evidence or artifacts are unavailable
- new evidence contradicts the current hypothesis or plan

## 🟡 markers

This section applies only to skai workflow documents (work specs, planning docs, unit test plans, working docs), not to arbitrary developer notes.

- 🟡 means TODO.
- 🟡 may also mean "implemented but pending approval"; it must not be cleared until the human approves.

### Default marker model

Default rule:
- At a planned gate, advance intent is the approval signal for clearing the guide-owned 🟡 markers completed by the phase that just finished.

Ordering rule:
- The agent first STOPs and waits at the gate.
- The agent removes the approved 🟡 markers only after the human gives advance intent.

### Gate/phase markers vs local working markers

Some workflows use two layers of markers:
- **Gate/phase markers**: track whether a broader phase or gate is still open.
- **Local working markers**: track unresolved items inside that phase (for example, discussion topics or open questions).

When both exist:
- Keep them distinct in the most natural workflow artifact.
- Local approvals clear local working markers.
- Advance intent clears the gate/phase marker.
- The workflow guide must document the exact human intent that authorizes removal for each marker type.

### Custom marker lifecycles

Workflows may define narrower marker lifecycles when the default model would misdescribe the artifact's real behavior.

When a guide does this, it must document:
- where the markers live
- which markers track gate/phase state vs local working state
- what human intent authorizes removing each kind
- that the agent removes them only after that human intent is received

### Marker update protocol

Every 🟡 marker must be individually updatable via a stable identifier on the same line. Do not rely on line numbers.

Do not propagate markers up the hierarchy by accident:
- Do NOT add 🟡 to a parent heading or line solely because it contains child bullets that have 🟡.
- Only put 🟡 on the specific unresolved item lines when the unresolved state belongs to those items.
- A section or heading may carry 🟡 only when it intentionally tracks real gate/phase state for that section, not merely because child items remain open.

Allowed stable identifiers (choose the best fit per artifact):
- Work specs: task number + title line (e.g., `3. **Do thing** 🟡`)
- Unit tests: function name (e.g., `@Test func testFoo() ... // 🟡`) or MARK section title
- Planning/working docs: explicit IDs (e.g., `🟡 [Q-03] ...`, `🟡 [DEC-02] ...`) or an intentional section/heading marker when that heading tracks gate/phase state

Tooling expectation:
- Individual updates: use editor search (scoped to the current working document), or CLI ripgrep (`rg`) to locate the identifier quickly before editing.
- Bulk updates (optional): constrain to a single working document or a clearly scoped section, never repo-wide.

