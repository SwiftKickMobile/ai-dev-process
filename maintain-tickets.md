Managed-By: ai-dev-process
Managed-Id: guide.ticket-implementation
Managed-Source: maintain-tickets.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-02-28

# Ticket implementation session

Purpose: work through process improvement tickets filed against this repo. The human initiates a session, the agent summarizes ready tickets, they discuss approach, and the agent implements with approval.

This is an internal maintenance workflow for the `ai-dev-process` repo, not a guide for host projects.

## Prerequisites

- You are working in the `ai-dev-process` repo.
- A GitHub MCP server configured with issue read/write permissions.
- The repo has open issues labeled `agent ready` (triaged and approved for agent implementation).

## Checkpoints

This guide follows the shared process-flow mechanics in `Guides/Core/process-flow.md` (checkpoints, advance intent, `auto`, and the standard gate line).

Workflow-specific gate points (this guide must STOP and wait at these checkpoints):
- After bucketing ready tickets into phases in `working-docs/ticket-planning.md` (human reviews the phase breakdown and open 🟡 items).
- At the start of each phase, after initializing that phase's discussion/proposals content (human reviews/decides on any 🟡 items before implementation begins).
- Before making repo changes for a phase (agent proposes the concrete file-change plan; human approves).
- After implementing a phase (agent reports what changed; human marks the phase `Status: COMPLETE` or requests changes).

## Procedure

### 1. Fetch ready tickets

Use the GitHub MCP server to list open issues labeled `agent ready`.

If no issues match, say **"No ready tickets found."** and stop.

### 2. Bucket into phases (default)

Default behavior: do not implement tickets one-by-one without first grouping them by theme.

The goal is to implement cohesive improvements (one phase at a time), not blindly apply each ticket's suggested solution in isolation.

Rules:
- Create or update a working planning document at `working-docs/ticket-planning.md`.
- Read all ready tickets, then bucket them by common theme.
- Reconcile before proceeding: ensure every open issue labeled `agent ready` is represented in the planning document (listed under exactly one phase, with overlaps as needed). If any are missing, add them before moving on.
- Treat the buckets as **execution phases** (Phase A, Phase B, etc.). Implement one phase fully before moving to the next.
- Capture overlaps explicitly so changes that touch the same area are coordinated.
- Seed each phase with proposals and 🟡 open items, and replace 🟡 inline as the human approves decisions (follow the planning protocol in `Guides/Spec/work-spec-creation.md`).
- Bucket content requirements (each phase must include):
  - Ticket lists include links: every ticket must be listed as a markdown link to the GitHub issue (not just `#123`).
  - For each ticket, paraphrase:
    - friction/problem (what is going wrong)
    - suggestion (what it proposes changing)
    - anything else of note (e.g., affected files, constraints, overlaps)
  - Keep paraphrases concise (1-3 bullets per ticket). Do not paste the full ticket body.

Format example:

```
Primary tickets:
- [#20 Hard prohibition on destructive git operations without explicit human approval](https://github.com/<owner>/<repo>/issues/20)
  - Friction: agent ran a destructive git command without permission and destroyed uncommitted work.
  - Suggestion: hard rule requiring explicit approval for destructive git commands, with scope/warning requirements.
  - Notes: should live in policy for universal enforcement.
```

Skip condition (optional):
- If there are only 1-2 tickets and they are obviously independent, you may skip bucketing and implement directly.

Checkpoint: STOP after updating `working-docs/ticket-planning.md` and wait for advance intent (use the standard gate line; see `Guides/Core/process-flow.md`).

## Commands

### Advance intent

**Definition:** Advance intent. See `Guides/Core/process-flow.md`.

**Behavior in this workflow:**
- If you are at a phase boundary (e.g., "move on to Phase D"): advance intent means "initialize the next phase's discussion content" in `working-docs/ticket-planning.md` (seed proposals/questions/🟡 under that phase) and then STOP at the phase-start checkpoint.
- If you are ready to implement a phase (phase discussion resolved): advance intent means "propose the concrete file-change plan for the phase" and STOP for approval before making repo changes.
- If you just finished implementing a phase: advance intent means "proceed to the next phase" (or conclude if no more phases remain).

Authorization recognition: see `Guides/Core/process-flow.md` ("Authorization recognition (gate-bound)").

### Advance intent + `auto`

Advance intent followed by `auto` (e.g., "next auto"). See `Guides/Core/process-flow.md` for shared `auto` semantics and universal STOP conditions.

In this workflow, `auto` may be used to batch work inside an already-approved phase, but it does not bypass universal STOP conditions (e.g., destructive operations, missing required tools, spec/plan deviation).

### 3. Summarize

Present a summary of all ready tickets, grouped if natural categories emerge. For each ticket, show:

- Issue number and title
- One-line summary of what needs to change
- Affected files (if noted in the ticket)

### 4. Discuss and prioritize

Ask the human which tickets to tackle in this session and in what order. The human may:

- Select specific tickets by number.
- Reorder priorities.
- Defer tickets to a future session.
- Ask questions or discuss implementation approach for specific tickets.

Do not proceed until the human confirms the selection.

### 5. Implement

For each selected ticket, in the agreed order:

1. **Read the maintainer runbook.** Before making changes, read `maintain-ai-dev-process.md` and follow its rules (safety defaults, bookkeeping, content rules).
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

After implementing a ticket, ask the human if the result looks good before moving to the next one.

### 6. Close tickets

Close is triggered by human approval.

Default approval signal (phase-based sessions):
- When working from `working-docs/ticket-planning.md` phases, the human marking a phase as `Status: COMPLETE` is approval of the implementation for that phase.
- Once a phase is marked COMPLETE, close all tickets implemented in that phase.

Non-phase sessions:
- After the human approves the implementation for an individual ticket, close it.

Mechanics:
- Use the GitHub MCP server to close the issue with a comment noting it was implemented.
- If the implementation will be committed and pushed in a follow-up step, note that in the close comment instead.

### 7. Post-session

After all selected tickets are implemented:

- Run the `maintain-retro.md` checklist and report any misses.
- Remind the human to commit if they haven't already.
