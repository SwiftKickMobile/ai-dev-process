Managed-By: ai-dev-process
Managed-Id: guide.ticket-implementation
Managed-Source: maintain-tickets.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-02-20

# Ticket implementation session

Purpose: work through process improvement tickets filed against this repo. The human initiates a session, the agent summarizes ready tickets, they discuss approach, and the agent implements with approval.

This is an internal maintenance workflow for the `ai-dev-process` repo, not a guide for host projects.

## Prerequisites

- You are working in the `ai-dev-process` repo.
- A GitHub MCP server configured with issue read/write permissions.
- The repo has open issues labeled `agent ready` (triaged and approved for agent implementation).

## Procedure

### 1. Fetch ready tickets

Use the GitHub MCP server to list open issues labeled `agent ready`.

If no issues match, say **"No ready tickets found."** and stop.

### 2. Summarize

Present a summary of all ready tickets, grouped if natural categories emerge. For each ticket, show:

- Issue number and title
- One-line summary of what needs to change
- Affected files (if noted in the ticket)

### 3. Discuss and prioritize

Ask the human which tickets to tackle in this session and in what order. The human may:

- Select specific tickets by number.
- Reorder priorities.
- Defer tickets to a future session.
- Ask questions or discuss implementation approach for specific tickets.

Do not proceed until the human confirms the selection.

### 4. Implement

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

### 5. Close tickets

After the human approves the implementation for a ticket, use the GitHub MCP server to close it with a comment noting it was implemented. If the implementation will be committed and pushed in a follow-up step, note that in the close comment instead.

### 6. Post-session

After all selected tickets are implemented:

- Run the `maintain-retro.md` checklist and report any misses.
- Remind the human to commit if they haven't already.
