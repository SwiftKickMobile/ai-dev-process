Managed-By: skai
Managed-Id: guide.ticket-implementation
Managed-Source: maintain-tickets.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-05-03

# Ticket implementation session

Purpose: work through process improvement tickets filed against this repo. The human initiates a session, the agent summarizes ready tickets, they discuss approach, and the agent implements with approval.

This is an internal maintenance workflow for the `skai` repo, not a guide for host projects.

## Prerequisites

- You are working in the `skai` repo.
- A GitHub MCP server configured with issue read/write permissions.
- The repo has open issues labeled `agent ready` (triaged and approved for agent implementation).

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

Workflow-specific gate notes:
- The phase-start gate is a non-standard planned gate. The human may review and resolve inline `🟡` proposal items there before the phase is ready for implementation planning. `Next` there means: the phase scope is settled enough to move to a concrete file-change plan.
- The phase-completion gate is also non-standard. `Next` there means: approve the implemented phase, clear the phase marker, and close the tickets covered by that phase before moving on.

Planned gates for this workflow:
- After creating/updating `working-docs/<branch-path>/<session-name>/ticket-planning.md` with the ready-ticket inventory and proposed phases, and after confirming which phases/tickets will be tackled in this session, but before initializing the first selected phase.
- At the start of each phase, after initializing that phase's discussion/proposals content and exposing any inline `🟡` items that still need human decisions.
- Before making repo changes for a phase, after proposing the concrete file-change plan.
- After implementing a phase, after reporting what changed and before approving the phase as complete.

## Procedure

### 1. Fetch ready tickets

Use the GitHub MCP server to list open issues labeled `agent ready`.

If no issues match, say **"No ready tickets found."** and stop.

### 2. Create the planning document

Choose a `session-name` for this ticket-maintenance session so multiple planning efforts on the same branch stay separate.

Create or update a working planning document at `working-docs/<branch-path>/<session-name>/ticket-planning.md` (path per `Guides/Core/working-doc-conventions.md`, session name: `<session-name>`, subpath: none, filename: `ticket-planning.md`).

For each ticket, include:
- A markdown link to the GitHub issue (not just `#123`).
- Paraphrased summary:
  - friction/problem (what is going wrong)
  - suggestion (what it proposes changing)
  - anything else of note (e.g., affected files, constraints, overlaps)
- Keep paraphrases concise (1-3 bullets per ticket). Do not paste the full ticket body.

Reconcile before proceeding: ensure every open issue labeled `agent ready` is represented in the planning document. If any are missing, add them before moving on.

### 3. Bucket into phases (when applicable)

Default behavior: group tickets by common theme into execution phases before implementing.

The goal is to implement cohesive improvements (one phase at a time), not blindly apply each ticket's suggested solution in isolation.

Rules:
- Treat the buckets as **execution phases** (Phase A, Phase B, etc.). Implement one phase fully before moving to the next.
- Capture overlaps explicitly so changes that touch the same area are coordinated.
- Seed each phase with proposals and 🟡 open items, and replace 🟡 inline as the human approves decisions (follow the planning protocol in `Guides/Spec/work-spec-creation.md`).

Format example:

```
### 🟡 Phase A: <theme> (#N, #M, ...)

Primary tickets:
- [#20 Hard prohibition on destructive git operations without explicit human approval](https://github.com/<owner>/<repo>/issues/20)
  - Friction: agent ran a destructive git command without permission and destroyed uncommitted work.
  - Suggestion: hard rule requiring explicit approval for destructive git commands, with scope/warning requirements.
  - Notes: should live in policy for universal enforcement.
```

Mark phases with 🟡 when TODO.

If there are only 1-2 tickets and they are obviously independent, you may make each one its own single-ticket phase rather than grouping multiple tickets into a broader theme.

## Advance intent

Advance intent moves past the current gate. Common signals: "next", "continue", "go ahead", "do it".

Rules:
- Recognized as approval to move past a gate only after you output a `⏳ GATE:` line.
- "we should...", "let's..." = discussion/context-setting, NOT authorization.
- Outside a gate, interpret "begin"/"next"/"continue" using the workflow's active-phase rules below. Do not use them to skip planning, phase discussion, or implementation approval.

