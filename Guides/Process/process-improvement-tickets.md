Managed-By: ai-dev-process
Managed-Id: guide.process-improvement-tickets
Managed-Source: Guides/Process/process-improvement-tickets.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-02

# Process improvement tickets

Purpose: file process improvement ticket drafts as GitHub issues on the `ai-dev-process` repo. Typically triggered after a dev retro or report-process-problem session, which produce drafts in the working file. Can also be used standalone.

## Checkpoints

This guide follows the shared process-flow mechanics in `Guides/Core/process-flow.md` (checkpoints, advance intent, `auto`, and the standard gate line).

Workflow-specific gate points (this guide must STOP and wait at these checkpoints):
- If GitHub MCP access is missing/broken: STOP and ask the human to set it up (or to fix permissions).
- After presenting 🟡 drafts for review: STOP and wait for the human to decide each entry (file, revise, or skip).
- After filing issues and updating the working file: report results and STOP (complete).

At checkpoints, end checkpoint output with the standard gate line (see `Guides/Core/process-flow.md`).

## Advance intent

Advance intent (and `auto`) semantics are defined in `Guides/Core/process-flow.md`.

## Confidentiality rule

This repo is public. Issues must **never** contain project names, client names, domain-specific details, or any information that identifies the source project. Every suggestion must be generalized into a process-level observation.

- Bad: "In Project Acme, the retro missed security edge cases in the auth module"
- Good: "The retro template does not prompt for security considerations, which can lead to gaps in auth-related work"

The human reviews all issue content before creation (step 3). This is the final gate, but the agent should make a good-faith effort to generalize first.

## Prerequisites

- A `process-tickets.md` working file with one or more 🟡 ticket drafts (typically produced by a dev retro or report-process-problem session).
- A GitHub MCP server configured with issue creation permissions.
- The `ai-dev-process` GitHub repo identifier (e.g., `owner/ai-dev-process`). Ask the human if you don't know it.

## Procedure

### 0. Check GitHub MCP access

Before drafting any issues, verify that a GitHub MCP server is configured and accessible.

If it is not available, tell the human that a GitHub MCP server is required for ticket creation and offer to help set one up (e.g., adding a GitHub MCP server to the project's MCP configuration with a personal access token that has repo issue permissions).

Do not proceed until GitHub MCP access is confirmed.

### 1. Locate the working file

Look for the `process-tickets.md` working file (path per `Guides/Core/working-doc-conventions.md`). This file is typically created by the retro's process reflection step or by the report-process-problem workflow.

If the file exists, read it and proceed to step 2.

If it does not exist (e.g., this was triggered standalone), create it and ask the human what process improvements they'd like to file. Write each entry as a 🟡 draft using the format below and proceed to step 2.

Draft format (for standalone use only -- producer guides write this format directly):

```
## 🟡 Ticket: <concise summary>

**Labels**: <labels>

**Friction**
<What was painful or suboptimal, described generically -- no project references>

**Evidence/example**
<What happened that shows the problem (generic; no project references)>

**Failure mode**
<What goes wrong if this repeats>

**Candidate approach** (optional)
<A possible fix direction, if confident>

**Affected files**
<Paths within the ai-dev-process repo, if known. "Unknown" is acceptable.>

**Verification** (optional)
<How to confirm the change worked next time>
```

For `**Labels**`: choose from this explicit list (source of truth for this workflow):
- `enhancement`
- `bug`
- `documentation`
- `help wanted`
- `question`
- `invalid`

### 2. Verify and present drafts

If all entries already use `## 🟡 Ticket:` headings (the normal case when coming from a retro or report-process-problem session), skip conversion. Otherwise, convert any remaining entries to 🟡 draft format.

Apply the confidentiality rule to all entries. Present the working file to the human for review.

### 3. Human review

For each 🟡 entry, the human decides: **file**, **revise**, or **skip**.

- **File**: proceed to step 4 for this entry.
- **Revise**: update the draft per the human's feedback, then re-present.
- **Skip**: remove the 🟡 marker and strike through the heading (e.g., `## ~~Ticket: <summary>~~`).

### 4. Create issues

For each entry the human approved for filing:

1. Use the GitHub MCP server to create the issue on the `ai-dev-process` repo with the labels from the draft.
2. Update the working file: remove the 🟡 marker from the heading and add `**Filed**: #<number>` below the heading.

Report the created issue numbers and URLs back to the human.