`auto` = advance intent that bypasses planned gates only. Blocked gates always require explicit human resolution.
`auto to <milestone>` = auto-advance but STOP before the named planned gate. Use stable, workflow-specific milestone names.

Progress tracking:
- Default rule: 🟡 = TODO or pending approval. Do not clear 🟡 without human approval.
- The workflow-owned artifact is `working-docs/<branch-path>/<session-name>/ticket-planning.md`.
- Use each phase heading as the durable phase marker, for example: `### 🟡 Phase A: <theme>`.
- Use inline `🟡` items inside a phase for open planning questions, proposal choices, or unresolved scope details.
- At the planning-document gate, STOP with the selected phase headings still marked `🟡`.
- At the phase-start gate, local human decisions may clear inline `🟡` proposal items inside that phase. The phase heading `🟡` remains until implementation for that phase is approved.
- At the file-change-plan gate, STOP with the phase heading `🟡` still present.
- At the phase-completion gate, STOP with the phase heading `🟡` still present.
- After advance intent at the phase-completion gate, clear the phase heading `🟡`, close the tickets implemented in that phase, and move to the next remaining `🟡` phase.

Workflow-specific advance behavior:
- After the planning-document gate, advance intent means: initialize the first selected `🟡` phase and stop at that phase's start gate.
- At the phase-start gate, `next` means the phase scope is settled enough to move to a concrete file-change plan.
- At the file-change-plan gate, `next` means implement the approved phase.
- At the phase-completion gate, `next` means approve the implemented phase, clear the phase heading marker, close its tickets, and continue.
- `auto` may batch work inside an already-approved phase, but it does not bypass the planning-document, phase-start, file-change-plan, or phase-completion gates.

## Procedure (continued)

### 4. Summarize

Present a summary of all ready tickets, grouped if natural categories emerge. For each ticket, show:

- Issue number and title
- One-line summary of what needs to change
- Affected files (if noted in the ticket)

### 5. Discuss and prioritize

Ask the human which tickets to tackle in this session and in what order. The human may:

- Select specific tickets by number.
- Reorder priorities.
- Defer tickets to a future session.
- Ask questions or discuss implementation approach for specific tickets.

Do not proceed until the human confirms the selection.

Gate: STOP after the planning document is updated, summarized, and the session's selected phases/tickets are confirmed. End with `⏳ GATE: Next: Initialize the first selected phase for discussion. Say "next" or what to change.`

### 6. Implement

For each selected ticket, in the agreed order:

1. **Read the maintainer runbook.** Before making changes, read `maintain-skai.md` and follow its rules (safety defaults, bookkeeping, content rules).
2. **Propose a plan.** State what files you will change and how. Wait for approval.
3. **Make the changes.** Follow the repo's conventions:
   - Guides under `Guides/`, policies under `Policies/`, templates under `Templates/`.
   - Managed headers on all repo-source files.
   - ASCII only (no smart quotes or Unicode dashes).
   - Keep examples project-agnostic.
4. **Update bookkeeping.** For each change set:
   - `assets.manifest.json` (if assets were added/moved/renamed)
   - `README.md` Usage section (if a developer-facing workflow changed)
   - `CHANGELOG.md` (under "Unreleased", prefixed with today's date)
5. **Report.** Summarize what changed and list touched files.

After implementing a phase, report what changed and STOP at the phase-completion gate before moving to the next phase.

### 7. Close tickets

Close is triggered by human approval.

Default approval signal (phase-based sessions):
- Advance intent after a phase implementation report is approval. Close all tickets implemented in that phase.

Mechanics:
- Use the GitHub MCP server to close the issue with a comment noting it was implemented.
- If the implementation will be committed and pushed in a follow-up step, note that in the close comment instead.

### 8. Post-session

After all selected tickets are implemented:

- Run the `maintain-retro.md` checklist and report any misses.
- Remind the human to commit if they haven't already.
